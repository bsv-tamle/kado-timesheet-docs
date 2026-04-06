# Tài Liệu Chi Tiết Màn Hình Hệ Thống Timesheet MVP

Tài liệu này mô tả chi tiết từng màn hình trong phạm vi MVP, bao gồm:
- Mục tiêu
- Field/chức năng chính
- Action
- Validation
- Ghi chú nghiệp vụ

## 1. Nhóm màn hình dùng chung

### SCR-01. Login
Mục tiêu:
- Cho người dùng đăng nhập vào hệ thống bằng tài khoản đã được cấp.

Field chính:
- `email_or_username`: tên đăng nhập hoặc email
- `password`: mật khẩu
- `remember_me`: ghi nhớ đăng nhập, nếu cần

Action:
- `Đăng nhập`
- `Quên mật khẩu`

Validation:
- Bắt buộc nhập `email_or_username`
- Bắt buộc nhập `password`
- Thông báo lỗi nếu sai thông tin đăng nhập
- Thông báo nếu tài khoản bị khóa/inactive

Ghi chú nghiệp vụ:
- Đăng nhập thành công sẽ điều hướng theo role
- `admin` vào khu vực quản trị
- `employee` vào employee dashboard

### SCR-02. Quên mật khẩu
Mục tiêu:
- Cho người dùng gửi yêu cầu reset mật khẩu qua email.

Field chính:
- `email`

Action:
- `Gửi link reset`
- `Quay lại đăng nhập`

Validation:
- Bắt buộc nhập email
- Email đúng định dạng
- Nếu email không tồn tại, thông báo theo cách an toàn, không làm lộ thông tin hệ thống

Ghi chú nghiệp vụ:
- Link reset có thời hạn
- Sử dụng email của user đã được tạo trong hệ thống

### SCR-03. Đặt lại mật khẩu
Mục tiêu:
- Cập nhật mật khẩu mới thông qua token reset.

Field chính:
- `new_password`
- `confirm_password`

Action:
- `Cập nhật mật khẩu`

Validation:
- Bắt buộc nhập đầy đủ 2 field
- `confirm_password` phải trùng `new_password`
- Kiểm tra token còn hiệu lực

Ghi chú nghiệp vụ:
- Sau khi đổi mật khẩu thành công có thể điều hướng về trang login

### SCR-04. Hồ sơ cá nhân / Đổi mật khẩu
Mục tiêu:
- Cho user cập nhật thông tin cá nhân cơ bản và đổi mật khẩu.

Field chính:
- `full_name`
- `email`
- `phone`
- `position`
- `department`
- `avatar`
- Tab đổi mật khẩu:
  - `current_password`
  - `new_password`
  - `confirm_password`

Action:
- `Lưu thông tin`
- `Đổi mật khẩu`

Validation:
- `full_name` bắt buộc
- `email` đúng định dạng và unique nếu cho phép sửa
- `phone` đúng định dạng nếu có quy định
- Đổi mật khẩu phải nhập đúng `current_password`
- `confirm_password` phải trùng khớp

Ghi chú nghiệp vụ:
- Role employee và admin có thể dùng chung màn hình này

## 2. Nhóm màn hình Admin

### SCR-05. Admin Dashboard
Mục tiêu:
- Hiển thị tổng quan tình hình nhập công và sử dụng hệ thống.

Field/Khối thông tin chính:
- KPI tổng số nhân viên active
- KPI tổng số project active
- KPI số nhân viên chưa nhập công trong ngày
- Biểu đồ tổng giờ theo tháng
- Danh sách nhân viên chưa nhập công
- Bộ lọc tháng

Action:
- `Lọc theo tháng`
- `Đi đến báo cáo`
- `Export`

Validation:
- Không có validation phức tạp trên form
- Nếu không có dữ liệu thì hiển thị empty state

Ghi chú nghiệp vụ:
- Chỉ admin được xem dữ liệu tổng hợp toàn hệ thống

### SCR-06. Quản lý Project
Mục tiêu:
- Quản lý danh sách project và thực hiện CRUD.

Field/Khối thông tin chính:
- Bảng danh sách:
  - `project_code`
  - `project_name`
  - `status`
  - `billable_flag`
  - `updated_at`
- Bộ lọc:
  - từ khóa
  - status
  - billable/non-billable

Action:
- `Tạo project`
- `Sửa`
- `Archive`
- `Khôi phục`
- `Tìm kiếm`
- `Lọc`

Validation:
- Không cho tạo trùng `project_code`
- Không cho xóa cứng/phá dữ liệu liên quan, ưu tiên archive

Ghi chú nghiệp vụ:
- Nên ưu tiên dùng `archive` thay vì xóa vật lý

### SCR-07. Form Tạo/Sửa Project
Mục tiêu:
- Nhập thông tin chi tiết cho 1 project.

Field chính:
- `project_code`
- `project_name`
- `status`
- `billable_flag`
- `description`
- Dự trù mở rộng:
  - `planned_man_days`
  - `unit_price_per_man_day`
  - `cost_per_man_day`

Action:
- `Lưu`
- `Lưu và tạo mới`
- `Hủy`

Validation:
- `project_code` bắt buộc, unique
- `project_name` bắt buộc
- `status` bắt buộc
- Các field số nếu nhập phải >= 0

Ghi chú nghiệp vụ:
- Các field tài chính là mở rộng, MVP có thể cho phép bỏ trống

### SCR-08. Quản lý Nhân viên / User
Mục tiêu:
- Quản lý danh sách nhân viên và tài khoản đăng nhập.

Field/Khối thông tin chính:
- Bảng danh sách:
  - `employee_code` nếu có
  - `full_name`
  - `email`
  - `department`
  - `position`
  - `status`
- Bộ lọc:
  - từ khóa
  - department
  - status

Action:
- `Tạo nhân viên`
- `Sửa`
- `Reset mật khẩu`
- `Khóa/Mở tài khoản`
- `Tìm kiếm`

Validation:
- `email` unique
- Không cho khóa tài khoản admin cuối cùng, nếu có rule này

Ghi chú nghiệp vụ:
- Mọi user employee sẽ có tài khoản login

### SCR-09. Form Tạo/Sửa Nhân viên
Mục tiêu:
- Nhập thông tin nhân viên và tài khoản liên quan.

Field chính:
- `full_name`
- `email`
- `phone`
- `department`
- `position`
- `avatar`
- `status`
- `role` mặc định là employee

Action:
- `Lưu`
- `Tạo tài khoản`
- `Hủy`

Validation:
- `full_name` bắt buộc
- `email` bắt buộc, đúng định dạng, unique
- `status` bắt buộc

Ghi chú nghiệp vụ:
- MVP chỉ cần role `employee`; role `admin` được cấp có chủ đích

### SCR-10. Assign Project Cho Nhân viên
Mục tiêu:
- Quản lý danh sách project mà từng nhân viên được phép nhập công.

Field chính:
- `employee`
- Danh sách project available
- Danh sách project assigned
- Có thể bổ sung:
  - `assigned_from`
  - `assigned_to` nếu muốn mở rộng

Action:
- `Gán project`
- `Thu hồi project`
- `Lưu thay đổi`

Validation:
- Bắt buộc chọn nhân viên
- Không gán trùng cùng 1 project cho 1 nhân viên

Ghi chú nghiệp vụ:
- Đây là màn hình quan trọng vì employee chỉ được thấy project đã assign

### SCR-11. Báo cáo / Thống kê
Mục tiêu:
- Cho admin xem và xuất báo cáo giờ làm việc theo nhiều chiều lọc.

Field/Khối thông tin chính:
- Bộ lọc:
  - `month`
  - `employee`
  - `project`
  - `department`
  - `billable_flag` nếu cần
- Bảng tổng hợp
- Chart tổng quan
- Số liệu summary

Action:
- `Áp dụng bộ lọc`
- `Reset bộ lọc`
- `Export Excel`
- `Export CSV`

Validation:
- `month` là bộ lọc quan trọng, nên có giá trị mặc định
- Nếu không có dữ liệu thì hiển thị empty state rõ ràng

Ghi chú nghiệp vụ:
- Báo cáo là nơi suy ra OT nếu tổng giờ/ngày > 8

### SCR-12. Danh mục phụ trợ
Mục tiêu:
- Quản lý dữ liệu danh mục dùng chung cho hệ thống.

Field/Khối thông tin chính:
- Tab `Phòng ban`
- Tab `Chức danh`
- Tab `Loại công việc`

Action:
- `Thêm`
- `Sửa`
- `Xóa`

Validation:
- Tên danh mục bắt buộc
- Không cho trùng tên trong cùng 1 loại danh mục nếu cần

Ghi chú nghiệp vụ:
- Có thể làm 1 màn hình với nhiều tab để giảm số route

## 3. Nhóm màn hình Employee

### SCR-13. Employee Dashboard
Mục tiêu:
- Cho employee xem tổng quan công việc và tình trạng nhập công của chính mình.

Field/Khối thông tin chính:
- Tổng giờ tháng hiện tại
- Số ngày đã nhập công
- Danh sách project có nhiều giờ nhất
- Nhật ký gần đây

Action:
- `Đến trang nhập timesheet`
- `Đến trang thống kê cá nhân`

Validation:
- Không có validation form

Ghi chú nghiệp vụ:
- Không hiển thị dữ liệu của người khác

### SCR-14. Nhập Timesheet
Mục tiêu:
- Cho employee xem dữ liệu timesheet theo tháng và thực hiện nhập giờ theo ngày, theo project đã được assign.

Field/Khối thông tin chính:
- `month`
- Khu vực danh sách dữ liệu timesheet đã có:
  - dữ liệu trả về từ API list
  - nhóm theo project hoặc theo ngày
  - hiển thị `project`, `work_date`, `hours_worked`, `note`
- Bảng timesheet:
  - dòng là project
  - cột là ngày trong tháng
  - ô hiển thị hoặc chỉnh sửa giá trị `hours_worked`
- Tổng giờ mỗi ngày
- Tổng giờ mỗi project
- Nút mở form nhập công

Action:
- `Lưu`
- `Chuyển tháng`
- `Thêm dòng nhập công`
- `Sửa entry`
- `Xóa entry` nếu cho phép
- `Copy từ ngày hôm trước` nếu muốn mở rộng

Validation:
- Không cho nhập ngày tương lai
- Cho phép nhập dữ liệu quá khứ
- Cho phép giá trị thập phân như `0.25`, `0.5`
- Giá trị phải >= 0
- Tổng giờ mỗi ngày không được > 24
- Chỉ được nhập trên project đã assign

Ghi chú nghiệp vụ:
- Đây là màn hình quan trọng nhất của employee
- Nên có cơ chế lưu nhập liệu rõ ràng và thông báo khi vượt 24h/ngày
- Mockup hiện tại của bảng timesheet có thể được hiểu là màn hình hiển thị dữ liệu sau khi gọi API list
- Cần bổ sung thêm form nhập liệu riêng để người dùng thêm/cập nhật từng entry dễ hơn trên mobile hoặc trong các trường hợp không phù hợp với grid lớn

### SCR-14A. Form Nhập Công Theo Entry
Mục tiêu:
- Cho employee nhập nhiều entry timesheet theo dạng form/danh sách tạm, rồi submit một lần.

Field chính:
- Form nhập 1 dòng:
  - `work_date`
  - `project`
  - `hours_worked`
  - `note`: ghi chú đã làm gì với project đó
- Danh sách tạm trên FE:
  - nhiều dòng entry chờ submit
  - có thao tác sửa/xóa dòng trước khi gửi

Action:
- `Thêm vào danh sách tạm`
- `Sửa/Xóa dòng trong danh sách tạm`
- `Submit tất cả`
- `Hủy`

Validation:
- Mỗi dòng bắt buộc có `work_date`, `project`, `hours_worked`
- Không cho chọn ngày tương lai
- `project` chỉ được chọn trong danh sách project đã assign
- `hours_worked` phải >= 0
- Cho phép số lẻ như `0.25`, `0.5`
- Tổng số giờ của user trong `work_date` sau khi cộng toàn bộ batch không được > 24
- `note` không bắt buộc, nhưng nên có giới hạn độ dài nếu lưu vào DB

Ghi chú nghiệp vụ:
- Form này có thể mở bằng modal hoặc route riêng
- Nên dùng cùng logic validation với màn hình timesheet grid
- Trường `note` giúp giải thích user đã làm gì trên project, hữu ích cho tra cứu hoặc mở rộng báo cáo sau này

### SCR-15. Thống kê cá nhân
Mục tiêu:
- Cho employee xem tổng hợp giờ làm việc của chính mình.

Field/Khối thông tin chính:
- Bộ lọc `month`
- Tổng giờ trong tháng
- Tổng giờ theo project
- Chart/Biểu đồ
- Bảng chi tiết

Action:
- `Lọc theo tháng`
- `Xem chi tiết`

Validation:
- `month` có giá trị mặc định

Ghi chú nghiệp vụ:
- Không được xem dữ liệu của người khác

### SCR-16. Profile Employee
Mục tiêu:
- Entry riêng trong menu employee để vào màn hình hồ sơ cá nhân.

Field, Action, Validation:
- Dùng chung nội dung với `SCR-04`

Ghi chú nghiệp vụ:
- Về kỹ thuật có thể dùng chung component/màn hình profile

## 4. Kiến nghị về UX và phạm vi implementation

Để tối ưu effort MVP:
- `SCR-07` có thể là modal/drawer trong `SCR-06`
- `SCR-09` có thể là modal/drawer trong `SCR-08`
- `SCR-16` có thể dùng chung route/component với `SCR-04`
- `SCR-12` nên là 1 page nhiều tab thay vì 3 page tách riêng

## 5. Thứ tự ưu tiên wireframe/màn hình

Ưu tiên 1:
- SCR-01 Login
- SCR-05 Admin Dashboard
- SCR-06 Quản lý Project
- SCR-08 Quản lý Nhân viên / User
- SCR-10 Assign Project
- SCR-11 Báo cáo / Thống kê
- SCR-14 Nhập Timesheet
- SCR-13 Employee Dashboard

Ưu tiên 2:
- SCR-04 Profile / Đổi mật khẩu
- SCR-02 Quên mật khẩu
- SCR-03 Đặt lại mật khẩu
- SCR-12 Danh mục phụ trợ
- SCR-15 Thống kê cá nhân
