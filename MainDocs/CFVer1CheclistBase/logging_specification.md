# Logging Specification – Checklist Framework

## 1. 🎯 Mục tiêu

Logging không chỉ để debug lỗi, mà là nền tảng cho việc:

- Trace hành vi người dùng, phân tích hành vi bất thường
- Audit trail để phục vụ bảo mật, forensic
- Tối ưu hiệu năng, detect race condition, retry
- Hỗ trợ APM, error monitoring, alerting

## 2. 🔍 Logging Context

| Thành phần         | Mô tả                                                |
|--------------------|-------------------------------------------------------|
| `traceId`          | UUID duy nhất mỗi request, xuyên suốt hệ thống       |
| `userId`           | ID người dùng nếu đã đăng nhập                       |
| `checklistId`      | (tùy context) – liên kết hành vi đến dữ liệu chính   |
| `action`           | Hành động chính: `SAVE_CHECKLIST`, `LOGIN_SUCCESS`  |
| `source`           | Gọi từ đâu: `WebApp`, `BackgroundJob`, etc.         |

## 3. 📄 Log Format (JSON Structured Log)

```json
{
  "timestamp": "2025-07-04T16:12:32.410Z",
  "level": "INFO",
  "traceId": "98fc4241-8cd5-4f41-9e99-f0cc7b104de7",
  "userId": "2af0aa0c-90ab-4e9f-95e3-78e4580a12d5",
  "action": "SAVE_CHECKLIST",
  "message": "Checklist saved successfully",
  "metadata": {
    "checklistId": "deac51a1-33db-4b0d-badc-40e708b9f28e",
    "saveVersion": 4,
    "durationMs": 423
  }
}
```

## 4. 📈 Logging Levels & Policy

| Level   | Khi nào dùng                               | Có traceId? | Sink                |
|---------|---------------------------------------------|-------------|----------------------|
| DEBUG   | Trong dev, log chi tiết                     | ✅           | Local only           |
| INFO    | Hành vi thành công, mutation quan trọng     | ✅           | Event Log            |
| WARN    | Bất thường, timeout, version conflict       | ✅           | App Log              |
| ERROR   | Exception, crash, bug logic                 | ✅           | App Log + Error Sink |

## 5. 🗂️ Log Sink (Where to log)

| Loại Log         | Mô tả                                        |
|------------------|-----------------------------------------------|
| App Log          | Console hoặc file `logs/app.log`              |
| Event Log        | DB table `event_logs`                         |
| Error Log        | Hệ thống như Sentry, ELK, Seq, Loki           |

## 6. 🧩 Enrichment Strategy

- `traceId`: gắn qua middleware, xuyên suốt
- `userId`: từ JWT middleware
- `requestPath`, `method`, `statusCode`: từ HttpContext
- `latencyMs`: nếu đo được thời gian xử lý request

### Tool gợi ý:

- ASP.NET Core + Serilog
- Dùng `UseSerilogRequestLogging()`
- `Enrich.FromLogContext()` để tự động inject `traceId`, `userId`

## 7. 🎯 Business Events to Log

| Event                        | Level | Metadata cần log                                  |
|-----------------------------|-------|--------------------------------------------------|
| Đăng ký tài khoản           | INFO  | email, userId, IP                                |
| Đăng nhập / thất bại        | INFO  | email, IP, success/fail                          |
| SAVE_CHECKLIST              | INFO  | checklistId, version, durationMs                 |
| VERSION_CONFLICT            | WARN  | checklistId, expectedVersion, currentVersion     |
| DELETE_CHECKLIST            | INFO  | checklistId, userId                              |
| Exception không xử lý       | ERROR | message, stackTrace, request info, traceId       |
| Auto Save trigger           | DEBUG | debounce lý do gọi save                          |

## 8. 🧱 event_logs Schema (DB Table)

```sql
CREATE TABLE event_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  trace_id UUID NOT NULL,
  user_id UUID,
  action TEXT NOT NULL,
  message TEXT,
  metadata JSONB,
  source TEXT,
  level TEXT CHECK (level IN ('DEBUG', 'INFO', 'WARN', 'ERROR')),
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_event_logs_trace_id ON event_logs(trace_id);
CREATE INDEX idx_event_logs_user_id ON event_logs(user_id);
```

## 9. 🔁 TraceId Propagation Design

| Tầng        | Cơ chế                                                                 |
|-------------|-------------------------------------------------------------------------|
| UI          | Tạo `traceId` đầu tiên (uuid v4), gửi kèm mỗi request vào header       |
| API Gateway | Forward `traceId` header sang backend (hoặc sinh mới nếu chưa có)     |
| Backend     | Middleware đọc traceId → inject vào logger context (Serilog/ILogger)   |
| DB Log      | Các log quan trọng insert vào `event_logs` có kèm `traceId`            |
| Response    | Backend trả `traceId` vào header để FE debug được                      |

### ASP.NET Core Middleware Mẫu (pseudo-code)

```csharp
app.Use(async (context, next) => {
  var traceId = context.Request.Headers["X-Trace-Id"].FirstOrDefault() ?? Guid.NewGuid().ToString();
  context.Items["TraceId"] = traceId;
  LogContext.PushProperty("TraceId", traceId);
  context.Response.Headers["X-Trace-Id"] = traceId;
  await next();
});
```

## 10. 🧠 Ghi nhớ

> Log = UI của backend developer
> 
> Bạn không debug bằng code, bạn debug bằng log – nếu log đủ tốt thì mới debug được production bug.

