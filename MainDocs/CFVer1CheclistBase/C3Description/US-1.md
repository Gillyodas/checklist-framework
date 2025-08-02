# US-1: Create Checklist as Guest or User

## Guest

## User

| Step | Component                   | Role                                                      | Layer                                                    |
|------| ----------------------------|-----------------------------------------------------------|----------------------------------------------------------|
| 1    | `ChecklistController`       | Nhận HTTP request từ FE, map sang DTO                     | `Presentation/Controllers/ChecklistController`           |
| 2    | `CreateChecklistModel`      | ViewModel (đầu vào cho Controller)                        | `Presentation/ViewModels/`                               |
| 3    | `CreateChecklistInput`      | Input Contract của Use Case                               | `Application/Contracts/CreateChecklist`                  |
| 4    | `ICreateChecklistUseCase `  | Interface của create Checklist                            | `Application/Contracts/CreateChecklist`                  |
| 5    | `CreateChecklistUseCase`    | Xử lý logic chính cho use case Create Checklist           | `Application/UseCases/CreateChecklistUseCase`            |
| 6    | `UserRepository`            | Lấy thông tin người dùng (để check quota, id, plan, ...)  | `Infrastructure/Persistence/UserRepository/Query`        |
| 7    | `ChecklistName`             | VO: Validate tên checklist                                | `Domain/ValueObjects/ChecklistName`                      |
| 8    | `Checklist`                 | Entity: sinh ID, default fields, kiểm soát business rule  | `Domain/Entities/`                                       |
| 9    | `Check`                     | Entity: chứa từng mục nhỏ trong checklist                 | `Domain/Entities/`                                       |
| 10   | `ChecklistRepository`       | Lưu checklist vào DB                                      | `Infrastructure/Persistence/ChecklistRepository/Command` |
| 11   | `CheckRepository`           | Lưu các Check vào DB                                      | `Infrastructure/Persistence/CheckRepository/Command`     |
| 12   | `CreateChecklistOutput`     | Output contract của use case                              | `Application/Contracts/`                                 |
| 13   | `CreateChecklistDTO`        | DTO trả về FE                                             | `Application/DTOs/`                                      |