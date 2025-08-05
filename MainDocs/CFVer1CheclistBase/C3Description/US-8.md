# US-8: Merge Guest Checklist on Sign In

| Step | Component                     | Role                                                    | Layer                                                    |
| ---- | ----------------------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| 1    | `AuthController`              | Nhận callback hoặc trigger sau khi login thành công     | `Presentation/Controllers/AuthController`                |
| 2    | `GuestChecklistMergeModel`    | ViewModel gửi kèm GuestChecklistId (hoặc auto-trigger)  | `Presentation/ViewModels/Auth/GuestChecklistMergeModel`  |
| 3    | `MergeGuestChecklistInput`    | Input contract chứa userId, guestChecklist (or ID)      | `Application/Contracts/Auth/MergeGuestChecklistInput`    |
| 4    | `IMergeGuestChecklistUseCase` | Interface của UseCase                                   | `Application/Contracts/Auth/IMergeGuestChecklistUseCase` |
| 5    | `MergeGuestChecklistUseCase`  | Xử lý toàn bộ nghiệp vụ merge checklist                 | `Application/UseCases/Auth/MergeGuestChecklistUseCase`   |
| 6    | `GuestChecklistStorage`       | Truy xuất dữ liệu guest checklist từ local/session      | `Infrastructure/Local/ChecklistStorage/Query`            |
| 7    | `GuestCheckStorage`           | Truy xuất các check liên quan từ local                  | `Infrastructure/Local/CheckStorage/Query`                |
| 8    | `ChecklistFactory`            | Sinh checklist mới gắn với userId từ checklist guest    | `Application/Factories/ChecklistFactory`                 |
| 9    | `CheckFactory`                | Sinh lại check từ dữ liệu guest (có thể cần map lại ID) | `Application/Factories/CheckFactory`                     |
| 10   | `ChecklistName`               | Validate checklist name                                 | `Domain/ValueObjects/ChecklistName`                      |
| 11   | `Checklist`                   | Checklist entity đã gắn với user                        | `Domain/Entities/Checklist`                              |
| 12   | `Check`                       | Check entity từ checklist guest                         | `Domain/Entities/Check`                                  |
| 13   | `ChecklistRepository`         | Ghi checklist vào DB với owner là user                  | `Infrastructure/Persistence/ChecklistRepository/Command` |
| 14   | `CheckRepository`             | Ghi các check vào DB                                    | `Infrastructure/Persistence/CheckRepository/Command`     |
| 15   | `MergeGuestChecklistOutput`   | Output contract → success, checklistId, action taken    | `Application/Contracts/Auth/MergeGuestChecklistOutput`   |
| 16   | `MergeGuestChecklistDTO`      | DTO phản hồi client                                     | `Application/DTOs/Auth/MergeGuestChecklistDTO`           |

## Context

> Use case này xảy ra ngay sau Sign In (US-6) nếu hệ thống phát hiện:

>> Người dùng trước đó có checklist dạng Guest (trên local/session)

>> Sau khi login → hệ thống hỏi merge, hoặc merge tự động

> Nó cần access cả local data (guest) lẫn persistent data (user).
