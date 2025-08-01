# 📄 TraceId Design Specification – Checklist Framework

---

## 1. 🎯 Mục tiêu

Thiết kế cơ chế sinh – lan truyền – gắn traceId nhằm:
- Trace xuyên suốt UI → API → Service → DB → Logs → Response
- Hỗ trợ debugging, audit, và correlate log với lỗi production
- Trả traceId về client để hỗ trợ theo dõi từ phía người dùng

---

## 2. 📌 Ngữ cảnh sử dụng TraceId

TraceId được sử dụng xuyên suốt mọi loại usecase để trace hành vi và sự kiện:

| Nhóm Usecase            | Ví dụ cụ thể                                          | Mục tiêu Trace                                         |
|--------------------------|--------------------------------------------------------|--------------------------------------------------------|
| 🧍 Auth + Security        | SignUp, Login, Refresh, Logout, ChangePassword        | Trace failed login, token replay, brute-force          |
| 📋 Checklist Logic        | Create, Save, Delete, AutoSave Checklist              | Trace conflict, retry, race condition                 |
| ✅ Check Actions          | Add/Update/Delete Check, Check/Uncheck                | Trace từng hành vi trong checklist → bug trace        |
| 🔧 Background Processes   | AutoSave, Retry Save, Email Verification              | Trace debounce, retry logic                           |
| ⚠️ Error Handling         | 403, 409, 422, 500 trong bất kỳ API nào               | Correlate error traceId để debug và support           |

---

## 3. 🧬 Cơ chế sinh TraceId (FE vs BE)

| Nguồn Request         | Hành động                             | Quy ước TraceId                              |
|------------------------|----------------------------------------|-----------------------------------------------|
| ✅ Frontend (browser)  | Sinh `UUID v4` mới mỗi hành động      | Prefix `FE_` + UUID                           |
| ⛔ Không có traceId    | BE sinh mới nếu không có header       | Prefix `BE_` + UUID                           |
| ♻️ Internal API call   | Giữ nguyên traceId từ header          | Không đổi, truyền tiếp                        |

> TraceId luôn là UUID v4, gắn tiền tố FE_/BE_ để phân biệt source.

---

## 4. 🔗 TraceId Propagation Flow

```plaintext
FE → BE Middleware → Controller → Service → DB (event_logs)
     ↘               ↘               ↘
       Logger        ErrorResponse   Logs
```

| Giai đoạn        | Hành động kỹ thuật                                          |
|------------------|--------------------------------------------------------------|
| FE               | Gửi request với `X-Trace-Id: FE_{uuid}`                     |
| Middleware       | Đọc hoặc sinh traceId → gắn vào context + header            |
| Controller       | Đọc traceId từ context để enrich log + error response       |
| Service Layer    | Truyền traceId xuống tầng gọi DB hoặc service nội bộ        |
| Repository/DB    | Lưu traceId vào `event_logs`, `action_logs` nếu applicable   |
| Logger           | Dùng LogContext để gắn traceId tự động vào log              |
| Response         | Header và body error luôn trả traceId                       |

> ⚠️ Retry logic như AutoSave cần giữ nguyên traceId cũ, không tạo mới.

---

## 5. 🧱 Header Convention

| Header         | Ý nghĩa                   | Bắt buộc? |
|----------------|----------------------------|-----------|
| X-Trace-Id     | TraceId xuyên hệ thống     | ✅         |

- BE chỉ chấp nhận `UUID v4` có prefix `FE_` hoặc `BE_`
- Nếu không hợp lệ → sinh mới và log cảnh báo

---

## 6. 🛠 Middleware Design (ASP.NET Core)

### ✳ TraceIdMiddleware

```csharp
public class TraceIdMiddleware
{
    private readonly RequestDelegate _next;
    public TraceIdMiddleware(RequestDelegate next) => _next = next;

    public async Task InvokeAsync(HttpContext context)
    {
        var traceId = context.Request.Headers["X-Trace-Id"].FirstOrDefault();
        if (string.IsNullOrWhiteSpace(traceId) || !Regex.IsMatch(traceId, "^(FE|BE)_[a-f0-9-]{36}$"))
        {
            traceId = $"BE_{Guid.NewGuid()}";
        }

        context.Items["TraceId"] = traceId;
        context.Response.Headers["X-Trace-Id"] = traceId;

        using (LogContext.PushProperty("TraceId", traceId))
        {
            await _next(context);
        }
    }
}
```

> Middleware phải được đặt trước mọi logic xử lý để đảm bảo mọi log đều có traceId.

---

## 7. 🪵 Logger Integration

- Dùng Serilog + `LogContext.PushProperty("TraceId", ...)`
- Gắn vào tất cả log line: INFO, WARN, ERROR
- Structured JSON log example:

```json
{
  "timestamp": "2025-07-04T15:35:42.102Z",
  "level": "INFO",
  "traceId": "FE_4c2b7e00-82f5-4bb3-81a4-6fcf0533c24e",
  "userId": "6ab7b010-499b-4cb3-bcf5-b4f0c8c36a6f",
  "action": "SAVE_CHECKLIST",
  "message": "Checklist saved successfully",
  "metadata": {
    "checklistId": "8f0dfbac-b510-4dfc-92a6-d672c33cb512",
    "saveVersion": 4,
    "durationMs": 381
  }
}
```

---

## 8. 🗃 DB Integration (event_logs)

| Field       | Type   | Ghi chú |
|-------------|--------|---------|
| trace_id    | UUID   | FE_/BE_ + UUID |
| user_id     | UUID   | Ai thực hiện hành vi |
| event_type  | string | e.g. LOGIN_FAILED, CHECKLIST_SAVE |
| context     | JSONB  | Payload tùy event |
| created_at  | timestamptz | |

→ TraceId dùng làm khóa chính để tìm lại mọi log liên quan event bất thường.

---

## 9. 📤 Error Response Format

Luôn trả traceId về FE khi có lỗi:

```json
{
  "errorCode": "VERSION_CONFLICT",
  "message": "Checklist của bạn đã cũ. Vui lòng tải lại.",
  "traceId": "BE_237c2f4f-9170-4e13-8f64-248b71cccf63"
}
```

→ FE nên hiển thị traceId nếu cần support từ backend hoặc CS.

---

## 10. 🔐 Best Practices

| Tình huống                         | Xử lý đề xuất |
|------------------------------------|----------------|
| FE gửi traceId sai định dạng       | BE sinh mới và log cảnh báo |
| Retry AutoSave                     | Giữ traceId gốc ban đầu |
| Background Job                     | Tự sinh BE_ traceId riêng |
| Unhandled Exception                | Bắt buộc log + trả traceId trong 500 |
| Không gắn traceId trong error log  | CI/CD test fail – enforced bằng test log |

---

> ✅ TraceId là nền tảng để observability có thể hoạt động. Nếu bạn không kiểm soát propagation, bạn không thể debug production một cách có hệ thống.

