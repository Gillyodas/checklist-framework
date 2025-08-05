# US-1: Create Checklist as Guest or User

## Guest

| Step | Component                    | Role                                                      | Layer                                                    |
|------|------------------------------|-----------------------------------------------------------|----------------------------------------------------------|
| 1    | `EditChecklistView`          | UI hiển thị cho người dùng                                | `Presentation/EditChecklist/`                            |
| 2    | `CreateChecklistEditor`      | Nhận input từ người dùng                                  | `Presentation/CreateChecklist/`                          |
| 3    | `CreateChecklistInput`       | Input Contract của UseCase                                | `Application/Contracts/CreateChecklist/`                 |
| 4    | `ChecklistFactory`           | Factory của Checklist                                     | `Application/Factories/`                                 |
| 5    | `CheckFactory` (Opt)         | Factory của Check                                         | `Application/Factories/`                                 |
| 6    | `GuestCreateChecklistUseCase`| Xử lý logic chính cho UseCase Create Checklist            | `Application/UseCases/`                                  |
| 7    | `IGuestStorage`              | Iterface cho Guest Storage                                | `Application/Interfaces/`                                |
| 8    | `GuestMetaStorage`           | Lấy thông tin người dùng (để check quota, id, plan, ...)  | `Infrastructure/Local/GuestStorage/Query`                |
| 9    | `GuestDTO`                   | DTO thông tin Guest                                       | `Application/DTOs/`                                      |
| 10   | `ChecklistName`              | VO: Validate tên Checklist                                | `Domain/ValueObjects/`                                   |
| 11   | `CheckName`                  | VO: Validate tên Check                                    | `Domain/ValueObjects/`                                   |
| 12   | `Checklist`                  | Entity: sinh ID, default fields, kiểm soát business rule  | `Domain/Entities/`                                       |
| 13   | `Check`                      | Entity: chứa từng mục nhỏ trong checklist                 | `Domain/Entities/`                                       |
| 14   | `GuestCreateChecklistOutput` | Output Contract của UseCase                               | `Application/Contracts/CreateChecklist/`                 |
| 15   | `GuestCreateChecklistDTO`    | DTO Guest create Checklist                                | `Application/DTOs/`                                      |
| 16   | `IChecklistStorage`          | Iterface cho Checklist Storage                            | `Application/Interfaces/`                                |
| 17   | `GuestChecklistStorage`      | Lưu Checklist vào LocalStorage                            | `Infrastructure/Local/ChecklistStorage/Command`          |
| 18   | `GuestCheckStorage`          | Lưu Check vào LocalStorage                                | `Infrastructure/Local/CheckStorage/Command`              |


## User

| Step | Component                 | Role                                                   | Layer                                                    |
| ---- | ------------------------- | ------------------------------------------------------ | -------------------------------------------------------- |
| 1    | `ChecklistController`     | Nhận HTTP request từ frontend                          | `Presentation/Controllers/ChecklistController`           |
| 2    | `CreateChecklistModel`    | ViewModel nhận input từ UI (POST body)                 | `Presentation/ViewModels/Checklist/`                     |
| 3    | `CreateChecklistInput`    | Input Contract cho UseCase                             | `Application/Contracts/CreateChecklist/`                 |
| 4    | `ICreateChecklistUseCase` | Interface UseCase cho phép injection và test           | `Application/Contracts/CreateChecklist/`                 |
| 5    | `CreateChecklistUseCase`  | Điều phối nghiệp vụ create checklist                   | `Application/UseCases/Checklist/CreateChecklistUseCase`  |
| 6    | `ChecklistFactory`        | Sinh entity Checklist từ input                         | `Application/Factories/ChecklistFactory`                 |
| 7    | `CheckFactory` (optional) | Tạo các Check từ text list                             | `Application/Factories/CheckFactory`                     |
| 8    | `ChecklistName`           | VO: validate tên checklist                             | `Domain/ValueObjects/ChecklistName`                      |
| 9    | `CheckName`               | VO: validate từng check nếu cần                        | `Domain/ValueObjects/CheckName`                          |
| 10   | `Checklist`               | Entity đại diện cho checklist                          | `Domain/Entities/Checklist`                              |
| 11   | `Check`                   | Entity cho từng mục trong checklist                    | `Domain/Entities/Check`                                  |
| 12   | `UserRepository`          | Truy vấn thông tin user hiện tại (quota, id, email...) | `Infrastructure/Persistence/UserRepository/Query`        |
| 13   | `ChecklistRepository`     | Ghi Checklist xuống DB                                 | `Infrastructure/Persistence/ChecklistRepository/Command` |
| 14   | `CheckRepository`         | Ghi các Check xuống DB                                 | `Infrastructure/Persistence/CheckRepository/Command`     |
| 15   | `CreateChecklistOutput`   | Output của UseCase (dạng logic)                        | `Application/Contracts/CreateChecklist/`                 |
| 16   | `CreateChecklistDTO`      | DTO trả về cho FE (response HTTP body)                 | `Application/DTOs/CreateChecklistDTO`                    |