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

- Tạo `formula` với biểu thức có `{{variable}}`
- Tạo tài khoản `buyer` với chỉ số `baseline` (điện, nước, số người) và tháng khởi điểm
- Gán `formula` cho `buyer`
- Tạo `container` để thu thập chỉ số cuối kỳ
- Gửi link container đến buyer
- Duyệt (approve/discard) các `submission` từ buyer
- Xuất bill cho buyer
- Kiểm tra tình trạng thanh toán

## buyer

- Nhận link `container` từ seller
- Submit chỉ số điện/nước vào container
- Nhận và xem bill
- Đóng tiền

## admin

- Có tất cả quyền như `seller`
- Tạo tài khoản `seller`
- Quản lý các `seller`
