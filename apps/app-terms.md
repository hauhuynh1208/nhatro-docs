THUẬT NGỮ: GIẢI THÍCH

admin: Người quản trị hệ thống, có quyền cao nhất để quản lý toàn bộ hệ thống và các `seller`.

seller: Chủ nhà trọ — người tạo và quản lý `buyer`, `variable`, `formula`, `usage record`, và `sheet config` trong hệ thống.

buyer: Người thuê trọ (tenant) — đơn vị tính bill do `seller` tạo. Mỗi buyer có chỉ số `baseline` (điện, nước, số người), `room_price` (tiền phòng), và có thể được gán 1 `formula` riêng (override). Các giá trị `people_count` và `room_price` có thể thay đổi theo thời gian nhưng không ảnh hưởng đến các `bill` đã tạo (giá trị được snapshot tại thời điểm phát hành bill). Buyer nhận link `usage record` để submit chỉ số, và nhận `bill` sau khi seller duyệt. Seller có thể đặt tên buyer tuỳ ý (ví dụ "Phòng A").

formula: Biểu thức do `seller` viết để tính `bill`. Có thể chứa `{{variable}}`, tham chiếu đến `formula` khác, số cụ thể, và điều kiện (if/else). Ví dụ: `if {{Số người}} > 2 then {{Số điện đã sử dụng}} * 3000 else {{Số điện đã sử dụng}} * 2500`. Buyer có thể được gán formula riêng để override formula từ `sheet config`.

variable: Biến có tên do `seller` định nghĩa (ví dụ: "Số điện đã sử dụng", "Tiền điện"). Được tham chiếu trong `formula` bằng cú pháp `{{tên biến}}`. Giá trị thực tế của variable được gắn vào qua `sheet config` tại thời điểm tạo bill.

usage record: Bảng ghi chỉ số do `seller` tạo, dùng để thu thập chỉ số điện/nước từ các `buyer`. Seller đặt tên cho bảng ghi (ví dụ "Tháng 4/2026"), rồi gửi link cho buyer để submit. Sau khi seller duyệt xong, bảng ghi lưu danh sách chỉ số điện/nước đã xác nhận của tất cả buyer.

submission: Bản ghi chỉ số (điện, nước) do `buyer` gửi vào `usage record`. Mỗi buyer có tối đa 1 submission per usage record. Sau khi `approve` (hoặc seller tự nhập thủ công), giá trị trở thành chỉ số hiện tại của buyer trong bảng ghi đó.

baseline: Chỉ số khởi điểm (điện, nước, số người) của `buyer`, được nhập khi seller tạo buyer lần đầu. Baseline là điểm xuất phát ban đầu và không tự động cập nhật sau mỗi kỳ — consumption về sau được tính từ 2 `usage record` (old và new) thông qua `sheet config`.

sheet config: Thiết lập bảng tính do `seller` tạo để cấu hình tính bill. Seller chọn `usage record` cũ và mới, từ đó gắn `variable` vào giá trị điện/nước đã sử dụng (tính từ chênh lệch old và new record). Seller cũng có thể gắn `formula` mặc định. Một sheet config gắn với 1 cặp usage record cụ thể và được dùng để tạo `bill`.

approve: Hành động của `seller` khi chấp nhận một `submission`, xác nhận giá trị điện/nước do buyer gửi. Nếu buyer không submit, seller có thể tự nhập thủ công.

discard: Hành động của `seller` khi từ chối một `submission` (không dùng để tính bill).

bill: Hóa đơn của một `buyer` cho một kỳ tính tiền, được tạo từ 1 `sheet config`. Hệ thống tính consumption (chênh lệch giữa old và new usage record), áp variable bindings và formula để ra tổng tiền. Giá trị `people_count` và `room_price` của buyer được snapshot vào `line_items` tại thời điểm tạo bill — thay đổi sau này của buyer không ảnh hưởng đến bill đã phát hành. Mỗi `buyer` chỉ có tối đa một `bill` cho mỗi kỳ.

billing cycle: Kỳ tính tiền (ví dụ: tháng dương lịch) dùng để gom `submission` và xuất `bill`.

export bill: Hành động xuất `bill` ra định dạng tĩnh (ảnh, PDF) để gửi cho `buyer`.

payment: Giao dịch / trạng thái thanh toán liên quan đến `bill` (pending, paid, failed).

audit log: Lịch sử các thao tác quan trọng (ai approve/discard/export/modify và khi nào).

notification: Cơ chế thông báo (in-app/email/SMS) gửi tới `buyer` hoặc `seller` về sự kiện hệ thống.

authentication / login: Quá trình xác thực người dùng để cho phép truy cập vào ứng dụng. Dựa trên `username` + `password`; cần lưu `password_hash`, `last_login`, `is_active` trong hồ sơ người dùng.

developer: Người phát triển / vận hành hệ thống (ops). Developer chịu trách nhiệm provision tài khoản `admin` ban đầu và có thể thực hiện tác vụ quản trị hệ thống ngoài ứng dụng (khôi phục, reset admin, v.v.).

Ghi chú: sử dụng chính xác các thuật ngữ trên trong tất cả tài liệu để tránh nhầm lẫn.
