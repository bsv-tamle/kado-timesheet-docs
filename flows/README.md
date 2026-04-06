# Bộ Tài Liệu Flow Nghiệp Vụ

Thư mục này quản lý flow theo nguyên tắc: **mỗi flow là 1 file**.

## Quy ước đặt tên file
- `<FLOW-CODE>-<short-name>.md`
- Ví dụ: `FLOW-AUTH-01-login.md`

## Khung chuẩn cho mỗi flow
1. Mục tiêu
2. Vai trò tham gia
3. Điều kiện đầu vào
4. Kết quả đầu ra
5. Luồng chính (happy path)
6. Luồng thay thế và lỗi
7. Business rules
8. API mapping
9. Điểm cần test

## Danh sách flow hiện có

### Nhóm A - Authentication và Account
- `FLOW-AUTH-01-login.md`
- `FLOW-AUTH-02-forgot-password.md`
- `FLOW-AUTH-03-reset-password.md`
- `FLOW-AUTH-04-change-password.md`
- `FLOW-AUTH-05-logout.md`

### Nhóm B - Quản trị người dùng và phân quyền
- `FLOW-ADMIN-USER-01-create-user.md`
- `FLOW-ADMIN-USER-02-update-user.md`
- `FLOW-ADMIN-USER-03-lock-unlock-user.md`
- `FLOW-ADMIN-USER-04-admin-reset-password.md`

### Nhóm C - Quản trị project và assign
- `FLOW-ADMIN-PROJECT-01-create-project.md`
- `FLOW-ADMIN-PROJECT-02-update-project.md`
- `FLOW-ADMIN-PROJECT-03-archive-restore-project.md`
- `FLOW-ADMIN-ASSIGN-01-assign-project.md`
- `FLOW-ADMIN-ASSIGN-02-unassign-project.md`

### Nhóm D - Timesheet employee
- `FLOW-TS-01-list-timesheet.md`
- `FLOW-TS-02-create-timesheet-entry.md`
- `FLOW-TS-03-update-timesheet-entry.md`
- `FLOW-TS-04-delete-timesheet-entry.md`
- `FLOW-TS-05-grid-quick-edit.md`

### Nhóm E - Báo cáo và dashboard
- `FLOW-REPORT-01-admin-dashboard.md`
- `FLOW-REPORT-02-report-filter.md`
- `FLOW-REPORT-03-export-report.md`
- `FLOW-REPORT-04-employee-dashboard-report.md`

### Nhóm F - Danh mục phụ trợ
- `FLOW-CAT-01-department-crud.md`
- `FLOW-CAT-02-position-crud.md`
- `FLOW-CAT-03-work-type-crud.md`

### Nhóm G - Notification Slack
- `FLOW-NOTI-01-schedule-job.md`
- `FLOW-NOTI-02-select-targets.md`
- `FLOW-NOTI-03-send-slack-message.md`
