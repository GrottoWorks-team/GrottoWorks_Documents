# Rà soát đồng bộ: plan.md ↔ Task Spec v1.2 ↔ API Contract v1.2 ↔ BA v1.1 ↔ ERD v3.5

**Ngày:** 08/10/2026 · **Phạm vi:** repo `GrottoWorks_Documents` (commit `a51100d`) và `GrottoWorks_BE/plan.md` + code trên `main` (`14cb733`)
> **Cập nhật 14/10/2026:** các điểm dưới đây đã được xử lý trong API Contract v1.3, ERD v3.6, BA v1.2, Spec v1.3 và `plan.md`. Riêng R-05: giữ nguyên các API parish đã code (không gỡ `createParish`/`listParishes`). R-06: Lâm đã bỏ DRAFT trong code (`41dac24`).

**Thứ tự ưu tiên khi lệch (Spec §0):** quyết định nhóm > BA v1.1 > ERD v3.5 > API Contract v1.2

## Kết quả chốt (08/10/2026)

| Điểm | Quyết định |
|---|---|
| A5 / R-05 | Hệ thống chỉ phục vụ **1 giáo xứ**, seed sẵn bằng migration với id cố định (`Parish:DefaultParishId`, Parish và Identity dùng chung). Bỏ `createParish`, `listParishes`; `getParish` = USER, `updateParish` = ADMIN, PARISH |
| B5 / R-06 | **Bỏ trạng thái DRAFT** của season: season tạo ra ở PLANNING → ACTIVE → READY → COMPLETED → ARCHIVED |
| C1–C3 / R-07 | Dùng `task.purchase-task.assigned` cho buyer, `contribution.receipt.confirmed` thay `contribution.donation.recorded`; danh mục event đầy đủ ở `plan.md` Mục 10.3 |
| B6 / R-08 | **Gọi đồng bộ** (phương án a): Task gọi Resource `POST /internal/support-transfers` khi Leader duyệt; thiếu tồn kho → 409 ngay |
| D1 / R-09 | Unique **cả** tên và năm season trong giáo xứ |

Ký hiệu mức độ: 🔴 đang chặn / gây conflict code · 🟠 sẽ gây conflict khi làm sprint tới · 🟡 lệch tài liệu, sửa cho sạch

---

## A. Nguyên nhân trực tiếp khiến Lâm bị conflict

| # | Mức | Vấn đề | plan.md / Spec nói | API Contract / thực tế repo | Đề xuất chốt |
|---|---|---|---|---|---|
| A1 | 🔴 | **Ownership Platform bị chồng** | PLT-01..06 là của Lâm (plan §3.1, §9: "Platform 0/9") | Vũ đã commit `09170a5` (docker-compose, YARP, BuildingBlocks.Web/Security/Messaging — F-PLT-02..06). Merge `5dd9d29` của Lâm làm mất `Program.cs` Parish → Lâm phải sửa lại ở `6a76c8c`, `14cb733` | Cập nhật plan: PLT-02..06 **đã có bản đầu (Vũ)**, Lâm là owner tiếp tục hoàn thiện/review. Từ nay không ai sửa code service/package của người kia nếu không có PR được tag review |
| A2 | 🔴 | **Hiện trạng repo trong plan §0 sai** | "ApiGateway, BuildingBlocks rỗng", commit `69d2f0f`, `lam-dev-1` chưa merge | Đã có gateway, 3 BuildingBlocks, Identity service + 49 test, `lam-dev-1` đã merge | Viết lại §0 theo `14cb733` (đã sửa trong plan v1.3) |
| A3 | 🔴 | **Quy tắc route Gateway mâu thuẫn với path của Contract** | PLT-06: route `/api/v1/{service-prefix}/**` | Contract dùng path theo resource, **không có prefix service**. 5 prefix bị nhiều service dùng chung: `/users/**` (5 service), `/seasons/**` (5), `/work-areas/**` (4), `/tasks/**` (Task + Resource), `/purchase-requests/**` (Resource + Task). Gateway hiện route `/users/**`, `/seasons/**`, `/work-areas/**` → 1 service duy nhất ⇒ sẽ 404 khi thêm Task/Resource | Giữ path Contract (FE/Mobile đã bám). Gateway dùng **route cụ thể có `Order` nhỏ hơn** cho các sub-path thuộc service khác (bảng ở Phụ lục 1). Bỏ quy tắc "service-prefix" khỏi plan |
| A4 | 🔴 | **Event envelope không khớp Contract §8** | plan §8 đánh dấu ✅ "đã thống nhất envelope"; `BuildingBlocks/README.md §2` + `IntegrationEventEnvelope` chỉ có `eventType, version, payload, aggregateType, aggregateId, occurredAt` | Contract §8 bắt buộc thêm `eventId, correlationId, actorId, parishId, seasonId, aggregateVersion`; consumer idempotent theo **`eventId`** (không có field này thì Inbox không làm được) | Sửa `IntegrationEventEnvelope` + README theo Contract §8 (giữ thêm `version` của payload). Bỏ dấu ✅ ở checklist cho tới khi sửa xong |
| A5 | 🔴 | **Role của endpoint Parish** | plan/Spec F-PAR-01: ADMIN/PARISH | Contract: `listParishes/createParish/getParish/updateParish` = **PARISH** only. BA §4.7: Admin quản lý "giáo xứ, cộng đoàn". Thực tế: user PARISH bắt buộc có `parish_id` (BR-67) ⇒ PARISH không thể tạo parish đầu tiên. Code Parish hiện **không có `RequireAuthorization`** | Theo BA: `createParish` = ADMIN; `list/get/updateParish` = ADMIN (mọi parish) + PARISH (chỉ parish của mình, `EnsureSameParish`); `createCommunity/updateCommunity` = ADMIN, PARISH. **Sửa Contract** |
| A6 | 🟠 | **Envelope trong quy trình chuẩn** | plan §1 bước 6: envelope `{ data, meta }` | Contract 4.2: collection = `{ data, page, meta }` | Sửa plan §1 (đã sửa) |

## B. Endpoint lệch giữa plan/Spec và Contract

| # | Mức | Function | plan.md / Spec | API Contract v1.2 | Đề xuất |
|---|---|---|---|---|---|
| B1 | 🟠 | F-TSK-17 (Lâm) | `POST /purchase-requests/{id}/purchase-task` | `POST /purchase-requests/{requestId}/purchase-tasks`, operationId `createPurchaseTask` | Theo Contract. Gateway route riêng → Task |
| B2 | 🟠 | F-RES-20 (Vũ) | `POST /support-transfers/{id}/confirm`, service Resource | `POST /support-response-materials/{itemId}/confirm-receipt` (`confirmSupportMaterialReceipt`), tag **Support** (= Task service của Lâm) | Giữ path Contract, **Resource** implement (vì sở hữu ledger); gateway route riêng → Resource. Ghi rõ ngoại lệ tag ≠ service |
| B3 | 🟠 | F-RES-12 (Vũ) | "Donor CRUD" | **Không có endpoint donor nào** | Bổ sung Contract: `GET/POST /donors`, `PATCH /donors/{donorId}` (MO, ADMIN) |
| B4 | 🟡 | F-RES-14 (Vũ) | `PATCH /donations/{id}` | Không có | Bổ sung Contract (P2) hoặc bỏ khỏi plan |
| B5 | 🟠 | F-PAR-06 (Lâm) | "+ nội bộ chuyển PLANNING, READY" | State DRAFT → PLANNING → ACTIVE nhưng **không có command DRAFT→PLANNING**; READY do readiness | Thêm `POST /seasons/{seasonId}/start-planning` (PARISH) **hoặc** cho `activateSeason` nhận DRAFT/PLANNING. Cần chốt |
| B6 | 🟠 | F-TSK-15b ↔ F-RES-20 | Task duyệt → event → Resource ghi TRANSFER_OUT | Contract: `approveSupportMaterial` "posts TRANSFER_OUT" (đồng bộ) + lỗi `MATERIAL_INSUFFICIENT_AVAILABLE` khi vượt unallocated | Task không biết unallocated. Chốt 1 trong 2: (a) Task gọi `POST /internal/support-transfers` của Resource (đồng bộ, trả 409 ngay), hoặc (b) async + event `support.transfer.failed`. Đề xuất (a) |
| B7 | 🟡 | `listMyContributions` | RES-13 (Resource) | Tag Contributions (Resource) — nhưng bảng `contribution` thuộc Reporting (ERD/Spec §3) | Chốt: Resource trả danh sách donation của user; điểm/contribution xem ở `getMyRecognition` (Reporting) |
| B8 | 🟡 | Idempotency-Key | Spec 2.1: record purchase, record donation, check-in/out, export | Contract 4.1: purchase, contribution receipt, attendance, export, **backup**; ví dụ 10.2 dùng cho `approvePurchaseRequest` | Gom 1 danh sách: `recordPurchase, recordDirectDonation, recordDonationReceipt, requestCheckIn, recordCheckOut, createReportExport, createSystemDataExport, startBackup, POST /internal/tasks`. Bỏ header ở ví dụ 10.2 |

## C. Event lệch tên / thiếu

| # | Mức | plan/Spec dùng | Contract §8 | Đề xuất |
|---|---|---|---|---|
| C1 | 🟠 | Resource lưu buyer bằng `task.assignment.responded` (lọc task PURCHASE) — plan RES-01, Spec 6.1 | `task.purchase-task.assigned` — "Buyer accepted a PURCHASE task" | Dùng **`task.purchase-task.assigned`** (Resource không phải lọc, set `BUYER_ASSIGNED`). Sửa plan RES-01, Spec 6.1 |
| C2 | 🟠 | `contribution.donation.recorded` (RES-13, RPT-02) | `contribution.receipt.confirmed` | ERD: contribution sinh từ **receipt**. Dùng `contribution.receipt.confirmed` cho mọi receipt (kể cả direct donation). Bỏ `donation.recorded`; giữ `contribution.donation.adjusted` (RES-14) |
| C3 | 🟡 | Có trong plan/Spec, **thiếu** trong Contract §8: `identity.user.unlocked`, `planning.season.completed/archived`, `planning.work-area.created/updated`, `task.materials.changed`, `task.status.changed`, `attendance.hours.approved`, `support.member.completed`, `material-report.submitted`, `borrowed-item.overdue`, `readiness.area.reopened` | — | Bổ sung vào Contract §8 (Spec 6.1 là danh sách đầy đủ) |
| C4 | 🟡 | Không dùng | `task.progress.submitted`, `attendance.request.submitted` mô tả "check-in **or check-out**" | Check-out không qua request (D-06) → sửa mô tả |

## D. Lệch dữ liệu / state giữa BA – ERD – Contract

| # | Mức | Nội dung | BA v1.1 | ERD v3.5 | Contract v1.2 | Spec/plan | Đề xuất |
|---|---|---|---|---|---|---|---|
| D1 | 🟠 | Unique season | BR-02: **tên** unique trong parish | unique `(parish_id, season_year)` | `name` unique; schema **không có `seasonYear`** | Spec §9: "unique cả năm và tên" (quyết định nhóm) | Theo quyết định nhóm: ERD thêm unique `(parish_id, season_name)`; Contract thêm `Season.seasonYear` |
| D2 | 🟡 | ReadinessItem | — | `item_status` PENDING, PASSED, FAILED, CORRECTIVE_ACTION, RECHECK | Schema `result` PENDING/PASSED/FAILED nhưng §7 dùng CORRECTIVE_ACTION → RECHECK | — | Contract schema đổi thành `status` 5 giá trị như ERD |
| D3 | 🟡 | MaterialRequirement.scopeType | — | Không có (unique work_area+material) | `WORK_AREA` hoặc `TASK` | Task-level nằm ở `task_material` | Bỏ `scopeType` khỏi Contract |
| D4 | 🟡 | `existing_quantity` | — | Có trong `material_requirement` | Công thức shortage không dùng | — | Bỏ cột hoặc ghi chú "chỉ để hiển thị", không dùng tính shortage |
| D5 | 🟡 | Parish schema | — | `parish_status` không enum | Không có schema Parish | ACTIVE/INACTIVE | Thêm schema Parish (id, name, address, description, status ACTIVE/INACTIVE, version) |

## E. Lỗi nội bộ plan.md / Spec

| # | Vị trí | Lỗi | Sửa |
|---|---|---|---|
| E1 | plan §2 S9, Spec §7 S9 | "E2E 17 kịch bản" | BA §17 có **16** kịch bản |
| E2 | plan §3.2 F-PAR-06 | Publish `planning.season.activated/completed/archived` | Đúng với Spec, nhưng Contract thiếu → xem C3 |
| E3 | Spec §1 bảng Vũ/Readiness | "Readiness template" | Đã bỏ theo D-04 |
| E4 | Spec §3 tiêu đề cột | "Bảng ERD v3.4" | ERD v3.5 |
| E5 | plan §8 | Checklist JWT + envelope ✅ | Envelope chưa đúng (A4) |
| E6 | Gateway `appsettings.json` | Thiếu route `/admin/**`, `/roles` | Thêm (Identity) — F-IDN-08 sẽ 404 qua gateway |

---

## Việc cần làm (đề xuất)

1. **Họp 15' chốt 5 điểm cần quyết định**: A5 (role Parish), B5 (DRAFT→PLANNING), B6 (sync/async transfer), C1/C2 (tên event), D1 (unique season).
2. **Vũ hoặc người giữ Docs**: phát hành **API Contract v1.3** với các mục A5, B1–B8, C1–C4, D1–D5.
3. **Lâm**: sửa `IntegrationEventEnvelope` + `BuildingBlocks/README.md §2` (A4); gateway route cụ thể (A3, E6); thêm `RequireAuthorization` cho Parish (A5).
4. **Cả hai**: từ nay đổi Contract → sửa Spec + plan **trong cùng PR**; không merge code đụng service/package của người kia khi chưa có review.

---

## Phụ lục 1 — Route Gateway theo Contract (route cụ thể, `Order` nhỏ hơn route chung)

| Route cụ thể (prefix `/api/v1`) | Service |
|---|---|
| `/users/me/tasks`, `/users/me/attendance` | task |
| `/users/me/contributions` | resource |
| `/users/me/recognition` | reporting |
| `/users/me/notification-preferences` | notify |
| `/users/**`, `/auth/**`, `/skills/**`, `/admin/**`, `/roles` | identity |
| `/seasons/{id}/material-needs`, `/seasons/{id}/contribution-needs` | resource |
| `/seasons/{id}/support-summary` | task |
| `/seasons/{id}/readiness` | readiness |
| `/seasons/{id}/recognition` | reporting |
| `/seasons/**` | parish |
| `/work-areas/{id}/tasks`, `/attendance-requests`, `/attendance-corrections`, `/working-hours` | task |
| `/work-areas/{id}/material-requirements`, `/shortages` | resource |
| `/work-areas/{id}/readiness-checks` | readiness |
| `/work-areas/**` | parish |
| `/tasks/{id}/material-reports` | resource |
| `/purchase-requests/{id}/purchase-tasks` | task |
| `/support-response-materials/{id}/confirm-receipt` | resource (ngoại lệ B2) |
| `/tasks/**`, `/task-applications/**`, `/task-assignments/**`, `/attendance/**`, `/attendance-requests/**`, `/attendance-corrections/**`, `/support-requests/**`, `/support-response-members/**`, `/support-response-materials/**` | task |
| `/purchase-requests/**`, `/purchases/**`, `/material-categories/**`, `/materials/**`, `/material-requirements/**`, `/material-allocations/**`, `/inventory/**`, `/inventory-adjustments/**`, `/task-material-reports/**`, `/donations/**`, `/donors/**`, `/loan-offers/**`, `/borrowed-items/**` | resource |
| `/readiness-checks/**` | readiness |
| `/dashboards/**`, `/reports/**`, `/report-exports/**`, `/contribution-types/**`, `/recognition-rules/**` | reporting |
| `/notifications/**`, `/notification-rules/**`, `/activity-logs/**`, `/resource-history/**`, `/system-data-exports/**`, `/backups/**`, `/files/**` | notify |
| `/internal/**` | **không expose** qua gateway |
