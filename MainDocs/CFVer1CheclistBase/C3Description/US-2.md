# US-2: Edit Checklist as Guest or User

## Guest

| Step | Component                   | Role                                            | Layer                                                      |
| ---- | --------------------------- | ----------------------------------------------- | ---------------------------------------------------------- |
| 1    | `EditChecklistView`         | Giao diện chỉnh sửa checklist                   | `Presentation/EditChecklist/`                              |
| 2    | `EditChecklistEditor`       | Nhận input từ người dùng, trigger lưu checklist | `Presentation/EditChecklist/`                              |
| 3    | `EditChecklistInput`        | Input Contract của UseCase                      | `Application/Contracts/EditChecklist/`                     |
| 4    | `ChecklistFactory`          | Tạo bản checklist mới từ dữ liệu cũ + input     | `Application/Factories/ChecklistFactory`                   |
| 5    | `CheckFactory` (optional)   | Tạo các Check từ input mới nếu cần              | `Application/Factories/CheckFactory`                       |
| 6    | `GuestEditChecklistUseCase` | Điều phối logic chỉnh sửa checklist guest       | `Application/UseCases/Checklist/GuestEditChecklistUseCase` |
| 7    | `IGuestStorage`             | Interface cho guest storage layer               | `Application/Interfaces/`                                  |
| 8    | `GuestChecklistStorage`     | Truy xuất và lưu checklist local                | `Infrastructure/Local/ChecklistStorage/Command+Query`      |
| 9    | `GuestCheckStorage`         | Truy xuất và lưu các Check local                | `Infrastructure/Local/CheckStorage/Command+Query`          |
| 10   | `ChecklistName`             | VO: validate tên checklist                      | `Domain/ValueObjects/`                                     |
| 11   | `CheckName`                 | VO: validate từng check                         | `Domain/ValueObjects/`                                     |
| 12   | `Checklist`                 | Entity checklist đã chỉnh sửa                   | `Domain/Entities/Checklist`                                |
| 13   | `Check`                     | Entity từng mục Check đã chỉnh sửa              | `Domain/Entities/Check`                                    |
| 14   | `GuestEditChecklistOutput`  | Output contract từ UseCase                      | `Application/Contracts/EditChecklist/`                     |
| 15   | `GuestEditChecklistDTO`     | DTO phản hồi cho FE sau chỉnh sửa checklist     | `Application/DTOs/EditChecklist/`                          |

## User

| Step | Component                 | Role                                              | Layer                                                    |
| ---- | ------------------------- | ------------------------------------------------- | -------------------------------------------------------- |
| 1    | `ChecklistController`     | Nhận HTTP request chỉnh sửa checklist từ frontend | `Presentation/Controllers/ChecklistController`           |
| 2    | `EditChecklistModel`      | ViewModel input từ FE (PUT/PATCH request)         | `Presentation/ViewModels/Checklist/`                     |
| 3    | `EditChecklistInput`      | Input contract cho UseCase                        | `Application/Contracts/EditChecklist/`                   |
| 4    | `IEditChecklistUseCase`   | Interface UseCase cho phép test & DI              | `Application/Contracts/EditChecklist/`                   |
| 5    | `EditChecklistUseCase`    | Xử lý logic chỉnh sửa checklist                   | `Application/UseCases/Checklist/EditChecklistUseCase`    |
| 6    | `ChecklistRepository`     | Tìm checklist hiện tại theo id + user             | `Infrastructure/Persistence/ChecklistRepository/Query`   |
| 7    | `CheckRepository`         | Tìm các check hiện tại của checklist              | `Infrastructure/Persistence/CheckRepository/Query`       |
| 8    | `ChecklistFactory`        | Tạo bản checklist mới (copy/update state)         | `Application/Factories/ChecklistFactory`                 |
| 9    | `CheckFactory` (optional) | Tạo check từ input mới                            | `Application/Factories/CheckFactory`                     |
| 10   | `ChecklistName`           | VO: validate tên checklist                        | `Domain/ValueObjects/ChecklistName`                      |
| 11   | `CheckName`               | VO: validate tên từng check                       | `Domain/ValueObjects/CheckName`                          |
| 12   | `Checklist`               | Checklist entity được cập nhật                    | `Domain/Entities/Checklist`                              |
| 13   | `Check`                   | Check entity được cập nhật                        | `Domain/Entities/Check`                                  |
| 14   | `ChecklistRepository`     | Ghi lại checklist đã chỉnh sửa                    | `Infrastructure/Persistence/ChecklistRepository/Command` |
| 15   | `CheckRepository`         | Ghi lại check đã chỉnh sửa                        | `Infrastructure/Persistence/CheckRepository/Command`     |
| 16   | `EditChecklistOutput`     | Output contract từ UseCase                        | `Application/Contracts/EditChecklist/`                   |
| 17   | `EditChecklistDTO`        | DTO phản hồi trả về FE                            | `Application/DTOs/EditChecklist/`                        |
