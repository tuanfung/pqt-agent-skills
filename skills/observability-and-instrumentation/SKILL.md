---
name: observability-and-instrumentation
description: Thiết lập instrumentation cho code để hành vi trong môi trường production có thể quan sát và chẩn đoán được. Sử dụng khi thêm logging, metrics, tracing, hoặc alerting. Sử dụng khi triển khai bất kỳ tính năng nào chạy trong production và bạn cần minh chứng rằng nó hoạt động. Sử dụng khi các sự cố production được báo cáo nhưng bạn không thể biết chuyện gì đã xảy ra từ dữ liệu hiện có.
---

# Khả năng quan sát (Observability) và Thiết lập đo lường (Instrumentation)

## Tổng quan

Code mà bạn không thể quan sát là code mà bạn không thể vận hành. Observability là khả năng trả lời câu hỏi "hệ thống đang làm gì và tại sao?" từ bên ngoài, bằng cách sử dụng telemetry mà code phát ra. Instrumentation không phải là một tiện ích bổ sung sau khi ra mắt — nó được viết song song với tính năng, tương tự như cách viết test. Nếu một tính năng được triển khai mà không có telemetry, bug đầu tiên do người dùng báo cáo sẽ trở thành một cuộc "khảo cổ học" thay vì một câu truy vấn đơn giản.

## Khi nào nên sử dụng

- Xây dựng bất kỳ tính năng nào sẽ chạy trong production
- Thêm một service, endpoint, background job, hoặc tích hợp bên ngoài mới
- Một sự cố production mất quá nhiều thời gian để chẩn đoán ("chúng tôi không thể biết chuyện gì đã xảy ra")
- Thiết lập hoặc xem xét các quy tắc alerting
- Xem xét một PR có thêm I/O, retries, queues, hoặc các cuộc gọi cross-service

**KHÔNG dùng cho:**
- Chẩn đoán một lỗi đang xảy ra ngay lúc này — hãy sử dụng skill `debugging-and-error-recovery` (observability là thứ giúp skill đó hoạt động nhanh hơn vào lần tới)
- Profiling và tối ưu hóa sự chậm chạp đã được đo lường — hãy sử dụng skill `performance-optimization`
- Danh sách kiểm tra giám sát ngày ra mắt và các trigger rollback — xem skill `shipping-and-launch`; skill này bao gồm phần instrumentation cung cấp dữ liệu cho chúng

## Quy trình

### 1. Xác định thế nào là "hoạt động" trước khi thiết lập instrumentation

Telemetry mà không có câu hỏi đi kèm thì chỉ là nhiễu. Trước khi thêm bất kỳ instrumentation nào, hãy viết ra 2–4 câu hỏi mà một kỹ sư on-call sẽ đặt ra về tính năng này:

```
TÍNH NĂNG: checkout payment retry
CÂU HỎI ON-CALL SẼ ĐẶT RA:
1. Tỷ lệ thanh toán thành công trong lần thử đầu tiên so với sau khi retry là bao nhiêu?
2. Khi một khoản thanh toán thất bại vĩnh viễn, tại sao? (lỗi provider? timeout? validation?)
3. Provider thanh toán có chậm hơn bình thường không?
→ Mọi tín hiệu dưới đây phải giúp trả lời một trong những câu hỏi này.
```

Nếu bạn không thể nêu tên các câu hỏi, bạn chưa sẵn sàng để thiết lập instrumentation — bạn sẽ log mọi thứ và không học được gì cả.

### 2. Chọn tín hiệu (signal) phù hợp cho mỗi câu hỏi

| Tín hiệu | Trả lời | Đặc điểm chi phí | Ví dụ |
|---|---|---|---|
| **Structured log** | "Điều gì đã xảy ra trong trường hợp cụ thể này?" | Theo từng sự kiện; tăng theo lưu lượng | `payment_failed` kèm mã lỗi của provider |
| **Metric** | "Tần suất / tốc độ, tính tổng thể?" | Cố định theo mỗi series; truy vấn rẻ | độ trễ p99 của các cuộc gọi provider |
| **Trace** | "Thời gian tiêu tốn ở đâu giữa các service?" | Theo mỗi request; thường được lấy mẫu (sampled) | Một lượt checkout chậm, được phân tích chi tiết theo từng chặng (hop) |

Quy tắc chung: metrics cho bạn biết **rằng** có gì đó sai, traces cho bạn biết **ở đâu**, và logs cho bạn biết **tại sao**.

### 3. Structured logging

Log các sự kiện, không log dạng văn xuôi. Mỗi dòng log là một đối tượng JSON với tên sự kiện ổn định và các trường (fields) máy có thể đọc được:

```typescript
// XẤU: string interpolation — không thể truy vấn, không nhất quán
logger.info(`Payment ${id} failed for user ${userId} after ${n} retries`);

// TỐT: tên sự kiện ổn định + các trường có cấu trúc
logger.warn({
  event: 'payment_failed',
  paymentId: id,
  provider: 'stripe',
  errorCode: err.code,
  attempt: n,
}, 'payment failed');
```

**Các mức log (Log levels) — hãy sử dụng chúng một cách nhất quán:**

| Mức | Ý nghĩa | Hành động của on-call |
|---|---|---|
| `error` | Vi phạm bất biến (invariant broken); có thể cần ai đó can thiệp | Điều tra |
| `warn` | Bị suy giảm nhưng đã được xử lý (retry thành công, sử dụng fallback) | Theo dõi xu hướng |
| `info` | Sự kiện kinh doanh quan trọng (đã đặt hàng, job đã hoàn thành) | Không có |
| `debug` | Chi tiết chẩn đoán | Mặc định tắt trong production |

**Correlation ID là bắt buộc.** Hãy tạo (hoặc tiếp nhận) một request ID tại ranh giới hệ thống và gắn nó vào mọi dòng log, span, và cuộc gọi outbound. Nếu không có nó, bạn không thể tái dựng lại một request duy nhất từ các log bị trộn lẫn:

```typescript
// Express: child logger cho mỗi request, ID được truyền xuống dưới
app.use((req, res, next) => {
  req.id = req.headers['x-request-id'] ?? crypto.randomUUID();
  req.log = logger.child({ requestId: req.id });
  res.setHeader('x-request-id', req.id);
  next();
});
```

**Tuyệt đối không log secrets, tokens, mật khẩu, hoặc toàn bộ PII.** Đây là quy tắc nghiêm ngặt từ skill `security-and-hardening` — các pipeline telemetry là đường dẫn rò rỉ dữ liệu điển hình. Hãy sử dụng allowlist cho các trường; đừng log toàn bộ body của request.

### 4. Metrics

Đối với các service hướng request, hãy thiết lập **RED** cho mọi endpoint và mọi dependency bên ngoài: **R**ate (số request/giây), **E**rrors (tỷ lệ lỗi), **D**uration (histogram độ trễ, không dùng trung bình). Đối với các tài nguyên (queues, pools, hosts), hãy sử dụng **USE**: **U**tilization (mức độ sử dụng), **S**aturation (mức độ bão hòa), **E**rrors (lỗi).

Tương tự như tracing, hướng tiếp cận trung lập với nhà cung cấp (vendor-neutral) là OpenTelemetry metrics API (cùng SDK và context như bước 5). Ví dụ dưới đây sử dụng `prom-client` của Prometheus — một lựa chọn backend phổ biến, không phải duy nhất; các quy tắc RED/USE và cardinality là giống nhau trong cả hai trường hợp.

```typescript
import { Histogram } from 'prom-client';

const httpDuration = new Histogram({
  name: 'http_request_duration_seconds',
  help: 'HTTP request duration',
  labelNames: ['method', 'route', 'status_class'],  // '2xx', không phải '200'
  buckets: [0.05, 0.1, 0.25, 0.5, 1, 2.5, 5],
});
```

**Cardinality là điểm dễ gây lỗi.** Mỗi tổ hợp nhãn (label) duy nhất là một time series riêng biệt. Các nhãn phải đến từ các tập hợp nhỏ, cố định (route template, status class, tên provider). Tuyệt đối không sử dụng user ID, URL thô, thông báo lỗi, hoặc các giá trị không giới hạn khác làm nhãn — những thứ đó thuộc về logs và traces.

```
OK khi làm nhãn:    route="/api/tasks/:id"   status_class="5xx"   provider="stripe"
KHÔNG BAO GIỜ làm nhãn:  user_id, email, request_id, full URL, văn bản thông báo lỗi
```

Đừng bao giờ theo dõi số trung bình, hãy luôn dùng phân vị (percentiles): số trung bình sẽ che giấu 1% người dùng đang gặp trải nghiệm tồi tệ. Hãy sử dụng histograms và đọc p50/p95/p99.

### 5. Distributed tracing

Sử dụng OpenTelemetry — đây là tiêu chuẩn trung lập với nhà cung cấp, và auto-instrumentation hỗ trợ HTTP, gRPC, và các DB client phổ biến với hầu như không cần viết code:

```typescript
// tracing.ts — phải được import trước bất kỳ thứ gì khác
import { NodeSDK } from '@opentelemetry/sdk-node';
import { getNodeAutoInstrumentations } from '@opentelemetry/auto-instrumentations-node';

const sdk = new NodeSDK({
  serviceName: 'checkout-service',
  instrumentations: [getNodeAutoInstrumentations()],
});
sdk.start();
```

Chỉ thêm manual spans xung quanh các đơn vị công việc nội bộ có ý nghĩa (ví dụ: `applyDiscounts`, `chargeProvider`) và gắn các attribute mà kỹ sư on-call sẽ dùng để lọc. Truyền context qua mọi ranh giới async — HTTP headers, metadata của queue message — nếu không trace sẽ bị ngắt quãng. Mặc định lấy mẫu (sample) dạng head-based với tỷ lệ thấp; giữ lại 100% lỗi nếu backend của bạn hỗ trợ tail sampling.

### 6. Alerting

Cảnh báo dựa trên **triệu chứng người dùng cảm nhận**, không dựa trên nguyên nhân:

```
TRIỆU CHỨNG (xứng đáng page):           NGUYÊN NHÂN (dashboard, không page):
error rate > 1% trong 5 phút             CPU ở mức 85%
p99 latency > 2s                        một pod được khởi động lại
queue age > 10 phút                      disk ở mức 70%
```

Các cảnh báo dựa trên nguyên nhân thường kích hoạt khi không có gì sai và bỏ lỡ những thất bại mà bạn không dự đoán trước. Các cảnh báo dựa trên triệu chứng kích hoạt chính xác khi người dùng bị ảnh hưởng, bất kể nguyên nhân là gì.

Quy tắc cho mọi cảnh báo bạn tạo ra:

1. **Phải có hành động cụ thể.** Nếu phản ứng là "cứ lờ đi, nó sẽ tự hồi phục", hãy xóa cảnh báo đó.
2. **Phải có link tới runbook** — dù chỉ là ba dòng: ý nghĩa của cảnh báo, câu truy vấn đầu tiên cần chạy, và quy trình báo cáo cấp trên (escalation path).
3. **Phải có ngưỡng (threshold) và thời gian duy trì (duration)** được căn cứ theo SLO hoặc dữ liệu lịch sử, không phải do đoán.
4. Chỉ sử dụng hai mức độ nghiêm trọng: **page** (ảnh hưởng người dùng, hành động ngay) và **ticket** (suy giảm chất lượng, hành động trong tuần này). Một mức thứ ba sẽ trở thành nhiễu và khiến mọi người quen với việc lờ đi tất cả.

### 7. Xác minh chính telemetry

Instrumentation cũng là code; nó có thể sai. Trước khi coi như hoàn thành, hãy kích hoạt các luồng xử lý và xem kết quả thực tế:

- Gây ra lỗi trong môi trường staging → tìm nó trong logs bằng `requestId`, xác nhận các trường đã được cấu trúc (không phải là `[object Object]`)
- Gửi traffic thử nghiệm → xác nhận các metric series xuất hiện với các nhãn mong đợi và giá trị hợp lý
- Theo dõi một request đi qua các service trong giao diện tracing → không có span nào bị đứt
- Kích hoạt mỗi cảnh báo mới một lần (hạ thấp ngưỡng tạm thời) → xác nhận nó gửi đến đúng channel và link runbook hoạt động

## Những lời bào chữa phổ biến

| Lời bào chữa | Thực tế |
|---|---|
| "Tôi sẽ thêm logging sau khi nó hoạt động" | "Sau đó" sẽ trở thành "sau sự cố đầu tiên", thời điểm đắt đỏ nhất để nhận ra bạn đang bị mù thông tin. Hãy thiết lập instrumentation khi bạn xây dựng. |
| "Càng nhiều logs = càng tăng khả năng quan sát" | Nhiễu không cấu trúc khiến việc xử lý sự cố chậm hơn, không phải nhanh hơn. Ba sự kiện có thể truy vấn tốt hơn ba trăm dòng văn xuôi. |
| "console.log là ổn trong lúc này" | Đầu ra không cấu trúc không thể lọc, tương quan hóa hoặc dùng để cảnh báo. Việc thiết lập structured logger chỉ tốn thêm năm phút một lần duy nhất. |
| "Chúng ta chỉ cần nhìn vào dashboard khi có gì đó hỏng" | Các dashboard được xây dựng mà không có câu hỏi xác định sẽ cho bạn thấy mọi thứ trừ câu trả lời. Hãy bắt đầu từ các câu hỏi on-call. |
| "Cảnh báo mọi thứ quan trọng, chúng ta sẽ tinh chỉnh sau" | Một thiết bị báo tin ồn ào sẽ khiến mọi người quen với việc lờ nó đi. Việc tinh chỉnh sẽ không bao giờ diễn ra; nhưng một lần bỏ lỡ cảnh báo thực sự thì chắc chắn sẽ xảy ra. |
| "Dùng User ID làm nhãn cho metric giúp debug dễ dàng hơn" | Nó cũng khiến backend metrics của bạn bị sập. Truy vấn cardinality cao thuộc về logs và traces. |
| "Tracing là quá mức cần thiết cho hai service của chúng ta" | Hai service đã có nghĩa là có những câu hỏi về độ trễ cross-service mà logs không thể trả lời. Auto-instrumentation khiến chi phí trở nên không đáng kể. |

## Dấu hiệu cảnh báo (Red Flags)

- Một PR tính năng có retries, queues, hoặc các cuộc gọi bên ngoài mà không có telemetry mới
- Các dòng log được tạo bằng string interpolation thay vì các trường có cấu trúc
- Không có correlation/request ID — mỗi dòng log là một "kẻ mồ côi"
- Metrics được gắn nhãn bằng user ID, URL thô, hoặc văn bản thông báo lỗi (bom cardinality)
- Độ trễ được theo dõi dưới dạng số trung bình mà không có phân vị (percentiles)
- Các cảnh báo kích hoạt hàng ngày và được xác nhận mà không có hành động khắc phục
- Cảnh báo dựa trên nguyên nhân (CPU, bộ nhớ) làm phiền con người trong khi tỷ lệ lỗi hướng người dùng không được giám sát
- Secrets, tokens, hoặc toàn bộ request body xuất hiện trong logs
- "Nó chạy trên máy tôi" là minh chứng duy nhất cho thấy một tính năng trong production đang khỏe mạnh

## Xác minh

Sau khi thiết lập instrumentation cho một tính năng, hãy xác nhận:

- [ ] Các câu hỏi on-call cho tính năng này đã được viết ra, và mỗi tín hiệu đều tương ứng với một câu hỏi
- [ ] Tất cả đầu ra log đều có cấu trúc (JSON), với tên sự kiện ổn định và có correlation ID trên mỗi dòng
- [ ] Không có secrets, tokens, hoặc PII chưa được che mờ trong bất kỳ dòng log nào (kiểm tra ngẫu nhiên đầu ra thực tế)
- [ ] Các metric RED tồn tại cho mọi endpoint mới và mọi dependency bên ngoài, với tập hợp nhãn bị giới hạn
- [ ] Độ trễ là một histogram; p95/p99 có thể truy vấn được
- [ ] Một request duy nhất có thể được theo dõi từ đầu đến cuối trong giao diện tracing mà không bị đứt span
- [ ] Mọi cảnh báo mới đều dựa trên triệu chứng, có link runbook và đã được chạy thử một lần
- [ ] Một lỗi cố ý tạo ra trong staging đã được định vị chỉ bằng telemetry, mà không cần đọc source code

Để xem phiên bản tóm tắt của danh sách này, bao gồm cả chốt chặn instrumentation trước khi ra mắt, hãy xem `references/observability-checklist.md`.
