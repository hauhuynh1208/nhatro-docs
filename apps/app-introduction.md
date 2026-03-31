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

- Tạo tài khoản buyer
- Tạo phòng cho thuê (phòng cho thuê gọi là house)
- Gán buyer vào phòng cho thuê
- Tạo bảng giá (bảng giá gọi là price)
- Gán bảng giá cho phòng cho thuê
- Ghi số điện / nước
- Approve số điện / nước được ghi từ buyer
- Tạo bill
- Kiểm tra tình trạng thanh toán

## buyer

- Nhận bill từ seller
- Gửi thông tin điện / nước đến cho buyer
- Đóng tiền phòng

## admin

- Có tất cả quyền như `seller`
- Tạo tài khoản `seller`
- Quản lý các `seller`
