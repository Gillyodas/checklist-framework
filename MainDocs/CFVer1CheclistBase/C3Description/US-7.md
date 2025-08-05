# US-7: Sign Out Securely

| Step | Component                  | Role                                                    | Layer                                            |
| ---- | -------------------------- | ------------------------------------------------------- | ------------------------------------------------ |
| 1    | `AuthController`           | Nhận HTTP request đăng xuất từ frontend                 | `Presentation/Controllers/AuthController`        |
| 2    | `SignOutModel`             | ViewModel chứa thông tin (ex: token, sessionId) nếu cần | `Presentation/ViewModels/Auth/SignOutModel`      |
| 3    | `SignOutInput`             | Contract đầu vào UseCase                                | `Application/Contracts/Auth/SignOutInput`        |
| 4    | `ISignOutUseCase`          | Interface cho SignOutUseCase                            | `Application/Contracts/Auth/ISignOutUseCase`     |
| 5    | `SignOutUseCase`           | Logic xử lý đăng xuất: xóa token/session                | `Application/UseCases/Auth/SignOutUseCase`       |
| 6    | `ISessionStore` (opt)      | Interface lưu phiên người dùng nếu dùng session token   | `Application/Interfaces/Auth/ISessionStore`      |
| 7    | `SessionStore`             | Xóa token/session theo user/sessionId/token             | `Infrastructure/Auth/SessionStore`               |
| 8    | `IRefreshTokenStore` (opt) | Nếu dùng refresh token → cần xóa token ra khỏi storage  | `Application/Interfaces/Auth/IRefreshTokenStore` |
| 9    | `RefreshTokenStore`        | Xóa hoặc revoke refresh token                           | `Infrastructure/Auth/RefreshTokenStore`          |
| 10   | `SignOutOutput`            | Output của UseCase: status, message                     | `Application/Contracts/Auth/SignOutOutput`       |
| 11   | `SignOutDTO`               | DTO phản hồi về FE: message, redirect, status           | `Application/DTOs/Auth/SignOutDTO`               |
