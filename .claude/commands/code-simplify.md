---
description: Đơn giản hóa mã nguồn để tăng cường độ rõ ràng và khả năng bảo trì — giảm độ phức tạp mà không làm thay đổi hành vi
---

Kích hoạt skill agent-skills:code-simplification.

Đơn giản hóa mã nguồn vừa thay đổi gần đây (hoặc phạm vi được chỉ định) trong khi vẫn giữ nguyên chính xác hành vi:

1. Đọc CLAUDE.md và nghiên cứu các quy ước của dự án
2. Xác định mã nguồn mục tiêu — các thay đổi gần đây trừ khi có phạm vi rộng hơn được chỉ định
3. Hiểu mục đích của mã nguồn, các callers, edge cases, và độ bao phủ của test trước khi can thiệp
4. Tìm kiếm các cơ hội đơn giản hóa:
   - Lồng nhau sâu (Deep nesting) → guard clauses hoặc extracted helpers
   - Hàm quá dài → chia nhỏ theo trách nhiệm
   - Nested ternaries → if/else hoặc switch
   - Tên chung chung → tên mô tả chi tiết hơn
   - Logic bị trùng lặp → các hàm chia sẻ
   - Dead code → xóa sau khi xác nhận
5. Áp dụng từng bước đơn giản hóa một cách tăng tiến — chạy test sau mỗi thay đổi
6. Xác minh tất cả các test đều vượt qua, build thành công và diff sạch sẽ

Nếu test thất bại sau một bước đơn giản hóa, hãy hoàn tác thay đổi đó và xem xét lại. Sử dụng `code-review-and-quality` để đánh giá kết quả.
