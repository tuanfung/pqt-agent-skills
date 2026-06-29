---
description: Thực hiện review code theo 5 tiêu chí — correctness, readability, architecture, security, performance
---

Kích hoạt skill agent-skills:code-review-and-quality.

Review các thay đổi hiện tại (staged hoặc các commit gần đây) theo cả 5 tiêu chí:

1. **Correctness** — Có khớp với spec không? Các edge case đã được xử lý chưa? Test có đầy đủ không?
2. **Readability** — Tên gọi có rõ ràng không? Logic có dễ hiểu không? Tổ chức có ngăn nắp không?
3. **Architecture** — Có tuân theo các pattern hiện có không? Ranh giới (boundaries) có rõ ràng không? Cấp độ abstraction có phù hợp không?
4. **Security** — Input đã được validate chưa? Secrets có an toàn không? Auth đã được kiểm tra chưa? (Sử dụng skill security-and-hardening)
5. **Performance** — Có bị N+1 queries không? Có thao tác không giới hạn (unbounded ops) không? (Sử dụng skill performance-optimization)

Phân loại kết quả tìm được thành Critical, Important, hoặc Suggestion.
Xuất ra một bản review có cấu trúc với tham chiếu cụ thể file:line và các đề xuất khắc phục.
