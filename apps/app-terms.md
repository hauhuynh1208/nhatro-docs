THUẬT NGỮ: GIẢI THÍCH

admin: Người quản trị hệ thống, có quyền cao nhất để quản lý toàn bộ hệ thống và các `seller`.

seller: Chủ nhà trọ (người đăng/quản lý `house` và tạo `price` / `formula`).

buyer: Người thuê trọ (tenant) — người nhận `bill`, gửi `electricity usage` / `water usage`.

group: Nhóm do `seller` tạo, phân biệt bằng `type`. `type=1` là nhóm house (house group): gom các phòng có chung `price` và `formula`; mỗi `house` chỉ thuộc tối đa 1 group loại này. `type=2` (tương lai) là nhóm user. Khi `house` thuộc `group`, nó kế thừa `price` và `formula` của `group` thay vì cấu hình riêng từng phòng.

house: Đơn vị cho thuê (phòng/nhà) mà `seller` tạo và quản lý.

price: Bảng giá (ví dụ: `electric price`, `water price`, `house price`) — chứa đơn vị, giá trên đơn vị, và metadata về effective period.

formula: Công thức/luật để tính `bill` từ các `usage` và `price` (có thể là biểu thức hoặc rules engine).

electricity usage: Bản ghi tiêu thụ điện do `buyer` hoặc `seller` tạo, gồm: meter id, current reading, previous reading (nếu có), ảnh meter, timestamp, và metadata.

water usage: Bản ghi tiêu thụ nước tương tự như `electricity usage`.

meter: Đồng hồ (điện hoặc nước) gắn với `house`. Có `meter id` duy nhất.

reading: Giá trị số đọc từ `meter` (số nguyên hoặc số thực tuỳ loại).

approve: Hành động của `seller` khi chấp nhận một `usage` làm giá trị cuối kỳ.

discard: Hành động của `seller` khi từ chối một `usage` (không dùng để tính bill).

bill: Hóa đơn của một `house` cho một `billing cycle` (tháng). Mỗi `house` chỉ có tối đa một `bill` cho mỗi kỳ.

billing cycle: Kỳ tính tiền (ví dụ: tháng dương lịch) dùng để gom `usage` và xuất `bill`.

export bill: Hành động xuất `bill` ra định dạng tĩnh (ảnh, PDF) để gửi cho `buyer`.

payment: Giao dịch / trạng thái thanh toán liên quan đến `bill` (pending, paid, failed).

audit log: Lịch sử các thao tác quan trọng (ai approve/discard/export/modify và khi nào).

notification: Cơ chế thông báo (in-app/email/SMS) gửi tới `buyer` hoặc `seller` về sự kiện hệ thống.

authentication / login: Quá trình xác thực người dùng để cho phép truy cập vào ứng dụng. Thông thường dựa trên `email` + `password` hoặc provider OAuth; cần lưu `password_hash`, `auth_provider`, `last_login`, `is_active` trong hồ sơ người dùng.

developer: Người phát triển / vận hành hệ thống (ops). Developer chịu trách nhiệm provision tài khoản `admin` ban đầu và có thể thực hiện tác vụ quản trị hệ thống ngoài ứng dụng (khôi phục, reset admin, v.v.).

Ghi chú: sử dụng chính xác các thuật ngữ trên trong tất cả tài liệu để tránh nhầm lẫn (ví dụ không đổi `seller` thành `owner` hoặc `host` ở chỗ khác).
