# Phân tích nghiệp vụ chi tiết hệ thống GrottoWorks

**Phiên bản:** 1.2 · **Cập nhật:** 14/10/2026

### Lịch sử thay đổi

| Phiên bản | Ngày | Nội dung |
|---|---|---|
| 1.0 | — | Baseline ban đầu |
| 1.1 | 07/10/2026 | Áp dụng 6 quyết định của nhóm: (D-01) Work Area Leader duyệt purchase request và giao volunteer đi mua bằng task mua; (D-02) khôi phục pledge quyên góp, xác nhận nhận nhiều lần; (D-03) support request gồm nhân lực và/hoặc vật tư; (D-04) không dùng mẫu readiness checklist; (D-05) volunteer nhập thông tin người cho mượn; (D-06) Leader duyệt từng yêu cầu check-in. Bổ sung BR-58 → BR-70, trạng thái, phân quyền, thông báo, ngoại lệ và tiêu chí nghiệm thu tương ứng. Đồng bộ với ERD v3.5 và API Contract v1.2 |
| 1.2 | 14/10/2026 | Season không còn trạng thái `Draft`: tạo mới ở `Planning` (đã áp dụng trong code). BR-02: tên **và** năm season duy nhất trong giáo xứ. Đồng bộ với ERD v3.6 và API Contract v1.3 |

## 1. Mục đích tài liệu

Tài liệu này mô tả đầy đủ nghiệp vụ của **GrottoWorks - Hệ thống quản lý làm hang đá và trang trí Giáng sinh tại giáo xứ**. Nội dung là cơ sở để nhóm phát triển:

- Software Requirement Specification.
- Use Case Diagram và Use Case Specification.
- Activity Diagram, Sequence Diagram và State Machine Diagram.
- Domain Model, Entity Relationship Diagram và thiết kế cơ sở dữ liệu.
- Phân quyền, API, giao diện, test case và acceptance criteria.

GrottoWorks quản lý xuyên suốt một mùa chuẩn bị Giáng sinh theo năm câu hỏi:

| Câu hỏi | Nhóm dữ liệu trả lời |
|---|---|
| Cần thực hiện việc gì? | Mùa Giáng sinh, khu vực, công việc và checklist |
| Ai thực hiện? | Người dùng, vai trò, tình nguyện viên, kỹ năng và phân công |
| Cần những nguồn lực nào? | Vật tư, tiền, dụng cụ, mua sắm, quyên góp và mượn |
| Tiến độ đến đâu? | Trạng thái task, chấm công, ảnh, ghi chú và phê duyệt |
| Đã sẵn sàng hay chưa? | Readiness checklist, kiểm tra an toàn và công việc khắc phục |

## 2. Bối cảnh và vấn đề nghiệp vụ

Mỗi dịp Giáng sinh, một giáo xứ có thể đồng thời triển khai nhiều công trình như hang đá, cây thông, hệ thống đèn, sân nhà thờ, sân khấu và các khu vực trang trí khác. Công việc thường được chia cho nhiều cộng đoàn và có sự tham gia của ban phụ trách, trưởng khu vực, người quản lý vật tư, tình nguyện viên và giáo dân đóng góp.

Trong cách làm thủ công, dữ liệu thường nằm rải rác trong nhóm chat, thông báo miệng, giấy tờ và bảng tính cá nhân. Điều này tạo ra các vấn đề:

- Không có một nguồn dữ liệu thống nhất về công việc, người phụ trách và thời hạn.
- Khó biết vật tư nào đang cần, đã có, đã mua, được tặng, được mượn hoặc còn thiếu.
- Không phân biệt rõ vật tư mới được cam kết với vật tư đã thực sự nhận.
- Không biết ai được giao đi mua, đã mua chưa, đã giao chưa và hóa đơn thuộc yêu cầu nào.
- Việc đăng ký và phân công tình nguyện viên chưa dựa trên kỹ năng hoặc lịch rảnh.
- Khó hỗ trợ chéo khi một cộng đoàn thiếu nhân lực hoặc kỹ năng nhưng cộng đoàn khác có thể hỗ trợ.
- Tiến độ thường được báo bằng lời, thiếu ảnh và thiếu bước nghiệm thu.
- Giờ làm việc, công sức và đóng góp của từng người khó tổng hợp minh bạch.
- Không có tiêu chí rõ ràng để kết luận một khu vực đã sẵn sàng và an toàn.

GrottoWorks đưa các dữ liệu trên vào một luồng thống nhất:

```text
Lập mùa Giáng sinh
  -> Tạo khu vực
  -> Chia thành công việc
  -> Lập kế hoạch người và vật tư
  -> Mua, nhận đóng góp hoặc mượn vật tư
  -> Thực hiện và chấm công
  -> Gửi bằng chứng tiến độ
  -> Trưởng khu vực nghiệm thu
  -> Kiểm tra readiness và an toàn
  -> Tổng kết, báo cáo và ghi nhận đóng góp
```

## 3. Phạm vi nghiệp vụ

### 3.1. Phạm vi cốt lõi phiên bản 1.0

1. Quản lý giáo xứ và cộng đoàn.
2. Quản lý mùa chuẩn bị Giáng sinh.
3. Quản lý khu vực hoặc công trình.
4. Quản lý công việc và yêu cầu kỹ năng.
5. Quản lý hồ sơ tình nguyện viên và kỹ năng.
6. Phân công hoặc đăng ký tham gia công việc.
7. Hỗ trợ nhân lực, kỹ năng và vật tư giữa các cộng đoàn.
8. Lập kế hoạch và theo dõi vật tư.
9. Yêu cầu mua vật tư, Work Area Leader phê duyệt, giao task mua cho volunteer và xác nhận giao hàng.
10. Ghi nhận cam kết quyên góp (pledge) và tiếp nhận quyên góp thực tế.
11. Quản lý vật tư hoặc dụng cụ cho mượn.
12. Phân bổ vật tư cho khu vực hoặc công việc.
13. Check-in có Leader duyệt, check-out và điều chỉnh giờ làm.
14. Cập nhật tiến độ, ảnh và nghiệm thu công việc.
15. Kiểm tra readiness và các hạng mục an toàn cơ bản.
16. Dashboard, báo cáo, lịch sử hoạt động và điểm ghi nhận.
17. Quản lý tài khoản, vai trò, quyền và danh mục cấu hình.

### 3.2. Ngoài phạm vi phiên bản 1.0

- Cổng thanh toán và xử lý giao dịch tài chính trực tuyến.
- Chat thời gian thực thay thế các ứng dụng nhắn tin.
- AI tự động ra quyết định phân công hoặc phê duyệt.
- AI kiểm tra chất lượng công trình từ hình ảnh.
- Dự báo nâng cao về giá, tồn kho hoặc tiến độ.
- Quản lý kế toán đầy đủ như một phần mềm tài chính.
- Thay thế việc kiểm tra an toàn bắt buộc theo quy định thực tế.

Hệ thống có thể ghi nhận khoản tiền hoặc vật tư đã thực tế tiếp nhận. Đây là dữ liệu đóng góp được nhập thủ công bởi nhân sự có quyền, không phải giao dịch thanh toán trực tuyến.

## 4. Cơ cấu tổ chức và vai trò

### 4.1. Giáo xứ

Giáo xứ là đơn vị quản lý cao nhất trong một tenant nghiệp vụ. Một giáo xứ có thể có nhiều cộng đoàn, nhiều người dùng và nhiều mùa Giáng sinh qua các năm.

### 4.2. Cộng đoàn

Cộng đoàn là nhóm địa phương trực thuộc giáo xứ. Một cộng đoàn có thể phụ trách nhiều khu vực trong cùng một mùa. Tình nguyện viên có một cộng đoàn gốc nhưng vẫn có thể hỗ trợ khu vực của cộng đoàn khác theo quy trình hỗ trợ chéo.

### 4.3. Ban phụ trách Giáng sinh

Ban phụ trách có góc nhìn toàn giáo xứ và chịu trách nhiệm:

- Tạo và kích hoạt mùa Giáng sinh.
- Tạo hoặc phân bổ khu vực cho các cộng đoàn.
- Chỉ định trưởng khu vực và cán bộ vật tư.
- Theo dõi tiến độ, thiếu hụt, yêu cầu hỗ trợ và readiness.
- Theo dõi purchase request, chi phí thực tế so với ngân sách dự kiến của mùa.
- Điều phối hỗ trợ nhân lực, kỹ năng và vật tư giữa các cộng đoàn.
- Xem và xuất báo cáo tổng kết.

Ban phụ trách không nhất thiết trực tiếp tạo hoặc thực hiện từng task.

### 4.4. Trưởng khu vực

Trưởng khu vực chịu trách nhiệm trong phạm vi khu vực được giao:

- Chia khu vực thành các task có thể thực hiện và nghiệm thu.
- Xác định thời hạn, số người, thời lượng, kỹ năng, vật tư và tiêu chí hoàn thành.
- Phân công tình nguyện viên hoặc xét duyệt đăng ký.
- Theo dõi tiến độ và xử lý công việc bị chậm hoặc bị chặn.
- Nghiệm thu bằng chứng do tình nguyện viên gửi.
- Yêu cầu sửa lại nếu kết quả chưa đạt.
- Duyệt từng yêu cầu check-in của volunteer và xét duyệt yêu cầu điều chỉnh giờ làm.
- Duyệt hoặc từ chối purchase request của work area; sau khi duyệt, tạo task mua và giao volunteer đi mua.
- Tạo support request khi work area thiếu người, kỹ năng hoặc vật tư.
- Tạo readiness checklist cho work area, thực hiện kiểm tra và tạo việc khắc phục.

### 4.5. Cán bộ vật tư và đóng góp

Cán bộ vật tư chịu trách nhiệm:

- Quản lý kho vật tư logic của work area được phân công; kho của mỗi work area được theo dõi tách biệt.
- Lập danh sách vật tư cho work area hoặc task.
- Theo dõi số lượng cần, hiện có, đã xác nhận nhận, đã phân bổ và còn thiếu.
- Tạo yêu cầu mua cho phần thiếu hụt và gửi Work Area Leader duyệt.
- Xác nhận số lượng, chi phí và tình trạng vật tư khi người mua giao hàng.
- Ghi nhận donation trực tiếp; kiểm tra và xác nhận từng lần nhận thực tế của pledge.
- Tiếp nhận hoặc từ chối loan offer do volunteer nhập; theo dõi vật tư hoặc dụng cụ mượn và hoàn trả.
- Đồng ý xuất vật tư khi work area của mình hỗ trợ cộng đoàn khác; xác nhận số lượng nhận được khi work area của mình được hỗ trợ.
- Phân bổ vật tư đã xác nhận cho work area hoặc task.
- Xác minh báo cáo vật tư sau khi task đã được Leader xác nhận hoàn thành.

### 4.6. Tình nguyện viên và người đóng góp

Người dùng mobile app có thể:

- Khai báo hồ sơ, cộng đoàn, kỹ năng và thời gian có thể tham gia.
- Xem task phù hợp hoặc yêu cầu hỗ trợ.
- Chấp nhận hoặc từ chối task được phân công.
- Đăng ký tham gia task và chờ trưởng khu vực duyệt.
- Gửi yêu cầu check-in và chờ Work Area Leader duyệt; check-out khi kết thúc; yêu cầu sửa giờ nếu có sai sót.
- Cập nhật tiến độ bằng phần trăm, ghi chú và hình ảnh.
- Nhận task mua (task loại `PURCHASE`), nhập thông tin mua và tải hóa đơn.
- Cam kết quyên góp vật tư hoặc tiền (pledge) cho nhu cầu đang thiếu.
- Nhập thông tin vật tư hoặc dụng cụ cho mượn, kể cả thay cho người cho mượn không có tài khoản.
- Có thể hỗ trợ task của cộng đoàn khác thông qua support request.
- Xem giờ làm, task hoàn thành, đóng góp và điểm ghi nhận của bản thân.

### 4.7. Quản trị viên hệ thống

Quản trị viên quản lý cấu hình kỹ thuật và dữ liệu nền, không trực tiếp điều hành công việc Giáng sinh:

- Tài khoản, gán giáo xứ cho tài khoản, khóa hoặc mở khóa tài khoản.
- Vai trò, quyền và phạm vi dữ liệu.
- Giáo xứ, cộng đoàn và các danh mục.
- Danh mục kỹ năng, loại vật tư, loại khu vực và loại đóng góp.
- Quy tắc tính điểm ghi nhận (có phiên bản và ngày hiệu lực).
- Cấu hình thông báo và lịch sử hoạt động.
- Hỗ trợ sao lưu và xuất dữ liệu.

## 5. Mô hình nghiệp vụ cốt lõi

```text
Parish
  |-- Communities
  |-- Users and Roles
  `-- Christmas Seasons
        |-- Work Areas
        |     |-- Tasks
        |     |     |-- Required Skills
        |     |     |-- Assignments
        |     |     |-- Attendance Requests, Sessions, Corrections
        |     |     |-- Progress Evidence
        |     |     `-- Task Material Reports
        |     |-- Material Requirements
        |     |-- Material Allocations
        |     `-- Readiness Checklists
        |-- Purchase Requests -> Purchase Tasks -> Purchases
        |-- Donations (Pledges and Receipts)
        |-- Borrowed Items
        |-- Support Requests (Staffing and/or Material)
        `-- Reports, Recognition, Notifications, Activity Log
```

Mùa Giáng sinh là aggregate hoặc business backbone chính. Mọi dữ liệu vận hành phải xác định được mình thuộc giáo xứ và mùa nào để không trộn dữ liệu giữa các năm.

## 6. Đối tượng nghiệp vụ

| Đối tượng | Ý nghĩa | Quan hệ chính |
|---|---|---|
| Parish | Giáo xứ sử dụng hệ thống | Có nhiều community, user và season |
| Community | Cộng đoàn thuộc giáo xứ | Có tình nguyện viên và có thể phụ trách nhiều work area |
| ChristmasSeason | Mùa chuẩn bị theo năm | Chứa toàn bộ dữ liệu vận hành của một mùa |
| WorkArea | Khu vực hoặc công trình | Thuộc season, có leader, officer và nhiều task |
| Task | Đơn vị công việc có thể giao và nghiệm thu | Thuộc work area; loại `WORK`, `PURCHASE` (mua theo purchase request) hoặc `CORRECTIVE` (khắc phục readiness) |
| Skill | Kỹ năng chuẩn hóa | Quan hệ nhiều-nhiều với task và volunteer |
| VolunteerProfile | Hồ sơ tham gia của người dùng | Có community, skill, availability và contribution |
| VolunteerAvailability | Khung giờ rảnh có cấu trúc | Dùng để cảnh báo xung đột lịch khi phân công |
| TaskAssignment | Liên kết volunteer với task | Ghi nguồn phân công hoặc đăng ký và trạng thái phản hồi |
| AttendanceRequest | Yêu cầu check-in của volunteer | Leader duyệt từng yêu cầu; được duyệt mới tạo Attendance |
| Attendance | Phiên làm việc check-in/check-out | Thuộc một assignment đã chấp nhận, mở từ một AttendanceRequest đã duyệt |
| AttendanceCorrection | Yêu cầu sửa giờ | Lưu giờ cũ, giờ đề nghị, lý do, người duyệt |
| TaskProgress | Lần cập nhật tiến độ | Có phần trăm, ghi chú, ảnh, người gửi và thời gian |
| Material | Danh mục vật tư hoặc dụng cụ | Có đơn vị tính và thuộc tính tái sử dụng |
| MaterialRequirement | Nhu cầu vật tư cấp work area | Thuộc work area, dùng để lập kế hoạch và tính shortage |
| TaskMaterial | Nhu cầu vật tư cấp task | Gắn material và required quantity với một task |
| PurchaseRequest | Yêu cầu mua phần vật tư thiếu | MO tạo, Work Area Leader duyệt; có số lượng, dự toán, lý do và trạng thái |
| Purchase | Kết quả mua thực tế | Gắn với task mua; có người mua, số lượng, giá, nhà cung cấp và hóa đơn |
| Donation | Một khoản đóng góp (pledge hoặc ghi nhận trực tiếp) | Có trạng thái PLEDGED / PARTIALLY_RECEIVED / RECEIVED / CANCELLED; MATERIAL gắn work area nhận |
| DonationItem | Một vật tư trong MATERIAL donation | Lưu pledged quantity và tổng received/unusable quantity |
| DonationReceipt | Một lần nhận thực tế | Ghi số tiền hoặc số lượng nhận, không dùng được; là căn cứ cộng stock |
| BorrowedItem | Vật tư hoặc dụng cụ mượn | Volunteer nhập thay người cho mượn; có tên/liên hệ chủ sở hữu, ngày trả dự kiến và tình trạng khi trả |
| MaterialAllocation | Phân bổ vật tư đã xác nhận | Gắn số lượng thực cấp với work area hoặc task; allocated quantity là dữ liệu lịch sử không bị ghi đè bởi báo cáo sau task |
| TaskMaterialReport | Báo cáo vật tư sau khi task hoàn thành | Volunteer báo cáo vật tư đã được cấp; Material Officer xác minh |
| TaskMaterialReportItem | Chi tiết báo cáo theo allocation item | Lưu remaining, damaged và lost ở mức reported và verified |
| SupportRequest | Yêu cầu hỗ trợ liên cộng đoàn | Loại STAFFING / MATERIAL / MIXED; chứa vị trí cần người và/hoặc vật tư cần, hạn và trạng thái |
| SupportRequestMaterial | Vật tư cần trong support request | Material và số lượng cần |
| SupportResponseMaterial | Đề nghị chuyển vật tư từ cộng đoàn khác | Work area nguồn, số lượng đề nghị/duyệt/nhận |
| ReadinessChecklist | Lần kiểm tra sẵn sàng | Leader tạo cho work area; gồm nhiều item bắt buộc hoặc tùy chọn |
| Contribution | Sự kiện đóng góp đã được công nhận | Là cơ sở tính báo cáo và điểm |
| RecognitionEntry | Điểm phát sinh từ contribution | Lưu quy tắc và phiên bản đã áp dụng |
| Notification | Thông báo nghiệp vụ | Gắn người nhận, sự kiện và trạng thái gửi |
| ActivityLog | Lịch sử hành động quan trọng | Lưu actor, thời gian, đối tượng và nội dung thay đổi |
| FileObject | Ảnh, hóa đơn, tài liệu đính kèm | Lưu metadata, người upload, mục đích và record được gắn |

## 7. Quy trình nghiệp vụ tổng thể

### 7.1. Giai đoạn lập kế hoạch

1. Ban phụ trách tạo mùa Giáng sinh ở trạng thái `Planning`.
2. Ban phụ trách xác nhận các cộng đoàn tham gia.
3. Ban phụ trách tạo các khu vực cần triển khai.
4. Mỗi khu vực được gán trưởng khu vực, cán bộ vật tư, lịch và hạn hoàn thành.
5. Trưởng khu vực chia công trình thành các task.
6. Mỗi task được gắn kỹ năng, số người, thời lượng, vật tư và tiêu chí hoàn thành.
7. Cán bộ vật tư lập nhu cầu và xác định thiếu hụt.
8. Mùa được kích hoạt khi tối thiểu có khu vực và các vai trò bắt buộc.

### 7.2. Giai đoạn huy động nguồn lực

1. Trưởng khu vực phân công trực tiếp hoặc mở task cho tình nguyện viên đăng ký.
2. Hệ thống hiển thị mức phù hợp theo kỹ năng, lịch rảnh, cộng đoàn và xung đột lịch.
3. Khi thiếu người, kỹ năng hoặc vật tư, trưởng khu vực tạo support request toàn giáo xứ.
4. Thiếu hụt vật tư được xử lý bằng một hoặc nhiều nguồn:
   - Vật tư hiện có trong kho logic của work area.
   - Mua mới (MO tạo request, Leader duyệt và giao task mua).
   - Pledge của volunteer và donation thực tế đã tiếp nhận.
   - Mượn.
   - Chuyển kho từ work area của cộng đoàn khác qua support request.
5. Chỉ vật tư thực tế đã nhận, còn dùng được và đã được Material Officer xác nhận mới được tính là khả dụng cho work area.

### 7.3. Giai đoạn thực hiện

1. Tình nguyện viên chấp nhận assignment hoặc được duyệt đăng ký.
2. Tình nguyện viên gửi yêu cầu check-in khi bắt đầu tại địa điểm làm việc; Leader duyệt từng yêu cầu thì phiên làm việc mới được mở.
3. Tình nguyện viên thực hiện task và có thể gửi nhiều lần cập nhật tiến độ.
4. Tình nguyện viên check-out khi kết thúc phiên làm.
5. Khi cho rằng task đã hoàn thành, tình nguyện viên gửi bằng chứng và chuyển task sang `Submitted`.
6. Trưởng khu vực nghiệm thu:
   - Đạt: chuyển `Completed`.
   - Chưa đạt: chuyển `Revision Required`, ghi lý do và yêu cầu sửa.
7. Sau khi task ở trạng thái `Completed`, volunteer phụ trách gửi material report nếu task có vật tư đã được phân bổ.
8. Material Officer của work area kiểm tra thực tế và xác minh material report trước khi hệ thống cập nhật số liệu vật tư chính thức.

### 7.4. Giai đoạn readiness và tổng kết

1. Trưởng khu vực tạo checklist cho work area và nhập các mục kiểm tra (bắt buộc hoặc tùy chọn).
2. Từng mục được đánh dấu `Passed` hoặc `Failed`, kèm ghi chú và bằng chứng nếu cần.
3. Mục bắt buộc bị fail phải tạo hoặc liên kết với corrective task.
4. Sau khi sửa, trưởng khu vực kiểm tra lại.
5. Work area chỉ được `Ready` khi mọi mục bắt buộc đã pass.
6. Ban phụ trách theo dõi readiness toàn mùa.
7. Khi công việc kết thúc, hệ thống tổng hợp chi phí, vật tư, giờ làm, đóng góp và điểm.
8. Ban phụ trách hoàn tất rồi lưu trữ mùa để tra cứu lịch sử.

## 8. Phân tích chi tiết các quy trình chính

### 8.1. Tạo và kích hoạt mùa Giáng sinh

**Tác nhân chính:** Ban phụ trách Giáng sinh.

**Điều kiện trước:** Giáo xứ và tài khoản ban phụ trách đang hoạt động.

**Luồng chuẩn:**

1. Nhập tên mùa, năm, ngày bắt đầu, ngày kết thúc, mô tả và ngân sách dự kiến.
2. Hệ thống kiểm tra tên hoặc năm trùng và thời gian không hợp lệ.
3. Lưu mùa ở trạng thái `Planning`.
4. Tạo các work area và chọn community phụ trách nếu có.
5. Chỉ định leader và material officer.
6. Nhập ngân sách dự kiến và các quy tắc áp dụng.
7. Kích hoạt mùa (`Planning` → `Active`).
8. Hệ thống gửi thông báo tới người được giao vai trò.

**Ngoại lệ:**

- Không kích hoạt nếu chưa có work area.
- Không kích hoạt nếu một work area đang hoạt động nhưng thiếu leader hoặc material officer.
- Ngày kết thúc phải sau ngày bắt đầu.
- Không cho phép hai mùa `Active` trùng thời gian nếu chính sách giáo xứ không cho phép.

### 8.2. Tạo task và phân công tình nguyện viên

**Tác nhân chính:** Trưởng khu vực.

**Luồng chuẩn:**

1. Chọn work area đang quản lý.
2. Tạo task với tên, mô tả, ngày, deadline, mức ưu tiên và tiêu chí hoàn thành.
3. Xác định số volunteer và thời lượng dự kiến.
4. Gắn một hoặc nhiều skill.
5. Gắn nhu cầu vật tư liên quan.
6. Công bố task hoặc chọn volunteer để phân công.
7. Hệ thống hiển thị skill, community, availability và cảnh báo xung đột.
8. Volunteer nhận thông báo và chọn `Accepted` hoặc `Declined`.

**Đăng ký tự nguyện:**

1. Volunteer xem available task.
2. Volunteer gửi đăng ký.
3. Assignment ở trạng thái `Pending`.
4. Leader duyệt hoặc từ chối.
5. Khi duyệt, capacity còn lại giảm tương ứng.

**Ngoại lệ:**

- Tài khoản bị khóa hoặc inactive không được tham gia.
- Khi task đã đủ người, đăng ký mới bị chặn hoặc đưa vào danh sách chờ.
- Leader có thể override skill mismatch nếu có quyền và bắt buộc nhập lý do.
- Không xóa cứng task đã có assignment; chỉ được cancel và lưu lý do.

### 8.3. Check-in, check-out và sửa giờ

**Luồng chuẩn:**

1. Volunteer mở task đã `Accepted`.
2. Chọn check-in trong cửa sổ thời gian cho phép.
3. Hệ thống tạo yêu cầu check-in `Pending`, lưu thời gian máy chủ tại lúc gửi (`requested_at`) và thông báo cho Leader của work area.
4. Leader xem danh sách yêu cầu và duyệt **từng yêu cầu của từng người** (không duyệt hàng loạt):
   - `Approved`: hệ thống mở attendance session với `check_in_at = requested_at`.
   - `Rejected`: bắt buộc nhập lý do; không tạo session, không tính giờ.
5. Volunteer chọn check-out khi kết thúc; check-out không cần duyệt.
6. Hệ thống lưu thời gian và tính số phút làm việc.

Volunteer có thể hủy yêu cầu check-in đang `Pending`. Nếu Leader duyệt trễ, giờ làm vẫn tính từ lúc volunteer gửi yêu cầu.

```text
Working Duration = Check-out Time - Check-in Time
```

**Sửa giờ:**

1. Volunteer nhập thời gian đề nghị và lý do.
2. Yêu cầu ở trạng thái `Pending`.
3. Leader kiểm tra và `Approved` hoặc `Rejected`.
4. Chỉ thời gian đã duyệt mới đi vào báo cáo và tính điểm.

**Quy tắc:**

- Không check-out nếu chưa có check-in đã được duyệt.
- Một assignment chỉ có tối đa một yêu cầu check-in đang chờ hoặc một phiên đang mở.
- Các phiên đã duyệt không được chồng thời gian.
- Hệ thống phải cảnh báo phiên mở quá thời lượng cấu hình.

### 8.4. Cập nhật và nghiệm thu tiến độ

1. Volunteer nhập phần trăm hoàn thành, ghi chú và ảnh.
2. Mỗi cập nhật là một bản ghi mới, không ghi đè lịch sử.
3. Khi hoàn thành, volunteer gửi task để nghiệm thu.
4. Leader xem tiêu chí, ảnh, ghi chú, giờ làm và vật tư liên quan.
5. Leader chọn:
   - `Approve`: task thành `Completed`.
   - `Request Revision`: task thành `Revision Required` và phải có lý do.
6. Khi volunteer sửa xong, task được submit lại.

`Volunteer Submitted` không đồng nghĩa với `Task Completed`.

### 8.5. Lập kế hoạch vật tư và tính thiếu hụt

Mỗi material requirement cần tối thiểu:

- Material.
- Đơn vị tính.
- Số lượng cần.
- Số lượng hiện có đủ điều kiện sử dụng.
- Phạm vi work area hoặc task.
- Ghi chú kỹ thuật hoặc chất lượng.

Công thức nghiệp vụ:

```text
Confirmed Available Quantity
  = Opening Usable Stock
  + Confirmed Purchase Delivery
  + Confirmed Usable Donation
  + Confirmed Borrowed Quantity
  + Verified Usable Return From Task
  - Disposed / Damaged / Lost Quantity

Unallocated Quantity
  = Confirmed Available Quantity - Active Allocated Quantity

Shortage Quantity
  = max(0, Required Quantity - Confirmed Available Quantity)
```

Số lượng đã mua nhưng chưa được xác nhận giao, donation chưa thực tế tiếp nhận hoặc vật tư không dùng được không được tính vào `Confirmed Available Quantity`.

### 8.6. Thiếu vật tư đến hoàn tất mua sắm

1. Material Officer xác định `Shortage Quantity > 0`.
2. Material Officer tạo purchase request gồm một hoặc nhiều vật tư, số lượng, dự toán và lý do, rồi gửi duyệt.
3. Work Area Leader của chính work area đó review và chọn `Approved` hoặc `Rejected` (từ chối phải có lý do).
4. Sau khi duyệt, Leader tạo **task mua** (task loại `PURCHASE`) gắn với purchase request và giao cho một volunteer theo luồng assignment thông thường.
5. Volunteer chấp nhận task mua. Nếu volunteer từ chối, Leader giao lại cho người khác.
6. Volunteer nhập số lượng thực tế, giá, nhà cung cấp, ngày mua và ảnh hóa đơn.
7. Trạng thái mua thành `Purchased`, nhưng stock chưa tăng.
8. Khi hàng được giao, Material Officer kiểm tra số lượng và tình trạng.
9. Material Officer xác nhận số lượng usable thực nhận.
10. Hệ thống ghi material transaction, cập nhật stock của đúng work area, tính lại shortage và tự động chuyển task mua sang `Completed`.

**Ngoại lệ:**

- Không tạo request khi shortage bằng 0, trừ khi có quyền override và lý do.
- Không tạo task mua khi purchase request chưa được Leader duyệt.
- Chi phí thực tế vượt dự toán đã duyệt phải được Leader duyệt bổ sung (budget exception) trước khi xác nhận.
- Hóa đơn không hợp lệ không làm mất record mua; record được chuyển sang trạng thái cần bổ sung.
- Giao thiếu được xác nhận theo số lượng thực nhận và request còn phần chưa hoàn thành.

### 8.7. Pledge và ghi nhận donation thực tế

Donation có thể bắt đầu theo hai cách:

- **Pledge:** volunteer cam kết quyên góp tiền hoặc vật tư qua mobile app cho nhu cầu đang thiếu. Donation ở trạng thái `Pledged`, chưa làm thay đổi stock hay tổng tiền đã nhận.
- **Ghi nhận trực tiếp:** Material Officer hoặc Admin nhập khoản đã nhận ngay (người đóng góp không dùng app). Donation được tạo ở trạng thái `Received` cùng một lần nhận.

Mỗi lần nhận thực tế là một `DonationReceipt`. Một pledge có thể được giao nhiều lần.

Một donation có một trong hai loại:

- `MONEY`: lưu số tiền cam kết (nếu là pledge) và tổng số tiền thực nhận.
- `MATERIAL`: có một hoặc nhiều `DonationItem` và phải chọn work area nhận vật tư.

Ví dụ:

```text
MATERIAL Donation
  |-- 20 m dây điện: received 20, unusable 2
  |-- 10 bóng đèn: received 10, unusable 1
  `-- 5 hộp sơn: received 5, unusable 0
```

Quy trình pledge:

1. Volunteer xem danh sách nhu cầu (vật tư thiếu hoặc nhu cầu tiền) của season.
2. Volunteer chọn `MONEY` (nhập số tiền cam kết) hoặc `MATERIAL` (chọn vật tư, số lượng cam kết, work area nhận).
3. Hệ thống tạo hoặc dùng lại donor liên kết với tài khoản volunteer và lưu donation `Pledged`.
4. Material Officer nhận được thông báo pledge mới.
5. Khi nhận thực tế, Material Officer tạo một receipt: số tiền nhận, hoặc từng material với received quantity và unusable quantity sau khi kiểm tra.
6. `Usable Quantity = Received Quantity - Unusable Quantity`.
7. Chỉ usable quantity của từng receipt được cộng vào stock của work area nhận, đúng một lần cho mỗi receipt.
8. Donation chuyển `Partially Received` khi còn thiếu so với cam kết, `Received` khi đủ.
9. Hệ thống tạo (ở lần nhận đầu tiên) hoặc cập nhật contribution summary để phục vụ báo cáo và recognition.
10. Volunteer hoặc Material Officer có thể hủy phần pledge chưa giao, bắt buộc nhập lý do; các receipt đã nhận vẫn giữ nguyên.

Quy trình ghi nhận trực tiếp:

1. Officer/Admin chọn hoặc tạo donor.
2. Chọn loại donation, nhập số tiền hoặc vật tư đã nhận (MATERIAL thì chọn work area nhận).
3. Hệ thống tạo donation `Received` và một receipt trong cùng transaction, rồi thực hiện bước 6, 7, 9 như trên.

Pledge (số tiền hoặc số lượng cam kết) không bao giờ được tính vào stock hay tổng tiền đã nhận. Nhận vượt cam kết được cảnh báo nhưng vẫn ghi theo số thực nhận.

### 8.8. Quản lý vật tư hoặc dụng cụ mượn

1. Volunteer (hoặc Officer) tạo loan offer: tên item, số lượng, tình trạng ban đầu, ngày trả dự kiến và **thông tin người cho mượn** (tên, số điện thoại, địa chỉ). Người cho mượn không cần có tài khoản; volunteer nhập giùm và là đầu mối liên hệ trong hệ thống.
2. Material Officer xem loan offer và chọn `Accepted` hoặc `Rejected`.
3. Khi nhận đồ, Officer ghi ngày mượn (`Issued`).
4. Item có thể được giao cho work area hoặc task (`In Use`). Đồ mượn không thuộc danh mục vật tư và không tính vào stock.
4. Hệ thống nhắc trước và sau ngày trả dự kiến.
5. Khi trả, officer ghi ngày trả và tình trạng:
   - Good.
   - Damaged.
   - Lost.
6. Nếu damaged hoặc lost, hệ thống ghi nhận sự cố và người có trách nhiệm xử lý; volunteer đã nhập loan offer được thông báo để liên hệ người cho mượn.

### 8.9. Phân bổ vật tư

Việc nhận vật tư không đồng nghĩa vật tư đã được dành cho một công việc cụ thể.

```text
100 m dây điện đã xác nhận
  -> 30 m cho Hang đá
  -> 40 m cho Đèn sân nhà thờ
  -> 20 m cho Sân khấu
  -> 10 m chưa phân bổ
```

Quy tắc:

- Chỉ phân bổ từ số lượng đã xác nhận và còn chưa phân bổ.
- Tổng allocation đang hoạt động không được vượt unallocated quantity.
- Mỗi allocation phải gắn work area hoặc task.
- `allocated_quantity` là số lượng thực tế đã cấp và phải được giữ nguyên như dữ liệu lịch sử.
- Việc hoàn trả vật tư sau task chỉ ảnh hưởng stock sau khi Material Officer xác minh material report.
- Vật tư tiêu hao và vật tư tái sử dụng cần cách xử lý cuối mùa khác nhau.

### 8.10. Báo cáo vật tư sau khi hoàn thành task

Điều kiện trước: task đã được Work Area Leader xác nhận `Completed` và task có material allocation.

Quy trình:

1. Hệ thống tải danh sách `material_allocation_item` đã thực tế cấp cho task; volunteer không tự chọn material ngoài danh sách này.
2. Volunteer phụ trách gửi material report cho từng allocation item, gồm:
   - Remaining quantity.
   - Damaged quantity.
   - Lost quantity.
   - Ghi chú nếu cần.
3. Material Officer của đúng work area kiểm tra số lượng và tình trạng vật tư thực tế.
4. Officer nhập hoặc xác nhận các giá trị verified.
5. Chỉ giá trị `verified_*` mới được dùng để cập nhật dữ liệu vật tư chính thức.
6. `Verified Remaining Quantity` còn sử dụng được được trả về available stock của **chính work area đó**.
7. `Verified Damaged Quantity` và `Verified Lost Quantity` không được trả lại usable stock.
8. Lượng đã sử dụng có thể được suy ra khi cần:

```text
Used Quantity
  = Allocated Quantity
  - Verified Remaining Quantity
  - Verified Damaged Quantity
  - Verified Lost Quantity
```

Material report không sửa lại `allocated_quantity`; allocation tiếp tục là lịch sử số lượng đã cấp.

### 8.11. Hỗ trợ nhân lực và vật tư giữa các cộng đoàn

Support request dùng cho **nhân lực, kỹ năng và/hoặc vật tư**. Loại request: `STAFFING` (chỉ người), `MATERIAL` (chỉ vật tư) hoặc `MIXED` (cả hai).

**Phần nhân lực:** một request có thể gồm nhiều vị trí (`position`). Mỗi position xác định:

- Tên vị trí hoặc loại công việc.
- Số volunteer cần.
- Một hoặc nhiều skill bắt buộc cho mỗi người nếu có.
- Deadline và ghi chú.

**Phần vật tư:** request liệt kê từng material và số lượng cần. Vật tư hỗ trợ là vật tư **chuyển từ kho work area của cộng đoàn khác**; volunteer cá nhân muốn tặng vật tư dùng luồng pledge (8.7).

Quy trình:

1. Work Area Leader tạo support request khi work area thiếu người, kỹ năng hoặc vật tư.
2. Request được công bố trong phạm vi giáo xứ.
3. Volunteer hoặc community khác gửi response:
   - Nhân lực: danh sách người có thể hỗ trợ theo position.
   - Vật tư: work area nguồn và số lượng đề nghị cho từng material; Material Officer của work area nguồn phải đồng ý.
4. Hệ thống kiểm tra kỹ năng, capacity, tránh gán trùng một người vào nhiều position đang active trong cùng request, và kiểm tra số lượng vật tư đề nghị không vượt unallocated stock của work area nguồn.
5. Leader của work area yêu cầu xác nhận từng người và từng dòng vật tư phù hợp.
6. Volunteer được xác nhận tham gia task/work area tương ứng thông qua assignment.
7. Vật tư được duyệt: hệ thống ghi `TRANSFER_OUT` ở work area nguồn; khi nhận hàng, Material Officer bên nhận xác nhận số lượng usable và hệ thống ghi `TRANSFER_IN` cho phần usable.
8. Khi đủ số người và đủ vật tư yêu cầu, request thành `Fulfilled`; nếu chỉ đáp ứng một phần thì `Partially Fulfilled`.

### 8.12. Readiness và an toàn

Checklist do Work Area Leader tạo cho từng work area (không dùng mẫu cấu hình sẵn). Leader nhập từng mục và đánh dấu bắt buộc hoặc tùy chọn. Ví dụ với hang đá:

- Công trình chính đã hoàn thành.
- Trang trí đã hoàn thành.
- Hệ thống đèn hoạt động.
- Không có dây điện hở.
- Khung và các chi tiết đứng vững.
- Không có cạnh sắc hoặc vật dễ đổ.
- Không cản trở lối đi.
- Khu vực đã được dọn sạch.
- Người có trách nhiệm đã kiểm tra cuối.

Nếu một mục bắt buộc fail:

```text
Failed Item
  -> Create Corrective Task (task loại CORRECTIVE, gắn vào item)
  -> Assign Responsible Person
  -> Fix and Upload Evidence
  -> Recheck
  -> Passed
```

## 9. Vòng đời và trạng thái

### 9.1. Christmas Season

```text
Planning -> Active -> Ready -> Completed -> Archived
```

| Trạng thái | Ý nghĩa |
|---|---|
| Planning | Trạng thái khi tạo mùa; đang nhập thông tin, tạo area, task và nguồn lực |
| Active | Đang triển khai thực tế |
| Ready | Các khu vực bắt buộc đã đạt readiness |
| Completed | Mùa đã tổng kết nghiệp vụ |
| Archived | Dữ liệu lịch sử, hạn chế chỉnh sửa |

### 9.2. Work Area

```text
Planned -> In Progress -> Readiness Review -> Ready -> Completed
                       `-> Needs Correction -> In Progress
```

### 9.3. Task

```text
Not Started -> In Progress -> Submitted -> Completed
                              `-> Revision Required -> In Progress

Not Started/In Progress -> Cancelled
```

Task loại `PURCHASE` không cần Leader nghiệm thu: hệ thống tự chuyển `Completed` khi purchase của task được Material Officer xác nhận giao hàng.

### 9.4. Task Assignment

```text
Pending -> Accepted -> In Progress -> Completed
       `-> Declined
       `-> Cancelled
```

### 9.4a. Attendance Request và Attendance Session

```text
Check-in Request: Pending -> Approved   (mở session)
                         `-> Rejected   (bắt buộc lý do)
                         `-> Cancelled  (volunteer hủy)

Attendance Session: Open -> Closed -> Corrected (khi correction được duyệt)
Attendance Correction: Pending -> Approved / Rejected
```

### 9.5. Purchase Request / Purchase

Mọi purchase request đều do Work Area Leader duyệt:

```text
Draft -> Pending Approval -> Approved -> Buyer Assigned -> Purchased -> Partially Received -> Completed
                         `-> Rejected                                `-> Completed
Draft/Pending Approval/Approved -> Cancelled
```

- `Buyer Assigned`: volunteer đã chấp nhận task mua.
- `Purchased`: volunteer đã gửi thông tin mua; stock chưa tăng.
- `Partially Received`: Material Officer đã xác nhận một phần; request còn phần chưa hoàn thành.

Mỗi lần mua (`Purchase`): `Submitted -> Confirmed`, hoặc `Submitted -> Needs Info` khi hóa đơn không hợp lệ.

`Purchased` chưa làm tăng stock. Chỉ sau khi Material Officer xác nhận delivery/usable quantity thì số lượng mới được ghi nhận chính thức.

### 9.6. Donation

```text
Pledged -> Partially Received -> Received
   |              |
   `--------------`-> Cancelled (bắt buộc lý do; receipt đã nhận giữ nguyên)

Ghi nhận trực tiếp -> Received
```

Mỗi lần nhận thực tế (`DonationReceipt`): kiểm tra số lượng/tình trạng -> cộng usable quantity vào stock -> cập nhật contribution summary.

Trạng thái của MATERIAL donation item (`Pledged`, `Partially Received`, `Fully Received`, `Unusable`) được suy ra từ pledged, received và unusable quantity, không lưu như workflow độc lập.

### 9.7. Borrowed Item

```text
Offered (volunteer nhập) -> Accepted (MO) -> Issued -> In Use -> Returned
                        `-> Rejected
Overdue = quá expected return date mà chưa Returned (suy ra, không lưu)
Returned -> Good / Damaged / Lost
```

### 9.7a. Support Request

```text
Draft -> Published -> Partially Fulfilled -> Fulfilled -> Closed
Draft/Published -> Cancelled
Support Response Material: Pending -> Approved -> Transferred
                                  `-> Rejected / Cancelled
```

### 9.8. Readiness Item

```text
Pending -> Passed
       `-> Failed -> Corrective Action -> Recheck -> Passed/Failed
```

### 9.9. Task Material Report

```text
Task Completed
  -> Material Report Submitted
  -> Material Officer Verification
       |-> Verified
       `-> Revision Required -> Resubmitted
```

Chỉ report đã được Material Officer xác minh mới làm thay đổi official material data.

## 10. Quy tắc nghiệp vụ

### 10.1. Tổ chức và mùa

- **BR-01:** Một giáo xứ có nhiều season nhưng dữ liệu của các season không được trộn lẫn.
- **BR-02:** Tên season và năm season đều phải duy nhất trong phạm vi giáo xứ.
- **BR-03:** Ngày kết thúc phải sau ngày bắt đầu.
- **BR-04:** Work area thuộc đúng một season.
- **BR-05:** Work area có thể là cấp giáo xứ hoặc thuộc một community.
- **BR-06:** Mỗi work area hoạt động có một leader và một material officer chính.
- **BR-07:** Dữ liệu season đã archive chỉ được thay đổi qua quyền đặc biệt và phải ghi log.

### 10.2. Task và volunteer

- **BR-08:** Task thuộc đúng một work area.
- **BR-09:** Lịch task phải nằm trong lịch của work area và season, trừ trường hợp override.
- **BR-10:** Một task có thể cần nhiều skill và nhiều volunteer.
- **BR-11:** Chỉ tài khoản active mới được assign hoặc đăng ký.
- **BR-12:** Assignment trực tiếp phải được volunteer phản hồi.
- **BR-13:** Assignment bị decline không được tính là contribution.
- **BR-14:** Không vượt capacity trừ khi leader có quyền override và nhập lý do.
- **BR-15:** Skill mismatch hoặc schedule conflict phải hiển thị cảnh báo.
- **BR-16:** Volunteer submit không tự động làm task completed.
- **BR-17:** Revision Required phải có lý do và không làm mất evidence cũ.

### 10.3. Attendance

- **BR-18:** Check-out phải có phiên đang mở được tạo từ yêu cầu check-in đã được Leader duyệt.
- **BR-19:** Check-out phải sau check-in.
- **BR-20:** Không cho phép hai attendance session đã duyệt bị chồng thời gian.
- **BR-21:** Chỉ giờ được duyệt mới dùng để báo cáo và tính điểm.
- **BR-22:** Mọi sửa giờ phải lưu thời gian cũ, thời gian mới, lý do và người duyệt.

### 10.4. Vật tư và mua sắm

- **BR-23:** Mỗi work area có một Material Officer chính chịu trách nhiệm quản lý kho vật tư logic của work area đó.
- **BR-24:** Stock của các work area được theo dõi tách biệt; vật tư của work area này không tự động trở thành available stock của work area khác.
- **BR-25:** Shortage không bao giờ âm trong báo cáo.
- **BR-26:** Vật tư đã mua nhưng chưa được Material Officer xác nhận giao không được tính vào stock.
- **BR-27:** Purchase request phải thuộc một work area, chứa ít nhất một material item khi submit và có lý do.
- **BR-28:** Mọi purchase request phải được Work Area Leader duyệt trước khi tạo task mua.
- **BR-29:** Leader chỉ duyệt purchase request của work area mình phụ trách.
- **BR-30:** Leader giao volunteer đi mua bằng task loại `PURCHASE` gắn với purchase request đã duyệt; người mua là volunteer có assignment `Accepted` trên task đó.
- **BR-31:** Từ chối phê duyệt phải có lý do.
- **BR-32:** Chi phí thực tế vượt dự toán đã duyệt phải được Leader duyệt bổ sung (budget exception) trước khi xác nhận.
- **BR-33:** Tổng allocation không vượt confirmed unallocated quantity.
- **BR-34:** `allocated_quantity` là lịch sử số lượng thực tế đã cấp và không được ghi đè bởi kết quả sử dụng sau task.

### 10.5. Donation và borrowing

- **BR-35:** Donation bắt đầu từ pledge của volunteer (`Pledged`) hoặc từ ghi nhận trực tiếp của Officer/Admin (`Received`); mỗi lần nhận thực tế là một DonationReceipt.
- **BR-36:** MONEY donation lưu số tiền cam kết (nếu có) và tổng tiền thực nhận; không có DonationItem.
- **BR-37:** MATERIAL donation phải có ít nhất một DonationItem và phải xác định work area nhận vật tư.
- **BR-38:** Chỉ `received_quantity - unusable_quantity` của từng receipt được cộng stock, đúng một lần cho mỗi receipt.
- **BR-39:** Số tiền hoặc số lượng cam kết (pledge) không được tính vào stock hay tổng tiền đã nhận.
- **BR-40:** Contribution bằng tiền ở phiên bản 1.0 không phải giao dịch thanh toán online.
- **BR-41:** Borrowed item bắt buộc có tên người cho mượn và expected return date; volunteer có thể nhập thông tin thay cho người cho mượn không có tài khoản.
- **BR-42:** Khi borrowed item returned phải ghi tình trạng.
- **BR-43:** Vật tư damaged hoặc lost không được đưa lại vào usable stock.

### 10.6. Báo cáo vật tư sau task

- **BR-44:** Work Area Leader phải xác nhận Task `Completed` trước khi volunteer gửi material report cuối task.
- **BR-45:** Material report chỉ được lập từ các `material_allocation_item` đã thực tế cấp cho task.
- **BR-46:** Volunteer chỉ báo cáo trạng thái vật tư; volunteer không trực tiếp cập nhật official stock.
- **BR-47:** Material Officer của đúng work area là người xác minh material report.
- **BR-48:** Chỉ các giá trị verified mới ảnh hưởng dữ liệu vật tư chính thức.
- **BR-49:** Verified usable remaining quantity phải quay lại available stock của cùng work area.
- **BR-50:** Verified damaged/lost quantity không được quay lại usable stock.
- **BR-51:** Việc verify material report không được sửa `allocated_quantity` lịch sử.

### 10.7. Readiness và ghi nhận

- **BR-52:** Work area không được Ready nếu còn mục mandatory bị thiếu hoặc fail.
- **BR-53:** Reopen readiness phải có quyền và lý do.
- **BR-54:** Contribution chỉ được tính từ sự kiện nghiệp vụ đã xác nhận.
- **BR-55:** Một sự kiện contribution không được tính điểm hai lần.
- **BR-56:** Điểm ghi nhận không có giá trị tiền mặt.
- **BR-57:** Báo cáo chỉ chứa dữ liệu người yêu cầu có quyền xem.

### 10.8. Bổ sung v1.1

- **BR-58:** Volunteer chỉ gửi yêu cầu check-in cho assignment `Accepted`; mỗi assignment tối đa một yêu cầu check-in `Pending` và không được gửi khi đang có phiên mở.
- **BR-59:** Work Area Leader của task duyệt từng yêu cầu check-in của từng người, không duyệt hàng loạt; từ chối phải có lý do.
- **BR-60:** Thời điểm check-in là thời điểm máy chủ nhận yêu cầu, không phải thời điểm Leader duyệt.
- **BR-61:** Yêu cầu check-in chưa được duyệt hoặc bị từ chối không tạo attendance và không tính giờ.
- **BR-62:** Support request phải có ít nhất một position (STAFFING), một dòng vật tư (MATERIAL) hoặc cả hai (MIXED) khi công bố.
- **BR-63:** Vật tư hỗ trợ chỉ lấy từ unallocated stock của work area nguồn và phải được Material Officer của work area nguồn đồng ý.
- **BR-64:** Chuyển kho hỗ trợ ghi `TRANSFER_OUT` ở work area nguồn khi được duyệt; stock bên nhận chỉ tăng phần usable sau khi Material Officer bên nhận xác nhận.
- **BR-65:** Task loại `PURCHASE` tự chuyển `Completed` khi purchase của task được Material Officer xác nhận giao hàng.
- **BR-66:** Mục readiness bắt buộc bị `Failed` phải gắn một corrective task (task loại `CORRECTIVE`).
- **BR-67:** Mọi tài khoản trừ System Admin phải gắn với một giáo xứ; người dùng chỉ truy cập dữ liệu của giáo xứ đó.
- **BR-68:** Recognition point phải lưu contribution gốc và phiên bản rule; một contribution không được tính điểm hai lần theo cùng một rule.
- **BR-69:** File chỉ được gắn vào record sau khi upload hoàn tất, bởi chính người upload; truy cập file qua URL ngắn hạn có kiểm tra quyền.
- **BR-70:** Khi volunteer có lịch rảnh, hệ thống cảnh báo nếu task nằm ngoài lịch rảnh hoặc trùng assignment khác.

## 11. Phân quyền nghiệp vụ

Ký hiệu: `M` quản lý, `A` phê duyệt, `S` gửi dữ liệu, `V` xem, `-` không có quyền. Dấu `*` nghĩa là chỉ trong phạm vi được giao.

| Chức năng | Committee | Leader | Material Officer | Volunteer | Admin |
|---|:---:|:---:|:---:|:---:|:---:|
| Season | M | V | V | V | A |
| Community và Work Area | M | M* | V* | V | A |
| Task | V | M* | V* | S/V* | A |
| Assignment | V | M/A* | V* | S/V bản thân | A |
| Attendance (duyệt check-in, sửa giờ) | V | A* | V* | S/M bản thân | A |
| Progress | V | A* | V* | S bản thân | A |
| Material Requirement | V | V* | M* | V | A |
| Purchase Request | V | A* (duyệt, tạo task mua) | M/S* | S/V task mua của bản thân | A |
| Donation | V | V* | M* (ghi nhận, xác nhận nhận) | S pledge bản thân / V | A/M |
| Borrowed Item | V | V* | M/A* | S loan offer (kể cả nhập giùm) / V bản thân | A |
| Support Request | V/điều phối | M* | M* (đồng ý xuất/xác nhận nhận vật tư) | S response | A |
| Allocation | V | V* | M* | V | A |
| Task Material Report | V | V* | A/M* | S bản thân* | A |
| Readiness | V/A | M/A* | V* | S bằng chứng | A |
| Report | V/Export | V* | V* | V bản thân | A |
| User, Role, Category | - | - | - | - | M |

## 12. Báo cáo và chỉ số

### 12.1. Dashboard Ban phụ trách

- Tổng số work area và trạng thái.
- Tiến độ toàn mùa và theo community.
- Số task chưa giao, trễ hạn, revision required và completed.
- Vật tư còn thiếu và giá trị dự toán.
- Purchase request chờ Leader duyệt, đang mua và chờ xác nhận giao.
- Pledge đang chờ nhận và donation thực tế đã nhận: money, material received, unusable và usable quantity.
- Support request đang mở (nhân lực và vật tư).
- Yêu cầu check-in đang chờ duyệt theo work area.
- Số volunteer, giờ làm và tỷ lệ tham gia.
- Readiness theo work area và số mandatory item failed.

### 12.2. Báo cáo cuối mùa

- Danh sách và trạng thái work area.
- Task theo trạng thái, deadline và người phụ trách.
- Vật tư cần, đã có, đã mua, được tặng, được mượn, đã phân bổ và còn dư.
- Ngân sách dự kiến của mùa, chi phí dự toán, được duyệt và thực tế.
- Donation theo donor, loại donation, số tiền hoặc material cam kết/received/usable.
- Vật tư chuyển kho giữa các cộng đoàn qua support request.
- Borrowed item và tình trạng hoàn trả.
- Attendance và giờ làm theo volunteer, community và work area.
- Readiness và lịch sử corrective action.
- Contribution và recognition point.

Mọi báo cáo cần ghi rõ giáo xứ, season, thời gian tạo, bộ lọc, đơn vị tính và phạm vi dữ liệu.

## 13. Điểm ghi nhận đóng góp

Điểm được tạo từ contribution event đã xác nhận. Ví dụ:

| Sự kiện | Điểm minh họa |
|---|---:|
| Hoàn thành một task được duyệt | 10 |
| Mỗi 5 giờ làm được duyệt | 20 |
| Donation thực tế đã nhận và được xác nhận | Theo loại hoặc giá trị cấu hình |
| Hoàn thành task mua (purchase được xác nhận giao) | 10 |
| Hỗ trợ khẩn cấp | 20 |
| Hỗ trợ community khác | 25 |

Các con số trên chỉ là ví dụ. Hệ thống phải dùng quy tắc có phiên bản và ngày hiệu lực. Khi tạo điểm phải lưu contribution gốc và phiên bản rule để giải thích được kết quả.

## 14. Thông báo nghiệp vụ

Hệ thống nên phát thông báo khi:

- Người dùng được gán vai trò cho work area.
- Volunteer được assign task hoặc có kết quả đăng ký.
- Assignment bị decline.
- Task bị revision hoặc được approved.
- Có yêu cầu check-in chờ duyệt (gửi Leader); check-in được duyệt hoặc bị từ chối (gửi volunteer).
- Attendance session mở quá lâu.
- Attendance correction được duyệt hoặc từ chối.
- Có purchase request chờ Leader duyệt; purchase request được duyệt hoặc bị từ chối.
- Task mua được giao hoặc sắp đến hạn.
- Purchase đã submit và chờ xác nhận giao.
- Có pledge mới; pledge được xác nhận nhận một phần, đủ, hoặc bị hủy.
- Donation thực tế được ghi nhận hoặc được điều chỉnh sau kiểm tra.
- Có loan offer mới; loan offer được chấp nhận hoặc từ chối.
- Borrowed item sắp đến hạn hoặc quá hạn.
- Có support request mới phù hợp với community, skill hoặc vật tư đang có; vật tư hỗ trợ được duyệt hoặc đã chuyển.
- Mandatory readiness item failed.
- Material report được volunteer gửi hoặc được Material Officer xác minh.
- Work area chuyển sang Ready.

Thông báo không được chứa thông tin nhạy cảm hoặc URL file riêng tư có hiệu lực lâu dài.

## 15. Dữ liệu và kiểm soát toàn vẹn

- Mọi record nghiệp vụ phải xác định được parish và season.
- Dùng server timestamp cho hành động có ảnh hưởng đến trạng thái hoặc thời lượng.
- Không xóa cứng record đã phát sinh giao dịch; dùng cancel, deactivate hoặc archive.
- Ảnh và hóa đơn phải liên kết đúng record, người upload và thời gian upload.
- Cập nhật stock, allocation, purchase confirmation, donation receipt, chuyển kho hỗ trợ, duyệt check-in, material report verification và recognition phải chạy trong transaction.
- Mỗi record nguồn (receipt, delivery, transfer, material report) chỉ được ghi vào sổ kho một lần.
- Phát hiện concurrent update để tránh ghi đè dữ liệu mới hơn.
- Log không chứa mật khẩu, token, URL file riêng tư hoặc dữ liệu cá nhân không cần thiết.
- Các danh mục đã được tham chiếu chỉ được deactivate.

## 16. Trường hợp biên và ngoại lệ cần kiểm thử

1. Kích hoạt season khi chưa có work area.
2. Work area thiếu leader hoặc material officer.
3. Task nằm ngoài khoảng thời gian season.
4. Hai volunteer cùng đăng ký vị trí cuối của task.
5. Leader assign volunteer thiếu skill.
6. Volunteer decline sau khi đã được giao.
7. Volunteer check-out khi chưa check-in.
8. Volunteer quên check-out và gửi correction.
9. Hai attendance session bị chồng thời gian.
10. Volunteer submit task nhưng thiếu evidence bắt buộc.
11. Leader yêu cầu revision nhiều lần.
12. Required quantity hoặc received quantity bị nhập âm.
13. Purchase request lớn hơn shortage.
14. Chi phí mua thực tế vượt budget đã duyệt.
15. Buyer upload file hóa đơn sai định dạng hoặc quá dung lượng.
16. Hàng đã mua nhưng giao thiếu hoặc không dùng được.
17. MATERIAL donation nhận 15 nhưng có 3 đơn vị unusable; chỉ 12 được cộng stock.
18. Volunteer báo remaining/damaged/lost vượt allocated quantity hoặc Material Officer verify số lượng không hợp lệ.
19. Hai officer đồng thời allocation phần stock cuối cùng.
20. Borrowed item quá hạn, damaged hoặc lost.
21. Mandatory readiness item failed ngay trước sự kiện.
22. Committee reopen work area đã Ready.
23. Thay đổi recognition rule giữa mùa.
24. Tài khoản bị khóa trong khi còn assignment mở.
25. User cố truy cập dữ liệu của parish hoặc work area ngoài phạm vi.
26. Export lớn và export có dữ liệu nhạy cảm.
27. Volunteer gửi yêu cầu check-in nhưng Leader duyệt trễ nhiều giờ.
28. Volunteer gửi hai yêu cầu check-in liên tiếp cho cùng assignment.
29. Leader từ chối check-in không nhập lý do.
30. Pledge được giao nhiều lần, có lần nhận vượt số lượng cam kết.
31. Hủy pledge đã nhận một phần.
32. Volunteer từ chối task mua; Leader giao lại cho người khác.
33. Tạo task mua khi purchase request chưa được duyệt.
34. Hỗ trợ vật tư vượt unallocated stock của work area nguồn hoặc hai request cùng lấy phần stock cuối.
35. Vật tư chuyển kho bị hỏng một phần khi đến nơi.
36. Volunteer nhập loan offer thay cho người cho mượn không có tài khoản.

## 17. Tiêu chí nghiệm thu nghiệp vụ cấp cao

Hệ thống đạt baseline nghiệp vụ khi chứng minh được các kịch bản sau:

1. Ban phụ trách tạo, cấu hình và kích hoạt một season hợp lệ.
2. Trưởng khu vực tạo task có skill và material requirement.
3. Volunteer có thể được assign hoặc đăng ký rồi được duyệt.
4. Volunteer gửi yêu cầu check-in, Leader duyệt từng người; volunteer cập nhật evidence, check-out và được nghiệm thu.
5. Hệ thống tính đúng shortage từ số lượng đã xác nhận.
6. Purchase request được Work Area Leader duyệt; chỉ request đã duyệt mới được tạo task mua và giao volunteer.
7. Buyer gửi thông tin mua và officer xác nhận delivery trước khi tăng stock.
8. Volunteer pledge được và Material Officer xác nhận nhận nhiều lần; donation trực tiếp được ghi nhận đúng theo MONEY/MATERIAL; chỉ usable material đã nhận làm tăng stock.
9. Allocation không thể vượt lượng còn khả dụng.
10. Volunteer nhập loan offer (kể cả thay người cho mượn); borrowed item được theo dõi đến khi trả và ghi tình trạng.
11. Community khác có thể hỗ trợ nhân lực/kỹ năng và chuyển vật tư qua support request; stock hai work area thay đổi đúng.
12. Sau khi task Completed, volunteer gửi material report và Material Officer verify; usable remainder quay lại đúng work-area stock.
13. Work area không thể Ready khi còn mandatory item failed.
14. Báo cáo tổng hợp đúng task, giờ làm, vật tư, chi phí và contribution.
15. Người dùng chỉ xem hoặc thay đổi dữ liệu đúng vai trò và phạm vi.
16. Các hành động phê duyệt, chỉnh sửa quan trọng và export có audit trail.

## 18. Phân rã module đề xuất

### A. Organization and Access Management

- Parish, Community.
- User, Role, Permission.
- Account lifecycle và authorization scope.

### B. Christmas Planning

- ChristmasSeason.
- WorkArea.
- Task, Skill, TaskSkill.
- TaskAssignment, task mua (`PURCHASE`) và task khắc phục (`CORRECTIVE`).
- SupportRequest, SupportRequestPosition, SupportPositionSkill, SupportRequestMaterial, SupportResponse, SupportResponseMember và SupportResponseMaterial.

### C. Volunteer Execution

- VolunteerProfile, VolunteerSkill và VolunteerAvailability.
- AttendanceRequest, Attendance và AttendanceCorrection.
- TaskProgress, ProgressEvidence và review.

### D. Resource Management

- Material và MaterialRequirement.
- PurchaseRequest và Purchase (người mua qua task mua).
- Donor, Donation, DonationItem, DonationReceipt và DonationReceiptItem.
- BorrowedItem và MaterialAllocation.
- TaskMaterialReport và TaskMaterialReportItem.

### E. Progress and Readiness

- Progress aggregation.
- ReadinessChecklist và ReadinessItem (không dùng template).
- Corrective task.

### F. Reporting and Recognition

- Dashboard và Report.
- Contribution, RecognitionRule và RecognitionEntry.
- Notification, NotificationPreference, ActivityLog và FileObject.

## 19. Quyết định đã chốt và vấn đề còn mở

### 19.1. Quyết định đã chốt (07/10/2026)

| ID | Vấn đề | Quyết định |
|---|---|---|
| D-01 | Purchase approval và người mua | Work Area Leader duyệt mọi purchase request của work area, sau đó tạo task mua và giao volunteer đi mua |
| D-02 | Quyên góp | Có pledge qua app; Material Officer xác nhận từng lần nhận thực tế; vẫn cho ghi nhận trực tiếp |
| D-03 | Hỗ trợ liên cộng đoàn | Support request gồm nhân lực/kỹ năng và/hoặc vật tư (chuyển kho giữa work area) |
| D-04 | Readiness checklist | Không dùng mẫu; Leader tạo checklist và mục kiểm tra cho từng work area |
| D-05 | Người cho mượn không có tài khoản | Volunteer nhập thông tin người cho mượn giùm và là đầu mối liên hệ |
| D-06 | Check-in | Leader duyệt từng yêu cầu check-in của từng người; check-out không cần duyệt |
| R-06 (14/10/2026) | Trạng thái season | Bỏ `Draft`; season tạo ở `Planning` |
| R-09 (08/10/2026) | Season trùng lặp | Tên và năm season đều duy nhất trong giáo xứ |

### 19.2. Vấn đề còn cần stakeholder xác nhận

| Vấn đề | Baseline đang dùng | Cần xác nhận |
|---|---|---|
| Xác nhận delivered material | Material Officer xác nhận usable quantity cho work area | Có yêu cầu hai người xác nhận cho hàng giá trị cao hay không |
| Check-in location | Thời gian bắt buộc, vị trí hoặc ảnh là cấu hình | Có dùng GPS hoặc geofence không |
| Monetary donation | Ghi actual money received thủ công; không có online payment | Quyền nhập/chỉnh sửa và chứng từ nội bộ cần lưu |
| Recognition | Điểm phi tài chính theo rule có phiên bản | Điểm, giới hạn, thời gian hiệu lực và xử lý khiếu nại |
| Media | File riêng tư liên kết record | Dung lượng, thời gian lưu và định dạng |
| Reusable material | Có carry-over bằng opening balance | Ai kiểm kê và xác nhận đầu mùa mới |
| Support request | Leader work area yêu cầu xác nhận response; MO work area nguồn đồng ý xuất vật tư | Trường hợp override skill |
| Check-out | Không cần Leader duyệt | Có cần bật duyệt check-out không (dữ liệu đã hỗ trợ `request_type = CHECK_OUT`) |

## 20. Kết luận

GrottoWorks không chỉ là hệ thống CRUD quản lý danh sách công việc. Giá trị nghiệp vụ chính nằm ở sáu chuỗi liên kết:

1. **Season -> Work Area -> Task** để quản lý kế hoạch.
2. **Task -> Skill -> Volunteer -> Assignment** để quản lý người thực hiện.
3. **Requirement -> Purchase (Leader duyệt, task mua)/Pledge & Donation/Borrowing/Support Transfer -> Confirmation -> Allocation** để quản lý nguồn lực theo từng work area.
4. **Task -> Allocation -> Material Report -> Officer Verification -> Stock Reconciliation** để kiểm soát vật tư sau khi công việc hoàn thành.
5. **Check-in Request -> Leader Approval -> Attendance -> Progress Evidence -> Leader Review** để chứng minh công việc thực tế.
6. **Readiness Checklist -> Corrective Task -> Recheck -> Ready** để đảm bảo kết quả cuối cùng.

Nếu thiết kế database, API, UI và test case luôn bám vào các chuỗi nghiệp vụ này, các module của hệ thống sẽ liên kết nhất quán và tránh trở thành những chức năng CRUD rời rạc.
