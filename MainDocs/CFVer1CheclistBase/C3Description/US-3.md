# US-3: View Checklist as Guest or User

## Guest

| Step | Component                   | Role                                                  | Layer                                                          |
| ---- | --------------------------- | ----------------------------------------------------- | -------------------------------------------------------------- |
| 1    | `ChecklistViewScreen`       | Giao diện hiển thị checklist cho guest                | `Presentation/ViewChecklist/ChecklistViewScreen`               |
| 2    | `ChecklistViewer`           | Logic lấy dữ liệu checklist guest và hiển thị         | `Presentation/ViewChecklist/ChecklistViewer`                   |
| 3    | `ViewChecklistInput`        | Contract đầu vào UseCase (chứa guestId + checklistId) | `Application/Contracts/ViewChecklist/`                         |
| 4    | `IGuestChecklistStorage`    | Interface truy xuất checklist cho guest               | `Application/Interfaces/Checklist/IGuestChecklistStorage`      |
| 5    | `GuestChecklistStorage`     | Truy xuất checklist từ localStorage                   | `Infrastructure/Local/ChecklistStorage/Query`                  |
| 6    | `IGuestCheckStorage`        | Interface truy xuất check cho guest                   | `Application/Interfaces/Checklist/IGuestCheckStorage`          |
| 7    | `GuestCheckStorage`         | Truy xuất check từ localStorage                       | `Infrastructure/Local/CheckStorage/Query`                      |
| 8    | `GuestViewChecklistUseCase` | Logic điều phối: lấy checklist + check                | `Application/UseCases/ViewChecklist/GuestViewChecklistUseCase` |
| 9    | `Checklist`                 | Checklist entity được dựng lại để hiển thị            | `Domain/Entities/Checklist`                                    |
| 10   | `Check`                     | Check entity được dựng lại để hiển thị                | `Domain/Entities/Check`                                        |
| 11   | `GuestViewChecklistOutput`  | Output của UseCase                                    | `Application/Contracts/ViewChecklist/`                         |
| 12   | `GuestChecklistDTO`         | DTO trả về cho FE                                     | `Application/DTOs/ViewChecklist/GuestChecklistDTO`             |

## User

| Step | Component               | Role                                                | Layer                                                       |
| ---- | ----------------------- | --------------------------------------------------- | ----------------------------------------------------------- |
| 1    | `ChecklistController`   | Nhận HTTP request từ FE (GET checklist/{id})        | `Presentation/Controllers/ChecklistController`              |
| 2    | `ViewChecklistModel`    | ViewModel input từ URL params / query (checklistId) | `Presentation/ViewModels/ViewChecklistModel`                |
| 3    | `ViewChecklistInput`    | Contract truyền vào UseCase                         | `Application/Contracts/ViewChecklist/`                      |
| 4    | `IViewChecklistUseCase` | Interface của UseCase để inject                     | `Application/Contracts/ViewChecklist/IViewChecklistUseCase` |
| 5    | `ViewChecklistUseCase`  | Logic: load checklist + check, verify ownership     | `Application/UseCases/ViewChecklist/ViewChecklistUseCase`   |
| 6    | `ChecklistRepository`   | Tìm checklist theo id + userId                      | `Infrastructure/Persistence/ChecklistRepository/Query`      |
| 7    | `CheckRepository`       | Tìm các check thuộc checklist đó                    | `Infrastructure/Persistence/CheckRepository/Query`          |
| 8    | `Checklist`             | Entity checklist được dựng lại từ DB                | `Domain/Entities/Checklist`                                 |
| 9    | `Check`                 | Entity check được dựng lại từ DB                    | `Domain/Entities/Check`                                     |
| 10   | `ViewChecklistOutput`   | Output thuần domain                                 | `Application/Contracts/ViewChecklist/ViewChecklistOutput`   |
| 11   | `ChecklistDTO`          | DTO trả về client                                   | `Application/DTOs/ViewChecklist/ChecklistDTO`               |

