# Use case

- Yêu cầu: người dùng phải đăng nhập (`login`) trước khi truy cập hoặc thao tác trong ứng dụng. Tài khoản `admin` được developer tạo (provisioned) trước.

## Thiết lập ban đầu

- admin mở tài khoản cho seller
- seller tạo các `variable` cần dùng (ví dụ: "Số điện đã sử dụng", "Số nước đã sử dụng", "Số người")
- seller tạo `formula` với biểu thức có thể chứa `{{variable}}`, tham chiếu `formula` khác, số cụ thể, và điều kiện (if/else); ví dụ: `{{Số điện đã sử dụng}} * 2500 + {{Số nước đã sử dụng}} * 14000`
- seller tạo tài khoản `buyer`, đặt tên tuỳ ý (ví dụ "Phòng A"), và nhập chỉ số `baseline` (điện, nước, số người) kèm tháng khởi điểm
- seller có thể gán `formula` riêng cho từng `buyer` để override formula từ `sheet config` (tuỳ chọn)

## Thay đồng hồ (khi cần)

Khi đồng hồ điện hoặc nước của 1 buyer bị thay mới, seller tạo 1 `replacement request` cho buyer đó:

- Chọn loại đồng hồ (điện hoặc nước)
- Nhập chỉ số đồng hồ vật lý cũ tại thời điểm thay (x)
- Nhập chỉ số ban đầu của đồng hồ mới sau khi lắp (a)

Khi tính bill cho kỳ có thay đồng hồ, seller áp replacement request vào sheet config. Lúc này lượng tiêu thụ được tính theo công thức `(b − a) + (x − y)`, trong đó y là chỉ số từ old usage record và b là chỉ số từ new usage record.

## Kỳ xuất bill

1. Seller tạo 1 `usage record` (bảng ghi) và đặt tên (ví dụ "Tháng 4/2026")
2. Seller gửi link usage record đến các `buyer`
3. Mỗi buyer truy cập link và submit chỉ số điện/nước hiện tại (`submission`)
4. Seller vào usage record, duyệt từng `submission`:
   - `approve`: chấp nhận chỉ số do buyer gửi
   - Seller có thể tự nhập thủ công nếu buyer không submit
5. Sau khi duyệt, usage record lưu danh sách chỉ số điện/nước đã xác nhận của tất cả buyer
6. Seller tạo `sheet config`:
   - Chọn `usage record` cũ (old) và mới (new)
   - Gắn `variable` vào giá trị điện/nước đã sử dụng (tính từ chênh lệch old và new record)
   - Gắn `formula` mặc định cho sheet config (tuỳ chọn)
7. Seller tạo `bill` bằng cách chọn `sheet config`:
   - Hệ thống tính consumption cho từng buyer: giá trị new record − old record
   - Áp variable bindings và formula (formula riêng của buyer nếu có, ngược lại dùng formula từ sheet config) để ra `total_amount`
   - Xuất bill cho từng buyer; bill có thể được copy dưới dạng hình ảnh để gửi cho buyer

Mỗi `buyer` chỉ có tối đa 1 `bill` cho mỗi `billing cycle`.
