# 📄 Error Code Reference – Checklist Framework

Generated at: `2025-07-08T04:31:25.478473Z`

## 🔖 Standardized Error Format

```json
{
  "errorCode": "VERSION_CONFLICT",
  "message": "Save checklist với version cũ hơn",
  "hint": "Reload checklist để lấy version mới nhất",
  "timestamp": "2025-07-08T04:31:25.478473Z",
  "traceId": "example-trace-id"
}
```

## ✅ AuthErrors

| Error Code | HTTP Status | Message | Hint |
|------------|--------------|---------|------|
| `INVALID_CREDENTIALS` | `401` | Email hoặc mật khẩu không đúng | Kiểm tra lại thông tin đăng nhập |
| `EMAIL_EXISTS` | `409` | Email đã tồn tại | Dùng email khác hoặc đăng nhập nếu đã có tài khoản |
| `INVALID_TOKEN` | `401` | Token không hợp lệ hoặc đã hết hạn | Vui lòng đăng nhập lại để nhận token mới |
| `MISSING_REFRESH_TOKEN` | `400` | Thiếu refresh token | Đảm bảo gửi kèm refresh token trong request |
| `TOKEN_REPLAYED` | `401` | Refresh token đã bị dùng lại | Đăng nhập lại để nhận token mới |
| `TOO_MANY_ATTEMPTS` | `429` | Gửi OTP hoặc xác thực quá nhiều lần | Thử lại sau vài phút |
| `WEAK_PASSWORD` | `400` | Mật khẩu yếu | Mật khẩu phải chứa chữ hoa, chữ thường và số |
| `USER_NOT_FOUND` | `404` | Không tìm thấy người dùng | Đảm bảo bạn đã đăng ký tài khoản đúng email |
| `EMAIL_NOT_VERIFIED` | `403` | Email chưa được xác thực | Kiểm tra email để xác thực tài khoản |

## ✅ OTPVerificationErrors

| Error Code | HTTP Status | Message | Hint |
|------------|--------------|---------|------|
| `INVALID_OTP` | `400` | OTP sai hoặc hết hạn | Bạn có thể yêu cầu gửi lại mã xác thực |
| `OTP_EXPIRED` | `400` | Mã OTP đã hết hạn | Yêu cầu gửi lại mã OTP mới |
| `OTP_ALREADY_USED` | `400` | Mã OTP đã được sử dụng | Vui lòng yêu cầu mã OTP mới |
| `OTP_TOO_MANY_RESEND` | `429` | Gửi lại OTP quá nhiều lần | Thử lại sau vài phút |
| `OTP_TOO_MANY_ATTEMPTS` | `429` | Nhập sai mã OTP quá nhiều lần | Tài khoản sẽ bị khóa tạm thời nếu tiếp tục |
| `OTP_BLOCKED` | `423` | Tài khoản bị khóa tạm thời do sai mã nhiều | Đợi một thời gian rồi thử lại hoặc liên hệ hỗ trợ |
| `OTP_NOT_FOUND` | `404` | Không tìm thấy mã OTP nào hợp lệ | Vui lòng yêu cầu gửi lại mã xác thực |

## ✅ ChecklistErrors

| Error Code | HTTP Status | Message | Hint |
|------------|--------------|---------|------|
| `VERSION_CONFLICT` | `409` | Save checklist với version cũ hơn | Reload checklist để lấy version mới nhất |
| `ACCESS_DENIED` | `403` | Không có quyền truy cập checklist này | Hãy kiểm tra lại quyền sở hữu hoặc đăng nhập tài khoản đúng |
| `CHECKLIST_NOT_FOUND` | `404` | Không tìm thấy checklist | ID checklist không tồn tại hoặc đã bị xóa |
| `MAX_CHECK_LIMIT_EXCEEDED` | `400` | Checklist đã có quá nhiều công việc | Không thể thêm công việc mới, đã vượt giới hạn cho phép |
| `CHECKLIST_CONTENT_INVALID` | `400` | Nội dung checklist không hợp lệ | Đảm bảo tiêu đề và nội dung checklist không trống hoặc sai định dạng |
| `CHECK_NOT_FOUND` | `404` | Không tìm thấy công việc trong checklist | ID công việc không tồn tại trong checklist |
| `CHECK_EMPTY` | `400` | Nội dung công việc không được để trống | Nhập nội dung công việc trước khi lưu |
