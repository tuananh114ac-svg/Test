# Phân tích các vi phạm SOLID trong ClubManager (God Class)

Dưới đây là 4 vi phạm nguyên tắc SOLID được tìm thấy trong class `ClubManager`:

### 1. Vi phạm SRP (Single Responsibility Principle - Nguyên tắc đơn trách nhiệm)
- **Vị trí:** Toàn bộ class `ClubManager` và cụ thể tại các method `CreateClub`, `AddMemberToClub`, `CreateEvent`.
- **Phân tích:** Class này đang ôm đồm quá nhiều trách nhiệm: quản lý logic nghiệp vụ (thêm CLB, thêm thành viên, tạo sự kiện), ghi log (viết vào file), gửi thông báo (email, slack) và cả xuất báo cáo (`PrintClubReport`).
- **Hậu quả:** Class phình to (God class), khó bảo trì, khó viết Unit Test. Bất kỳ thay đổi nào về cách ghi log hay cách gửi email cũng sẽ buộc class quản lý nghiệp vụ CLB phải thay đổi theo.

### 2. Vi phạm OCP (Open/Closed Principle - Nguyên tắc Đóng/Mở)
- **Vị trí:** Method `CreateClub` (chuỗi `if/else` kiểm tra đuôi email) và `PrintClubReport`.
- **Phân tích:** 
  - Trong `CreateClub`, logic gửi thông báo đang bị hardcode với `if (presidentEmail.Contains("@gmail.com"))`. Nếu muốn hỗ trợ thêm nhà mạng khác hoặc kênh khác như SMS/Zalo, ta bắt buộc phải mở file này ra và sửa/thêm code. 
  - Trong `PrintClubReport`, báo cáo đang bị fix cứng việc in ra `Console`. Muốn xuất ra file CSV hay PDF thì lại phải nhảy vào sửa method này.
- **Hậu quả:** Dễ sinh ra lỗi (regression bug) khi tái cấu trúc hoặc thêm tính năng mới vì phải can thiệp trực tiếp vào code cũ đang chạy ổn định.

### 3. Vi phạm DIP (Dependency Inversion Principle - Nguyên tắc đảo ngược phụ thuộc)
- **Vị trí:** Lệnh `File.AppendAllText(_logFilePath, ...)` nằm rải rác ở hầu hết các method.
- **Phân tích:** `ClubManager` (class module cấp cao chứa business logic) đang phụ thuộc trực tiếp vào `File` (module cấp thấp xử lý I/O của hệ thống). Nó đáng lẽ phải phụ thuộc vào một Abstraction (ví dụ: `IAppLogger`).
- **Hậu quả:** Gắn chặt (tight-coupling) logic nghiệp vụ với cơ sở hạ tầng. Cực kỳ khó để mock dữ liệu khi viết Unit Test (mỗi lần test đều sinh ra file text thật trên ổ cứng), và không thể chuyển đổi linh hoạt sang ghi log trên Cloud hoặc Database nếu cần.

### 4. Vi phạm SRP / Khía cạnh Coupling (Gắn chặt Logic Thông báo vào Data)
- **Vị trí:** Method `CreateEvent`.
- **Phân tích:** Vừa chịu trách nhiệm khởi tạo object `Event`, method này vừa lặp qua danh sách `_members` để tự thực hiện hành vi gửi thông báo (`Console.WriteLine($"Email to...")`).
- **Hậu quả:** Nếu có 10.000 thành viên, việc tạo sự kiện sẽ bị block (treo) cho đến khi gửi xong 10.000 email. Logic thông báo không được tách rời khiến luồng chạy của ứng dụng thiếu tính bất đồng bộ và khó mở rộng.