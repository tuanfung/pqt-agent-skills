# Danh sách Kiểm tra Khả năng Quan sát (Observability Checklist)

Tài liệu tham khảo nhanh để thiết lập công cụ quan sát cho mã nguồn production. Sử dụng cùng với skill `observability-and-instrumentation`.

## Mục lục

- [Các Câu hỏi cho Nhân viên On-Call (Bắt đầu tại đây)](#on-call-questions-start-here)
- [Logging Cấu trúc (Structured Logging)](#structured-logging)
- [Chỉ số (Metrics)](#metrics)
- [Truy vết Phân tán (Distributed Tracing)](#distributed-tracing)
- [Cảnh báo (Alerting)](#alerting)
- [Bảng điều khiển (Dashboards)](#dashboards)
- [Xác minh Telemetry](#verify-the-telemetry)
- [Cửa kiểm soát Trước khi Ra mắt (Pre-Launch Gate)](#pre-launch-gate)

## Các Câu hỏi cho Nhân viên On-Call (Bắt đầu tại đây)

Telemetry mà không có câu hỏi đi kèm thì chỉ là nhiễu. Trước khi thiết lập công cụ quan sát:

- [ ] Đã viết ra 2–4 câu hỏi mà một kỹ sư on-call sẽ đặt ra về tính năng này
- [ ] Mọi tín hiệu bên dưới đều tương ứng với một trong những câu hỏi đó
- [ ] Mỗi câu hỏi được khớp với loại tín hiệu phù hợp: metrics cho biết **điều gì** sai, traces cho biết sai **ở đâu**, logs cho biết **tại sao**

## Logging Cấu trúc (Structured Logging)

- [ ] Logs có cấu trúc (JSON) với tên event ổn định — không phải là các chuỗi ký tự tự do
- [ ] Mỗi dòng log đều mang một correlation/request ID, được tạo hoặc chấp nhận tại biên hệ thống
- [ ] Correlation ID được truyền đi trong mọi lời gọi outbound và biên bất đồng bộ (HTTP headers, metadata của queue)
- [ ] Các mức độ log (log levels) nhất quán: `error` = vi phạm bất biến (invariant), cần có người can thiệp; `warn` = bị suy giảm nhưng đã được xử lý; `info` = sự kiện kinh doanh quan trọng; `debug` = tắt trong môi trường production
- [ ] Không có secrets, tokens, mật khẩu, hoặc PII chưa được ẩn danh trong bất kỳ dòng log nào (quy tắc nghiêm ngặt từ `security-and-hardening`)
- [ ] Các trường (fields) nằm trong danh sách cho phép (allowlist) — không bao gồm toàn bộ nội dung request/response, không có auth headers
- [ ] Các lời gọi dịch vụ bên ngoài chỉ được log với metadata: endpoint, status, latency, số lần thử (attempt count), các định danh đã được làm sạch
- [ ] Kiểm tra xác suất đầu ra log thực tế: các trường có cấu trúc, không phải là `[object Object]`

## Chỉ số (Metrics)

- [ ] Triển khai **RED** cho mọi endpoint và mọi phụ thuộc bên ngoài: Rate, Errors, Duration
- [ ] Triển khai **USE** cho mọi tài nguyên (queues, pools, hosts): Utilization, Saturation, Errors
- [ ] Latency là một histogram; p50/p95/p99 có thể truy vấn — tuyệt đối không dùng giá trị trung bình
- [ ] Tất cả các labels đều đến từ các tập hợp nhỏ, cố định (route template, status class, provider name)
- [ ] Không có giá trị label không giới hạn: không dùng user IDs, tenant IDs, emails, raw URLs, request IDs, hoặc nội dung thông báo lỗi
- [ ] Các mã trạng thái (status codes) được nhóm theo class (`5xx`, không phải `503`)
- [ ] Độ sâu hàng đợi (queue depth) và thời gian xử lý được theo dõi cho mọi worker/queue

## Truy vết Phân tán (Distributed Tracing)

- [ ] OpenTelemetry (hoặc tương đương) được khởi tạo khi dịch vụ bắt đầu, trước các import khác
- [ ] Kích hoạt auto-instrumentation cho các HTTP, gRPC và DB clients
- [ ] Trace context được truyền đi trong mọi lời gọi outbound (W3C `traceparent`/`tracestate`) và được trích xuất từ mọi request inbound
- [ ] Context tồn tại qua các biên bất đồng bộ — các tin nhắn trong queue mang theo trace metadata
- [ ] Chỉ tạo manual spans xung quanh các đơn vị công việc nội bộ có ý nghĩa, với các attributes mà nhân viên on-call sẽ dùng để lọc
- [ ] Không dùng secrets hoặc PII làm span attributes
- [ ] Áp dụng head-based sampling với tỷ lệ mặc định thấp; giữ lại 100% lỗi nếu có tail sampling

## Cảnh báo (Alerting)

- [ ] Mọi cảnh báo đều dựa trên triệu chứng (symptom-based: error rate, p99 latency, queue age) — các nguyên nhân (CPU, disk, restarts) sẽ đưa lên dashboards, không đưa vào pagers
- [ ] Mọi cảnh báo đều phải có hành động khắc phục; các cảnh báo kiểu "cứ lờ đi, nó sẽ tự phục hồi" cần bị xóa
- [ ] Mọi cảnh báo đều dẫn link đến một runbook — tối thiểu ba dòng: ý nghĩa của cảnh báo, truy vấn đầu tiên cần chạy, quy trình leo thang (escalation path)
- [ ] Ngưỡng (thresholds) và thời gian (durations) được xác định dựa trên SLO hoặc dữ liệu lịch sử, không phải do dự đoán
- [ ] Chỉ có hai mức độ nghiêm trọng: **page** (ảnh hưởng người dùng, xử lý ngay) và **ticket** (suy giảm hiệu năng, xử lý trong tuần này)
- [ ] Mỗi cảnh báo mới được chạy thử một lần: đảm bảo nó gửi đến đúng channel và link runbook hoạt động
- [ ] Không để tồn tại các cảnh báo kích hoạt hàng ngày và được xác nhận (acknowledged) mà không có hành động xử lý

## Bảng điều khiển (Dashboards)

- [ ] Tồn tại dashboard theo dõi sức khỏe dịch vụ: error rate, latency p99, traffic, saturation
- [ ] Bảng theo dõi sức khỏe phụ thuộc hiển thị error rates và latency cho từng dịch vụ
- [ ] Dashboard trả lời được các câu hỏi on-call ở đầu danh sách kiểm tra này — không phải là "hiển thị mọi thứ trừ câu trả lời"
- [ ] Khoảng thời gian mặc định hợp lý (1h–6h, không phải 30d)

## Xác minh Telemetry

Thiết lập công cụ quan sát cũng là viết mã; nó có thể sai:

- [ ] Gây ra lỗi trong môi trường staging → tìm thấy lỗi trong logs bằng correlation ID
- [ ] Gửi traffic thử nghiệm → các metric series xuất hiện với labels mong đợi và giá trị hợp lý
- [ ] Theo dõi một request từ đầu đến cuối trong giao diện tracing → không có span nào bị gãy
- [ ] Một lỗi giả lập được chẩn đoán chỉ dựa trên telemetry mà không cần đọc mã nguồn

## Cửa kiểm soát Trước khi Ra mắt (Pre-Launch Gate)

Trước khi một tính năng được triển khai lên production, tất cả các điều sau phải đúng:

- [ ] Structured logs được đổ về log aggregator
- [ ] RED metrics hiển thị trên dashboards cho mọi endpoint và phụ thuộc mới
- [ ] Có ít nhất một cảnh báo symptom-based được cấu hình, kèm theo runbook và đã chạy thử
- [ ] Một request có thể được truy vết qua mọi dịch vụ mà nó đi qua
- [ ] Nhân viên on-call biết runbooks nằm ở đâu

Để biết trình tự giám sát ngày ra mắt và các điều kiện kích hoạt rollback, hãy xem skill `shipping-and-launch`.
