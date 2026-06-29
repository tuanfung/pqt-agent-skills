---
description: Chạy quy trình TDD — viết các test thất bại, thực thi, xác minh. Đối với các lỗi, hãy sử dụng pattern Prove-It.
---

Kích hoạt skill agent-skills:test-driven-development.

Đối với các tính năng mới:
1. Viết các test mô tả hành vi mong đợi (các test này phải THẤT BẠI)
2. Thực thi mã nguồn để các test này vượt qua
3. Refactor trong khi vẫn giữ cho các test ở trạng thái green

Đối với việc sửa lỗi (pattern Prove-It):
1. Viết một test tái hiện lỗi (phải THẤT BẠI)
2. Xác nhận test thất bại
3. Thực thi bản sửa lỗi
4. Xác nhận test vượt qua
5. Chạy toàn bộ test suite để kiểm tra regression

Đối với các vấn đề liên quan đến trình duyệt, hãy kích hoạt thêm agent-skills:browser-testing-with-devtools để xác minh với Chrome DevTools MCP.
