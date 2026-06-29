---
description: Rào chắn chống trùng lặp khi thêm hoặc thay đổi các skill
paths:
  - "skills/**"
---

# Thêm hoặc thay đổi một skill

Repo này đã bao gồm hầu hết các vòng đời phát triển, vì vậy hầu hết các ý tưởng skill mới sẽ trùng lặp với một skill hiện có hoặc một PR đang mở. Trước khi tạo thư mục `skills/<name>/` mới hoặc chỉnh sửa đáng kể một thư mục hiện có:

- Thực hiện các kiểm tra chuẩn bị (pre-flight checks) trong [CONTRIBUTING.md](../../CONTRIBUTING.md#before-proposing-a-new-skill): tìm kiếm trong danh mục, kiểm tra các PR đang mở (`gh pr list --state open`), và giải trình về khoảng trống cần bổ sung.
- Ưu tiên mở rộng một skill hiện có thay vì thêm một skill gần như trùng lặp. Nếu ý tưởng trùng lặp với một skill hiện có, hãy chỉnh sửa skill đó thay vì thêm một thư mục mới.
- Giữ file `SKILL.md` tuân theo [docs/skill-anatomy.md](../../docs/skill-anatomy.md), và tuyệt đối không trùng lặp nội dung giữa các skill, thay vào đó hãy tham chiếu đến skill kia.

CONTRIBUTING.md là nguồn sự thật duy nhất (single source of truth) cho toàn bộ workflow; quy tắc này trỏ đến đó thay vì nhắc lại danh sách kiểm tra.
