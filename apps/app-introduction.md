# Tên ứng dụng

NhaTro

# Mô tả

App quản lý nhà trọ

# Yêu cầu truy cập

- Ứng dụng yêu cầu đăng nhập (authentication) để truy cập. Mọi thao tác quản lý và xem thông tin đều cần tài khoản đã đăng nhập.
- Tài khoản `admin` được provision bởi developer (developer tạo/sắp xếp tài khoản `admin` ban đầu). `admin` sẽ tiếp tục tạo và quản lý các `seller`.

# Đối tượng sử dụng

Có 3 đối tượng sử dụng app này: `admin`, `seller`, `buyer` (xem `app-terms.md` để biết định nghĩa).

# Tính năng

## seller

- Tạo `variable` (biến tham chiếu cho formula)
- Tạo `formula` với biểu thức có `{{variable}}`, hỗ trợ điều kiện (if/else) và tham chiếu formula khác
- Tạo tài khoản `buyer` với số người (`people_count`) và tiền phòng (`room_price`)
- Gán `formula` riêng cho `buyer` (tuỳ chọn, override formula từ sheet config)
- Tạo `usage record` (bảng ghi) để thu thập chỉ số cuối kỳ
- Gửi link usage record đến buyer
- Duyệt (approve/manual) các `submission` từ buyer
- Tạo `sheet config`: chọn usage record cũ/mới, gắn variable và formula
- Tạo `bill` từ sheet config
- Kiểm tra tình trạng thanh toán

## buyer

- Nhận link `usage record` từ seller
- Submit chỉ số điện/nước vào usage record
- Nhận và xem bill
- Đóng tiền

## admin

- Có tất cả quyền như `seller`
- Tạo tài khoản `seller`
- Quản lý các `seller`
