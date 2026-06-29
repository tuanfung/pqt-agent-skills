---
description: Chia nhỏ công việc thành các task có thể kiểm chứng được với tiêu chí chấp nhận và thứ tự phụ thuộc
---

Gọi skill agent-skills:planning-and-task-breakdown.

Đọc spec hiện có (SPEC.md hoặc tương đương) và các phần liên quan trong codebase. Sau đó:

1. Vào chế độ lập kế hoạch (plan mode) — chỉ đọc, không thay đổi code
2. Xác định sơ đồ phụ thuộc (dependency graph) giữa các component
3. Chia nhỏ công việc theo chiều dọc (một luồng hoàn chỉnh cho mỗi task, thay vì chia theo lớp ngang)
4. Viết các task kèm theo tiêu chí chấp nhận (acceptance criteria) và các bước kiểm chứng
5. Thêm các điểm kiểm tra (checkpoints) giữa các giai đoạn
6. Trình bày kế hoạch để con người review

Lưu kế hoạch vào tasks/plan.md và danh sách task vào tasks/todo.md.
