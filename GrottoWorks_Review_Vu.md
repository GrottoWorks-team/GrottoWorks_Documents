# Theo dõi review — task của Vũ (GrottoWorks_BE)

- **Phạm vi:** Identity, Resource, Readiness, Reporting
- **Ngày review:** 05/11/2026
- **Nhánh sửa:** `Vu-dev` ở cả hai repo
  - GrottoWorks_BE: base `8e80225` → `05ee01e` → `a37c776` → `9832b99` → `940d863`
  - GrottoWorks_Documents: cập nhật BA 1.2.3 và Spec (xem V-06)
- **Kết quả test sau khi sửa:** 1.346/1.346 unit test pass trên toàn solution; build 0 error, 0 cảnh báo

## Tổng quan

| ID | Mức độ | Vấn đề | Service | Trạng thái |
|---|---|---|---|---|
| V-01 | 🔴 Nghiêm trọng | Event outbox có thể mất vì không commit cùng thay đổi nghiệp vụ | Resource, Readiness | ✅ Đã sửa (`a37c776`) |
| V-02 | 🔴 Nghiêm trọng | Shortage bị tính thiếu sau khi task trả vật tư về kho | Resource | ✅ Đã sửa (`05ee01e`) |
| V-03 | 🟠 Trung bình | `recognition.entry.created` publish sau khi save | Reporting | ✅ Đã sửa (`a37c776`) |
| V-04 | 🟠 Trung bình | Vật tư cấp cho task bị hủy bị kẹt vĩnh viễn | Resource | ✅ Đã sửa (`9832b99`), chờ Lâm làm consumer thông báo |
| V-05 | 🟠 Trung bình | Allocation cấp work area (không gắn task) không có đường trả về kho | Resource | ✅ Đã chốt: xem như đã tiêu thụ (R-26) |
| V-06 | 🟡 Thấp | Công thức BA 8.5 chưa khớp cách tính trong code | Tài liệu | ✅ Đã cập nhật BA 1.2.3 và Spec, chờ báo Lâm |
| V-07 | 🟡 Thấp | Một số transaction DB giữ trong lúc gọi HTTP sang service khác | Resource, Readiness | 📝 Theo dõi |
| V-08 | 🟡 Thấp | Chưa có test tích hợp với DB thật và chưa có CI | Toàn repo | 📝 Theo dõi |
| V-09 | ⚪ Ghi chú | Access token vẫn hợp lệ tới 15 phút sau khi lock user | Identity | 📝 Chấp nhận trong baseline |
| V-10 | ⚪ Ghi chú | Cảnh báo build EF1003 và NU1510 | Resource, Readiness, Reporting, Identity | ✅ Đã sửa (`940d863`) |

---

## Chi tiết

### V-01 — Event outbox có thể mất ✅

**Nguyên nhân:** `OutboxIntegrationEventPublisher.PublishAsync` tự gọi `SaveChangesAsync` riêng. Nhiều handler lại save thay đổi nghiệp vụ trước rồi mới publish, nên hai lần ghi nằm ở hai transaction khác nhau.

**Hậu quả:** nếu crash hoặc bước ghi outbox lỗi ở giữa hai lần ghi:
- Tồn kho đã tăng nhưng mất `purchase.delivery.confirmed`, nên task PURCHASE bên TaskExecution không bao giờ đóng.
- Checklist đã READY nhưng mất `readiness.area.ready`, nên work area kẹt ở `READINESS_REVIEW` (ảnh hưởng demo #13).
- Mất `material.shortage.changed`, nên Reporting hiển thị sai.

**Đã sửa:**
- Thêm `IUnitOfWork` (`Application/Abstractions/IUnitOfWork.cs`) và `EfUnitOfWork` (`Infrastructure/Data/EfUnitOfWork.cs`). Nếu đã có transaction đang mở thì dùng chung, nên stock lease và `InTransactionAsync` của các store vẫn chạy đúng.
- Các handler đã gói trong unit of work:
  - **Resource:** confirm delivery, create/update requirement, opening stock, inventory adjustment, direct donation, donation receipt, pledge material/money, adjust donation, loan offer, job nhắc trả đồ mượn, submit/verify material report.
  - **Readiness:** ghi kết quả/recheck (khi item failed), confirm, reopen.

**Kiểm chứng:**
- Thêm unit test hồi quy: `Delivery_stock_and_events_are_written_in_one_unit_of_work` và `Failed_result_confirm_and_reopen_write_state_and_event_in_one_unit_of_work`.
- Chạy test tạm trên PostgreSQL 17 (không commit), cả 3 kịch bản đều đạt: lỗi sau khi save thì ledger rollback; ledger và outbox commit cùng nhau; stock lock giữ tới khi commit.

**Lưu ý khi viết handler mới:** command nào save rồi publish thì phải chạy trong `IUnitOfWork`.

### V-02 — Shortage bị tính thiếu sau khi task trả vật tư ✅

**Nguyên nhân:** `InventoryCalculator` tính `Shortage = Required − tổng ledger`. Phần thừa của task được ghi thêm một dòng `+TASK_RETURN` vào ledger, trong khi allocation không bao giờ được giải phóng (BR-51). Vì vậy phần trả về bị tính như hàng mới nhập lần thứ hai.

**Ví dụ:** cần 120, nhận 100, cấp 30 cho task, task trả về 10 → shortage hiện **10** (đúng phải là **20**).

**Hậu quả:** sai demo #5 ("Tính đúng shortage"). Có thể chặn tạo purchase request vì shortage hiện thấp hơn thực tế.

**Đã sửa:**
- `InventoryCalculator.Calculate(..., taskReturned)`: shortage tính trên ledger đã trừ các dòng `TASK_RETURN`. Unallocated giữ nguyên (vật tư trả về vẫn dùng lại được).
- `IInventoryStore.GetTaskReturnTotalsAsync` (EF và fake); `InventoryReader` truyền giá trị này vào calculator.

**Kiểm chứng:** test `Task_returns_free_stock_but_never_cover_the_requirement_twice` và `Verified_remainder_does_not_lower_the_shortage`.

### V-03 — Recognition event publish sau khi save (Reporting) ✅

**Nguyên nhân:** `IntegrationEventHandler` gọi `recorder.PublishAsync(points)` sau `SaveAsync`. Nếu lỗi ở giữa thì điểm đã ghi nhưng mất `recognition.entry.created`.

**Đã sửa:**
- Publisher của Reporting chỉ stage dòng outbox, không tự save nữa.
- Event được stage trước lần `SaveChanges` duy nhất, nên điểm, dòng inbox và event commit cùng nhau.

**Kiểm chứng:** test `Recognition_events_are_staged_before_the_points_commit`.

### V-04 — Vật tư cấp cho task bị hủy bị kẹt ✅

**Hiện trạng trước khi sửa:** không có luồng nào giải phóng allocation, và material report chỉ nộp được khi task `COMPLETED` (BR-44). Vì vậy vật tư đã cấp cho task `CANCELLED` không bao giờ quay lại unallocated.

**Đã chốt:** phương án C — thu hồi bằng material report của task bị hủy (R-25).
- Không chọn tự trả toàn bộ về kho khi hủy, vì task có thể đã dùng một phần vật tư; chỉ số lượng đã verify mới vào kho (BR-49).
- Không phải đổi schema hay thêm endpoint.

**Đã sửa (`9832b99`):**
- `submitTaskMaterialReport` nhận cả task `CANCELLED`: assignee **hoặc MO của work area** gửi. Task hủy trước khi có người nhận thì không có assignee, nên MO gửi. MO verify như thường.
- Task `COMPLETED` vẫn chỉ assignee gửi; MO không gửi thay được.
- Endpoint cho phép thêm role `MATERIAL_OFFICER`.
- Khi `task.status.changed` chuyển task sang `CANCELLED` mà task còn allocation, Resource phát **một lần** event `material.allocation.recall-required` (taskId, work area, season, parish, MO, các dòng allocation). Event này đi trong unit of work.
- Test: `Officer_recovers_materials_of_a_cancelled_task_through_the_report`, `Cancelled_task_is_reported_by_an_assignee_or_its_own_officer_only`, `Cancelling_a_task_that_holds_materials_asks_the_officer_to_recall_them_once`.

**Còn lại:**
- [ ] Lâm: thêm consumer và template thông báo cho `material.allocation.recall-required` ở NotificationAudit (bảng mã event trong `Services/NotificationAudit/README.md`).
- [ ] Lâm: cập nhật API Contract (docx) cho role mới của `submitTaskMaterialReport` và event mới.
- [ ] FE/Mobile: hiện nút "Báo cáo thu hồi vật tư" cho MO trên task bị hủy còn allocation.

### V-05 — Allocation cấp work area không có đường trả về ✅

**Đã chốt (R-26):** allocation chỉ gắn work area (không gắn task) được xem là **đã tiêu thụ** khi cấp, không có material report. Muốn thu hồi phần dư thì cấp theo task.
- Lý do: thường là vật tư dùng chung, lượng dư nhỏ, và mùa Giáng sinh kết thúc là đóng sổ.
- Không sửa code; đã ghi vào BA 8.9 và `Services/Resource/README.md`.
- [ ] Nếu sau này cần thu hồi: mở rộng cách làm của V-04 (report theo allocation).

### V-06 — Công thức BA 8.5 chưa khớp code ✅

**Đã cập nhật** `GrottoWorks_Phan_Tich_Nghiep_Vu_Chi_Tiet_v1.2.md` lên **phiên bản 1.2.3** (nhánh `Vu-dev` của GrottoWorks_Documents):
- Mục 8.5, công thức mới:
  - Confirmed Available = tổng ledger (có cộng support transfer in/out).
  - Allocated = tổng đã cấp, không bao giờ giảm.
  - Unallocated = max(0, Available − Allocated).
  - **Shortage = max(0, Required − (Available − Verified Task Return))**.
  - Có ví dụ 120/100/30/10.
- Bỏ "Confirmed Borrowed Quantity" khỏi công thức: đồ mượn không vào sổ kho, khớp với F-RES-18.
- Mục 8.9 thêm R-26; mục 8.10 và BR-44 thêm trường hợp task bị hủy; danh sách thông báo thêm nhắc thu hồi; bảng quyết định thêm R-24, R-25, R-26.
- Spec phân công: F-RES-17 và bảng event 6.1 thêm `material.allocation.recall-required`.
- [ ] Báo Lâm về R-24, R-25, R-26 và event mới.

### V-07 — Transaction DB giữ trong lúc gọi HTTP 📝

**Hiện trạng:** vài handler giờ chạy toàn bộ trong unit of work, kể cả lúc gọi HTTP. Ví dụ khi pledge thì tìm donor qua Identity; trong `InspectAsync` thì gọi service File.

**Ảnh hưởng:** kết quả vẫn đúng, nhưng giữ kết nối và lock DB lâu hơn.

**Việc cần làm:**
- **Đã chốt:** để nguyên.
- [ ] Sau demo, hoặc khi đo thấy chậm: chuyển phần gọi HTTP ra trước transaction, chỉ gói phần save và publish.

### V-08 — Thiếu test tích hợp và CI 📝

**Hiện trạng:**
- Unit test dùng fake store, nên không bắt được lỗi transaction hoặc concurrency.
- Repo chưa có `.github/workflows` (F-PLT-09 thuộc phần của Lâm).

**Việc cần làm:**
- **Đã chốt:** làm ở S8.
- [ ] Thêm project test tích hợp dùng Testcontainers PostgreSQL. Ưu tiên test: allocation tranh phần cuối, xoay refresh token song song, rollback outbox.
- [ ] Chạy full stack bằng Docker để thử luồng purchase (record → confirm → task PURCHASE đóng) và luồng readiness (confirm → work area READY).

### V-09 — Access token sau khi lock user 📝

Lock user đã thu hồi refresh token, nhưng access token đã cấp vẫn dùng được tới khi hết hạn (≤ 15 phút). Code đã ghi chú việc này.
- **Đã chốt:** chấp nhận trong baseline và ghi vào phần giới hạn đã biết.
- [ ] Làm Redis denylist khi có Redis.

### V-10 — Cảnh báo build ✅

**Đã sửa (`940d863`):**
- Bỏ PackageReference thừa (NU1510) ở Identity, Resource, Readiness, Reporting.
- Thêm `#pragma warning disable EF1003` kèm lý do cho câu SQL lấy batch outbox (chỉ ghép hằng số).
- Build lại toàn bộ (`--no-incremental`): 0 cảnh báo.
- Ghi chú: NotificationAudit và TaskExecution (phần của Lâm) cũng có reference `Configuration.Abstractions` tương tự; mình không sửa.

---

## Những phần đã kiểm tra và ổn

- **Identity:**
  - Xoay refresh token bằng UPDATE có điều kiện nên atomic; dùng lại token cũ thì thu hồi cả chuỗi.
  - Reset password thu hồi refresh token; token reset chỉ dùng một lần.
  - Lock user thu hồi token và đưa event vào outbox trong cùng transaction.
- **Tồn kho:**
  - Các thao tác xuất kho đều khoá advisory lock theo work area.
  - Ledger chống ghi trùng bằng unique index theo `reference_type, reference_id`.
- **Reporting:** inbox `processed_event` chặn xử lý trùng event; contribution ứng với donation theo quan hệ 1-1.
- **Readiness:** đã kiểm tra BR-52 (còn hạng mục bắt buộc chưa pass thì không confirm), BR-53 (reopen bắt buộc có lý do), BR-66 (cần corrective task trước khi recheck), BR-73 (chỉ confirm khi work area ở `READINESS_REVIEW`).
