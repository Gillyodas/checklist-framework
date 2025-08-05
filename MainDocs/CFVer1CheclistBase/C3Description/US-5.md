# US-5: Sign Up with Email

| Step | Component                | Role                                                            | Layer                                                     |
| ---- | ------------------------ | --------------------------------------------------------------- | --------------------------------------------------------- |
| 1    | `AuthController`         | Nhận HTTP request đăng ký từ FE                                 | `Presentation/Controllers/AuthController`                 |
| 2    | `SignUpModel`            | ViewModel chứa input email + password + confirm                 | `Presentation/ViewModels/Auth/SignUpModel`                |
| 3    | `SignUpInput`            | Contract cho UseCase                                            | `Application/Contracts/Auth/SignUpInput`                  |
| 4    | `ISignUpUseCase`         | Interface UseCase                                               | `Application/Contracts/Auth/ISignUpUseCase`               |
| 5    | `SignUpUseCase`          | Điều phối logic đăng ký, kiểm tra email, băm mật khẩu, lưu user | `Application/UseCases/Auth/SignUpUseCase`                 |
| 6    | `EmailAddress`           | VO kiểm tra định dạng email hợp lệ                              | `Domain/ValueObjects/EmailAddress`                        |
| 7    | `PlainPassword`          | VO validate password                                            | `Domain/ValueObjects/PlainPassword`                       |
| 8    | `User`                   | Entity người dùng                                               | `Domain/Entities/User`                                    |
| 9    | `PasswordHasher`         | Service băm mật khẩu                                            | `Domain/Services/PasswordHasher`                          |
| 10   | `UserRepository`         | Kiểm tra email tồn tại và lưu user                              | `Infrastructure/Persistence/UserRepository/Command+Query` |
| 11   | `ISendVerificationEmail` | Interface gửi email xác thực                                    | `Application/Interfaces/ISendVerificationEmail`           |
| 12   | `EmailSender`            | Gửi email chứa mã xác thực hoặc link                            | `Infrastructure/External/EmailSender`                     |
| 13   | `SignUpOutput`           | Output contract (status, message, next step)                    | `Application/Contracts/Auth/SignUpOutput`                 |
| 14   | `SignUpDTO`              | DTO trả về FE                                                   | `Application/DTOs/Auth/SignUpDTO`                         |
