# Use case

- Yêu cầu: người dùng phải đăng nhập (`login`) trước khi truy cập hoặc thao tác trong ứng dụng. Tài khoản `admin` được developer tạo (provisioned) trước.

## Thiết lập ban đầu

- admin mở tài khoản cho seller
- seller tạo `formula` với biểu thức có chứa `{{variable}}`, ví dụ: `{{Số điện đã sử dụng}} * 2500 + {{Số nước đã sử dụng}} * 14000`
- seller tạo tài khoản `buyer`, đặt tên tuỳ ý (ví dụ "Phòng A"), và nhập chỉ số `baseline` (điện, nước, số người) kèm tháng khởi điểm
- seller gán 1 `formula` cho mỗi `buyer`

## Kỳ xuất bill

1. Seller tạo 1 `container` cho billing cycle (ví dụ tháng 4/2026)
2. Seller gửi link container đến các `buyer`
3. Mỗi buyer truy cập link và submit chỉ số điện/nước hiện tại (`submission`)
4. Seller vào container, duyệt từng `submission`:
   - `approve`: chấp nhận chỉ số — trở thành giá trị current
   - `discard`: từ chối — không dùng để tính bill
5. Với từng buyer đã được approve, hệ thống tính consumption:
   - `{{Số điện đã sử dụng}}` = `electricity_current` − `electricity_baseline`
   - `{{Số nước đã sử dụng}}` = `water_current` − `water_baseline`
   - Áp các giá trị này vào `formula` của buyer để ra `total_amount`
6. Seller xuất bill cho từng buyer. Bill có thể được copy dưới dạng hình ảnh để gửi cho buyer.
7. Sau khi xuất bill, giá trị current đã được approve tự động trở thành `baseline` mới của buyer cho kỳ kế tiếp.

Mỗi `buyer` chỉ có tối đa 1 `bill` cho mỗi `billing cycle`.
