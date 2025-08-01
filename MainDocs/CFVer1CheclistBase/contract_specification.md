# 📄 Contract Specification – Checklist Framework

Tài liệu này mô tả chi tiết các contract API giữa client và backend cho hệ thống Checklist Framework, bao gồm mục tiêu, request/response, validation rules và behavior production.

---

## ✅ Checklists APIs

3. Contract
✅ 3.1 PUT /checklists/:id
🎯 Mục tiêu
Save nội dung hiện tại của Checklist, có versioning và updateAt. Backend phải xử lý:
- Kiểm tra Checklist Owner.
- Lưu nội dung Checklist.
- Phản hồi kết quả (Nếu thành công trả về version mới nhất cho Checklist).
✅ Request
Endpoint: PUT /checklists/:id
Headers: Authorization: Bearer {token}
Query Param: auto | manual
Body:
{
	“checklistId”: “e5b08db3-9a3d-4f3f-b2b4-7efce9a21b63”,
	"updateAt": "2025-07-01T13:45:23.000Z"
	“currentVersion”: 3
	“checklistName” : “Example”,
	“description” : “This is example contract”,
	"checks": [
  {
    “checkId” : “e5b08db3-9a3d-4f3f-b2b4-7e234fds21b63”
    “content”: “Check 1”,
    “isChecked”: true,
    “note”: “Note”
   “isDeleted” : false
  }
]
}
✅ Contract Spec
Field	Loại	Bắt buộc?	Mô tả
checklistId	UUID	✅	ID Checklist
updateAt	ISO8601	✅	Thời điểm cập nhật
checklistName	string	❌	Tên Checklist
description	string	❌	Mô tả Checklist
checks[]	checks[]: array of Check	❌	Danh sách các Check của Checklist
content	string	✅	Nội dung của Check
isChecked	boolean	❌	Đã Check hay chưa
note	string	❌	Ghi chú của Check
currentVersion	int	✅	Phiên bản hiện tại của Checklist
checkId	 UUID	✅	ID Check
isDeleted	boolean	✅	Hỗ trọ delete Check cụ thể

✅ Validation Rules
Trường	Rule
currentVersion	- Client gửi ‘currentVersion’ (phiên bản đang hiển thị)
- Backend:
  • Nếu version hiện tại trong DB khác → 409 Conflict
  • Nếu khớp → chấp nhận, lưu version mới
updateAt	Phải sau lần cập nhật gần nhất, nếu không → 400
checklistId	Phải đúng quyền sở hữu (AuthZ)
checks[]	Tối đa 100 items / checklist
content	Không được để trống

✅ Response
200 OK
{
	“checklistId”: “e5b08db3-9a3d-4f3f-b2b4-7efce9a21b63”,
	“status”: “success”,
	“updateAt”: “2025-07-01T13:45:23.000Z”,
	“saveVersion”: 4
}
400 Bad Request for updateAt
{
	“error”: “updatedAt must be later than last save time.”
}
400 Bad Request for Version
{
	“error”: “Invalid version. Client version cannot be higher than server.”
}

409 Conflict
{
  “error”: “Version mismatch. Please reload checklist.”
}
403 Forbidden
{
  “error”: “Checklist does not belong to the user.”
}
422 Unprocessable for Content
{
  “error”: “Checklist content invalid.”
}
422 Unprocessable Entity
{
  “error”: “Missing required field: currentVersion”
}
500 Internal Server Error

3.2 GET /checklists
🎯 Mục tiêu
Lấy danh sách Checklist thuộc về người dùng hiện tại, không cần gửi userId, xác thực bằng JWT.
✅ Request
Endpoint: GET /checklists
Headers: Authorization: Bearer {token}
Body:
Không có body.
UserId được extract từ token.
✅ Response
200 OK
{
  "status": "success",
  "checklists": [
    {
      "checklistId": "e5b08db3-9a3d-4f3f-b2b4-7efce9a21b63",
      "checklistName": "Daily Tasks",
      "lastUpdatedAt": "2025-07-01T13:40:22.000Z",
      "version": 3
    },
    {
      "checklistId": "cfbde93e-1a2d-48bc-a1ef-cf8d9f238fba",
      "checklistName": "Project A",
      "lastUpdatedAt": "2025-06-29T10:12:00.000Z",
      "version": 1
    }
  ]
}
🔒 AuthZ Note:
•	Token phải hợp lệ (JWT, HMAC/SHA256, expiry check)
•	UserId được lấy từ sub claim trong token
•	Backend không chấp nhận query param hoặc body chứa userId

401 Unauthorized
{
  “error”: “Missing or invalid token.”
}
204 No Content
{
“status”: “204 No Content”
}
403 Forbidden
{
	“error”: “You are not allowed to access this resource.”
}
429 Too Many Requests
{
	“error”: “Rate limit exceeded. Try again later.”
}
📌 Hệ thống production có throttling/rate limiting thường áp dụng tại gateway hoặc API layer.
500 Internal Server Error
{
	“error”: “Something went wrong. Please try again later.”
}
📌 Phải log đầy đủ userId, traceId, stack trace phía server – không show ra client.
3.3 GET /checklists/:id
________________________________________
🎯 Mục tiêu
Trả về toàn bộ nội dung chi tiết của 1 checklist cụ thể của người dùng hiện tại.
Backend phải xử lý:
•	Kiểm tra quyền sở hữu Checklist (checklistId phải thuộc user hiện tại)
•	Truy vấn nội dung checklist và toàn bộ check
•	Trả về version hiện tại và updatedAt để hỗ trợ Auto Save + Manual Save phía frontend
________________________________________
✅ Request
•	Endpoint: GET /checklists/:id
•	Headers: Authorization: Bearer {token}
•	Path Param: :id = UUID của checklist
________________________________________
✅ Response – 200 OK
{
  "status": "success",
  "checklist": {
    "checklistId": "e5b08db3-9a3d-4f3f-b2b4-7efce9a21b63",
    "checklistName": "Example",
    "description": "This is example contract",
    "updatedAt": "2025-07-01T13:45:23.000Z",
    "version": 3,
    "checks": [
      {
        "checkId": "e5b08db3-9a3d-4f3f-b2b4-7e234fds21b63",
        "content": "Check 1",
        "isChecked": true,
        "note": "Note",
        "isDeleted": false
      }
    ]
  }
}
________________________________________
✅ Contract Spec
Field	Loại	Bắt buộc?	Mô tả
checklistId	UUID	✅	ID duy nhất của checklist
checklistName	string	✅	Tên checklist
description	string	❌	Mô tả checklist
updatedAt	ISO8601	✅	Thời điểm cập nhật gần nhất
version	int	✅	Phiên bản hiện tại
checks[]	array	✅	Danh sách các check
checkId	UUID	✅	ID của mỗi check
content	string	✅	Nội dung check
isChecked	boolean	❌	Trạng thái checked
note	string	❌	Ghi chú nếu có
isDeleted	boolean	✅	Cờ soft-delete cho check
________________________________________
✅ Validation Rules
Trường	Rule
checklistId	Phải đúng quyền sở hữu (check AuthZ)
checks[]	Trả tối đa 100 item – có thể phân trang ở version 2
content	Không được null hoặc chuỗi rỗng

________________________________________
✅ Response Error
404 Not Found – Không tìm thấy Checklist
{
  "error": "Checklist not found."
}
403 Forbidden – Checklist không thuộc user hiện tại
{
  "error": "Checklist does not belong to the user."
}
401 Unauthorized – Thiếu token hoặc token sai
{
  "error": "Missing or invalid token."
}
429 Too Many Requests – Quá giới hạn gọi API
{
  "error": "Rate limit exceeded. Try again later."
}
500 Internal Server Error
{
  "error": "Something went wrong. Please try again later."
}
✅ 3.4 DELETE /checklists/:id
________________________________________
🎯 Mục tiêu
Xoá một checklist cụ thể thuộc về người dùng hiện tại.
Backend cần đảm bảo:
•	Kiểm tra quyền sở hữu (AuthZ)
•	Thực hiện soft delete để giữ audit trail và tránh mất dữ liệu vĩnh viễn
•	Trả về trạng thái xoá
________________________________________
✅ Request
•	Endpoint: DELETE /checklists/:id
•	Headers: Authorization: Bearer {token}
•	Path Param: :id = UUID của checklist
•	Body: Không có
________________________________________
✅ Behavior mặc định:
•	Đây là Soft Delete → trường isDeleted = true, không xoá khỏi DB
•	Nếu checklist đã bị xoá trước đó, thao tác này là idempotent (xoá nhiều lần vẫn trả về 200)
________________________________________
✅ Response – 200 OK
{
  "status": "deleted",
  "checklistId": "e5b08db3-9a3d-4f3f-b2b4-7efce9a21b63",
  "deletedAt": "2025-07-02T10:30:00.000Z"
}
________________________________________
✅ Contract Spec
Field	Loại	Bắt buộc?	Mô tả
checklistId	UUID	✅	ID của checklist cần xoá
deletedAt	ISO8601	✅	Thời điểm hệ thống thực hiện soft delete
________________________________________
✅ Validation Rules
Trường	Rule
checklistId	Phải là checklist thuộc quyền sở hữu của user hiện tại (AuthZ)
isDeleted	Nếu đã xoá rồi → không thực hiện lại, nhưng vẫn trả 200
________________________________________
✅ Error Responses
404 Not Found – Checklist không tồn tại hoặc đã bị hard delete
{
  "error": "Checklist not found."
}
403 Forbidden – Checklist không thuộc user hiện tại
{
  "error": "Checklist does not belong to the user."
}
500 Internal Server Error
{
  "error": "Something went wrong. Please try again later."
}
✅ 3.5 POST /auth/signup
________________________________________
🎯 Mục tiêu
Cho phép người dùng tạo tài khoản mới với thông tin cơ bản.
Hệ thống phải đảm bảo:
•	Validate định dạng dữ liệu (email, mật khẩu, tên)
•	Kiểm tra email đã tồn tại hay chưa
•	Lưu tài khoản mới một cách an toàn (mã hóa mật khẩu)
•	Trả về JWT token để đăng nhập ngay nếu cần
________________________________________
✅ Request
•	Endpoint: POST /auth/signup
•	Headers: Content-Type: application/json
•	Body:
{
  "email": "user@example.com",
  "password": "SecurePass123!",
  "fullName": "Nguyen Van A"
}
________________________________________
✅ Response – 201 Created
{
  "status": "created",
  "userId": "d6f8c9f0-8c2b-4377-b8a9-f4a43259d81e",
  "email": "user@example.com",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6..."
}
________________________________________
✅ Contract Spec
Field	Type	Required	Mô tả
email	string	✅	Địa chỉ email, unique
password	string	✅	Mật khẩu phải đủ mạnh
fullName	string	✅	Tên đầy đủ của người dùng
________________________________________

✅ Validation Rules
Trường	Rule
email	Đúng định dạng RFC 5322, chưa tồn tại trong hệ thống
password	≥ 8 ký tự, phải có chữ thường + chữ hoa + số, nên có ký tự đặc biệt
fullName	Không để trống, tối đa 100 ký tự
________________________________________
❌ 400 Bad Request – Invalid Format
{
  "error": "Invalid email format."
}
❌ 409 Conflict – Email already exists
{
  "error": "Email already in use."
}



❌ 422 Unprocessable Entity – Weak password
{
  "error": "Password must include uppercase, lowercase, and number."
}
❌ 500 Internal Server Error
{
  "error": "Unable to process signup. Please try again later."
}
________________________________________
🔐 Bảo mật & Production Notes
Yếu tố	Giải thích
🔒 Mã hóa mật khẩu	Dùng bcrypt hoặc Argon2, không bao giờ lưu plaintext
🧾 Ghi log hợp lý	Log sự kiện tạo tài khoản (không log password)
🔐 JWT tạo ngay	Cho phép người dùng đăng nhập ngay sau khi tạo tài khoản (optional)
⛔ Giới hạn rate	Cần chống spam đăng ký bằng Rate Limiting hoặc Captcha nếu mở public API

✅ 3.6 POST /auth/login
🎯 Mục tiêu
Cho phép người dùng đã đăng ký đăng nhập vào hệ thống bằng email và mật khẩu.
Backend phải đảm bảo:
•	Kiểm tra tồn tại của tài khoản
•	So sánh mật khẩu một cách an toàn (hash)
•	Trả về JWT token nếu xác thực thành công
✅ Request
•	Endpoint: POST /auth/login
•	Headers: Content-Type: application/json
•	Body:
{
  "email": "user@example.com",
  "password": "SecurePass123!"
}
________________________________________
✅ Response – 200 OK
{
  "status": "success",
  "userId": "d6f8c9f0-8c2b-4377-b8a9-f4a43259d81e",
  "email": "user@example.com",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
________________________________________
✅ Contract Spec
Field	Type	Required	Mô tả
email	string	✅	Địa chỉ email đã đăng ký
password	string	✅	Mật khẩu trùng với người dùng
________________________________________
✅ Validation Rules
Trường	Rule
email	Phải đúng format RFC 5322
password	Không để trống, tối đa 100 ký tự
________________________________________
❌ 400 Bad Request – Dữ liệu thiếu hoặc format sai
{
  "error": "Invalid email or password format."
}
❌ 401 Unauthorized – Sai thông tin đăng nhập
{
  "error": "Incorrect email or password."
}
❌ 429 Too Many Requests – Login quá nhanh (rate limited)
{
  "error": "Too many login attempts. Please try again later."
}
❌ 500 Internal Server Error
{
  "error": "Unable to process login at the moment."
}
________________________________________
🔐 Bảo mật & Best Practice
Vấn đề	Cách xử lý an toàn
🔒 Không bao giờ log mật khẩu	Log email + IP + status thôi
⏱️ Rate Limit	Bảo vệ endpoint bằng throttling
🤐 Trả lỗi chung	Không nói rõ email sai hay password sai
🧂 Hash password	Bcrypt (>= 10 rounds) hoặc Argon2
🧾 Audit log	Ghi log sự kiện đăng nhập (thành công/thất bại)

✅ 3.7 POST /auth/refresh-token
🎯 Mục tiêu
Cho phép người dùng gia hạn phiên đăng nhập bằng cách sử dụng refresh token hợp lệ, thay vì phải đăng nhập lại bằng mật khẩu.
Backend cần đảm bảo:
•	Kiểm tra refresh token có hợp lệ và còn hạn hay không
•	Tạo access token (JWT) mới
•	Có thể cấp refresh token mới (tùy thiết kế)
________________________________________
✅ Flow kiến trúc chuẩn
User login → nhận: Access Token + Refresh Token
        ↓
Access Token hết hạn sau ~15 phút
        ↓
Client gửi Refresh Token → POST /auth/refresh-token
        ↓
Backend verify token → cấp Access Token mới (và có thể Refresh mới)
________________________________________
✅ Request
•	Endpoint: POST /auth/refresh-token
•	Headers: Content-Type: application/json
•	Body:
{
  "refreshToken": "fadc3c94-f87b-4d2f-a9b3-xxxxxxxxxxxx"
}
✅ Response – 200 OK
{
  "status": "success",
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "a8d3723a-3f12-4e9b-bf53-xxxxxxxxxxxx",  // nếu cấp mới
  "expiresIn": 900
}
________________________________________
✅ Contract Spec
Field	Type	Required	Mô tả
refreshToken	string (UUID)	✅	Mã định danh duy nhất của refresh token đã cấp
________________________________________
✅ Validation Rules	
Trường	Rule
refreshToken	✅ Phải là chuỗi UUID hợp lệ (theo định dạng v4)
refreshToken	✅ Không được để trống hoặc null
refreshToken	✅ Phải tồn tại trong hệ thống (DB/Redis)
refreshToken	✅ Phải còn hạn sử dụng (expiresAt > now())
refreshToken	✅ Không bị thu hồi (isRevoked == false)
refreshToken	⛔ Không thuộc user khác (nếu thiết kế gắn với userId)
refreshToken	✅ Không bị dùng lại (nếu dùng cơ chế rotate – single-use only)

📌 Giải thích từng rule:
Loại rule	Mục đích
Format validation	Bảo vệ API khỏi input rác hoặc tấn công injection
Existence check	Chỉ chấp nhận token do hệ thống cấp, đã được lưu trữ
Expiry check	Tránh việc token hết hạn vẫn được dùng (hạn refresh thường là 7–30 ngày)
Revocation check	Dùng để logout toàn bộ hoặc khi bị nghi ngờ rò rỉ
Rotation check	Nếu dùng cơ chế "mỗi refresh token chỉ được dùng 1 lần" thì phải xoá/mark used ngay sau khi cấp access token
________________________________________
❌ 401 Unauthorized – Token sai hoặc đã bị thu hồi
{
  "error": "Invalid or expired refresh token."
}
❌ 422 Unprocessable Entity – Thiếu token
{
  "error": "Missing refreshToken in request."
}
❌ 429 Too Many Requests
{
  "error": "Too many refresh attempts. Please wait."
}

✅ 3.8 POST /auth/logout
🎯 Mục tiêu
Cho phép người dùng đăng xuất khỏi hệ thống bằng cách thu hồi refresh token, từ đó vô hiệu hóa khả năng gia hạn phiên.
Access token vẫn hết hạn tự động, còn Refresh token sẽ bị thu hồi ngay.
________________________________________
✅ Request
•	Endpoint: POST /auth/logout
•	Headers:
o	Content-Type: application/json
o	Authorization: Bearer {accessToken}
•	Body:
{
  "refreshToken": "fadc3c94-f87b-4d2f-a9b3-xxxxxxxxxxxx"
}
✅ Response – 200 OK
{
  "status": "logged_out",
  "revokedAt": "2025-07-02T14:50:00.000Z"
}
________________________________________
✅ Contract Spec
Field	Type	Required	Mô tả
refreshToken	UUID	✅	Refresh token đã cấp cho phiên này
Authorization	Bearer	✅	Access token hiện tại để xác thực người dùng
________________________________________
✅ Validation Rules
Trường	Rule
refreshToken	Phải là UUID v4, không để trống
refreshToken	Phải tồn tại, chưa bị thu hồi, chưa hết hạn
userId	Lấy từ access token → refreshToken phải thuộc cùng userId
________________________________________
✅ Behavior backend
•  Tìm refreshToken trong DB/Redis
•  Kiểm tra: có tồn tại, còn hạn, chưa revoked
•  Đánh dấu revoked = true, lưu revokedAt
•  Log lại event logout
________________________________________
❌ 401 Unauthorized – Thiếu access token hoặc sai
{
  "error": "Missing or invalid access token."
}
❌ 404 Not Found – Token không tồn tại
{
  "error": "Refresh token not found."
}
❌ 409 Conflict – Token đã bị logout trước đó (optional)
{
  "error": "Token already revoked."
}
✅ Server-side Notes
Tác vụ	Giải thích
🔒 Revoke token	Đảm bảo token không thể tiếp tục refresh được nữa
🧾 Log logout	Ghi log userId, IP, refreshToken, thời gian logout
💥 Revoke toàn bộ	(tuỳ chọn) nếu muốn logout khỏi mọi thiết bị – xóa toàn bộ token userId đó
🚫 Không xóa access token	Access token vẫn có thể còn dùng được nếu chưa hết hạn, và hệ thống không cần revoke vì stateless

✅ 3.9 GET /auth/profile
🎯 Mục tiêu
Cung cấp thông tin người dùng hiện tại từ token.
Backend sẽ:
•	Giải mã Access Token (Bearer JWT)
•	Truy vấn user từ DB (nếu cần)
•	Trả về thông tin cơ bản để frontend hiển thị UI (tên, email, vai trò…)
________________________________________
✅ Request
•	Endpoint: GET /auth/profile
•	Headers:
o	Authorization: Bearer {accessToken}
________________________________________
✅ Response – 200 OK
{
  "userId": "d6f8c9f0-8c2b-4377-b8a9-f4a43259d81e",
  "email": "user@example.com",
  "fullName": "Nguyen Van A",
  "role": "user",
  "createdAt": "2025-07-01T13:00:00.000Z"
}
________________________________________
✅ Contract Spec
Field	Type	Required	Mô tả
userId	UUID	✅	ID người dùng, lấy từ token/DB
email	string	✅	Email
fullName	string	✅	Tên đầy đủ
role	string	✅	user / admin (tùy hệ thống)
createdAt	ISO8601	✅	Thời gian tạo tài khoản
________________________________________
✅ Validation Rules
Trường	Rule
Authorization header	✅ Bắt buộc, phải theo format Bearer {token}
token	✅ Phải là JWT hợp lệ, decode được (có signature đúng)
exp (token)	✅ Phải chưa hết hạn (exp > now())
sub (token)	✅ Phải tồn tại → ánh xạ được userId trong hệ thống
userId	✅ Phải tìm thấy trong DB (nếu cần DB query, không dùng token 100%)
user.status	✅ Không được là banned, inactive, deleted (tuỳ business rule)
________________________________________
✅ Backend Logic
AccessToken → middleware decode → get userId
↓
Truy vấn DB (optional nếu claim đủ)
↓
Trả về thông tin: userId, email, fullName, role, createdAt
________________________________________

❌ 401 Unauthorized – Thiếu hoặc sai token
{
  "error": "Missing or invalid access token."
}
❌ 404 Not Found – Token hợp lệ nhưng không tìm thấy user
{
  "error": "User not found."
}
________________________________________
🔐 Bảo mật & Best Practice
Thực hành tốt	Lý do
✅ Lấy thông tin từ token	Không phụ thuộc vào frontend
✅ Không cho sửa ở endpoint này	Đây là read-only
✅ Chỉ gửi thông tin cần thiết	Tránh leak dữ liệu nhạy cảm
✅ Gắn thêm exp, iat nếu frontend cần hiển thị thời gian	
✅ 3.10 POST /auth/change-password
🎯 Mục tiêu
Cho phép người dùng đang đăng nhập đổi mật khẩu.
Backend phải đảm bảo:
•	Xác thực đúng người dùng từ access token
•	Xác minh đúng currentPassword
•	Đảm bảo newPassword đủ mạnh và không trùng với mật khẩu cũ
•	Cập nhật mật khẩu an toàn (bằng hash), ghi log bảo mật
________________________________________
✅ Request
•	Endpoint: POST /auth/change-password
•	Headers:
o	Authorization: Bearer {accessToken}
o	Content-Type: application/json
•	Body:
{
  "currentPassword": "OldPass123!",
  "newPassword": "NewSecurePass456!"
}
✅ Response – 200 OK
{
  "status": "password_changed",
  "changedAt": "2025-07-02T15:35:00.000Z"
}
________________________________________
✅ Contract Spec
Field	Type	Required	Mô tả
currentPassword	string	✅	Mật khẩu hiện tại (để xác minh trước khi đổi)
newPassword	string	✅	Mật khẩu mới
Authorization	Bearer	✅	Access token để xác thực người dùng hiện tại
________________________________________
✅ Validation Rules
Trường	Rule
currentPassword	✅ Phải đúng với mật khẩu đã hash của user hiện tại
newPassword	✅ Không trùng với currentPassword
newPassword	✅ Tối thiểu 8 ký tự, có chữ hoa, chữ thường, số, ký tự đặc biệt
token (Authorization)	✅ Phải là JWT hợp lệ, còn hạn
userId (from token)	✅ Phải tìm thấy trong DB, không bị ban/khóa
________________________________________
❌ 400 Bad Request – Format sai
{
  "error": "New password must be different from current password."
}
❌ 401 Unauthorized – Token thiếu hoặc sai
{
  "error": "Missing or invalid access token."
}
❌ 403 Forbidden – Mật khẩu hiện tại sai
{
  "error": "Current password is incorrect."
}
❌ 422 Unprocessable Entity – Mật khẩu mới yếu
{
  "error": "New password must be at least 8 characters, include uppercase, lowercase, number."
}
❌ 500 Internal Server Error
{
  "error": "Unable to change password at the moment."
}
________________________________________
🔐 Production Behavior
Xử lý	Chi tiết
🧂 Hash new password	Bcrypt (>= 10 rounds) hoặc Argon2
🧾 Ghi log sự kiện	Log userId, IP, time (không log password!)
🔄 Optional: Revoke token	Nếu muốn logout mọi phiên đang hoạt động sau khi đổi mật khẩu

✅ 3.11 POST /auth/reset-password-request
🎯 Mục tiêu
Khi người dùng quên mật khẩu, họ nhập email.
Hệ thống sẽ:
•	Kiểm tra email có tồn tại
•	Tạo mã OTP hoặc token reset
•	Gửi email khôi phục (1 link chứa token hoặc mã số)
________________________________________
✅ Request
•	Endpoint: POST /auth/reset-password-request
•	Headers: Content-Type: application/json
•	Body:
{
  "email": "user@example.com"
}	
✅ Response – 200 OK (luôn trả về giống nhau)
{
  "status": "email_sent"
}
________________________________________

✅ Contract Spec
Field	Type	Required	Mô tả
email	string	✅	Email người dùng đã đăng ký
________________________________________
✅ Validation Rules
Trường	Rule
email	✅ Định dạng hợp lệ (RFC 5322)
email	✅ Nếu không tồn tại → vẫn trả 200 (không lộ user tồn tại)
________________________________________
❌ 422 – Format lỗi
{
  "error": "Invalid email format."
}
________________________________________
🔐 Production Behavior
Xử lý	Mục đích
🧾 Log IP + email gửi	Giúp phát hiện spam
📤 Gửi email có token	Gửi link dạng /reset-password?token=abcd...
🔐 Token dạng JWT / UUID, mã hóa, expiry ≤ 15 phút	Bảo mật cao
⏱️ Rate limit endpoint	Tránh brute-force spam email

✅ 3.12 POST /auth/reset-password-confirm
🎯 Mục tiêu
Người dùng truy cập link từ email (hoặc nhập mã OTP), đặt lại mật khẩu mới.
Hệ thống sẽ:
•	Xác thực token/mã đúng và còn hạn
•	Cập nhật mật khẩu mới
•	Thu hồi mọi session cũ nếu cần (optional)
________________________________________
✅ Request
•	Endpoint: POST /auth/reset-password-confirm
•	Headers: Content-Type: application/json
•	Body (token-based):
{
  "resetToken": "fadc3c94-f87b-4d2f-a9b3-xxxxxxxxxxxx",
  "newPassword": "NewSecurePass123!"
}
✅ Response – 200 OK
{
  "status": "password_reset",
  "resetAt": "2025-07-02T16:00:00.000Z"
}
________________________________________
✅ Contract Spec
Field	Type	Required	Mô tả
resetToken	string	✅	Token gửi trong email
newPassword	string	✅	Mật khẩu mới
________________________________________

✅ Validation Rules
Trường	Rule
resetToken	✅ Có trong DB hoặc decode được (JWT)
resetToken	✅ Chưa hết hạn, chưa bị dùng
newPassword	✅ Khác với mật khẩu cũ, đủ mạnh (≥8 ký tự, có số, chữ hoa…)
________________________________________
❌ 400 – Token đã dùng hoặc sai format
{
  "error": "Invalid or expired reset token."
}
❌ 422 – Mật khẩu mới yếu
{
  "error": "New password must be at least 8 characters, with uppercase, lowercase, and number."
}
________________________________________
🔐 Production Behavior
Xử lý	Mục đích
✅ So sánh mật khẩu cũ	Ngăn việc đặt lại y chang mật khẩu trước
🧾 Ghi log reset	UserId, IP, thời gian
🚫 Xoá hoặc vô hiệu hóa token	Ngăn dùng lại token
🔄 Thu hồi refresh token	Optional: logout mọi thiết bị khác


✅ 3.13 POST /auth/request-email-verification
🎯 Mục tiêu
Gửi mã OTP (One-Time Password) hoặc mã xác thực tới email người dùng để xác nhận rằng email là thật và đang hoạt động.
Được sử dụng cho:
•	Xác thực email trong quá trình đăng ký
•	Xác thực lại email cũ
•	Phòng chống spam / email ảo
________________________________________
✅ Request
•	Endpoint: POST /auth/request-email-verification
•	Headers: Content-Type: application/json
•	Body:
{
  "email": "user@example.com"
}
✅ Response – 200 OK (dù email có tồn tại hay không)
{
  "status": "verification_code_sent"
}
📌 Luôn trả thành công → không cho attacker biết email nào đã tồn tại trong hệ thống.
________________________________________
✅ Contract Spec
Field	Type	Required	Mô tả
email	string	✅	Email người dùng cần xác thực
________________________________________
✅ Validation Rules
Trường	Rule
email	✅ Format hợp lệ (RFC 5322)
email	✅ Có thể gửi lại nhiều lần, nhưng phải có rate limit
email	✅ Không được gửi quá N lần trong 1 giờ (tùy config)
________________________________________
❌ 422 – Invalid Format
{
  "error": "Invalid email format."
}
❌ 429 – Quá nhiều yêu cầu gửi OTP	
{
  "error": "Too many verification attempts. Try again later."
}
________________________________________
🔐 Backend Behavior
Tác vụ	Giải thích
✅ Tạo mã OTP random (6 chữ số hoặc token)	Dạng "124578" hoặc UUID
✅ Gửi qua email với template rõ ràng	"Your verification code is: 124578"
✅ Lưu vào DB (hoặc Redis) kèm TTL 5–10 phút	Có trạng thái unused, used, expired
✅ Tạo trace log	Ghi lại IP, email, thiết bị nếu cần
✅ Gắn rate-limit per IP/email	Tránh spam / abuse

✅ 3.14 POST /auth/verify-email-otp
________________________________________
🎯 Mục tiêu
Xác minh rằng người dùng đang sở hữu email đã đăng ký, thông qua việc nhập đúng mã OTP đã được gửi qua email.
Sau khi xác minh thành công, hệ thống sẽ:
•	Đánh dấu email là đã xác thực
•	Cho phép bước tiếp theo trong luồng đăng ký / reset password
________________________________________
✅ Request
•	Endpoint: POST /auth/verify-email-otp
•	Headers: Content-Type: application/json
•	Body:
{
  "email": "user@example.com",
  "otp": "124578"
}
✅ Response – 200 OK
{
  "status": "email_verified",
  "email": "user@example.com",
  "verifiedAt": "2025-07-02T16:15:00.000Z"
}
________________________________________
✅ Contract Spec
Field	Type	Required	Mô tả
email	string	✅	Email nhận mã OTP
otp	string	✅	Mã OTP gồm 6 số hoặc mã xác thực
________________________________________
✅ Validation Rules
Trường	Rule
email	✅ Format hợp lệ
otp	✅ Không để trống, độ dài 6 ký tự
otp	✅ Khớp với bản ghi chưa hết hạn trong DB
otp	✅ Trạng thái là pending, chưa bị used hoặc expired
otp	✅ Số lần thử không vượt quá giới hạn (3–5 lần)
________________________________________
❌ 400 – OTP sai hoặc đã hết hạn
{
  "error": "Invalid or expired verification code."
}
❌ 422 – Format sai
{
  "error": "Verification code must be 6 digits."
}
❌ 429 – Quá nhiều lần nhập sai
{
  "error": "Too many failed attempts. Please request a new code."
}
________________________________________
✅ Backend Behavior
Hành vi	Chi tiết
Truy DB với email + otp	Tìm bản ghi còn hiệu lực
Kiểm tra expiresAt	Mã OTP còn hạn sử dụng?
Kiểm tra status	pending hay đã used
Nếu đúng → đánh dấu là used	Ghi verifiedAt, đổi status = used
Nếu sai → tăng failedAttempts	Nếu quá 5 lần → blocked mã

✅ 3.15 POST /auth/resend-email-verification
🎯 Mục tiêu
Cho phép người dùng đã từng gửi OTP trước đó yêu cầu gửi lại mã OTP xác minh email, nếu chưa nhập hoặc bị mất.
________________________________________
✅ Request
•	Endpoint: POST /auth/resend-email-verification
•	Headers: Content-Type: application/json
•	Body:
{
  "email": "user@example.com"
}
✅ Response – 200 OK
{
  "status": "verification_code_resent",
  "resendCount": 2
}
________________________________________

✅ Contract Spec
Field	Type	Required	Mô tả
email	string	✅	Địa chỉ email cần gửi lại mã xác thực
________________________________________
✅ Validation Rules
Trường	Rule
email	✅ Format hợp lệ
email	✅ Phải tồn tại bản ghi pending OTP chưa dùng
resend count	✅ Không quá 3 lần trong 1 giờ
lastSentAt	✅ Phải cách lần gửi trước ≥ 30 giây
________________________________________
❌ 429 – Quá giới hạn
{
  "error": "You have reached the resend limit. Please try again later."
}
________________________________________
✅ Backend Behavior
•	Truy bản ghi OTP gần nhất trạng thái pending
•	Nếu hợp lệ → tạo OTP mới hoặc reuse OTP cũ (nếu chưa hết hạn)
•	Gửi lại email, log event type: RESEND_VERIFICATION
•	Tăng resendCount, cập nhật lastSentAt

Standardized error response
✅ Chuẩn structure
{
  "errorCode": "INVALID_OTP",
  "message": "The OTP you entered is incorrect or has expired.",
  "hint": "You can request a new verification code.",
  "timestamp": "2025-07-02T15:55:00Z",
  "traceId": "0eaf0d24-2f4d-4bc2-8b73-8b322cfd1531"
}
________________________________________
🔖 Convention đề xuất
Field	Type	Mô tả
errorCode	STRING	Mã lỗi chuẩn hoá (INVALID_TOKEN, EMAIL_EXISTS)
message	STRING	Thông báo người dùng có thể hiểu
hint	STRING?	Gợi ý hành động tiếp theo (tuỳ)
timestamp	ISO8601	Thời gian xảy ra lỗi
traceId	UUID	ID dùng để trace log backend (gắn vào logger)

---