# GrottoWorks Backend — Spec phân chia task theo function

**Phiên bản:** 1.3 · **Ngày:** 14/10/2026 · *v1.3: đồng bộ path / operationId / event với API Contract v1.3; season tạo ở PLANNING (bỏ DRAFT); duyệt vật tư hỗ trợ gọi đồng bộ; danh mục event thống nhất. v1.2 (07/10/2026): chốt D-01 → D-06*
**Team BE:** Nguyễn Tiến Lâm (Leader) · Nguyễn Trí Vũ
**Thời gian dự án:** 09/2026 → 03/2027

**Tài liệu nguồn**

| Tài liệu | Vai trò trong spec này |
|---|---|
| `GrottoWorks_Phan_Tich_Nghiep_Vu_Chi_Tiet_v1.2.md` (BA v1.2) | Nguồn chuẩn cho nghiệp vụ, state machine, business rule (BR-01 → BR-70) |
| `ERD_Grottowork_v3.6.txt` (ERD v3.6) | Nguồn chuẩn cho data model |
| `GrottoWorks_API_Contract_v1.3.docx` (API v1.3) | Nguồn chuẩn cho endpoint, operationId, schema, error format |
| `FA26SE067_..._thinhdp2.docx` | Phạm vi đăng ký đồ án |
| Repo `GrottoWorks_BE` (main) | Hiện trạng: đã có `ParishCoordination` (Clean Architecture, .NET 10, EF Core + Npgsql, Minimal API) |

> **Ba tài liệu trên đã được đồng bộ theo quyết định nhóm (Mục 9).** Quy tắc khi lệch:
> 1. **Endpoint, path, operationId, role, tên event:** theo API Contract. Spec và `plan.md` không tự đặt path/tên riêng.
> 2. **API đã implement:** giữ nguyên tài liệu của API đó; nếu cần đổi thì đổi code trước, sau đó cập nhật tài liệu trong cùng PR.
> 3. Nghiệp vụ còn lại: quyết định nhóm > BA > ERD > API Contract, và cập nhật lại tài liệu trong cùng PR.

---

## 1. Tóm tắt phân công

| Owner | Service / Module | Phạm vi function |
|---|---|---|
| **Lâm** | Platform | API Gateway (YARP), BuildingBlocks, docker-compose, RabbitMQ/MassTransit, Outbox/Inbox, JWT validation chung |
| **Lâm** | `ParishCoordination` (đang làm) | Parish, Community, Season, Work Area, Work Area Category, gán Leader/Officer |
| **Lâm** | `TaskExecution` | Task, Skill yêu cầu, Assignment/Application, Progress & Evidence, Review, Attendance & Correction, Support Request (nhân lực + vật tư, D-03) |
| **Lâm** | `NotificationAudit` | Notification, Activity Log, File upload/download |
| **Vũ** | `Identity` | Đăng ký/đăng nhập/JWT/refresh, password reset, User/Role, Volunteer Profile, Skill catalog |
| **Vũ** | `Resource` | Material catalog, Requirement, Inventory ledger, Shortage, Allocation, Task Material Report, Purchase, Donor/Donation, Borrowed Item |
| **Vũ** | `Readiness` | Checklist, Item result, Corrective task, Recheck, Confirm/Reopen |
| **Vũ** | `Reporting` | Contribution, Recognition rule/point, Dashboard, Report, Export |

**Lý do chia như vậy**

- Chia theo **bounded context**, mỗi người sở hữu trọn service (code + DB + migration + test), tránh 2 người cùng sửa một service.
- Lâm đã làm `ParishCoordination` và là leader → giữ luôn phần nền tảng (gateway, messaging) và chuỗi nghiệp vụ **Season → Work Area → Task → Assignment → Attendance → Progress** (chuỗi 1, 2, 5 trong BA mục 20).
- Vũ sở hữu chuỗi **Requirement → Purchase/Donation/Borrow → Confirm → Allocation → Material Report** (chuỗi 3, 4), **Readiness** (chuỗi 6) và **Reporting** — các module chủ yếu tiêu thụ event.
- Ước lượng khối lượng: Lâm ≈ 95 endpoint + hạ tầng; Vũ ≈ 100 endpoint, logic tồn kho nặng hơn. Hai bên tương đương.

---

## 2. Kiến trúc mục tiêu

```text
Web (Committee/Leader/MO/Admin)    Mobile (Volunteer)
                 │  HTTPS /api/v1
                 ▼
        ┌──────────────────┐
        │  ApiGateway (YARP)│  JWT validate, rate limit, X-Correlation-Id
        └────────┬─────────┘
   ┌──────┬──────┼────────┬──────────┬──────────┬───────────┐
   ▼      ▼      ▼        ▼          ▼          ▼           ▼
Identity Parish  Task   Resource  Readiness  Reporting  NotificationAudit
 (Vũ)   (Lâm)  (Lâm)    (Vũ)       (Vũ)       (Vũ)        (Lâm)
   │      │      │        │          │          │           │
 idDb  parishDb taskDb resourceDb readyDb  reportDb    notifyDb   ← PostgreSQL, 1 DB/service
   └──────┴──────┴────────┴────┬─────┴──────────┴───────────┘
                               ▼
                    RabbitMQ (integration events)
                    Redis (cache, rate limit, refresh-token denylist)
                    MinIO (file object storage)
```

### 2.1 Quy ước bắt buộc (cả 2 người)

| Hạng mục | Quy ước |
|---|---|
| Cấu trúc service | `Services/<Name>/<Name>.Api / .Application / .Domain / .Infrastructure / .Contracts` + `tests/` — theo mẫu `ParishCoordination` hiện tại |
| API | Minimal API endpoint group, base path `/api/v1`, `WithName(operationId)` khớp API Contract, `WithTags(<Tag>)` |
| Response | Envelope `{ data, meta }` / `{ data, page, meta }` theo API Contract 4.2 |
| Lỗi | `application/problem+json` + `code` ổn định + `correlationId` (vd. `TASK_INVALID_STATE`), 422 cho validation, 409 cho vi phạm business rule |
| Enum JSON | `SNAKE_CASE_UPPER` (đã cấu hình trong Program.cs của Parish) |
| Thời gian | `timestamptz`, UTC, timestamp do server sinh cho mọi thay đổi trạng thái |
| ID | UUID; **không FK chéo service**, chỉ lưu ID tham chiếu |
| Concurrency | Cột `version` (xmin hoặc int) trên aggregate có thể sửa; `If-Match` sai → 412 |
| Idempotency | `Idempotency-Key` bắt buộc cho (API Contract 4.1): `recordPurchase`, `recordDirectDonation`, `recordDonationReceipt`, `requestCheckIn`, `recordCheckOut`, `createReportExport`, `createSystemDataExport`, `startBackup`; nội bộ: `POST /internal/tasks`, `POST /internal/support-transfers` |
| Xóa | Không xóa cứng record đã có giao dịch → cancel / deactivate / archive |
| Event | Envelope theo API Contract mục 8 (`eventId, eventType, occurredAt, correlationId, actorId, parishId, seasonId, aggregateType, aggregateId, aggregateVersion, payload`); publish qua Outbox; consumer idempotent theo `eventId` |
| Authorization | Role check từ JWT **+** scope check (parish / work area / self) trong Application layer |
| Branch | `main` ← PR bắt buộc review chéo; branch `feat/<service>/<function-id>` (vd. `feat/resource/F-RES-05`) |
| Commit | `<type>(<service>): <mô tả>` — vd. `feat(Resource): confirm purchase delivery` |
| DoD mỗi function | Endpoint chạy qua gateway · validation + ProblemDetails · scope check · unit test cho domain rule · integration test happy path + 1 lỗi chính · OpenAPI hiển thị đúng · event (nếu có) publish qua outbox |

### 2.1a Tag API Contract → service

| Tag (API Contract) | Service triển khai |
|---|---|
| User | Identity |
| Parish | ParishCoordination |
| Tasks, Attendance, Support | TaskExecution |
| Materials, Purchases, Contributions | Resource |
| Readiness | Readiness |
| Reports | Reporting |
| NotificationAudit | NotificationAudit |

**Ngoại lệ:** `confirmSupportMaterialReceipt` (tag Support) do **Resource** host vì ghi ledger TRANSFER_IN. Gateway giữ đúng path Contract (không thêm prefix service); prefix dùng chung giữa nhiều service (`/users`, `/seasons`, `/work-areas`, `/tasks`, `/purchase-requests`) khai báo route cụ thể trước route chung.

### 2.2 Xác thực và phân quyền

- Identity (Vũ) phát hành JWT: `sub` (userId), `role` (đúng **1 role**/tài khoản theo ERD), `parishId`, `communityId` (nếu có).
- Role code: `ADMIN`, `PARISH` (Committee), `LEADER`, `MO`, `VOLUNTEER`.
- Scope work area **không nằm trong token**. Task/Resource/Readiness giữ **bảng projection** `work_area_scope(work_area_id, season_id, parish_id, leader_user_id, material_officer_user_id, status)` cập nhật từ event `planning.work-area.*` của Parish.
- Gateway và từng service đều validate JWT (dùng chung package `BuildingBlocks.Security` do Lâm viết).

---

## 3. Danh sách service và data ownership

| Service | Owner | Bảng ERD v3.6 sở hữu | Bảng bổ sung (cần chốt, xem Mục 9) |
|---|---|---|---|
| Identity | Vũ | `app_user` (+ `parish_id`), `role`, `volunteer_profile`, `volunteer_availability`, `skill`, `volunteer_skill` | `refresh_token`, `password_reset_token` (kỹ thuật) |
| ParishCoordination | Lâm | `parish`, `community`, `christmas_season` (+ `estimated_budget`), `work_area_category`, `work_area` | — |
| TaskExecution | Lâm | `task` (+ `task_type`, `purchase_request_id`, `completion_criteria`), `task_skill`, `task_material`, `task_assignment`, `task_progress`, `progress_evidence`, `attendance_request`, `attendance`, `attendance_correction`, `support_request`, `support_request_position`, `support_position_skill`, `support_request_material`, `support_response`, `support_response_member`, `support_response_material` | — |
| Resource | Vũ | `material_category`, `material`, `material_requirement`, `material_transaction`, `material_allocation`, `material_allocation_item`, `task_material_report`, `task_material_report_item`, `purchase_request`, `purchase_request_item`, `purchase`, `purchase_item`, `donor`, `donation`, `donation_item`, `donation_receipt`, `donation_receipt_item`, `borrowed_item`, `borrow_transaction` | — |
| Readiness | Vũ | `readiness_checklist`, `readiness_item` (+ `corrective_task_id`) | — |
| Reporting | Vũ | `contribution_type`, `contribution`, `recognition_rule` (có version), `recognition_point` (+ `contribution_id`) | `report_export_job`, read-model tổng hợp |
| NotificationAudit | Lâm | `notification`, `notification_preference`, `activity_log`, `file_object` | `outbox/inbox` (mỗi service tự có) |

> `task_material` thuộc TaskExecution (Leader khai báo khi tạo task). Resource nhận bản sao qua event `task.materials.changed` để tính nhu cầu và kiểm tra allocation.

---

## 4. Function của Lâm

Ký hiệu ưu tiên: **P0** = bắt buộc cho demo luồng chính, **P1** = cần cho baseline nghiệm thu, **P2** = nếu kịp.

### 4.1 Platform (PLT)

| ID | Function | Mô tả / Acceptance criteria | Ưu tiên | Sprint |
|---|---|---|---|---|
| F-PLT-01 | Dọn repo & chuẩn hóa template service | Xóa `weatherforecast` khỏi Parish; thêm `.Contracts`, `tests/`; README hướng dẫn tạo service mới | P0 | S0 |
| F-PLT-02 | docker-compose local | PostgreSQL (1 instance, nhiều DB), RabbitMQ (management UI), Redis, MinIO; `docker compose up` chạy được toàn bộ | P0 | S0 |
| F-PLT-03 | BuildingBlocks.Web | ProblemDetails + error code chuẩn, response envelope, paging/sort parser (size ≤ 100, field lạ → 400), correlation-id middleware, health check `/health/live` `/health/ready` | P0 | S0 |
| F-PLT-04 | BuildingBlocks.Security | JWT validation, `ICurrentUser` (userId, role, parishId), policy theo role | P0 | S1 |
| F-PLT-05 | BuildingBlocks.Messaging | MassTransit + RabbitMQ, event envelope, Outbox (EF Core), Inbox/idempotent consumer, retry 3 lần → DLQ | P0 | S1 |
| F-PLT-06 | ApiGateway (YARP) | Route theo path API Contract (bảng 2.1a, route cụ thể cho prefix dùng chung), validate JWT, rate limit cho `/auth/login`, `/auth/register`, `/auth/password/*`, `/files/upload-requests`, `/report-exports`, `/system-data-exports`; `/internal/**` không expose | P0 | S1 |
| F-PLT-07 | Idempotency-Key middleware | Lưu key + response 24h trong Redis; dùng chung cho các service | P1 | S3 |
| F-PLT-08 | OpenAPI tổng hợp | Gom OpenAPI từng service qua gateway (Scalar/Swagger UI) cho FE/Mobile | P1 | S2 |
| F-PLT-09 | CI | GitHub Actions: build + test + format check cho mọi PR | P1 | S2 |

### 4.2 ParishCoordination (PAR) — tag `Parish`

| ID | Function | Endpoint (operationId) | Business rule / AC | Ưu tiên | Sprint |
|---|---|---|---|---|---|
| F-PAR-01 | Quản lý Parish | `listParishes` ✅, `createParish` ✅, `getParish`, `updateParish` | Chỉ ADMIN/PARISH; status ACTIVE/INACTIVE | P0 | S0 |
| F-PAR-02 | Quản lý Community | `listCommunities`, `createCommunity`, `updateCommunity` | Deactivate thay vì xóa; tên duy nhất trong parish | P0 | S1 |
| F-PAR-03 | ~~Parish member~~ | **Bỏ** — parish là thuộc tính `app_user.parish_id` (Identity). Danh sách thành viên dùng `listParishUsers` (F-IDN-10) | — | — | — |
| F-PAR-04 | Work area category | `listWorkAreaCategories` (USER), `createWorkAreaCategory`, `updateWorkAreaCategory` (ADMIN) | Đã được tham chiếu → chỉ deactivate | P1 | S1 |
| F-PAR-05 | Season CRUD | `listSeasons`, `createSeason`, `getSeason`, `updateSeason` | BR-01, BR-02 (tên/năm duy nhất trong parish), BR-03; season tạo ở PLANNING; `updateSeason` chỉ khi PLANNING | P0 | S1 |
| F-PAR-06 | Season lifecycle | `activateSeason`, `completeSeason`, `archiveSeason` | Season tạo ở PLANNING (không có DRAFT); PLANNING → ACTIVE → READY (nội bộ, khi readiness đạt) → COMPLETED → ARCHIVED. Không activate khi chưa có work area hoặc work area thiếu leader/officer (BA 8.1); không 2 season ACTIVE trùng thời gian; BR-07 archive → read-only. Event `planning.season.activated/completed/archived` | P0 | S2 |
| F-PAR-07 | Work area CRUD | `listWorkAreas`, `createWorkArea`, `getWorkArea`, `updateWorkArea` | BR-04, BR-05; lịch nằm trong season | P0 | S1 |
| F-PAR-08 | Gán community / leader / officer | `assignWorkAreaCommunity`, `assignWorkAreaLeader`, `assignMaterialOfficer` | BR-06; user phải ACTIVE (gọi Identity hoặc projection user); event `planning.work-area.assigned` → Notification | P0 | S2 |
| F-PAR-09 | Work area status | Nhận event từ Task/Readiness để chuyển PLANNED → IN_PROGRESS → READINESS_REVIEW → NEEDS_CORRECTION/READY → COMPLETED | Consume `task.status.changed`, `readiness.area.ready`, `readiness.area.reopened` | P1 | S5 |
| F-PAR-10 | Work area progress | `getWorkAreaProgress` | Read-model % task completed, nhận từ event Task | P1 | S5 |

### 4.3 TaskExecution (TSK) — tag `Tasks`, `Attendance`, `Support`

| ID | Function | Endpoint (operationId) | Business rule / AC | Ưu tiên | Sprint |
|---|---|---|---|---|---|
| F-TSK-01 | Projection scope | Consumer `planning.work-area.*`, `planning.season.*`, `identity.user.locked` | Bảng `work_area_scope`, `user_status_projection` | P0 | S2 |
| F-TSK-02 | Task CRUD | `listTasks`, `createTask`, `getTask`, `updateTask` | BR-08, BR-09 (override cần lý do); LEADER đúng work area; season ARCHIVED → chặn | P0 | S2 |
| F-TSK-03 | Publish / cancel task | `publishTask`, `cancelTask` | Cancel bắt buộc reason; task có assignment không xóa cứng | P0 | S2 |
| F-TSK-04 | Skill & material & criteria của task | `replaceTaskSkills`, `replaceTaskMaterials`, `replaceCompletionCriteria` | BR-10; event `task.materials.changed` cho Resource | P0 | S3 |
| F-TSK-05 | Volunteer matching | `listVolunteerMatches` | Gợi ý theo skill, community, availability, xung đột lịch (BR-15) — đọc profile từ Identity qua HTTP (có cache Redis) | P1 | S4 |
| F-TSK-06 | Assign trực tiếp | `assignVolunteer`, `listTaskAssignments`, `acceptTaskAssignment`, `declineTaskAssignment` | BR-11, BR-12, BR-13, BR-14 (capacity, override + reason); event `task.assignment.created/responded` | P0 | S3 |
| F-TSK-07 | Đăng ký tự nguyện | `applyForTask`, `listTaskApplications`, `approveTaskApplication`, `rejectTaskApplication` | Assignment PENDING; duyệt giảm capacity; 2 người tranh slot cuối → optimistic concurrency (edge case #4) | P0 | S3 |
| F-TSK-08 | Task của tôi | `listMyTasks` | `TaskFilter` (status, taskType, seasonId) | P0 | S3 |
| F-TSK-09 | Progress & evidence | `listTaskProgress`, `addTaskProgress` | Mỗi lần là record mới, không ghi đè; file phải là fileId đã `complete` | P0 | S4 |
| F-TSK-10 | Submit / review | `submitTaskForReview`, `getTaskReviewPackage`, `approveTaskCompletion`, `requestTaskRevision`, `resubmitTask` | BR-16, BR-17; approve set `approved_by/at` + task COMPLETED atomically (ERD check `task_progress_completion_approval`); event `task.progress.submitted` (khi submit/resubmit), `task.completion.reviewed` (Resource, Reporting, Parish consume) | P0 | S4 |
| F-TSK-11 | Check-in có Leader duyệt (D-06) | `requestCheckIn`, `cancelCheckInRequest`, `listAttendanceRequests`, `approveAttendanceRequest`, `rejectAttendanceRequest`, `recordCheckOut` | BR-18, BR-19, BR-58 → BR-61: chỉ assignment ACCEPTED; tối đa 1 request PENDING hoặc 1 phiên OPEN; Leader duyệt **từng request**, không có endpoint duyệt hàng loạt; reject bắt buộc lý do; `check_in_at = requested_at`; check-out trực tiếp; tính `working_hours`; event `attendance.request.submitted/reviewed` | P0 | S4 |
| F-TSK-12 | Attendance correction | `requestAttendanceCorrection`, `listAttendanceCorrections`, `approveAttendanceCorrection`, `rejectAttendanceCorrection` | BR-20, BR-21, BR-22 (lưu giờ cũ/mới/lý do/người duyệt); event `attendance.hours.approved` | P1 | S5 |
| F-TSK-13 | Báo cáo giờ & phiên mở lâu | `listMyAttendance`, `getWorkingHours`, `listLongOpenSessions` + job định kỳ | Event `attendance.session.open-too-long` | P1 | S5 |
| F-TSK-14 | Support request | `listSupportRequests`, `createSupportRequest`, `getSupportRequest`, `updateSupportRequest`, `publishSupportRequest`, `closeSupportRequest` | `support_type` = STAFFING / MATERIAL / MIXED (quyết định D-03). Phần nhân lực: nhiều position, mỗi position nhiều skill (AND). Phần vật tư: danh sách material + số lượng. Submit phải có ≥ 1 position **hoặc** ≥ 1 material item | P1 | S6 |
| F-TSK-15 | Support response — nhân lực | `respondToSupportRequest`, `listSupportResponses`, `approveSupportMember`, `rejectSupportMember` | Kiểm tra skill, capacity, 1 người ≤ 1 position active/request; approve → tạo assignment | P1 | S6 |
| F-TSK-15b | Support response — vật tư | `respondToSupportRequest` (phần materials, chỉ MO của work area nguồn), `approveSupportMaterial`, `rejectSupportMaterial` | BR-62, BR-63: Leader bên cần duyệt → Task **gọi đồng bộ** Resource `POST /internal/support-transfers` (ghi TRANSFER_OUT, thiếu unallocated → 409 `MATERIAL_INSUFFICIENT_AVAILABLE` trả thẳng cho Leader); chỉ khi Resource trả 200 mới set APPROVED và publish `support.material.approved` (thông báo). MO bên nhận xác nhận bằng `confirmSupportMaterialReceipt` (F-RES-20, Resource host). Request tự chuyển FULFILLED / PARTIALLY_FULFILLED khi đủ người **và** đủ vật tư | P1 | S7 |
| F-TSK-17 | Task mua hàng | `createPurchaseTask` — `POST /purchase-requests/{requestId}/purchase-tasks` (Leader) — tạo task `task_type = PURCHASE` gắn `purchase_request_id`, dùng lại `assignVolunteer` / `acceptTaskAssignment` / `declineTaskAssignment` | Quyết định D-01: thay cho purchase assignment riêng. Chỉ tạo khi PR đã APPROVED (nhận từ event `purchase.request.decided`). Buyer accept → publish `task.purchase-task.assigned`. Khi nhận `purchase.delivery.confirmed` → tự chuyển task COMPLETED | P0 | S4 |
| F-TSK-16 | Support summary | `getCrossCommunitySupportSummary` | PARISH xem toàn season | P2 | S7 |

### 4.4 NotificationAudit (NTF) — tag `NotificationAudit` + File

| ID | Function | Endpoint (operationId) | Business rule / AC | Ưu tiên | Sprint |
|---|---|---|---|---|---|
| F-NTF-01 | File upload | `createUploadRequest`, `completeUpload`, `getDownloadUrl`, `deleteUnattachedFile` + bảng `file_object` | BR-69; presigned URL MinIO ngắn hạn; giới hạn type/size theo API Contract mục 9; gắn file qua `reference_type/reference_id` | P0 | S3 |
| F-NTF-02 | Notification từ event | Consumer cho danh sách sự kiện BA mục 14 | Mapping event → template; không chứa dữ liệu nhạy cảm | P1 | S5 |
| F-NTF-03 | API notification | `listNotifications`, `markNotificationRead`, `markAllNotificationsRead`, `get/replaceNotificationPreferences`, `updateNotificationRule` | — | P1 | S5 |
| F-NTF-04 | Activity log | Consumer ghi log mọi event approve/confirm/reject/lock/export; `listActivityLogs`, `getActivityLog`, `getResourceHistory` | Append-only; không log password/token/URL file | P1 | S6 |
| F-NTF-05 | Push notification mobile | FCM cho volunteer | — | P2 | S8 |
| F-NTF-06 | System export / backup | `createSystemDataExport`, `startBackup`, `getBackupStatus` | 202 + job | P2 | S8 |

---

## 5. Function của Vũ

### 5.1 Identity (IDN) — tag `User`

| ID | Function | Endpoint (operationId) | Business rule / AC | Ưu tiên | Sprint |
|---|---|---|---|---|---|
| F-IDN-01 | Scaffold service + seed role | Service Identity theo template; seed 5 role, tài khoản ADMIN mặc định | — | P0 | S0 |
| F-IDN-02 | Register / login | `registerUser`, `login` | Hash mật khẩu (ASP.NET Identity `PasswordHasher` hoặc BCrypt); email unique, normalize; tài khoản LOCKED/INACTIVE không login được; event `identity.user.registered` | P0 | S1 |
| F-IDN-03 | Token | `refreshToken`, `logout` | Access token ngắn hạn (15'), refresh token rotate, phát hiện reuse → revoke cả chuỗi | P0 | S1 |
| F-IDN-04 | Password reset | `requestPasswordReset`, `resetPassword` | Token 1 lần, hết hạn; revoke refresh token sau reset | P1 | S2 |
| F-IDN-05 | Profile của tôi | `getMyProfile`, `updateMyProfile` | — | P0 | S1 |
| F-IDN-06 | Skill catalog | `listSkills`, `createSkill`, `updateSkill` | Đã tham chiếu → deactivate | P0 | S1 |
| F-IDN-07 | Volunteer profile, skill, lịch rảnh | `replaceMySkills`, `replaceMyAvailability`, community | `volunteer_skill` unique (user, skill); `volunteer_availability` có `available_to > available_from` (BR-70) | P0 | S2 |
| F-IDN-08 | Admin user | `listUsers`, `createUser`, `updateUser`, `lockUser`, `unlockUser`, `assignUserRole`, `listAssignableRoles` | 1 role/account; `parish_id` bắt buộc trừ ADMIN (BR-67); lock → revoke token + event `identity.user.locked` | P0 | S2 |
| F-IDN-09 | Internal user lookup | `GET /internal/users?ids=` (chỉ service-to-service) | Trả userId, tên, status, role, communityId, skills; dùng cho Parish/Task/Resource | P0 | S2 |
| F-IDN-10 | Danh sách user trong parish | `listParishUsers` (`GET /users`) | PARISH/LEADER/MO chỉ thấy user cùng parish; filter community, role, skill | P1 | S2 |

### 5.2 Resource (RES) — tag `Materials`, `Purchases`, `Contributions`

| ID | Function | Endpoint (operationId) | Business rule / AC | Ưu tiên | Sprint |
|---|---|---|---|---|---|
| F-RES-01 | Projection scope | Consumer `planning.work-area.*`, `planning.season.*`, `task.materials.changed`, `task.completion.reviewed`, `task.purchase-task.assigned` | Bảng `work_area_scope`, `task_projection` | P0 | S2 |
| F-RES-02 | Material category & catalog | `listMaterialCategories`, `createMaterialCategory`, `updateMaterialCategory`, `listMaterials`, `createMaterial`, `updateMaterial` | Code unique; đơn vị tính lấy từ material | P0 | S2 |
| F-RES-03 | Material requirement | `listMaterialRequirements`, `createMaterialRequirement`, `updateMaterialRequirement` | Unique (work_area, material); quantity > 0; MO đúng work area | P0 | S3 |
| F-RES-04 | Inventory ledger + opening stock | `recordOpeningStock`, `listInventory`, `getAllocationHistory` | **Mọi thay đổi stock đều qua `material_transaction`** (append-only). Stock tách theo work area (BR-23, BR-24) | P0 | S3 |
| F-RES-05 | Shortage | `getWorkAreaShortages`, `listMaterialNeeds` | Công thức BA 8.5; shortage ≥ 0 (BR-25); event `material.shortage.changed` | P0 | S3 |
| F-RES-06 | Purchase request | `listPurchaseRequests`, `createPurchaseRequest`, `getPurchaseRequest`, `updatePurchaseRequest`, `submitPurchaseRequest`, `getPurchaseRequestHistory` | BR-27; MO tạo, nhiều item; không tạo khi shortage = 0 trừ override + lý do; mọi request submit đều chờ Leader duyệt | P0 | S4 |
| F-RES-07 | Duyệt purchase request | `approvePurchaseRequest`, `rejectPurchaseRequest` (**role LEADER** của đúng work area — quyết định D-01) | Reject bắt buộc lý do (BR-31); event `purchase.request.submitted/decided` → Task service mở cho Leader tạo task mua | P0 | S4 |
| F-RES-08 | ~~Giao người mua~~ | **Bỏ** (API v1.1 cũ có endpoint giao người mua riêng) | Gộp vào task mua F-TSK-17 (Lâm). Resource consume `task.purchase-task.assigned` để lưu buyer, PR → `BUYER_ASSIGNED` | — | — |
| F-RES-09 | Ghi nhận mua | `recordPurchase`, `attachPurchaseReceipt` | Chỉ volunteer đang giữ assignment ACCEPTED của task PURCHASE tương ứng mới được ghi; status PURCHASED, **stock chưa tăng** (BR-26); file hóa đơn từ F-NTF-01; Idempotency-Key | P0 | S5 |
| F-RES-10 | Xác nhận giao hàng | `confirmPurchaseDelivery`, `requestPurchaseInfo` | MO nhập usable quantity → ghi `material_transaction` + tính lại shortage trong **1 transaction**; giao thiếu → request còn phần dở; event `purchase.delivery.confirmed` | P0 | S5 |
| F-RES-11 | Vượt dự toán | `approveCostOverrun` (LEADER) | BR-32: `total_cost` > dự toán đã duyệt → chặn `confirmPurchaseDelivery` cho tới khi Leader duyệt; lưu `purchase.cost_overrun_approved_by/at` | P1 | S6 |
| F-RES-12 | Donor | `listDonors`, `createDonor`, `updateDonor` (MO, ADMIN) | Không auto-merge theo tên/sđt; link `user_id` tùy chọn | P0 | S5 |
| F-RES-13 | Donation ghi nhận trực tiếp | `listDonations`, `recordDirectDonation` (`POST /donations`), `getDonation`, `listMyContributions` | MO ghi khoản đã nhận ngay: tạo donation status RECEIVED + 1 receipt trong 1 transaction; MONEY: amount > 0; MATERIAL: chỉ usable vào `material_transaction` của **work area đích**; event `contribution.receipt.confirmed` (cho receipt đầu tiên) | P0 | S5 |
| F-RES-13b | Pledge (D-02) | `listContributionNeeds`, `pledgeMaterialDonation`, `pledgeMonetarySupport` (Volunteer), `cancelDonationPledge` | Donation PLEDGED; donor tự tạo/link theo `user_id`; MATERIAL bắt buộc `work_area_id`; hủy cần lý do; **pledge không bao giờ tăng stock** (BR-39); event `contribution.pledged` | P1 | S6 |
| F-RES-13c | Ghi nhận lần nhận | `recordDonationReceipt` (`POST /donations/{id}/receipts`) | BR-38: mỗi lần nhận = 1 `donation_receipt` (+ items, unusable nhập ngay trên receipt); cộng dồn received/unusable; PARTIALLY_RECEIVED → RECEIVED; nhận vượt pledge → cảnh báo, ghi theo thực nhận; post stock 1 lần/receipt (unique `reference_type, reference_id`); event `contribution.receipt.confirmed` | P1 | S6 |
| F-RES-14 | Điều chỉnh donation | `adjustDonation` — `PATCH /donations/{donationId}` | Reconcile delta với stock và contribution (ERD note); event `contribution.donation.adjusted` | P2 | S8 |
| F-RES-20 | Chuyển kho hỗ trợ vật tư | `POST /internal/support-transfers` (Task gọi đồng bộ) + `confirmSupportMaterialReceipt` — `POST /support-response-materials/{itemId}/confirm-receipt` (MO bên nhận) | Quyết định D-03. Internal: khóa tồn kho nguồn, kiểm tra unallocated, ghi `TRANSFER_OUT` (409 `MATERIAL_INSUFFICIENT_AVAILABLE` nếu thiếu), idempotent theo itemId. Confirm: ghi `TRANSFER_IN` usable cho work area nhận; `reference_type = SUPPORT_RESPONSE_MATERIAL`; event `support.transfer.confirmed` | P1 | S7 |
| F-RES-15 | Allocation | `allocateMaterial`, `listMaterialAllocations` | BR-33: tổng allocation ≤ unallocated; 2 MO cùng lấy phần cuối → lock row / serializable; lỗi `MATERIAL_INSUFFICIENT_AVAILABLE` | P0 | S6 |
| F-RES-16 | Điều chỉnh tồn kho | `createInventoryAdjustment` | Damaged/lost/disposed không quay lại usable (BR-43) | P1 | S6 |
| F-RES-17 | Task material report | `submitTaskMaterialReport` (Volunteer), `listTaskMaterialReports`, `verifyTaskMaterialReport`, `requestMaterialReportRevision` (MO) | BR-44 → BR-51: chỉ khi task COMPLETED; chỉ từ allocation item của task; remaining+damaged+lost ≤ allocated; chỉ `verified_*` trả remaining về stock cùng work area; không sửa `allocated_quantity`; event `material-report.submitted`, `material-report.verified` | P0 | S7 |
| F-RES-18 | Loan offer & borrowed item (D-05) | `offerItemForLoan` (Volunteer nhập giùm người cho mượn), `listLoanOffers`, `acceptLoanOffer`, `rejectLoanOffer`, `listBorrowedItems`, `issueBorrowedItem`, `assignBorrowedItem`, `returnBorrowedItem`, `recordBorrowedItemIncident`, `listOverdueBorrowedItems` | BR-41 → BR-43: bắt buộc `owner_name` + `expected_return_date`; `offered_by_user_id` = volunteer nhập, là đầu mối liên hệ; đồ mượn không vào stock; job nhắc trước/sau hạn → event `borrowed-item.return-due` | P1 | S7 |
| F-RES-19 | ~~Loan offer~~ | Gộp vào F-RES-18 | — | — | — |

### 5.3 Readiness (RDY) — tag `Readiness`

| ID | Function | Endpoint (operationId) | Business rule / AC | Ưu tiên | Sprint |
|---|---|---|---|---|---|
| F-RDY-01 | ~~Template~~ | Bỏ khỏi baseline (nhóm dùng `readiness_checklist` + `readiness_item` có sẵn) | Chỉ làm nếu sau này bổ sung bảng template (xem Mục 10, mục tùy chọn) | P2 | — |
| F-RDY-02 | Tạo checklist | `createReadinessCheck`, `replaceReadinessItems`, `getReadinessCheck` | D-04: Leader tự nhập item, `is_required`; chỉ sửa item khi DRAFT | P1 | S6 |
| F-RDY-03 | Ghi kết quả & evidence | `recordReadinessResult`, `attachReadinessEvidence` | Failed → event `readiness.item.failed` | P1 | S7 |
| F-RDY-04 | Corrective task | `linkCorrectiveTask` | Gọi Task service tạo task (HTTP, có Idempotency-Key) hoặc link task có sẵn; mandatory failed bắt buộc có corrective task | P1 | S7 |
| F-RDY-05 | Recheck / confirm / reopen | `recheckReadinessItem`, `confirmAreaReadiness`, `reopenReadinessCheck` | BR-52 (còn mandatory chưa pass → 409), BR-53 (reopen cần lý do); event `readiness.area.ready/reopened` | P1 | S7 |
| F-RDY-06 | Readiness toàn season | `getSeasonReadiness` | — | P1 | S8 |

### 5.4 Reporting (RPT) — tag `Reports`

| ID | Function | Endpoint (operationId) | Business rule / AC | Ưu tiên | Sprint |
|---|---|---|---|---|---|
| F-RPT-01 | Contribution type & recognition rule | `list/create/updateContributionType`, `list/createRecognitionRule`, `deactivateRecognitionRule` | Đổi điểm = tạo version mới, có `effective_from/to` (BR-68) | P1 | S6 |
| F-RPT-02 | Sinh contribution & point | Consumer `task.completion.reviewed`, `attendance.hours.approved`, `purchase.delivery.confirmed`, `contribution.receipt.confirmed`, `contribution.donation.adjusted`, `support.member.completed` | BR-54, BR-55 (không tính 2 lần — unique theo eventId/reference), BR-56; lưu rule version; event `recognition.entry.created` | P1 | S7 |
| F-RPT-03 | Recognition | `getMyRecognition`, `getSeasonRecognition` | Donor ngoài không có account → không có point | P1 | S7 |
| F-RPT-04 | Dashboard | `viewSeasonDashboard`, `viewWorkAreaDashboard`, `viewMaterialDashboard` | Read-model build từ event; load < 3s (NFR) | P1 | S8 |
| F-RPT-05 | Báo cáo | `getTaskReport`, `getMaterialReport`, `getPurchaseReport`, `getContributionReport`, `getAttendanceReport`, `getReadinessReport` | BR-57; ghi rõ parish, season, thời gian tạo, bộ lọc | P1 | S8 |
| F-RPT-06 | Export | `createReportExport`, `getReportExport` | 202 + job; file xlsx/pdf lưu MinIO | P2 | S9 |

---

## 6. Hợp đồng tích hợp giữa 2 người

### 6.1 Integration event

| Event | Publisher (owner) | Consumer | Ghi chú |
|---|---|---|---|
| `identity.user.registered` | Identity (Vũ) | NotificationAudit | — |
| `identity.user.locked` / `unlocked` | Identity (Vũ) | Task, Parish, NotificationAudit | Task chặn assign user bị khóa (edge case #24) |
| `planning.season.activated` / `completed` / `archived` | Parish (Lâm) | Task, Resource, Readiness, Reporting | Archived → các service chặn ghi |
| `planning.work-area.created` / `updated` / `assigned` | Parish (Lâm) | Task, Resource, Readiness, Reporting, NotificationAudit | Payload gồm leader/officer/community/category |
| `task.materials.changed` | Task (Lâm) | Resource | Danh sách `task_material` |
| `task.assignment.created` / `responded` | Task (Lâm) | NotificationAudit | Chỉ để thông báo |
| `task.purchase-task.assigned` | Task (Lâm) | Resource | Buyer accept task PURCHASE → Resource lưu buyer, PR `BUYER_ASSIGNED` |
| `task.progress.submitted` | Task (Lâm) | NotificationAudit | Volunteer submit task để nghiệm thu |
| `task.status.changed` | Task (Lâm) | Parish, Reporting | — |
| `task.completion.reviewed` | Task (Lâm) | Resource, Reporting, Parish, NotificationAudit | Resource mở cho volunteer gửi material report |
| `attendance.hours.approved` | Task (Lâm) | Reporting | Chỉ giờ đã duyệt (BR-21) |
| `attendance.session.open-too-long` | Task (Lâm) | NotificationAudit | — |
| `support.request.published` / `support.member.completed` | Task (Lâm) | NotificationAudit, Reporting | — |
| `support.material.approved` | Task (Lâm) | NotificationAudit | Chỉ thông báo; trừ kho nguồn đã làm đồng bộ qua `POST /internal/support-transfers` |
| `support.transfer.confirmed` | Resource (Vũ) | Task | Task cập nhật số lượng đã đáp ứng của support request |
| `contribution.pledged` / `contribution.receipt.confirmed` | Resource (Vũ) | Reporting, NotificationAudit | Chỉ receipt mới tính contribution/điểm; receipt của donation trực tiếp cũng publish `contribution.receipt.confirmed` |
| `material.shortage.changed` | Resource (Vũ) | Reporting | — |
| `purchase.request.submitted` / `decided` | Resource (Vũ) | Task, NotificationAudit | Task chỉ cho tạo task PURCHASE khi APPROVED |
| `purchase.delivery.confirmed` | Resource (Vũ) | Task, Reporting, NotificationAudit | Task tự đóng task PURCHASE |
| `contribution.donation.adjusted` | Resource (Vũ) | Reporting | Reporting điều chỉnh contribution 1-1 với donation |
| `material-report.submitted` / `verified` | Resource (Vũ) | NotificationAudit, Reporting | — |
| `borrowed-item.return-due` / `overdue` | Resource (Vũ) | NotificationAudit | — |
| `loan.offer.submitted` | Resource (Vũ) | NotificationAudit | Báo MO có loan offer mới |
| `attendance.request.submitted` / `reviewed` | Task (Lâm) | NotificationAudit, Reporting | Báo Leader có check-in chờ duyệt; báo volunteer kết quả |
| `readiness.item.failed` / `area.ready` / `area.reopened` | Readiness (Vũ) | Parish, NotificationAudit, Reporting | Parish chuyển status work area |
| `recognition.entry.created` | Reporting (Vũ) | NotificationAudit | — |

> Danh mục trên khớp API Contract v1.3 Mục 8; **không dùng** `contribution.donation.recorded` và không lọc `task.assignment.responded` để lấy buyer. Event contract đặt trong `<Service>.Contracts` (record C#, có `Version`). Khi đổi payload phải tăng version và thông báo người kia trong PR.

### 6.2 HTTP gọi đồng bộ

| Caller | Callee | Endpoint | Lý do |
|---|---|---|---|
| Parish, Task, Resource | Identity | `GET /internal/users?ids=` | Kiểm tra user ACTIVE, lấy skill/community khi assign |
| Readiness | Task | `POST /internal/tasks` (corrective task) | Cần taskId ngay |
| Task | Resource | `POST /internal/support-transfers` | `approveSupportMaterial` cần biết ngay đủ tồn kho nguồn hay không (R-08) |
| Task, Resource, Readiness | NotificationAudit | `GET /internal/files/{id}` | Xác thực fileId đã complete và thuộc user |

Mọi HTTP client: typed `HttpClient` + timeout + retry (chỉ GET) + circuit breaker.

### 6.3 Mốc phụ thuộc (blocking)

| Mốc | Ai giao | Ai cần | Hạn |
|---|---|---|---|
| docker-compose + BuildingBlocks.Web | Lâm | Vũ | Hết S0 |
| JWT phát hành (login) + format claim | Vũ | Lâm (gateway, security) | Giữa S1 |
| BuildingBlocks.Security + Messaging + Gateway | Lâm | Vũ | Hết S1 |
| Event `planning.work-area.*` | Lâm | Vũ (Resource, Readiness) | Hết S2 |
| `/internal/users` | Vũ | Lâm (assign leader, assign volunteer) | Hết S2 |
| File upload (F-NTF-01) | Lâm | Vũ (hóa đơn), Lâm (evidence) | Hết S3 |
| `task.materials.changed`, `task.completion.reviewed` | Lâm | Vũ (allocation, material report) | Hết S4 |
| `POST /internal/tasks` | Lâm | Vũ (corrective task) | Hết S6 |
| Event `purchase.request.decided` + `task.purchase-task.assigned` (buyer = assignee của task PURCHASE) | Vũ ↔ Lâm | Cả hai | Giữa S4 |
| `POST /internal/support-transfers` | Vũ | Lâm (F-TSK-15b) | Đầu S7 |
| `confirmSupportMaterialReceipt` + event `support.transfer.confirmed` | Vũ | Lâm | Giữa S7 |

---

## 7. Kế hoạch sprint (2 tuần/sprint)

| Sprint | Thời gian | Lâm | Vũ | Demo cuối sprint |
|---|---|---|---|---|
| S0 | 07/10 – 18/10/2026 | F-PLT-01..03, F-PAR-01 | F-IDN-01, chốt Mục 9 cùng Lâm | `docker compose up`, tạo parish |
| S1 | 19/10 – 01/11 | F-PLT-04..06, F-PAR-02, 04, 05, 07 | F-IDN-02, 03, 05, 06 | Login → gọi API qua gateway |
| S2 | 02/11 – 15/11 | F-PAR-06, 08, F-TSK-01..03, F-PLT-08, 09 | F-IDN-04, 07, 08, 09, 10, F-RES-01, 02 | Tạo season + work area + gán leader/officer, activate |
| S3 | 16/11 – 29/11 | F-TSK-04, 06, 07, 08, F-NTF-01, F-PLT-07 | F-RES-03, 04, 05 | Tạo task, assign/apply, xem shortage |
| S4 | 30/11 – 13/12 | F-TSK-05, 09, 10, 11, 17 | F-RES-06, 07 | Check-in → Leader duyệt từng người, progress, nghiệm thu; MO tạo PR → Leader duyệt → tạo task mua |
| S5 | 14/12 – 27/12 | F-TSK-12, 13, F-PAR-09, 10, F-NTF-02, 03 | F-RES-09, 10, 12, 13 | Volunteer mua → MO xác nhận giao → stock tăng, task mua tự đóng; ghi donation |
| S6 | 28/12 – 10/01/2027 | F-TSK-14, 15, F-NTF-04 | F-RES-11, 13b, 13c, 15, 16, F-RDY-02, F-RPT-01 | Support nhân lực; pledge → nhận một phần; allocation |
| S7 | 11/01 – 24/01 | F-TSK-15b, 16, hỗ trợ tích hợp FE/Mobile | F-RES-17, 18, 20, F-RDY-03..05, F-RPT-02, 03 | Support vật tư → chuyển kho; material report → verify; readiness → Ready; điểm |
| S8 | 25/01 – 07/02 | F-NTF-05, 06, fix bug | F-RDY-06, F-RES-14, F-RPT-04, 05 | Dashboard + báo cáo |
| — | ~06/02 – 14/02 | Nghỉ Tết (buffer) | Nghỉ Tết (buffer) | — |
| S9 | 15/02 – 28/02 | Hardening, E2E 16 kịch bản BA mục 17, performance | F-RPT-06, hardening, test | Full flow demo |
| — | 03/2027 | Tài liệu, deploy, chuẩn bị bảo vệ | Tài liệu, deploy, chuẩn bị bảo vệ | — |

---

## 8. Ma trận kịch bản nghiệm thu (BA mục 17) → owner

| # | Kịch bản | Owner chính | Phối hợp |
|---|---|---|---|
| 1 | Tạo, cấu hình, kích hoạt season | Lâm | — |
| 2 | Leader tạo task có skill + material | Lâm | Vũ (material catalog) |
| 3 | Assign / đăng ký và duyệt volunteer | Lâm | Vũ (user, skill) |
| 4 | Check-in, evidence, check-out, nghiệm thu | Lâm | — |
| 5 | Tính đúng shortage | Vũ | — |
| 6 | Leader duyệt purchase request, giao task mua cho volunteer | Vũ (duyệt) | Lâm (task mua) |
| 7 | Buyer gửi thông tin, MO xác nhận giao trước khi tăng stock | Vũ | Lâm (file, đóng task mua) |
| 8 | Donation MONEY/MATERIAL (trực tiếp hoặc từ pledge), chỉ usable đã nhận tăng stock | Vũ | — |
| 9 | Allocation không vượt khả dụng | Vũ | — |
| 10 | Borrowed item đến khi trả | Vũ | — |
| 11 | Hỗ trợ nhân lực hoặc vật tư liên cộng đoàn | Lâm | Vũ (chuyển kho vật tư) |
| 12 | Material report sau task, MO verify, remainder về đúng kho | Vũ | Lâm (event task completed) |
| 13 | Work area không Ready khi còn mandatory failed | Vũ | Lâm (status work area) |
| 14 | Báo cáo tổng hợp đúng | Vũ | Lâm (event) |
| 15 | Đúng vai trò và phạm vi | Cả hai (mỗi người cho service của mình) | — |
| 16 | Audit trail cho approve/sửa/export | Lâm | Vũ (publish event đủ thông tin) |

---

## 9. Quyết định đã chốt (07/10/2026)

| ID | Vấn đề | Quyết định | Function bị ảnh hưởng |
|---|---|---|---|
| D-01 | Duyệt mua & giao người mua | Work Area Leader duyệt mọi purchase request; sau đó tạo task `PURCHASE` và giao volunteer qua luồng assignment thường. Không có purchase assignment riêng | F-RES-06, 07, 09, 10, 11 (Vũ), F-TSK-17 (Lâm) |
| D-02 | Quyên góp | Có pledge qua app; MO ghi từng lần nhận (`donation_receipt`); vẫn có ghi nhận trực tiếp | F-RES-13, 13b, 13c (Vũ) |
| D-03 | Hỗ trợ liên cộng đoàn | Nhân lực và/hoặc vật tư; vật tư = chuyển kho giữa work area | F-TSK-14, 15, 15b (Lâm), F-RES-20 (Vũ) |
| D-04 | Readiness | Không có template; Leader tự tạo checklist + item | F-RDY-02 (Vũ) |
| D-05 | Đồ mượn | Volunteer nhập thông tin người cho mượn giùm (`owner_name/phone/address`), là đầu mối liên hệ | F-RES-18 (Vũ) |
| D-06 | Check-in | Leader duyệt **từng** yêu cầu check-in của từng người; check-out trực tiếp | F-TSK-11 (Lâm) |
| R-07 | Tên event (08/10/2026) | Danh mục event thống nhất (Mục 6.1, API Contract v1.3 Mục 8): `task.purchase-task.assigned` cho buyer; `contribution.receipt.confirmed` thay `contribution.donation.recorded` | F-TSK-17, F-RES-01, 08, 13, 13c, F-RPT-02 |
| R-08 | Duyệt vật tư hỗ trợ (08/10/2026) | Task gọi **đồng bộ** Resource `POST /internal/support-transfers`; thiếu tồn → 409 ngay | F-TSK-15b (Lâm), F-RES-20 (Vũ) |
| R-06 | Trạng thái season (14/10/2026) | Bỏ DRAFT; season tạo ở PLANNING (đã code) | F-PAR-05, 06 |
| R-09 | Season trùng (08/10/2026) | Tên **và** năm season đều unique trong parish (đã code) | F-PAR-05 |
| — | Các điểm khác đã áp dụng | 1 role/account; parish là `app_user.parish_id`; gộp ghi nhận + xác nhận giao hàng thành `confirmPurchaseDelivery`; unique cả năm và tên season trong parish | F-IDN-08, 10, F-PAR-03, 05, F-RES-10 |

---

## 10. Tình trạng tài liệu trước khi code

| Tài liệu | Phiên bản mới | Nội dung đã cập nhật |
|---|---|---|
| ERD | `ERD_Grottowork_v3.5.txt` → **v3.6** (14/10/2026: season không DRAFT, unique tên + năm) | 56 bảng (đã parse DBML hợp lệ). Thêm `attendance_request`, `attendance_correction`, `donation_receipt(_item)`, `support_request_material`, `support_response_material`, `volunteer_availability`, `notification`, `notification_preference`, `activity_log`, `file_object`; thêm cột cho task mua, pledge, đồ mượn, corrective task, parish của user, ngân sách season, version rule, cost overrun. Ghi chú thay đổi ở cuối file |
| BA | `GrottoWorks_Phan_Tich_Nghiep_Vu_Chi_Tiet_v1.1.md` → **v1.2** (14/10/2026: season 8.1, 9.1, BR-02) | Lịch sử thay đổi; sửa vai trò (4.x), quy trình 8.3, 8.6, 8.7, 8.8, 8.11, 8.12; state 9.x (thêm Attendance Request, Support Request); BR-18, 28–32, 35–39, 41 sửa; thêm BR-58 → BR-70; ma trận quyền; dashboard; thông báo; edge case 27–36; tiêu chí nghiệm thu; mục 19 quyết định |
| API Contract | `GrottoWorks_API_Contract_v1.2.docx` → **v1.3** (14/10/2026: season không DRAFT + `seasonYear` + schema Parish; donor, `adjustDonation`; `approveSupportMaterial` đồng bộ; danh mục event; Idempotency-Key; readiness item status) | Mục "Changes in v1.2"; sửa endpoint Purchases, Contributions, Support, Attendance, Readiness; thêm task material report, skill, availability, work area category, recognition rule, contribution type, `listParishUsers`; chuyển file API sang NotificationAudit; cập nhật schema, state, công thức tồn kho, event, payload mẫu |

Các bản cũ (BA v1/v1.1, ERD v3.4/v3.5, API v1.1/v1.2) vẫn giữ nguyên để đối chiếu.

---

## 11. Checklist khởi động (tuần này)

- [x] Chốt D-01 → D-06 (07/10/2026).
- [x] Cập nhật ERD v3.5, BA v1.1, API Contract v1.2.
- [ ] Cả hai: đọc lại 3 tài liệu mới, comment chỗ chưa đồng ý trước khi mở S1.
- [ ] Lâm: merge `lam-dev-1`, xóa code mẫu weatherforecast, tạo docker-compose.
- [ ] Vũ: tạo `Services/Identity` theo đúng template `ParishCoordination`.
- [ ] Cả hai: thống nhất format JWT claim (`sub`, `role`, `parishId`, `communityId`) và event envelope, viết vào `BuildingBlocks/README.md`.
- [ ] Tạo GitHub Project board, mỗi Function ID = 1 issue, gắn label `owner:lam` / `owner:vu`, `P0/P1/P2`, `sprint:Sx`.
