THUẬT NGỮ: GIẢI THÍCH

admin: Người quản trị hệ thống, có quyền cao nhất để quản lý toàn bộ hệ thống và các `seller`.

seller: Chủ nhà trọ — người tạo và quản lý `buyer`, `variable`, `formula`, `usage record`, và `sheet config` trong hệ thống.

buyer: Người thuê trọ (tenant) — đơn vị tính bill do `seller` tạo. Mỗi buyer có `people_count` (số người), `room_price` (tiền phòng), và có thể được gán 1 `formula` riêng (override). Các giá trị `people_count` và `room_price` có thể thay đổi theo thời gian nhưng không ảnh hưởng đến các `bill` đã tạo (giá trị được snapshot tại thời điểm phát hành bill). Buyer nhận link `usage record` để submit chỉ số, và nhận `bill` sau khi seller duyệt. Seller có thể đặt tên buyer tuỳ ý (ví dụ "Phòng A").

formula: Biểu thức do `seller` viết để tính `bill`. Có thể chứa `{{variable}}`, tham chiếu đến `formula` khác, số cụ thể, và điều kiện (if/else). Ví dụ: `if {{Số người}} > 2 then {{Số điện đã sử dụng}} * 3000 else {{Số điện đã sử dụng}} * 2500`. Buyer có thể được gán formula riêng để override formula từ `sheet config`.

variable: Biến có tên do `seller` định nghĩa (ví dụ: "Số điện đã sử dụng", "Tiền điện"). Được tham chiếu trong `formula` bằng cú pháp `{{tên biến}}`. Giá trị thực tế của variable được gắn vào qua `sheet config` tại thời điểm tạo bill.

usage record: Bảng ghi chỉ số do `seller` tạo, dùng để thu thập chỉ số điện/nước từ các `buyer`. Seller đặt tên cho bảng ghi, rồi gửi link cho buyer để submit. Sau khi seller duyệt xong, bảng ghi lưu danh sách chỉ số điện/nước đã xác nhận của tất cả buyer. Tên usage record phải unique trong cùng 1 seller.

submission: Bản ghi chỉ số (điện, nước) do `buyer` gửi vào `usage record`. Mỗi buyer có tối đa 1 submission per usage record. Sau khi `approve` (hoặc seller tự nhập thủ công), giá trị trở thành chỉ số hiện tại của buyer trong bảng ghi đó.

baseline: Chỉ số khởi điểm điện/nước của `buyer` dùng để tính consumption. Baseline được cung cấp bởi `usage record` cũ (old record) trong `sheet config` — không phải được lưu trữ trực tiếp trên entity buyer. Consumption được tính từ chênh lệch giữa old và new usage record.

sheet config: Biểu mẫu tính bill do `seller` tạo. Seller định nghĩa các cột (column): các cột có sẵn (people_count, room_price, chỉ số điện/nước cũ, mới, lượng điện/nước đã dùng) và các cột tự thêm. Mỗi cột có thể được gán 1 `variable` (biến nhận giá trị của cột đó) hoặc 1 `formula` (hiển thị kết quả tính của formula). Khi tạo sheet config, seller dùng dữ liệu dummy để preview kết quả — không có `usage record` nào được gắn vào lúc này. Khi tạo `bill`, seller chọn sheet config kèm old và new `usage record`; hệ thống thay thế các cột chỉ số bằng dữ liệu thực tế của từng `buyer`.

replacement request: Bản ghi sự kiện thay đồng hồ điện hoặc nước của 1 `buyer`, do `seller` tạo. Lưu loại đồng hồ (điện/nước), chỉ số đồng hồ vật lý cũ tại thời điểm thay (x), và chỉ số ban đầu của đồng hồ mới sau khi lắp (a). Khi có replacement request, lượng tiêu thụ trong kỳ tính bill được tính theo công thức `(b − a) + (x − y)`, trong đó b là chỉ số đồng hồ mới từ new usage record, y là chỉ số đồng hồ cũ từ old usage record. Điều này đảm bảo tổng tiêu thụ gồm cả phần dùng trên đồng hồ cũ (chưa được ghi nhận) và phần dùng trên đồng hồ mới.

approve: Hành động của `seller` khi chấp nhận một `submission`, xác nhận giá trị điện/nước do buyer gửi. Nếu buyer không submit, seller có thể tự nhập thủ công.

discard: Hành động của `seller` khi từ chối một `submission` (không dùng để tính bill).

bill: Hóa đơn của một `buyer` cho một kỳ tính tiền, được tạo từ 1 `sheet config`. Seller đặt tên cho bill (tên phải unique trong cùng 1 seller). Hệ thống tính consumption (chênh lệch giữa old và new usage record), áp variable bindings và formula để ra tổng tiền. Giá trị `people_count` và `room_price` của buyer được snapshot vào `line_items` tại thời điểm tạo bill — thay đổi sau này của buyer không ảnh hưởng đến bill đã phát hành. Mỗi `buyer` chỉ có tối đa một `bill` cho mỗi kỳ.

billing cycle: Kỳ tính tiền (ví dụ: tháng dương lịch) dùng để gom `submission` và xuất `bill`.

export bill: Hành động xuất `bill` ra định dạng tĩnh (ảnh, PDF) để gửi cho `buyer`.

payment: Giao dịch / trạng thái thanh toán liên quan đến `bill` (pending, paid, failed).

audit log: Lịch sử các thao tác quan trọng (ai approve/discard/export/modify và khi nào).

notification: Cơ chế thông báo (in-app/email/SMS) gửi tới `buyer` hoặc `seller` về sự kiện hệ thống.

authentication / login: Quá trình xác thực người dùng để cho phép truy cập vào ứng dụng. Dựa trên `username` + `password`; cần lưu `password_hash`, `last_login`, `is_active` trong hồ sơ người dùng.

developer: Người phát triển / vận hành hệ thống (ops). Developer chịu trách nhiệm provision tài khoản `admin` ban đầu và có thể thực hiện tác vụ quản trị hệ thống ngoài ứng dụng (khôi phục, reset admin, v.v.).

Ghi chú: sử dụng chính xác các thuật ngữ trên trong tất cả tài liệu để tránh nhầm lẫn.
