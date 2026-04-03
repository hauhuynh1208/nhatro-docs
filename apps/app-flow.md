# Use case

- Yêu cầu: người dùng phải đăng nhập (`login`) trước khi truy cập hoặc thao tác trong ứng dụng. Tài khoản `admin` được developer tạo (provisioned) trước.

- admin mở tài khoản cho seller
- seller tạo house
- seller tạo `group` (tuỳ chọn) để gom nhiều house có chung giá và công thức tính tiền
- seller tạo price, ví dụ như `electric price`, `water price` hoặc `house price`
- seller tạo formula (xem `app-terms.md`) dùng để tính `bill`
- seller tạo tài khoản buyer
- Tại màn hình quản lý `house` của seller, seller gắn `price`, `formula` và `buyer` vào `house`; hoặc gán `house` vào `group` để kế thừa `price` và `formula` của nhóm
- Khi đến cuối kỳ tính tiền, sẽ có các use cases như sau:
  - buyer sẽ tạo thông tin điện và nước (`electricity usage` và `water usage`) cuối kỳ, bao gồm hình chụp số trên đồng hồ (meter id), số đọc, ảnh meter, và timestamp. Sau đó gửi về cho seller thông qua app
  - seller cũng có thể tự tạo thông tin điện nước cho một house nào đó
  - Tại màn hình quản lý `house`, seller sẽ review các `usage` do `buyer` hoặc `seller` tạo. Khi review, seller có thể `approve` hoặc `discard`. `approve` có nghĩa chuyển `usage` thành giá trị cuối kỳ; `discard` nghĩa là từ chối và không sử dụng `usage` đó để tính `bill`.
  - Sau khi review và approve / discard, seller sẽ bấm xuất bill. Khi xuất bill, 1 tờ bảng kê khai sẽ xuất hiện, và user có thể copy bảng kê khai đó dưới dạng hình ảnh để gửi cho buyer.

-- Với bảng tính `bill`, seller có thể lưu lại thành `bill` của tháng để truy cứu. Mỗi `house` chỉ có tối đa 1 `bill` cho mỗi `billing cycle`.
