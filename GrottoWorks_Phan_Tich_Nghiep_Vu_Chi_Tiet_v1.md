# Phân tích nghiệp vụ chi tiết hệ thống GrottoWorks

**Phiên bản:** 1.0

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
7. Hỗ trợ nhân lực và kỹ năng giữa các cộng đoàn.
8. Lập kế hoạch và theo dõi vật tư.
9. Yêu cầu mua vật tư, phê duyệt, giao người mua và xác nhận giao hàng.
10. Ghi nhận và tiếp nhận quyên góp thực tế.
11. Quản lý vật tư hoặc dụng cụ cho mượn.
12. Phân bổ vật tư cho khu vực hoặc công việc.
13. Check-in, check-out và điều chỉnh giờ làm.
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
- Phê duyệt yêu cầu mua lớn, vượt ngân sách hoặc thuộc trường hợp đặc biệt.
- Điều phối hỗ trợ nhân lực và kỹ năng giữa các cộng đoàn.
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
- Xét duyệt yêu cầu điều chỉnh giờ làm.
- Thực hiện readiness checklist và tạo việc khắc phục.

### 4.5. Cán bộ vật tư và đóng góp

Cán bộ vật tư chịu trách nhiệm:

- Quản lý kho vật tư logic của work area được phân công; kho của mỗi work area được theo dõi tách biệt.
- Lập danh sách vật tư cho work area hoặc task.
- Theo dõi số lượng cần, hiện có, đã xác nhận nhận, đã phân bổ và còn thiếu.
- Tạo yêu cầu mua cho phần thiếu hụt.
- Sau khi purchase request đủ điều kiện mua hoặc được phê duyệt nếu thuộc trường hợp cần phê duyệt, chọn và giao volunteer đi mua.
- Xác nhận số lượng, chi phí và tình trạng vật tư khi được giao.
- Ghi nhận, kiểm tra và xác nhận donation thực tế đã tiếp nhận.
- Theo dõi vật tư hoặc dụng cụ mượn và hoàn trả.
- Phân bổ vật tư đã xác nhận cho work area hoặc task.
- Xác minh báo cáo vật tư sau khi task đã được Leader xác nhận hoàn thành.

### 4.6. Tình nguyện viên và người đóng góp

Người dùng mobile app có thể:

- Khai báo hồ sơ, cộng đoàn, kỹ năng và thời gian có thể tham gia.
- Xem task phù hợp hoặc yêu cầu hỗ trợ.
- Chấp nhận hoặc từ chối task được phân công.
- Đăng ký tham gia task và chờ trưởng khu vực duyệt.
- Check-in, check-out và yêu cầu sửa giờ nếu có sai sót.
- Cập nhật tiến độ bằng phần trăm, ghi chú và hình ảnh.
- Nhận nhiệm vụ đi mua, nhập thông tin và tải hóa đơn.
- Có thể hỗ trợ task của cộng đoàn khác thông qua quy trình hỗ trợ nhân lực/kỹ năng.
- Xem giờ làm, task hoàn thành, đóng góp và điểm ghi nhận của bản thân.

### 4.7. Quản trị viên hệ thống

Quản trị viên quản lý cấu hình kỹ thuật và dữ liệu nền, không trực tiếp điều hành công việc Giáng sinh:

- Tài khoản, khóa hoặc mở khóa tài khoản.
- Vai trò, quyền và phạm vi dữ liệu.
- Giáo xứ, cộng đoàn và các danh mục.
- Danh mục kỹ năng, loại vật tư, loại khu vực và loại đóng góp.
- Mẫu readiness checklist.
- Quy tắc tính điểm ghi nhận.
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
        |     |     |-- Attendance
        |     |     |-- Progress Evidence
        |     |     `-- Task Material Reports
        |     |-- Material Requirements
        |     |-- Material Allocations
        |     `-- Readiness Checklists
        |-- Purchases
        |-- Donations
        |-- Borrowed Items
        |-- Staffing Support Requests
        `-- Reports and Recognition
```

Mùa Giáng sinh là aggregate hoặc business backbone chính. Mọi dữ liệu vận hành phải xác định được mình thuộc giáo xứ và mùa nào để không trộn dữ liệu giữa các năm.

## 6. Đối tượng nghiệp vụ

| Đối tượng | Ý nghĩa | Quan hệ chính |
|---|---|---|
| Parish | Giáo xứ sử dụng hệ thống | Có nhiều community, user và season |
| Community | Cộng đoàn thuộc giáo xứ | Có tình nguyện viên và có thể phụ trách nhiều work area |
| ChristmasSeason | Mùa chuẩn bị theo năm | Chứa toàn bộ dữ liệu vận hành của một mùa |
| WorkArea | Khu vực hoặc công trình | Thuộc season, có leader, officer và nhiều task |
| Task | Đơn vị công việc có thể giao và nghiệm thu | Thuộc work area, yêu cầu kỹ năng, người và vật tư |
| Skill | Kỹ năng chuẩn hóa | Quan hệ nhiều-nhiều với task và volunteer |
| VolunteerProfile | Hồ sơ tham gia của người dùng | Có community, skill, availability và contribution |
| TaskAssignment | Liên kết volunteer với task | Ghi nguồn phân công hoặc đăng ký và trạng thái phản hồi |
| Attendance | Phiên check-in/check-out | Thuộc một assignment đã chấp nhận |
| TaskProgress | Lần cập nhật tiến độ | Có phần trăm, ghi chú, ảnh, người gửi và thời gian |
| Material | Danh mục vật tư hoặc dụng cụ | Có đơn vị tính và thuộc tính tái sử dụng |
| MaterialRequirement | Nhu cầu vật tư cấp work area | Thuộc work area, dùng để lập kế hoạch và tính shortage |
| TaskMaterial | Nhu cầu vật tư cấp task | Gắn material và required quantity với một task |
| PurchaseRequest | Yêu cầu mua phần vật tư thiếu | Có số lượng, dự toán, người duyệt và trạng thái |
| Purchase | Kết quả mua thực tế | Có người mua, số lượng, giá, nhà cung cấp và hóa đơn |
| Donation | Một lần đóng góp thực tế đã tiếp nhận | MONEY lưu số tiền trực tiếp; MATERIAL có thể chứa nhiều DonationItem |
| DonationItem | Một vật tư trong MATERIAL donation | Lưu received quantity, unusable quantity và optional pledged quantity chỉ để tham chiếu |
| BorrowedItem | Vật tư hoặc dụng cụ mượn | Có chủ sở hữu, ngày trả dự kiến và tình trạng khi trả |
| MaterialAllocation | Phân bổ vật tư đã xác nhận | Gắn số lượng thực cấp với work area hoặc task; allocated quantity là dữ liệu lịch sử không bị ghi đè bởi báo cáo sau task |
| TaskMaterialReport | Báo cáo vật tư sau khi task hoàn thành | Volunteer báo cáo vật tư đã được cấp; Material Officer xác minh |
| TaskMaterialReportItem | Chi tiết báo cáo theo allocation item | Lưu remaining, damaged và lost ở mức reported và verified |
| SupportRequest | Yêu cầu hỗ trợ nhân lực liên cộng đoàn | Chứa các vị trí cần người, số lượng, kỹ năng yêu cầu, hạn và trạng thái |
| ReadinessChecklist | Lần kiểm tra sẵn sàng | Gồm nhiều item bắt buộc hoặc tùy chọn |
| Contribution | Sự kiện đóng góp đã được công nhận | Là cơ sở tính báo cáo và điểm |
| RecognitionEntry | Điểm phát sinh từ contribution | Lưu quy tắc và phiên bản đã áp dụng |
| Notification | Thông báo nghiệp vụ | Gắn người nhận, sự kiện và trạng thái gửi |
| ActivityLog | Lịch sử hành động quan trọng | Lưu actor, thời gian, đối tượng và nội dung thay đổi |

## 7. Quy trình nghiệp vụ tổng thể

### 7.1. Giai đoạn lập kế hoạch

1. Ban phụ trách tạo mùa Giáng sinh ở trạng thái `Draft`.
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
3. Khi thiếu người hoặc kỹ năng, trưởng khu vực tạo support request toàn giáo xứ.
4. Cán bộ vật tư xử lý thiếu hụt bằng một hoặc nhiều nguồn:
   - Vật tư hiện có trong kho logic của work area.
   - Mua mới.
   - Donation thực tế đã tiếp nhận.
   - Mượn.
5. Chỉ vật tư thực tế đã nhận, còn dùng được và đã được Material Officer xác nhận mới được tính là khả dụng cho work area.

### 7.3. Giai đoạn thực hiện

1. Tình nguyện viên chấp nhận assignment hoặc được duyệt đăng ký.
2. Tình nguyện viên check-in khi bắt đầu tại địa điểm làm việc.
3. Tình nguyện viên thực hiện task và có thể gửi nhiều lần cập nhật tiến độ.
4. Tình nguyện viên check-out khi kết thúc phiên làm.
5. Khi cho rằng task đã hoàn thành, tình nguyện viên gửi bằng chứng và chuyển task sang `Submitted`.
6. Trưởng khu vực nghiệm thu:
   - Đạt: chuyển `Completed`.
   - Chưa đạt: chuyển `Revision Required`, ghi lý do và yêu cầu sửa.
7. Sau khi task ở trạng thái `Completed`, volunteer phụ trách gửi material report nếu task có vật tư đã được phân bổ.
8. Material Officer của work area kiểm tra thực tế và xác minh material report trước khi hệ thống cập nhật số liệu vật tư chính thức.

### 7.4. Giai đoạn readiness và tổng kết

1. Trưởng khu vực mở checklist áp dụng cho loại khu vực.
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

1. Nhập tên mùa, ngày bắt đầu, ngày kết thúc, mô tả và ngân sách dự kiến.
2. Hệ thống kiểm tra tên trùng và thời gian không hợp lệ.
3. Lưu mùa ở trạng thái `Draft`.
4. Tạo các work area và chọn community phụ trách nếu có.
5. Chỉ định leader và material officer.
6. Cấu hình checklist và các quy tắc áp dụng.
7. Chuyển mùa sang `Planning`, sau đó `Active`.
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
3. Hệ thống lưu thời gian máy chủ và tạo attendance session đang mở.
4. Volunteer chọn check-out khi kết thúc.
5. Hệ thống lưu thời gian và tính số phút làm việc.

```text
Working Duration = Check-out Time - Check-in Time
```

**Sửa giờ:**

1. Volunteer nhập thời gian đề nghị và lý do.
2. Yêu cầu ở trạng thái `Pending`.
3. Leader kiểm tra và `Approved` hoặc `Rejected`.
4. Chỉ thời gian đã duyệt mới đi vào báo cáo và tính điểm.

**Quy tắc:**

- Không check-out nếu chưa check-in.
- Một assignment chỉ có tối đa một phiên đang mở.
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
2. Material Officer tạo purchase request gồm một hoặc nhiều vật tư, số lượng, dự toán và lý do.
3. Nếu request thuộc trường hợp cần phê duyệt, ví dụ mua lớn hoặc vượt ngưỡng/budget được cấu hình, Parish Committee review và chọn `Approved` hoặc `Rejected`.
4. Request không thuộc trường hợp cần phê duyệt có thể tiếp tục theo chính sách cấu hình của giáo xứ.
5. Khi request đủ điều kiện mua, Material Officer chọn và giao một volunteer đi mua.
6. Volunteer chấp nhận nhiệm vụ mua.
7. Volunteer nhập số lượng thực tế, giá, nhà cung cấp, ngày mua và ảnh hóa đơn.
8. Trạng thái mua thành `Purchased`, nhưng stock chưa tăng.
9. Khi hàng được giao, Material Officer kiểm tra số lượng và tình trạng.
10. Material Officer xác nhận số lượng usable thực nhận.
11. Hệ thống ghi material transaction, cập nhật stock của đúng work area và tính lại shortage.

**Ngoại lệ:**

- Không tạo request khi shortage bằng 0, trừ khi có quyền override và lý do.
- Request thuộc trường hợp bắt buộc approval không được assign buyer trước khi được duyệt.
- Chi phí thực tế vượt hạn mức hoặc budget đã được duyệt phải quay lại bước phê duyệt hoặc xử lý ngoại lệ.
- Hóa đơn không hợp lệ không làm mất record mua; record được chuyển sang trạng thái cần bổ sung.
- Giao thiếu được xác nhận theo số lượng thực nhận và request còn phần chưa hoàn thành.

### 8.7. Ghi nhận donation thực tế

Donation trong baseline v1 là **bản ghi một lần tiếp nhận thực tế**, do Material Officer hoặc Admin có quyền nhập. Donation không dùng header như một workflow pledge/draft/approval.

Một donation có một trong hai loại:

- `MONEY`: lưu trực tiếp số tiền thực tế đã nhận.
- `MATERIAL`: có một hoặc nhiều `DonationItem`.

Ví dụ:

```text
MATERIAL Donation
  |-- 20 m dây điện: received 20, unusable 2
  |-- 10 bóng đèn: received 10, unusable 1
  `-- 5 hộp sơn: received 5, unusable 0
```

Quy trình:

1. Officer/Admin chọn hoặc tạo donor.
2. Chọn loại donation là `MONEY` hoặc `MATERIAL`.
3. Với `MONEY`, nhập số tiền thực tế đã nhận và ngày nhận.
4. Với `MATERIAL`, nhập từng material, received quantity và unusable quantity sau khi kiểm tra thực tế.
5. `Usable Quantity = Received Quantity - Unusable Quantity`.
6. Chỉ usable quantity của MATERIAL donation được cộng vào stock của work area liên quan thông qua material transaction phù hợp.
7. Hệ thống tạo hoặc cập nhật contribution summary để phục vụ báo cáo và recognition nếu áp dụng.
8. Nếu cùng donor giao thêm ở thời điểm khác, tạo một donation record mới cho lần tiếp nhận mới.

`pledged_quantity` nếu có chỉ là thông tin tham chiếu lịch sử/ngoài hệ thống, không tạo pledge workflow và không được tính vào stock.

### 8.8. Quản lý vật tư hoặc dụng cụ mượn

1. Officer ghi tên item, chủ sở hữu, thông tin liên hệ, số lượng và tình trạng ban đầu.
2. Ghi ngày mượn và ngày trả dự kiến.
3. Item có thể được phân bổ cho work area hoặc task.
4. Hệ thống nhắc trước và sau ngày trả dự kiến.
5. Khi trả, officer ghi ngày trả và tình trạng:
   - Good.
   - Damaged.
   - Lost.
6. Nếu damaged hoặc lost, hệ thống ghi nhận sự cố và người có trách nhiệm xử lý.

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

### 8.11. Hỗ trợ nhân lực giữa các cộng đoàn

Support request trong baseline v1 chỉ dùng cho **nhân lực và kỹ năng**, không dùng như workflow chung cho material, donation, borrowing hoặc transport.

Một request có thể gồm nhiều vị trí (`position`). Mỗi position xác định:

- Tên vị trí hoặc loại công việc.
- Số volunteer cần.
- Một hoặc nhiều skill bắt buộc cho mỗi người nếu có.
- Deadline và ghi chú.

Quy trình:

1. Work Area Leader tạo support request khi work area thiếu người hoặc kỹ năng.
2. Request được công bố trong phạm vi giáo xứ.
3. Volunteer hoặc community khác gửi response và danh sách người có thể hỗ trợ.
4. Hệ thống kiểm tra kỹ năng, capacity và tránh gán trùng một người vào nhiều position đang active trong cùng request.
5. Người có quyền xác nhận response phù hợp.
6. Volunteer được xác nhận tham gia task/work area tương ứng thông qua assignment.
7. Khi đủ số người yêu cầu, request thành `Fulfilled`; nếu chỉ đáp ứng một phần thì `Partially Fulfilled`.

Thiếu vật tư được xử lý bằng các workflow riêng: purchase, donation, borrowing và material allocation.

### 8.12. Readiness và an toàn

Checklist được cấu hình theo loại work area. Ví dụ với hang đá:

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
  -> Create Corrective Task
  -> Assign Responsible Person
  -> Fix and Upload Evidence
  -> Recheck
  -> Passed
```

## 9. Vòng đời và trạng thái

### 9.1. Christmas Season

```text
Draft -> Planning -> Active -> Ready -> Completed -> Archived
```

| Trạng thái | Ý nghĩa |
|---|---|
| Draft | Đang nhập thông tin ban đầu |
| Planning | Đang tạo area, task và nguồn lực |
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

### 9.4. Task Assignment

```text
Pending -> Accepted -> In Progress -> Completed
       `-> Declined
       `-> Cancelled
```

### 9.5. Purchase Request / Purchase

Purchase request có thể đi qua approval hoặc không tùy policy:

```text
Draft -> Submitted
          |-> Approval Required -> Pending Approval -> Approved
          |                                      `-> Rejected
          `-> No Approval Required
                    |
                    v
              Ready for Purchase
                    -> Assigned Buyer
                    -> Purchased
                    -> Delivered
                    -> Confirmed
```

`Purchased` chưa làm tăng stock. Chỉ sau khi Material Officer xác nhận delivery/usable quantity thì số lượng mới được ghi nhận chính thức.

### 9.6. Donation

Donation trong baseline v1 không có lifecycle pledge/status. Mỗi record đại diện cho một lần tiếp nhận thực tế.

```text
Actual Receipt
  -> Officer/Admin Records Donation
  -> Inspect Quantity / Condition
  -> Post Usable Quantity
  -> Contribution Summary
```

Trạng thái của MATERIAL donation item được suy ra từ `received_quantity` và `unusable_quantity`, không lưu như một workflow độc lập.

### 9.7. Borrowed Item

```text
Offered -> Accepted -> Issued -> In Use -> Return Due -> Returned
                                             `-> Overdue
Returned -> Good / Damaged / Lost
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
- **BR-02:** Tên season phải duy nhất trong phạm vi giáo xứ.
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

- **BR-18:** Check-out phải có check-in mở tương ứng.
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
- **BR-28:** Request thuộc trường hợp bắt buộc phê duyệt không được assign buyer trước khi Parish Committee approve.
- **BR-29:** Parish Committee chỉ review/approve các purchase request thuộc trường hợp cần phê duyệt theo policy, ví dụ mua lớn hoặc vượt ngưỡng/budget.
- **BR-30:** Material Officer quyết định volunteer nào được giao đi mua sau khi request đủ điều kiện mua.
- **BR-31:** Từ chối phê duyệt phải có lý do.
- **BR-32:** Chi phí thực tế vượt hạn mức đã được duyệt phải có quyết định bổ sung hoặc xử lý ngoại lệ.
- **BR-33:** Tổng allocation không vượt confirmed unallocated quantity.
- **BR-34:** `allocated_quantity` là lịch sử số lượng thực tế đã cấp và không được ghi đè bởi kết quả sử dụng sau task.

### 10.5. Donation và borrowing

- **BR-35:** Donation header ghi nhận một lần tiếp nhận thực tế; không dùng làm pledge/draft/approval workflow.
- **BR-36:** MONEY donation lưu amount trực tiếp và không có DonationItem.
- **BR-37:** MATERIAL donation phải có ít nhất một DonationItem với received quantity hợp lệ.
- **BR-38:** Chỉ `received_quantity - unusable_quantity` của MATERIAL donation được cộng stock.
- **BR-39:** `pledged_quantity` nếu có chỉ là optional historical context, không được tính vào stock và không tạo pledge workflow.
- **BR-40:** Contribution bằng tiền ở phiên bản 1.0 không phải giao dịch thanh toán online.
- **BR-41:** Borrowed item bắt buộc có owner và expected return date theo policy dữ liệu áp dụng.
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

## 11. Phân quyền nghiệp vụ

Ký hiệu: `M` quản lý, `A` phê duyệt, `S` gửi dữ liệu, `V` xem, `-` không có quyền. Dấu `*` nghĩa là chỉ trong phạm vi được giao.

| Chức năng | Committee | Leader | Material Officer | Volunteer | Admin |
|---|:---:|:---:|:---:|:---:|:---:|
| Season | M | V | V | V | A |
| Community và Work Area | M | M* | V* | V | A |
| Task | V | M* | V* | S/V* | A |
| Assignment | V | M/A* | V* | S/V bản thân | A |
| Attendance | V | A* | V* | M bản thân | A |
| Progress | V | A* | V* | S bản thân | A |
| Material Requirement | V | V* | M* | V | A |
| Purchase Request | A khi cần approval | V* | M/S* | V bản thân | A |
| Donation | V | V* | M* | V nếu liên kết tài khoản | A/M |
| Borrowed Item | V | V* | M/A* | S/V bản thân | A |
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
- Purchase request chờ duyệt, đang mua và chờ xác nhận giao.
- Donation thực tế đã nhận: money, material received, unusable và usable quantity.
- Support request đang mở.
- Số volunteer, giờ làm và tỷ lệ tham gia.
- Readiness theo work area và số mandatory item failed.

### 12.2. Báo cáo cuối mùa

- Danh sách và trạng thái work area.
- Task theo trạng thái, deadline và người phụ trách.
- Vật tư cần, đã có, đã mua, được tặng, được mượn, đã phân bổ và còn dư.
- Chi phí dự toán, được duyệt và thực tế.
- Donation theo donor, loại donation, số tiền hoặc material received/usable.
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
| Hoàn thành purchase assignment | 10 |
| Hỗ trợ khẩn cấp | 20 |
| Hỗ trợ community khác | 25 |

Các con số trên chỉ là ví dụ. Hệ thống phải dùng quy tắc có phiên bản và ngày hiệu lực. Khi tạo điểm phải lưu contribution gốc và phiên bản rule để giải thích được kết quả.

## 14. Thông báo nghiệp vụ

Hệ thống nên phát thông báo khi:

- Người dùng được gán vai trò cho work area.
- Volunteer được assign task hoặc có kết quả đăng ký.
- Assignment bị decline.
- Task bị revision hoặc được approved.
- Attendance session mở quá lâu.
- Attendance correction được duyệt hoặc từ chối.
- Có purchase request chờ duyệt.
- Purchase assignment được giao hoặc sắp đến hạn.
- Purchase đã submit và chờ xác nhận giao.
- Donation thực tế được ghi nhận hoặc được điều chỉnh sau kiểm tra.
- Borrowed item sắp đến hạn hoặc quá hạn.
- Có support request mới phù hợp với community hoặc skill.
- Mandatory readiness item failed.
- Material report được volunteer gửi hoặc được Material Officer xác minh.
- Work area chuyển sang Ready.

Thông báo không được chứa thông tin nhạy cảm hoặc URL file riêng tư có hiệu lực lâu dài.

## 15. Dữ liệu và kiểm soát toàn vẹn

- Mọi record nghiệp vụ phải xác định được parish và season.
- Dùng server timestamp cho hành động có ảnh hưởng đến trạng thái hoặc thời lượng.
- Không xóa cứng record đã phát sinh giao dịch; dùng cancel, deactivate hoặc archive.
- Ảnh và hóa đơn phải liên kết đúng record, người upload và thời gian upload.
- Cập nhật stock, allocation, purchase confirmation, donation receipt reconciliation, material report verification và recognition phải chạy trong transaction.
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

## 17. Tiêu chí nghiệm thu nghiệp vụ cấp cao

Hệ thống đạt baseline nghiệp vụ khi chứng minh được các kịch bản sau:

1. Ban phụ trách tạo, cấu hình và kích hoạt một season hợp lệ.
2. Trưởng khu vực tạo task có skill và material requirement.
3. Volunteer có thể được assign hoặc đăng ký rồi được duyệt.
4. Volunteer check-in, cập nhật evidence, check-out và được nghiệm thu.
5. Hệ thống tính đúng shortage từ số lượng đã xác nhận.
6. Purchase request cần approval được Committee xử lý đúng policy; request đủ điều kiện mới được Material Officer assign buyer.
7. Buyer gửi thông tin mua và officer xác nhận delivery trước khi tăng stock.
8. Donation thực tế được ghi nhận đúng theo MONEY/MATERIAL; chỉ usable material received làm tăng stock.
9. Allocation không thể vượt lượng còn khả dụng.
10. Borrowed item được theo dõi đến khi trả và ghi tình trạng.
11. Community khác có thể hỗ trợ nhân lực/kỹ năng qua staffing support request.
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
- TaskAssignment.
- Staffing SupportRequest, SupportRequestPosition và SupportPositionSkill.

### C. Volunteer Execution

- VolunteerProfile và VolunteerSkill.
- Attendance và AttendanceCorrection.
- TaskProgress, ProgressEvidence và review.

### D. Resource Management

- Material và MaterialRequirement.
- PurchaseRequest, PurchaseAssignment và Purchase.
- Donation và DonationItem.
- BorrowedItem và MaterialAllocation.
- TaskMaterialReport và TaskMaterialReportItem.

### E. Progress and Readiness

- Progress aggregation.
- ReadinessTemplate, ReadinessChecklist và ReadinessItem.
- Corrective task.

### F. Reporting and Recognition

- Dashboard và Report.
- Contribution, RecognitionRule và RecognitionEntry.
- Notification và ActivityLog.

## 19. Các quyết định cần stakeholder xác nhận

| Vấn đề | Baseline đang dùng | Cần xác nhận |
|---|---|---|
| Purchase approval policy | Committee chỉ duyệt request thuộc trường hợp cần approval; Material Officer assign buyer sau khi request đủ điều kiện | Ngưỡng tiền/budget nào bắt buộc Committee approval |
| Xác nhận delivered material | Material Officer xác nhận usable quantity cho work area | Có yêu cầu hai người xác nhận cho hàng giá trị cao hay không |
| Check-in location | Thời gian bắt buộc, vị trí hoặc ảnh là cấu hình | Có dùng GPS hoặc geofence không |
| Monetary donation | Ghi actual money received thủ công; không có online payment | Quyền nhập/chỉnh sửa và chứng từ nội bộ cần lưu |
| Recognition | Điểm phi tài chính theo rule có phiên bản | Điểm, giới hạn, thời gian hiệu lực và xử lý khiếu nại |
| Media | File riêng tư liên kết record | Dung lượng, thời gian lưu và định dạng |
| Reusable material | Có carry-over bằng opening balance | Ai kiểm kê và xác nhận đầu mùa mới |
| Staffing support | Support request chỉ cho người/kỹ năng | Ai được quyền xác nhận community response và trường hợp override skill |

## 20. Kết luận

GrottoWorks không chỉ là hệ thống CRUD quản lý danh sách công việc. Giá trị nghiệp vụ chính nằm ở sáu chuỗi liên kết:

1. **Season -> Work Area -> Task** để quản lý kế hoạch.
2. **Task -> Skill -> Volunteer -> Assignment** để quản lý người thực hiện.
3. **Requirement -> Purchase/Donation/Borrowing -> Confirmation -> Allocation** để quản lý nguồn lực theo từng work area.
4. **Task -> Allocation -> Material Report -> Officer Verification -> Stock Reconciliation** để kiểm soát vật tư sau khi công việc hoàn thành.
5. **Attendance -> Progress Evidence -> Leader Review** để chứng minh công việc thực tế.
6. **Readiness Checklist -> Corrective Task -> Recheck -> Ready** để đảm bảo kết quả cuối cùng.

Nếu thiết kế database, API, UI và test case luôn bám vào các chuỗi nghiệp vụ này, các module của hệ thống sẽ liên kết nhất quán và tránh trở thành những chức năng CRUD rời rạc.
