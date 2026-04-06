THUẬT NGỮ: GIẢI THÍCH

admin: Người quản trị hệ thống, có quyền cao nhất để quản lý toàn bộ hệ thống và các `seller`.

seller: Chủ nhà trọ — người tạo và quản lý `buyer`, `formula`, và `container` trong hệ thống.

buyer: Người thuê trọ (tenant) — đơn vị tính bill do `seller` tạo. Mỗi buyer có chỉ số `baseline` (điện, nước, số người) và được gán 1 `formula`. Buyer nhận link `container` để submit chỉ số, và nhận `bill` sau khi seller duyệt. Seller có thể đặt tên buyer tuỳ ý (ví dụ "Phòng A").

formula: Biểu thức do `seller` viết để tính `bill`. Có thể chứa `{{variable}}` (ví dụ: `{{Số điện đã sử dụng}} * 2500 + {{Số nước đã sử dụng}} * 14000`). Mỗi buyer được gán 1 formula duy nhất.

variable: Giá trị tham chiếu trong biểu thức `formula`, viết dưới dạng `{{tên biến}}` (ví dụ: `{{Số điện đã sử dụng}}`). Được giải quyết tại thời điểm xuất bill: giá trị = current − baseline.

container: Phiên thu thập chỉ số do `seller` tạo cho 1 `billing cycle`. Seller gửi link container đến các buyer để họ submit chỉ số. Sau khi seller duyệt và xuất bill, container đóng lại.

submission: Bản ghi chỉ số (điện, nước) do `buyer` gửi vào `container` trong 1 billing cycle. Mỗi buyer có tối đa 1 submission per container. Sau khi `approve`, giá trị trở thành current value và sẽ replace `baseline` cho kỳ kế tiếp.

baseline: Chỉ số khởi điểm (điện, nước, số người) của `buyer` tại một tháng cụ thể. Được khởi tạo khi seller tạo buyer, và tự động cập nhật thành current value sau mỗi billing cycle.

approve: Hành động của `seller` khi chấp nhận một `submission`. Giá trị được approve trở thành current value và được dùng để tính bill (consumption = current − baseline).

discard: Hành động của `seller` khi từ chối một `submission` (không dùng để tính bill).

bill: Hóa đơn của một `buyer` cho một `billing cycle` (tháng). Mỗi `buyer` chỉ có tối đa một `bill` cho mỗi kỳ.

billing cycle: Kỳ tính tiền (ví dụ: tháng dương lịch) dùng để gom `submission` và xuất `bill`.

export bill: Hành động xuất `bill` ra định dạng tĩnh (ảnh, PDF) để gửi cho `buyer`.

payment: Giao dịch / trạng thái thanh toán liên quan đến `bill` (pending, paid, failed).

audit log: Lịch sử các thao tác quan trọng (ai approve/discard/export/modify và khi nào).

notification: Cơ chế thông báo (in-app/email/SMS) gửi tới `buyer` hoặc `seller` về sự kiện hệ thống.

authentication / login: Quá trình xác thực người dùng để cho phép truy cập vào ứng dụng. Dựa trên `username` + `password`; cần lưu `password_hash`, `last_login`, `is_active` trong hồ sơ người dùng.

developer: Người phát triển / vận hành hệ thống (ops). Developer chịu trách nhiệm provision tài khoản `admin` ban đầu và có thể thực hiện tác vụ quản trị hệ thống ngoài ứng dụng (khôi phục, reset admin, v.v.).

Ghi chú: sử dụng chính xác các thuật ngữ trên trong tất cả tài liệu để tránh nhầm lẫn.
