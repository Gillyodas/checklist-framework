# US-6: Sign In with Email

| Step | Component        | Role                                                | Layer                                                         |
| ---- | ---------------- | --------------------------------------------------- | ------------------------------------------------------------- |
| 1    | `AuthController` | Nhận HTTP request đăng nhập từ FE                   | `Presentation/Controllers/AuthController`                     |
| 2    | `SignInModel`    | ViewModel chứa email, password từ người dùng        | `Presentation/ViewModels/Auth/SignInModel`                    |
| 3    | `SignInInput`    | Input contract truyền vào UseCase                   | `Application/Contracts/Auth/SignInInput`                      |
| 4    | `ISignInUseCase` | Interface của UseCase                               | `Application/Contracts/Auth/ISignInUseCase`                   |
| 5    | `SignInUseCase`  | Logic xử lý xác thực người dùng và sinh token       | `Application/UseCases/Auth/SignInUseCase`                     |
| 6    | `EmailAddress`   | VO kiểm tra định dạng email                         | `Domain/ValueObjects/EmailAddress`                            |
| 7    | `PlainPassword`  | VO validate password input                          | `Domain/ValueObjects/PlainPassword`                           |
| 8    | `UserRepository` | Tìm user theo email và lấy hashed password          | `Infrastructure/Persistence/UserRepository/Query`             |
| 9    | `PasswordHasher` | So sánh password input với hashed password trong DB | `Domain/Services/PasswordHasher`                              |
| 10   | `TokenGenerator` | Sinh JWT token sau khi xác thực thành công          | `Domain/Services/TokenGenerator` (hoặc `Infrastructure/Auth`) |
| 11   | `SignInOutput`   | Output của UseCase: token, user info, expiration    | `Application/Contracts/Auth/SignInOutput`                     |
| 12   | `SignInDTO`      | DTO trả về cho client: token, userId, role...       | `Application/DTOs/Auth/SignInDTO`                             |
