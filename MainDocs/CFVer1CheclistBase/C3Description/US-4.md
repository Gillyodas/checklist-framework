# US-4: Manage Checklist Items Guest or User

## Guest

| Step | Component                 | Role                                                          | Layer                                                      |
| ---- | ------------------------- | ------------------------------------------------------------- | ---------------------------------------------------------- |
| 1    | `ChecklistController`     | Nhận HTTP request cho add/update/toggle/delete check          | `Presentation/Controllers/ChecklistController`             |
| 2    | `ManipulateCheckModel`    | ViewModel input cho từng action (PUT, PATCH, DELETE, POST...) | `Presentation/ViewModels/ManipulateCheck/`                 |
| 3    | `ManipulateCheckInput`    | Input contract truyền vào UseCase                             | `Application/Contracts/ManipulateCheck/`                   |
| 4    | `IManipulateCheckUseCase` | Interface UseCase                                             | `Application/Contracts/ManipulateCheck/`                   |
| 5    | `ManipulateCheckUseCase`  | Điều phối logic chỉnh check: add/update/toggle/delete         | `Application/UseCases/Checklist/ManipulateCheckUseCase`    |
| 6    | `ChecklistRepository`     | Load checklist theo checklistId + userId                      | `Infrastructure/Persistence/ChecklistRepository/Query`     |
| 7    | `CheckRepository`         | Load, save hoặc xóa các check                                 | `Infrastructure/Persistence/CheckRepository/Command+Query` |
| 8    | `CheckFactory` (opt)      | Sinh mới hoặc copy check                                      | `Application/Factories/CheckFactory`                       |
| 9    | `Checklist`               | Checklist entity, quản lý danh sách check                     | `Domain/Entities/Checklist`                                |
| 10   | `Check`                   | Check entity được tạo hoặc update                             | `Domain/Entities/Check`                                    |
| 11   | `CheckName`               | VO validate tên check                                         | `Domain/ValueObjects/CheckName`                            |
| 12   | `ManipulateCheckOutput`   | Output logic của UseCase                                      | `Application/Contracts/ManipulateCheck/`                   |
| 13   | `ChecklistDTO`            | DTO checklist phản hồi về client sau khi thay đổi             | `Application/DTOs/ViewChecklist/ChecklistDTO`              |

## User

| Step | Component                 | Role                                                          | Layer                                                      |
| ---- | ------------------------- | ------------------------------------------------------------- | ---------------------------------------------------------- |
| 1    | `ChecklistController`     | Nhận HTTP request cho add/update/toggle/delete check          | `Presentation/Controllers/ChecklistController`             |
| 2    | `ManipulateCheckModel`    | ViewModel input cho từng action (PUT, PATCH, DELETE, POST...) | `Presentation/ViewModels/ManipulateCheck/`                 |
| 3    | `ManipulateCheckInput`    | Input contract truyền vào UseCase                             | `Application/Contracts/ManipulateCheck/`                   |
| 4    | `IManipulateCheckUseCase` | Interface UseCase                                             | `Application/Contracts/ManipulateCheck/`                   |
| 5    | `ManipulateCheckUseCase`  | Điều phối logic chỉnh check: add/update/toggle/delete         | `Application/UseCases/Checklist/ManipulateCheckUseCase`    |
| 6    | `ChecklistRepository`     | Load checklist theo checklistId + userId                      | `Infrastructure/Persistence/ChecklistRepository/Query`     |
| 7    | `CheckRepository`         | Load, save hoặc xóa các check                                 | `Infrastructure/Persistence/CheckRepository/Command+Query` |
| 8    | `CheckFactory` (opt)      | Sinh mới hoặc copy check                                      | `Application/Factories/CheckFactory`                       |
| 9    | `Checklist`               | Checklist entity, quản lý danh sách check                     | `Domain/Entities/Checklist`                                |
| 10   | `Check`                   | Check entity được tạo hoặc update                             | `Domain/Entities/Check`                                    |
| 11   | `CheckName`               | VO validate tên check                                         | `Domain/ValueObjects/CheckName`                            |
| 12   | `ManipulateCheckOutput`   | Output logic của UseCase                                      | `Application/Contracts/ManipulateCheck/`                   |
| 13   | `ChecklistDTO`            | DTO checklist phản hồi về client sau khi thay đổi             | `Application/DTOs/ViewChecklist/ChecklistDTO`              |
