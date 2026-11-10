# Trạng thái triển khai & tổng hợp tài liệu — 11/11/2026

**Phạm vi:** `GrottoWorks_BE` · `GrottoWorks_FE` (web) · `GrottoWokrs_Mobile` (TNV)
**Mục đích:** một chỗ duy nhất trả lời "cái gì **đã làm**, cái gì **chưa**, tài liệu nào còn đúng".
**Thay thế:** bảng số liệu trong `API-GAP-STATUS-2026-11-04.md` §1 (số liệu mobile đã thay đổi 10–11/11).
**Đọc kèm:** `DOCS-INDEX.md` (danh mục tài liệu) · `API-GAP-STATUS-2026-11-04.md` (chi tiết gap API) ·
`GrottoWorks_BE/docs/BE-TODO-demo-2026-11-10.md` (việc BE phải làm cho 3 tính năng mới).

Ký hiệu: ✅ đủ (BE có API + UI nối thật) · 🟡 một phần / chỉ đọc / dùng dữ liệu cục bộ · ❌ chưa có · ⛔ BE không có endpoint (đã ẩn UI).

---

## 1. Bảng tổng hợp theo module

| # | Module | BE | Web | Mobile | Ghi chú / tài liệu |
|---|---|---|---|---|---|
| 1 | Đăng nhập, refresh, logout, quên/đổi mật khẩu | ✅ | ✅ | ✅ | `Identity`; mobile chỉ cho role VOLUNTEER |
| 2 | Tài khoản & phân quyền (admin) | ✅ | ✅ | — | ADMIN bị 403 ở `/users` → web fallback `/admin/users` (A10) |
| 3 | Hồ sơ TNV: tên/SĐT/giới thiệu, kỹ năng, lịch rảnh | ✅ | ✅ | ✅ | `PATCH /users/me`, `PUT /users/me/skills|availability` |
| 4 | Giáo xứ & cộng đoàn | ✅ | ✅ | 🟡 (đọc) | Mobile chỉ đọc cộng đoàn để khai hồ sơ |
| 5 | Mùa: tạo/sửa, activate/complete/archive | 🟡 | 🟡 | 🟡 (đọc) | `complete` chỉ nhận READY, thiếu ACTIVE→READY (A3) ⇒ nút "Chốt mùa" bị khoá |
| 6 | **Tạo mùa mới từ mùa cũ (clone)** | ❌ | ❌ | — | Không có `POST /seasons/{id}/clone`; đặc tả: FE spec map/clone §1 |
| 7 | Khu vực công tác (CRUD, gán leader/MO/cộng đoàn) | ✅ | ✅ | 🟡 (đọc) | Tạo + gán tách 2 request (A12) |
| 8 | **Sơ đồ giáo xứ (ảnh + ghim khu vực)** | ❌ | 🟡 | ❌ | **Demo FE-only**: `features/map/localStore.ts` (localStorage + IndexedDB); chỉ ADMIN (FE `PARISH`) sửa, ban hành giáo xem. Cần `GET/PUT/DELETE /parishes/{id}/map` (BE-TODO §1) |
| 9 | Nhiệm vụ: CRUD, kỹ năng, tiêu chí, phát hành | ✅ | ✅ | 🟡 (đọc + nhận/từ chối) | Chỉ task OPEN mới sửa (409 `TASK_INVALID_STATE`) |
| 10 | Phân công TNV (mời/accept/decline) | ✅ | ✅ | ✅ | `GET /users/me/assignments` (A8 đã đóng 05/11) |
| 11 | Đơn đăng ký tham gia (application) | 🟡 | ✅ (duyệt) | 🟡 (nộp) | BE có POST + list theo task + approve/reject; **thiếu** list/huỷ cho TNV (A5) ⇒ mobile cache cục bộ `api/applicationsCache.ts` |
| 12 | Tiến độ, gửi duyệt, nghiệm thu, yêu cầu sửa | ✅ | ✅ | ✅ | Mobile upload minh chứng qua presigned URL |
| 13 | Chấm công: yêu cầu vào ca, duyệt, check-out, sửa giờ | ✅ | ✅ | ✅ | `attendance-requests`, `attendance/{id}/correction-requests` |
| 14 | Vật tư: danh mục, tồn kho, yêu cầu theo khu, thiếu hụt, cấp phát | ✅ | ✅ | 🟡 (danh mục) | Mobile chỉ đọc `/materials` để hiện tên vật tư trong task |
| 15 | Mua hàng: đề xuất, duyệt, mua, nhập kho, hoá đơn | ✅ | ✅ | — | Kèm `purchase-tasks`, `confirm-delivery`, `receipts` |
| 16 | Quyên góp (vật tư/tiền), nhà tài trợ | ✅ | ✅ | ✅ (cam kết + của tôi) | |
| 17 | Đồ mượn (loan offer, issue/return/incident) | ✅ | ✅ | ✅ (chào cho mượn) | |
| 18 | Hỗ trợ chéo (support request/response) | ✅ | ✅ | — | TNV không tạo support được: route yêu cầu LEADER (A15) |
| 19 | Sẵn sàng / checklist trước mùa | ✅ | ✅ | — | `readiness-checks`, evidence, corrective task |
| 20 | Báo cáo & xuất file | ✅ | ✅ | — | `/reports/*`, `report-exports` |
| 21 | Vinh danh / điểm đóng góp | ✅ | ✅ | ✅ | `contribution-types`, `recognition-rules`, `users/me/recognition` |
| 22 | Thông báo (in-app, đã đọc, tuỳ chọn) | ✅ | ✅ | ✅ | `/notifications`, `read-all`, preferences |
| 23 | Nhật ký hoạt động (audit) | ✅ | ✅ (ADMIN) | ❌ | `/activity-logs` **chỉ ADMIN** ⇒ mobile đã bỏ màn/hook hoạt động |
| 24 | Sao lưu & job hệ thống | ✅ | ✅ | — | `/backups`, `/system-data-exports` |
| 25 | Dashboard | ✅ | ✅ | — | Web tổng hợp từ dữ liệu thật |
| 26 | Design Review 2D/3D (bản vẽ, phiên bản, bình luận) | ❌ | 🟡 | ❌ | **Demo FE-only**: `features/designs/localStore.ts` + `lib/blobStore.ts` |
| 27 | 3D Layer Builder (dựng cảnh, lớp, liên kết task) | ❌ | ✅ (local) | 🟡 | Web lưu localStorage (`features/builder`); TNV xem **lớp gắn với nhiệm vụ** trong app bằng scene mock (`features/volunteer/mockScene.ts`) |
| 28 | Đăng ký tình nguyện công khai (`/join`) + danh sách giáo xứ public | ❌ | ⛔ | — | Không có `POST /registrations` / `GET /public/parishes` (A4) ⇒ trang chỉ báo "đang chờ" |
| 29 | Đơn vị vật tư (material units) | ❌ | ⛔ | — | A14: đã bỏ UI lưu localStorage |
| 30 | Cấu hình notification rules | 🟡 | — | — | Có PATCH theo id, thiếu GET danh sách (A7) |
| 31 | Xoá dữ liệu (work area/task/community/skill/user) | ❌ | ⛔ | — | BE **0 route DELETE** (A1) ⇒ toàn bộ nút xoá đã ẩn |

---

## 2. Thay đổi mới nhất (10–11/11) đã có trong code

| Thay đổi | Nơi thực hiện | Tài liệu tương ứng |
|---|---|---|
| Sơ đồ + khu vực do **ADMIN (FE `PARISH`)** quản lý, ban hành giáo chỉ xem | `ParishMapPage.tsx` (`canEdit`), `AreaListPage.tsx` (`canEdit`) | FE spec map §2.4 (đã sửa 11/11) |
| **Tạo khu vực + ghim vị trí trong cùng form** (pin tuỳ chọn) | `AreaFormDialog.tsx` + `features/map/components/MapPinPicker.tsx` | FE spec map §2.5 (đã sửa 11/11) |
| TNV xem **lớp 3D gắn với nhiệm vụ** ngay trong app (không mở trình duyệt) | `features/volunteer/TaskModelScreen.tsx` + `sceneHtml.ts` + `mockScene.ts` (scene mock) | `BE-TODO-demo-2026-11-10.md` §3 |
| Bỏ trang chia sẻ công khai `/share/scene/:areaId` | FE: xoá `SceneSharePage`/`demoScene`, bỏ `WEB_BASE` ở mobile | — |
| **Bỏ toàn bộ mock mobile** (seed users/areas/tasks/timesheets, `withFallback`, `mockLogin`, `logActivity`) | `GrottoWokrs_Mobile/src/api/index.ts`, xoá `src/mocks/` | Mobile README (đã sửa 11/11) |
| Mobile gọi BE cho vật tư (`GET /materials`) | `real.ts: materialsReal()` | — |
| Bỏ stub `useVolunteerRegs()` trả mảng rỗng | FE: xoá `features/volunteers/api.ts` | — |
| **Mobile: thêm “Tuỳ chọn thông báo”** (in-app/push/email) | `features/volunteer/profile/NotificationSettingsScreen.tsx` + `GET/PUT /users/me/notification-preferences` | — |

### 2.1. Kết quả rà soát "endpoint BE có mà client chưa dùng" (11/11)

Quét 224 route public của `*Endpoints.cs` và đối chiếu toàn bộ `src/**/*.ts|tsx` của web + mobile:
**chỉ còn 3 route không có client**, trong đó 2 route health là hạ tầng:

| Route | Vì sao chưa dùng |
|---|---|
| `GET /health/live`, `GET /health/ready` | Hạ tầng, không cần UI |
| `PATCH /notification-rules/{ruleId}` | Chỉ có PATCH, **không có GET danh sách** (A7) nên không dựng được màn sửa |

⇒ Không còn tính năng nào "BE đã có endpoint mà FE/mobile chưa nối". Các phần còn thiếu đều do
**BE chưa có endpoint** (§3.1, §3.2), không phải thiếu UI.

---

## 3. Việc còn mở (theo mức ưu tiên)

### 3.1. BE phải làm để 3 tính năng mới chạy thật
1. `parish.map_*` (migration) + `GET/PUT/DELETE /parishes/{id}/map`, policy **ADMIN-only** cho ghi.
2. Đổi policy `POST/PATCH/PUT /work-areas*` về **ADMIN** (hiện còn cho BE `PARISH`).
3. `work_area_scene` + `scene_version` + `GET/PUT /work-areas/{id}/design-scene`; quyền đọc cho VOLUNTEER đã nhận việc (lọc theo `taskIds`).
4. Tuỳ chọn: gộp `mapPin` vào `POST /seasons/{id}/work-areas` (1 transaction).
Chi tiết + ước lượng: `GrottoWorks_BE/docs/BE-TODO-demo-2026-11-10.md`.

### 3.2. Gap API cũ còn mở (không đổi so với 04/11)
- A1 không có DELETE · A2 thiếu `GET /materials/{id}` · A3 thiếu ACTIVE→READY · A4 thiếu đăng ký công khai · A5 thiếu list/huỷ đơn của TNV · A6 thiếu design-scene · A7 thiếu GET notification-rules · A10 ADMIN 403 ở `/users`, `/purchase-requests`, dashboards · A11 thiếu `communityId` trong request admin · A12 thiếu leader/officer khi tạo khu · A13 thiếu field DTO · A14 đơn vị vật tư · A15 support cho VOLUNTEER.
- Vận hành: B1 backfill `work_area_scope/season_scope`, B2 rebuild image, B3 pending model changes `task_db`, B5/B8 MinIO endpoint & CORS, B6 rate limit, B7 charset.

### 3.3. Việc FE/mobile còn thiếu UI
- Web: màn clone mùa; dùng `PUT /parishes/{id}/map` khi BE xong; bỏ `mapStore`/`designStore`/scene local khi có API.
- Mobile: màn danh sách + huỷ đơn đăng ký (chờ A5); renderer 3D thật khi có `.glb` (hiện là canvas 2D mô phỏng).

---

## 4. Rà soát tài liệu: cái nào còn đúng, cái nào lỗi thời

| Tài liệu | Vai trò | Trạng thái 11/11 | Việc cần làm |
|---|---|---|---|
| `DOCS-INDEX.md` | Danh mục tài liệu | ✅ đã cập nhật 11/11 | Trỏ tới file này |
| `API-GAP-STATUS-2026-11-04.md` | Gap API chi tiết | 🟡 §1 sai số liệu mobile (đã sửa 11/11), §3 vẫn đúng | Giữ làm tham chiếu gap |
| `api-contracts.md` | Đối chiếu FE ↔ BE | 🟡 còn liệt kê vài path legacy nhưng nội dung vẫn đúng hướng | Rà lại khi BE xong map/design |
| `GrottoWorks_FE/docs/specs/2026-11-03-clone-season-and-parish-map.md` | Đặc tả clone + sơ đồ | 🟡 §2.4/§2.5 lỗi thời (quyền + luồng ghim) | **Đã sửa 11/11**; phần clone vẫn là đề xuất (BE chưa làm) |
| `GrottoWorks_FE/docs/specs/2026-10-27-3d-layer-builder.md` | Đặc tả builder + API scene | ✅ (ghi rõ giai đoạn A xong, B/C chờ BE) | Bổ sung màn TNV xem lớp khi BE có API |
| `GrottoWorks_FE/docs/specs/2026-10-27-design-review-2d-3d.md` | Đặc tả review 2D/3D | 🟡 vẫn là FE-local | Giữ tới khi có `design-*` API |
| `GrottoWorks_FE/docs/UI-TEST-SCENARIO.md` | Kịch bản test UI | 🟡 dòng bản đồ ghi "COMMUNITY + PARISH sửa" | **Đã sửa 11/11**: chỉ ADMIN sửa |
| `GrottoWokrs_Mobile/README.md` | Hướng dẫn app TNV | ⛔ mô tả app chạy offline + thư mục `mocks/` | **Đã sửa 11/11**: chỉ gọi BE; thêm màn mô hình 3D |
| `GrottoWorks_BE/docs/BE-TODO-demo-2026-11-10.md` | Việc BE cho 3 tính năng | ✅ | Cập nhật khi BE bắt đầu làm |
| `GrottoWorks_Documents/*` (Ra_Soat_SRS_SDD, Phuong_An, Review_Vu, UI-E2E-Test-Report) | Ghi chép point-in-time | 🗄️ lịch sử | Không sửa; đọc theo `DOCS-INDEX` |
| `GrottoWorks_Phan_Tich_Nghiep_Vu_Chi_Tiet_v1.2.4.md`, `.../v1.2.md`, BE spec v1 | Bản cũ | ⛔ đã bị thay thế | Giữ nguyên nhãn cũ |

### 4.1. Ba chỗ tài liệu từng nói sai so với code (đã sửa hôm nay)
1. "Ban hành giáo (COMMUNITY) tải ảnh sơ đồ + ghim" — nay **chỉ ADMIN** ghi; ban hành giáo chỉ xem.
2. "Ghim là hành động **sau khi** khu vực tồn tại" — nay **ghim nằm trong form tạo/sửa khu vực** (pin tuỳ chọn).
3. Mobile "chạy offline với mock seed" — nay **chỉ gọi BE**, không còn `src/mocks/`.

---

## 5. Kiểm chứng nhanh (không cần đọc hết tài liệu)

```bash
# BE: danh sách route (đối chiếu A1..A15)
Get-ChildItem -Recurse GrottoWorks_BE\Services -Include *Endpoints.cs -File |
  Select-String -Pattern '(MapGet|MapPost|MapPut|MapPatch|MapDelete)\("([^"]+)"' -AllMatches

# Web
npm --prefix GrottoWorks_FE run lint    # tsc -b
npm --prefix GrottoWorks_FE run test    # 39 file / 154 test

# Mobile
npm --prefix GrottoWokrs_Mobile run typecheck
```

Grep nhanh để xác nhận "còn mock hay không":
`grep -r "loadDb\|withFallback\|mock" GrottoWokrs_Mobile/src` → chỉ còn `mockScene.ts` (3D, BE chưa có API).
