# Thiết kế API (Bản nháp v1)

## 1. Mục tiêu tài liệu
Tài liệu này mô tả API theo góc nhìn nghiệp vụ để:
- Team BE implement đúng flow đã chốt.
- Team FE có contract rõ để tích hợp.
- Làm nguồn đầu vào cho OpenAPI/Swagger sinh từ code Laravel.

Phạm vi v1:
- AUTH
- ADMIN-USER
- ADMIN-PROJECT
- ADMIN-ASSIGN
- TIMESHEET (entry + details)
- PROFILE (employee tự sửa thông tin cá nhân cơ bản)
- NOTIFICATION: job nội bộ, không có API public cho FE

## 2. Quy ước chung
- Base URL: `/api/v1`
- Auth: JWT Bearer
- Content-Type: `application/json`
- Timezone nghiệp vụ: `Asia/Tokyo` (JST)
- Format ngày: `YYYY-MM-DD`
- Format tháng: `YYYY-MM`

Chuẩn response thành công (gợi ý):
```json
{
  "data": {},
  "message": "Success"
}
```

Chuẩn response lỗi (gợi ý):
```json
{
  "message": "Validation error",
  "errors": {
    "field_name": ["The field is required."]
  }
}
```

Mã lỗi dùng chung:
- `400` bad request
- `401` unauthenticated / token hết hạn
- `403` forbidden
- `404` not found
- `409` conflict
- `422` validation/business rule error
- `500` internal server error

## 3. Matrix quyền truy cập
- `admin`: toàn quyền các nhóm `ADMIN-*`, xem báo cáo toàn hệ thống
- `employee`: nhập công của chính mình, xem thống kê của chính mình, sửa profile của chính mình

## 4. Nhóm AUTH

### AUTH-01 Login
- `POST /auth/login`
- Public

Request:
```json
{
  "email": "employee@company.com",
  "password": "Secret123!"
}
```

Response:
```json
{
  "data": {
    "access_token": "jwt_token",
    "token_type": "Bearer",
    "expires_in": 3600,
    "user": {
      "id": 101,
      "full_name": "Tran Thi Hoa",
      "email": "employee@company.com",
      "role": "employee",
      "must_change_password": true
    }
  },
  "message": "Login success"
}
```

Rules:
- Account phải `active`.
- Nếu `must_change_password=true`, FE điều hướng trang đổi mật khẩu bắt buộc.

### AUTH-02 Forgot Password
- `POST /auth/forgot-password`
- Public

Request:
```json
{
  "email": "employee@company.com"
}
```

Response:
```json
{
  "data": {
    "sent": true
  },
  "message": "If this email exists, reset instructions have been sent."
}
```

### AUTH-03 Reset Password
- `POST /auth/reset-password`
- Public (dùng token reset)

Request:
```json
{
  "email": "employee@company.com",
  "token": "reset_token",
  "password": "NewSecret123!",
  "password_confirmation": "NewSecret123!"
}
```

Response:
```json
{
  "data": {
    "reset": true
  },
  "message": "Password reset successfully"
}
```

### AUTH-04 Change Password
- `POST /auth/change-password`
- `employee|admin`

Request:
```json
{
  "current_password": "OldSecret123!",
  "new_password": "NewSecret123!",
  "new_password_confirmation": "NewSecret123!"
}
```

Response:
```json
{
  "data": {
    "changed": true
  },
  "message": "Password changed successfully"
}
```

Ghi chú:
- Khi đổi mật khẩu thành công, set `must_change_password=false`.

### AUTH-05 Logout
- Client-side logout (không yêu cầu API trong MVP)
- FE xoá access token/local session.

## 5. Nhóm ADMIN-USER

### ADMIN-USER-01 List Users
- `GET /admin/users`
- `admin`
- Query: `keyword`, `status`, `department_id`, `position_id`, `page`, `per_page`

### ADMIN-USER-02 Create User
- `POST /admin/users`
- `admin`

Request:
```json
{
  "full_name": "Tran Thi Hoa",
  "email": "tran.hoa@company.com",
  "phone": "0901234567",
  "department_id": 3,
  "position_id": 5,
  "role": "employee",
  "status": "active",
  "send_invitation_email": true
}
```

Response:
```json
{
  "data": {
    "id": 101,
    "full_name": "Tran Thi Hoa",
    "email": "tran.hoa@company.com",
    "role": "employee",
    "status": "active",
    "must_change_password": true,
    "onboarding": {
      "temp_password_generated": true,
      "invitation_email_sent": true
    }
  },
  "message": "User created successfully"
}
```

Rules:
- `email` unique.
- Sinh mật khẩu tạm + lưu hash.
- Mặc định `must_change_password=true`.

### ADMIN-USER-03 Update User
- `PUT /admin/users/{id}`
- `admin`

Request:
```json
{
  "full_name": "Tran Thi Hoa Updated",
  "phone": "0909999999",
  "department_id": 4,
  "position_id": 6,
  "status": "active"
}
```

### ADMIN-USER-04 Lock/Unlock User
- `PATCH /admin/users/{id}/status`
- `admin`

Request:
```json
{
  "status": "locked"
}
```

Rules:
- Trạng thái hợp lệ: `active|inactive|locked`.

### ADMIN-USER-05 Admin Reset Password
- `POST /admin/users/{id}/reset-password`
- `admin`

Request:
```json
{
  "send_invitation_email": true
}
```

Response:
```json
{
  "data": {
    "reset": true,
    "must_change_password": true
  },
  "message": "Temporary password has been reset"
}
```

## 6. Nhóm ADMIN-PROJECT

### ADMIN-PROJECT-01 List Projects
- `GET /admin/projects`
- `admin`
- Query: `keyword`, `status`, `billable_flag`, `page`, `per_page`

### ADMIN-PROJECT-02 Create Project
- `POST /admin/projects`
- `admin`

Request:
```json
{
  "project_code": "PJ-10963",
  "project_name": "WORK DESIGN PLATFORM_2026/4",
  "status": "active",
  "billable_flag": true,
  "description": "Dự án WDP tháng 04/2026"
}
```

Rules:
- `project_code` unique.
- `status` thuộc `active|inactive|archived`.

### ADMIN-PROJECT-03 Update Project
- `PUT /admin/projects/{id}`
- `admin`

### ADMIN-PROJECT-04 Archive/Restore Project
- `PATCH /admin/projects/{id}/status`
- `admin`

Request ví dụ archive:
```json
{
  "status": "archived"
}
```

## 7. Nhóm ADMIN-ASSIGN

### ADMIN-ASSIGN-01 List Assignments by Employee
- `GET /admin/employee-projects?employee_id={id}`
- `admin`

Response gợi ý:
```json
{
  "data": {
    "employee_id": 101,
    "assigned_projects": [
      { "id": 10227, "project_code": "PJ-10227", "project_name": "KHAI HUNG CO LTD" }
    ]
  },
  "message": "Success"
}
```

### ADMIN-ASSIGN-02 Assign Projects
- `POST /admin/employee-projects/assign`
- `admin`

Request:
```json
{
  "employee_id": 101,
  "project_ids": [10227, 10963, 7451]
}
```

Response:
```json
{
  "data": {
    "assigned_count": 2,
    "skipped_count": 1,
    "skipped_project_ids": [7451]
  },
  "message": "Assignment updated"
}
```

Rules:
- Không tạo mapping trùng active.
- Chỉ cho assign project `active`.

### ADMIN-ASSIGN-03 Unassign Projects
- `POST /admin/employee-projects/unassign`
- `admin`

Request:
```json
{
  "employee_id": 101,
  "project_ids": [10963]
}
```

Rules:
- Thu hồi không xóa dữ liệu timesheet cũ.

## 8. Nhóm TIMESHEET (Employee)

### TS-01 List Timesheet by Month
- `GET /timesheets?month=YYYY-MM`
- `employee` (chỉ dữ liệu của chính mình)

Response gợi ý:
```json
{
  "data": {
    "month": "2026-04",
    "entries": [
      {
        "entry_id": 9012,
        "work_date": "2026-04-03",
        "total_hours": 7.0,
        "details": [
          { "id": 1, "project_id": 10227, "project_name": "KHAI HUNG CO LTD", "hours_worked": 2.5, "note": "..." }
        ]
      }
    ]
  },
  "message": "Success"
}
```

### TS-02 Create Timesheet Entry (entry + details)
- `POST /timesheets`
- `employee`

Request:
```json
{
  "work_date": "2026-04-03",
  "details": [
    {
      "project_id": 10227,
      "hours_worked": 2.5,
      "note": "Kiểm thử giao diện"
    },
    {
      "project_id": 10963,
      "hours_worked": 3.0,
      "note": "Regression test WDP"
    }
  ]
}
```

Rules:
- Không cho ngày tương lai.
- `hours_worked` cho phép số lẻ (0.25, 0.5...).
- Tổng giờ/ngày/user không vượt 24.
- Project phải thuộc danh sách assign active của user.
- Dùng transaction all-or-nothing cho MVP.

### TS-03 Update Timesheet Entry
- `PUT /timesheets/{entry_id}`
- `employee` (chỉ entry của chính mình)
- Cấu trúc body tương tự create, thay mới `details`.

### TS-04 Delete Timesheet Entry
- `DELETE /timesheets/{entry_id}`
- `employee` (chỉ entry của chính mình)

### TS-05 Quick Update Detail (tuỳ chọn)
- `PATCH /timesheet-details/{detail_id}`
- `employee`

Request:
```json
{
  "hours_worked": 1.75,
  "note": "Update note"
}
```

Ghi chú:
- Nếu backend chỉ dùng `PUT /timesheets/{entry_id}` để cập nhật toàn bộ thì có thể bỏ API này.

### TS-06 Get Assigned Projects for Timesheet Form
- `GET /timesheets/assigned-projects`
- `employee`

Response:
```json
{
  "data": [
    { "id": 10227, "project_code": "PJ-10227", "project_name": "KHAI HUNG CO LTD" },
    { "id": 10963, "project_code": "PJ-10963", "project_name": "WORK DESIGN PLATFORM_2026/4" }
  ],
  "message": "Success"
}
```

## 9. Nhóm PROFILE (Employee)

### PROFILE-01 Get My Profile
- `GET /me`
- `employee|admin`

### PROFILE-02 Update My Profile
- `PUT /me`
- `employee|admin`

Request:
```json
{
  "full_name": "Tran Thi Hoa",
  "phone": "0901234567",
  "avatar_url": "https://cdn.example.com/avatar-101.png"
}
```

Ghi chú:
- Không cho user tự đổi `role`, `status`.

## 10. Nhóm danh mục phụ (CAT)

### CAT-01 Departments
- `GET /admin/departments`
- `POST /admin/departments`
- `PUT /admin/departments/{id}`
- `DELETE /admin/departments/{id}`

### CAT-02 Positions
- `GET /admin/positions`
- `POST /admin/positions`
- `PUT /admin/positions/{id}`
- `DELETE /admin/positions/{id}`

### CAT-03 Work Types
- `GET /admin/work-types`
- `POST /admin/work-types`
- `PUT /admin/work-types/{id}`
- `DELETE /admin/work-types/{id}`

Quyền: tất cả `admin`.

## 11. Notification (nội bộ hệ thống)
- Không cung cấp API public cho FE ở MVP.
- Thực thi bằng scheduler + command nội bộ:
  - `php artisan timesheet:notify-missing`
- Chạy lúc `20:00 JST`, bỏ qua Thứ Bảy/Chủ Nhật.

## 12. Nội dung sẽ làm ở bản v2
- Chuẩn schema OpenAPI chi tiết cho từng endpoint (`components/schemas` đầy đủ).
- Chuẩn pagination response.
- Chuẩn error code business riêng (ví dụ `TS_001_FUTURE_DATE`).
- Nhóm REPORT API (admin dashboard, filter nâng cao, export CSV/Excel).
