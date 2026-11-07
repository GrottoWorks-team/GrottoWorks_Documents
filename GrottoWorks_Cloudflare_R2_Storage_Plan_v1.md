# GrottoWorks — Kế hoạch tích hợp Cloudflare R2 (ảnh tiến độ task + file thiết kế 3D)

**Ngày lập:** 06/11/2026 · **Phạm vi:** `GrottoWorks_BE` (NotificationAudit, TaskExecution, ParishCoordination, docker), `GrottoWorks_FE` (`lib/files.ts`, `features/designs`, `features/builder`), tài liệu ERD/Contract
**Trạng thái:** Phần A đã implement trên `GrottoWorks_BE@Vu-dev` (06/11/2026, chưa chạy thử với bucket R2 thật); B, C chờ chốt các quyết định ở §8

---

## 1. Hiện trạng (đã khảo sát code)

### 1.1 BE — đã có pipeline file hoàn chỉnh, chỉ đang chạy trên MinIO

| Thành phần | File | Ghi chú |
|---|---|---|
| API upload 3 bước | `NotificationAudit.Api/Endpoints/NotificationAuditEndpoints.cs` | `POST /api/v1/files/upload-requests` → presigned PUT; `POST /files/{id}/complete`; `GET /files/{id}/download-url`; `DELETE /files/{id}` |
| Abstraction storage | `NotificationAudit.Application/Files/FileUpload.cs` → `IFileObjectStorage` | `CreateUploadRequestAsync`, `InspectAsync`, `CopyToPrivateAsync`, `CreateDownloadRequestAsync`, `DeleteAsync` |
| Implementation | `NotificationAudit.Infrastructure/Files/MinioFileObjectStorage.cs` | Minio SDK, 2 client (internal/public endpoint), `MinioBucketInitializer` tự tạo bucket |
| Cũng dùng Minio SDK trực tiếp | `SystemJobs/SystemArtifactStorage.cs`, `SystemJobs/SystemObjectCleanupWorker.cs`, `NotificationAudit.BackupWorker/DatabaseBackupArtifactBuilder.cs`, `tools/Verification/RecoveryObjects.cs` | Phải chuyển cùng lúc |
| Key layout | `pending/{userId}/{fileId}` → copy sang `completed/{userId}/{fileId}/{attempt}` khi complete; `system-jobs/` | Trình duyệt chỉ ghi được `pending/` |
| Kiểm tra sau upload | `InspectAsync` | Đọc **toàn bộ object vào RAM** (giới hạn 20 MB), SHA-256, nhận diện magic bytes jpeg/png/webp/pdf |
| Gắn nghiệp vụ | `/internal/files/{id}/attachment-reservations[/finalize|/release]`, `record-download-url` | Whitelist service: `TaskExecution, Resource, Readiness, NotificationAudit` |
| Purpose | `FilePurposeDto` | `ProgressEvidence` (ảnh ≤ 10 MB), `ReadinessEvidence`, `PurchaseReceipt`, `Other` |
| Ảnh tiến độ task | `TaskExecution` (`TaskProgress`, `ProgressEvidence`, `TaskFileGateway`, `FileAttachmentRecovery`) | **Đã nối xong** với file service: `POST /tasks/{id}/progress` nhận `evidence[{fileId, caption}]`, reserve → finalize, `getTaskEvidenceDownloadUrl` |
| Config | `ObjectStorage__InternalEndpoint/PublicEndpoint/AccessKey/SecretKey/Bucket` trong `docker-compose.override.yml` | Service `notificationaudit.api` + backup worker |

### 1.2 FE

| Phần | Hiện trạng |
|---|---|
| `src/lib/files.ts` | `uploadFile(file, purpose)` đã gọi đúng 3 bước presigned; dùng cho checklist (`ReadinessEvidence`). Không có progress %, không gửi checksum |
| Ảnh tiến độ task (web) | Chỉ **xem** (`TaskDetailPage`, `getTaskEvidenceDownloadUrl`); nộp tiến độ do app mobile (role VOLUNTEER) |
| Thiết kế 2D/3D (`features/designs`) | **FE-only**: metadata ở `localStorage` (`designStore`), file ảnh/`.glb` ở **IndexedDB** (`lib/blobStore.ts`, `UploadVersionDialog` → `putBlob`). `ModelReviewCanvas` dùng `GLTFLoader.load(objectURL)`. Giới hạn: ảnh 8 MB, `.glb` 75 MB (`designs/config.ts`) |
| Trình dựng 3D (`features/builder`) | Scene JSON ở `localStorage grotto-builder-scene-v1:<areaId>`; `designLink.ts` upsert `DesignVersion` với `fileKey = scene:<areaId>` |
| Khoảng trống đã ghi | `docs/DESIGN_FLOW.md` §3–§5 (G1–G4): chưa có BE cho design/version/comment, file ở IndexedDB không chia sẻ được |

**Kết luận:** R2 tương thích S3 nên **không phải viết lại luồng upload**. Việc chính gồm: (A) thay adapter MinIO bằng adapter S3 chạy được cả MinIO (dev) lẫn R2 (staging/prod); (B) mở rộng file service cho file 3D lớn; (C) làm BE cho hồ sơ thiết kế và chuyển FE từ IndexedDB sang file service.

---

## 2. Kiến trúc mục tiêu

```text
 FE / Mobile                Gateway (YARP)            NotificationAudit (file service)          Cloudflare R2 (bucket private)
 ───────────                ──────────────            ───────────────────────────────          ──────────────────────────────
 1. POST /files/upload-requests ───────────────────►  tạo file_object(PENDING_UPLOAD)
                                                      presign PUT pending/{u}/{f}  ─────────►
 2. PUT <presigned URL> ─────────────────────────────────────────────────────────────────────►  pending/{u}/{f}
 3. POST /files/{id}/complete ─────────────────────►  CopyObject → completed/{u}/{f}/{n}  ──►  
                                                      stream SHA-256 + kiểm tra định dạng
 4. POST /tasks/{id}/progress {evidence:[fileId]}  ─► TaskExecution ── reserve/finalize ──► file_object ATTACHED
    POST /design-versions {fileId}                ─► (service sở hữu Design) ── reserve/finalize
 5. GET  .../download-url  ────────────────────────►  scope check ở service nghiệp vụ → presign GET (5–15')
    <img src> / GLTFLoader.load(url) ────────────────────────────────────────────────────────►  completed/...
```

Nguyên tắc giữ nguyên: bucket **private**, không bật `r2.dev` public; mọi truy cập bằng presigned URL ngắn hạn; DB chỉ lưu `object_key`, không lưu URL.

---

## 3. Phần A — Adapter S3 dùng chung MinIO/R2 (BE, NotificationAudit)

### A1. Thay Minio SDK bằng `AWSSDK.S3`
- Lý do: Cloudflare có hướng dẫn chính thức cho AWS SDK .NET; Minio SDK hay tự gọi `GetBucketLocation`/region và tạo bucket — token R2 giới hạn theo bucket sẽ bị từ chối.
- Tạo `S3FileObjectStorage : IFileObjectStorage` (giữ nguyên interface ⇒ Application/Domain/test không đổi).
- `AmazonS3Config`: `ServiceURL = Endpoint`, `ForcePathStyle = true`, `AuthenticationRegion = "auto"` (R2) / `"us-east-1"` (MinIO).
- R2 với AWSSDK v4: đặt `RequestChecksumCalculation = WHEN_REQUIRED`, `ResponseChecksumValidation = WHEN_REQUIRED`; `PutObjectRequest.DisablePayloadSigning = true` (theo docs R2).
- Presign: `GetPreSignedURL` (Verb PUT, `ContentType` được ký ⇒ client phải gửi đúng header `Content-Type`), Verb GET kèm `ResponseHeaderOverrides.ContentDisposition` (tên file gốc) khi cần tải về.
- `CopyObject`, `GetObject`, `DeleteObject`, `GetObjectMetadata` thay cho Stat — R2 đều hỗ trợ.
- Chuyển luôn `SystemArtifactStorage`, `SystemObjectCleanupWorker` (`ListObjectsV2` theo prefix), `DatabaseBackupArtifactBuilder`, `tools/Verification/RecoveryObjects.cs` sang một factory `IS3ClientFactory` dùng chung.

> **Đã làm (06/11/2026):** `S3FileObjectStorage`, `ObjectStorageOptions`/`ObjectStorageClients` (singleton), `ObjectStorageInitializer`; đã chuyển `SystemArtifactStorage`, `SystemObjectCleanupWorker`, `DatabaseBackupArtifactBuilder`, `tools/Verification/RecoveryObjects.cs`; hướng dẫn ở `GrottoWorks_BE/docs/ops/cloudflare-r2.md`. Khác so với đề xuất ban đầu: giữ tên key `InternalEndpoint` (không đổi thành `Endpoint`) để không phải sửa compose cũ; với R2, `PublicEndpoint` bị bỏ qua vì presigned URL của R2 chỉ hợp lệ trên endpoint S3. Chưa làm: readiness probe kiểm tra bucket, integration test bằng Testcontainers (môi trường hiện chưa có Docker; đã kiểm tra luồng end-to-end bằng một S3 server cục bộ).

### A2. Cấu hình
```jsonc
"ObjectStorage": {
  "Provider": "R2",                 // "MinIO" | "R2"
  "Endpoint": "https://<ACCOUNT_ID>.r2.cloudflarestorage.com",
  "PublicEndpoint": "",             // rỗng = dùng Endpoint (R2). MinIO dev: http://localhost:9000
  "Region": "auto",
  "Bucket": "grottoworks-files",
  "AccessKey": "",                  // lấy từ secret/env, KHÔNG commit
  "SecretKey": "",
  "ManageBucket": false             // true chỉ cho MinIO local (tạo bucket + CORS)
}
```
- Giữ tương thích ngược: map `InternalEndpoint` cũ → `Endpoint` trong 1 sprint rồi bỏ.
- `MinioBucketInitializer` → `ObjectStorageInitializer`: nếu `ManageBucket=false` chỉ `HeadBucket` để health check, không tạo bucket.
- Thêm readiness probe `/health/ready` kiểm tra `HeadBucket` (đã có `IReadinessProbe`).
- `docker-compose.override.yml`: giữ service `minio` cho local; thêm `.env.example` các biến `OBJECT_STORAGE_PROVIDER`, `R2_ACCOUNT_ID`, `R2_ACCESS_KEY_ID`, `R2_SECRET_ACCESS_KEY`, `R2_BUCKET`.

### A3. Thiết lập Cloudflare R2 (làm 1 lần, ghi vào `docs/ops/r2-setup.md`)
1. Tạo bucket `grottoworks-files` (prod) và `grottoworks-files-staging`; location hint APAC.
2. API token: **Object Read & Write**, chỉ áp cho 2 bucket trên; mỗi môi trường 1 token; lưu vào secret của server/CI.
3. CORS bucket (bắt buộc vì trình duyệt PUT/GET trực tiếp):
   ```json
   [{
     "AllowedOrigins": ["https://<domain-fe>", "http://localhost:5173"],
     "AllowedMethods": ["GET", "PUT", "HEAD"],
     "AllowedHeaders": ["Content-Type", "Content-Length"],
     "ExposeHeaders": ["ETag", "Content-Length"],
     "MaxAgeSeconds": 3600
   }]
   ```
4. Lifecycle rule: xóa object prefix `pending/` sau 1 ngày; abort multipart chưa xong sau 1 ngày. (Worker dọn dẹp hiện có vẫn giữ làm lớp thứ hai.)
5. Không bật public `r2.dev`, không gắn custom domain public trong giai đoạn này.
6. Theo dõi chi phí: free tier 10 GB lưu trữ, 1 triệu thao tác Class A, 10 triệu Class B/tháng, egress miễn phí. Ước tính: 10 khu × 20 phiên bản × 75 MB ≈ 15 GB + ảnh tiến độ ≈ vài GB ⇒ chi phí phát sinh rất nhỏ ($0,015/GB-tháng). Bật cảnh báo billing.

### A4. Test
- Unit: giữ nguyên test của `FileCompletionTests`/`FileFinalizationTests` (chạy qua interface).
- Integration: Testcontainers MinIO cho `S3FileObjectStorage` (cùng API S3).
- Smoke script `tools/Verification` chạy với bucket staging R2: presign PUT → PUT → complete → presign GET → so SHA-256 → delete.

**Tiêu chí xong A:** `docker compose up` vẫn chạy với MinIO; đổi env sang R2 staging thì luồng checklist (`ReadinessEvidence`) và ảnh tiến độ chạy end-to-end không sửa code FE.

---

## 4. Phần B — Ảnh tiến độ task (`PROGRESS_EVIDENCE`)

BE nghiệp vụ đã xong, nên phần này chủ yếu là kiểm thử trên R2 và hoàn thiện client.

| # | Việc | Repo |
|---|---|---|
| B1 | Kiểm tra lại luồng `addTaskProgress` / `submitTask` với `evidence[]` trên R2 staging: reserve → finalize → `getTaskEvidenceDownloadUrl` → ảnh hiển thị trong `TaskDetailPage` và review package của Leader | BE + FE |
| B2 | `lib/files.ts`: thêm `onProgress` (dùng `XMLHttpRequest` cho PUT để có % upload), tính `checksum` SHA-256 bằng `crypto.subtle` (ảnh ≤ 10 MB, rẻ) để BE đối chiếu | FE |
| B3 | Nén ảnh phía client trước khi upload (cạnh dài ≤ 2560 px, WebP/JPEG q≈0,82) — giảm dung lượng R2 và thời gian upload trên 4G | FE/Mobile |
| B4 | Hiển thị thumbnail: tạm dùng chính URL presigned; nếu chậm thì phase sau sinh thumbnail (ngoài phạm vi) | FE |
| B5 | Thông báo cho team mobile: chỉ cần `PUT` đúng `headers` BE trả về (đặc biệt `Content-Type`), không đổi API | Mobile |
| B6 | Xử lý URL hết hạn: khi `<img>` lỗi 403 → gọi lại `download-url` một lần | FE |

---

## 5. Phần C — File thiết kế 3D (`.glb`) và ảnh thiết kế 2D

### C1. Mở rộng file service (NotificationAudit)
- Purpose mới (Domain + Contracts + ERD `file_object.purpose`):
  - `DESIGN_IMAGE`: `image/png|jpeg|webp`, ≤ 10 MB.
  - `DESIGN_MODEL`: `model/gltf-binary`, đuôi `.glb`, ≤ 75 MB (khớp `MAX_MODEL_BYTES` FE).
  - (tùy chọn, xem §8-Q2) `DESIGN_SCENE`: `application/json`, ≤ 5 MB.
- `ValidateFile`: thêm nhánh cho các purpose trên; FE phải gửi `contentType = "model/gltf-binary"` tường minh vì trình duyệt thường trả `File.type = ""` cho `.glb`.
- **Viết lại `InspectAsync` theo kiểu streaming** (bắt buộc cho file 75 MB):
  - Đọc stream từ `GetObject`, cập nhật `IncrementalHash` SHA-256 theo từng block, chỉ giữ 12 byte đầu để nhận diện; không nạp cả file vào RAM.
  - Giới hạn byte đọc theo purpose (thay hằng `MaximumInspectionBytes = 20 MB`).
  - `DetectContentType`: thêm GLB — magic `glTF` (`0x67 0x6C 0x54 0x46`), version `2`, trường `length` (uint32 LE, byte 8–11) phải bằng `SizeBytes`; JSON: parse thử với `Utf8JsonReader` cho `DESIGN_SCENE`.
- `DownloadUrlMinutes`: cho phép override theo purpose (model 15 phút) vì tải 75 MB trên mạng yếu.
- Whitelist `ValidateRecord`: thêm service sở hữu Design (xem §8-Q1), referenceType `DESIGN_VERSION`.
- Rate limit gateway `/files/upload-requests` đã có — giữ nguyên.

### C2. BE hồ sơ thiết kế (đóng G1/G2/G4 trong `FE/docs/DESIGN_FLOW.md`)
Bảng (ERD, đặt trong DB của service sở hữu):

| Bảng | Cột chính |
|---|---|
| `design` | `design_id`, `work_area_id`, `title`, `description`, `design_type` (`TWO_D`/`THREE_D`), `current_version_id`, `created_by_user_id`, `created_at`, `updated_at`, `version` (concurrency) |
| `design_version` | `design_version_id`, `design_id`, `version_number` (unique theo design), `source` (`UPLOAD`/`BUILDER_SCENE`), `file_id` (nullable, file_object ATTACHED), `scene_json` jsonb (nullable), `file_name`, `mime_type`, `size_bytes`, `change_note`, `status` (`DRAFT/IN_REVIEW/APPROVED/CHANGES_REQUESTED`), `uploaded_by_user_id`, `uploaded_at`, `approved_by_user_id`, `approved_at` |
| `design_comment` | như type `DesignComment` FE (`position_type`, `position_2d`, `position_3d`, `object_name`, `task_id`…) |
| `design_scene` | `work_area_id` (pk), `scene_json` jsonb, `schema_version`, `version`, `updated_by`, `updated_at` — scene đang chỉnh của builder |
| `attachment_intent` | tái dùng pattern outbox reserve/finalize như `TaskExecution`/`Readiness` (`FileAttachmentRecovery`) |

Endpoint (theo đề xuất DESIGN_FLOW §4, chốt lại path khi cập nhật Contract):
- `GET/POST /api/v1/work-areas/{workAreaId}/designs`, `PATCH /api/v1/designs/{id}`
- `GET/POST /api/v1/designs/{id}/versions` — body `{ fileId, changeNote }` hoặc `{ source: "BUILDER_SCENE" }`; BE reserve → lưu version → finalize trong cùng luồng outbox
- `PATCH /api/v1/design-versions/{id}` — duyệt / yêu cầu chỉnh sửa (`If-Match`)
- `GET /api/v1/design-versions/{id}/file-url` — scope check `work_area_scope` rồi gọi `record-download-url`
- `GET/PUT /api/v1/work-areas/{workAreaId}/design-scene` — `PUT` bắt buộc `If-Match`
- `GET/POST /api/v1/design-versions/{id}/comments`, `PATCH/DELETE /api/v1/design-comments/{id}`
- Quyền: tạo/upload = LEADER của khu + PARISH; duyệt = PARISH (chốt với BA); audit qua event → `activity_log`.
- Gateway: thêm route `/api/v1/designs/**`, `/api/v1/design-versions/**`, `/api/v1/design-comments/**`, `/api/v1/work-areas/{id}/design*`.

### C3. FE — bỏ IndexedDB, dùng file service + API thật
| # | Việc | File |
|---|---|---|
| C3.1 | `FilePurpose` thêm `DesignImage`, `DesignModel`; `uploadFile(file, purpose, { contentType?, onProgress? })` | `src/lib/files.ts` |
| C3.2 | `UploadVersionDialog`: thay `newBlobKey`+`putBlob` bằng `uploadFile` → `fileId`; thanh tiến trình % cho `.glb`; nếu tạo version lỗi → `deleteFile(fileId)` (thay `deleteBlob`) | `features/designs/components/UploadVersionDialog.tsx` |
| C3.3 | Type `DesignVersion`: `fileKey` → `fileId?` + `source`; giữ đọc `fileKey` cũ trong giai đoạn chuyển tiếp | `src/types/index.ts` |
| C3.4 | `api.ts`: thay `designStore.*` bằng `api(...)` (giữ nguyên hook surface như comment trong file đã dự kiến); query key theo `areaId`/`designId` | `features/designs/api.ts` |
| C3.5 | `DesignReviewPage`: thay `getBlob` + `createObjectURL` bằng `GET /design-versions/{id}/file-url`; truyền URL thẳng cho `<img>` / `ModelReviewCanvas` (`GLTFLoader.load(url)` cần CORS GET trên R2 — §A3) ; retry 1 lần khi 403 hết hạn | `features/designs/pages/DesignReviewPage.tsx`, `components/ModelReviewCanvas.tsx` |
| C3.6 | Builder: `SceneRepository` gọi `GET/PUT design-scene` (If-Match); `designLink.syncSceneDesign` gọi API tạo version `BUILDER_SCENE` thay vì `designStore` | `features/builder/repository.ts`, `designLink.ts` |
| C3.7 | Asset `.glb` do người dùng import vào builder (nếu làm): upload `DesignModel`, scene lưu `fileId` thay vì URL `public/models` | `features/builder/assets.ts` |
| C3.8 | Feature flag `VITE_DESIGN_STORAGE=server|local` để demo offline vẫn chạy; nút "Đẩy dữ liệu cục bộ lên server" (đọc IndexedDB → upload) cho dữ liệu demo đã có — tùy chọn | `features/designs` |
| C3.9 | Cập nhật test (`designLink.test.ts`, `repository.test.ts`) dùng mock API; cập nhật `docs/DESIGN_FLOW.md` §2 | FE |

Ngoài phạm vi: `ParishMapPage` (ảnh bản đồ giáo xứ) cũng đang dùng `blobStore` — làm sau bằng cùng cơ chế (purpose `OTHER`/`PARISH_MAP`).

---

## 6. Cập nhật tài liệu (làm **trước** khi code C2, theo quy trình plan.md §1)
- ERD v3.6 → v3.7: purpose mới của `file_object`; bảng `design`, `design_version`, `design_comment`, `design_scene`.
- API Contract v1.3 → v1.4: endpoint §5-C2, mã lỗi mới (`FILE_MODEL_INVALID`, `DESIGN_VERSION_CONFLICT`, …), purpose mới trong schema `CreateUploadRequest`.
- BA: use case review thiết kế + quyền duyệt.
- BE `plan.md`: thêm function ID (đề xuất `F-PLT-10` R2 adapter, `F-NTF-07` file 3D, `F-DSG-01..05` design).

---

## 7. Lộ trình đề xuất

| Sprint | Thời gian | Hạng mục | Kết quả demo |
|---|---|---|---|
| S3 | 16/11 – 29/11 | A1–A4 (adapter S3, config, setup R2 staging, smoke test); §6 tài liệu; B1 | Checklist + ảnh tiến độ upload lên R2 staging |
| S4 | 30/11 – 13/12 | B2–B6; C1 (purpose mới + inspection streaming + GLB); C2 phần design/version/file-url | Upload `.glb` 75 MB lên R2, xem trong ModelReviewCanvas qua presigned URL |
| S5 | 14/12 – 27/12 | C2 comment + design-scene; C3.1–C3.6; C3.9 | Thiết kế 2D/3D chia sẻ được giữa các máy, quyền kiểm ở server |
| S6 | 28/12 – 10/01 | C3.7–C3.8, prod bucket + token prod, cảnh báo billing, hardening | Chuyển prod sang R2 |

---

## 8. Quyết định cần chốt

| ID | Câu hỏi | Đề xuất |
|---|---|---|
| Q1 | Design/scene thuộc service nào? | Đặt trong **ParishCoordination** (đã sở hữu `work_area`, scope và vai trò PARISH/LEADER) để không tạo thêm service/DB; nếu nhóm muốn tách thì tạo service `Design` riêng — đều dùng chung file service |
| Q2 | Scene builder lưu ở đâu? | Scene đang chỉnh: `jsonb` trong DB (nhỏ, cần `If-Match`). Khi tạo version từ builder: snapshot `scene_json` vào `design_version` (không cần R2). Chỉ dùng `DESIGN_SCENE` trên R2 nếu scene vượt ~1 MB |
| Q3 | Giới hạn `.glb` | 75 MB (khớp FE). R2 cho phép PUT đơn tới 5 GB nên không cần multipart ở giai đoạn này |
| Q4 | Giữ MinIO cho local? | Có — không phụ thuộc mạng/tài khoản khi dev; R2 cho staging/prod |
| Q5 | Xóa phiên bản cũ? | Không xóa (lịch sử review); xem xét chính sách lưu trữ sau mùa (archive/xóa sau khi season `CLOSED`) |
| Q6 | Ai giữ token R2 prod? | Leader team; lưu ở secret của server deploy, không đưa vào repo/CI log |

---

## 9. Rủi ro & cách giảm

| Rủi ro | Giảm thiểu |
|---|---|
| Trình duyệt bị CORS khi PUT/GET R2 | Cấu hình CORS §A3 trước khi test FE; header `Content-Type` phải khớp chữ ký presign |
| Inspection 75 MB làm tràn RAM container | Inspection streaming (C1), không buffer |
| Client upload file giả mạo đuôi `.glb` | Kiểm tra header GLB + length sau upload; sai thì 422 và xóa object |
| Object rác ở `pending/` | Lifecycle rule 1 ngày + `SystemObjectCleanupWorker` |
| Lộ access key | Token scope theo bucket, xoay vòng định kỳ, chỉ qua env/secret |
| Presigned URL hết hạn khi đang xem | TTL 5' ảnh / 15' model + retry gọi lại `file-url` |
| Phụ thuộc mạng ngoài trong test | Integration test dùng MinIO; chỉ smoke test dùng R2 staging |
