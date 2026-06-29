---
description: Chạy kiểm tra hiệu suất web thông qua persona web-performance-auditor
---

`/webperf` nhắm mục tiêu cụ thể đến các ứng dụng web. Không sử dụng cho các thư viện tiện ích, CLI hoặc mã chỉ chạy trên server mà không có đầu ra hiển thị trên trình duyệt.

## Xác định chế độ (mode)

**Deep mode** — kích hoạt khi có bất kỳ điều kiện nào sau đây:
- Một tệp báo cáo Lighthouse JSON (ví dụ: `npx lighthouse <url> --output json --output-path ./report.json`, hoặc `npx -p chrome-devtools-mcp chrome-devtools lighthouse_audit --output-format=json` từ Chrome DevTools MCP CLI)
- Một phản hồi PageSpeed Insights JSON (bao gồm Lighthouse + CrUX)
- Một phản hồi CrUX API (yêu cầu `CRUX_API_KEY` hoặc `GOOGLE_API_KEY`)
- Một DevTools performance trace
- Một URL trực tiếp cùng với `chrome-devtools` MCP server được cấu hình trong harness (agent có thể thu thập các số liệu trực tiếp thông qua các công cụ `lighthouse_audit` và `performance_*`)
- Chrome DevTools MCP CLI được gọi cục bộ (qua `npx -p chrome-devtools-mcp chrome-devtools <tool>` hoặc sau khi `npm i -g chrome-devtools-mcp`) — người dùng chạy các lệnh như `chrome-devtools lighthouse_audit --output-format=json` và gửi kết quả JSON cho agent

**Quick mode** — mặc định khi không có điều kiện nào ở trên. Agent sẽ quét mã nguồn để tìm các anti-pattern về cấu trúc và gắn nhãn cho mỗi phát hiện là `potential impact`.

## Chạy kiểm tra (audit)

Khởi tạo subagent `web-performance-auditor`. Truyền cho nó một cách rõ ràng:

- Các tệp, component hoặc diff đang được xem xét
- Bất kỳ đường dẫn artifact nào (Lighthouse JSON, PSI JSON, phản hồi CrUX, trace) hoặc nội dung JSON được dán vào
- URL mục tiêu hoặc tên trang nếu biết
- Một ghi chú về chế độ bạn mong muốn (Quick hoặc Deep), để agent có thể thông báo nếu thiếu đầu vào khi chế độ Deep được dự định

Subagent sẽ trả về một bảng điểm (scorecard - chỉ được điền các giá trị có nguồn xác định), một danh sách các phát hiện được xếp hạng, các quan sát tích cực và các đề xuất chủ động.

## Đầu ra (Output)

Trả về báo cáo kiểm tra đầy đủ cho người dùng. Không cần bước tổng hợp hay hợp nhất — đây là lệnh cho một persona duy nhất.
