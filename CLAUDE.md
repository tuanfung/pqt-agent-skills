# agent-skills

Đây là dự án agent-skills — một bộ sưu tập các kỹ năng kỹ thuật cấp độ production dành cho các AI coding agent.

## Cấu trúc dự án

```
skills/       → Các kỹ năng cốt lõi (mỗi thư mục có một file SKILL.md)
agents/       → Các persona agent có thể tái sử dụng (code-reviewer, test-engineer, security-auditor, web-performance-auditor)
hooks/        → Các session lifecycle hook
.claude/commands/ → Các slash command (/spec, /plan, /build, /test, /review, /code-simplify, /ship; cùng với /webperf specialist audit)
references/   → Các checklist bổ trợ (testing, performance, security, accessibility, observability)
docs/         → Hướng dẫn cài đặt cho các công cụ khác nhau
```

## Kỹ năng theo giai đoạn

**Define:** interview-me, idea-refine, spec-driven-development
**Plan:** planning-and-task-breakdown
**Build:** incremental-implementation, test-driven-development, context-engineering, source-driven-development, doubt-driven-development, frontend-ui-engineering, api-and-interface-design
**Verify:** browser-testing-with-devtools, debugging-and-error-recovery
**Review:** code-review-and-quality, code-simplification, security-and-hardening, performance-optimization
**Ship:** git-workflow-and-versioning, ci-cd-and-automation, deprecation-and-migration, documentation-and-adrs, observability-and-instrumentation, shipping-and-launch

## Quy ước

- Mỗi skill nằm trong `skills/<name>/SKILL.md`
- YAML frontmatter với các trường `name` và `description`
- Phần description bắt đầu bằng việc mô tả skill đó làm gì (ngôi thứ ba), sau đó là các điều kiện kích hoạt ("Use when...")
- Mỗi skill bao gồm: Overview, When to Use, Process, Common Rationalizations, Red Flags, Verification
- Các tài liệu tham khảo (references) nằm trong `references/`, không nằm trong các thư mục skill
- Các file hỗ trợ chỉ được tạo khi nội dung vượt quá 100 dòng

## Đóng góp

Trước khi thêm một skill mới hoặc chỉnh sửa đáng kể một skill hiện có, hãy chạy các pre-flight check trong [CONTRIBUTING.md](CONTRIBUTING.md#before-proposing-a-new-skill): tìm kiếm trong danh mục, kiểm tra các PR đang mở, xác nhận ý tưởng phù hợp với [docs/skill-anatomy.md](docs/skill-anatomy.md), và giải thích lý do cho khoảng trống đó. Ưu tiên mở rộng một skill hiện có thay vì thêm một skill gần như trùng lặp. CONTRIBUTING.md là nguồn sự thật duy nhất (single source of truth) cho quy trình này; không viết lại checklist của nó ở đây hoặc bất kỳ nơi nào khác, hãy dẫn link tới đó.

## Câu lệnh

- `npm test` — Không áp dụng (đây là dự án tài liệu)
- Validate: Kiểm tra xem tất cả các file SKILL.md có YAML frontmatter hợp lệ với name và description hay không

## Pull Requests

Các PR nhắm tới branch mặc định của upstream repository. Trong thiết lập fork điển hình, upstream remote là `upstream` và fork của bạn là `origin`, nhưng tên remote chính xác không quan trọng ở đây.

- Trước khi mở một PR, hãy tìm kiếm các PR và issue đang mở của upstream repository để xem có công việc nào tác động đến cùng các file hoặc quy tắc hay không. Nếu có bất kỳ sự trùng lặp nào, hãy phối hợp (xây dựng dựa trên đó, điều chỉnh quy tắc cho khớp, hoặc rebase sau khi nó được merge) thay vì mở một PR gây xung đột.
- Ưu tiên các PR nhỏ, tập trung thay vì thực hiện refactor lớn trên các file được chia sẻ rộng rãi (ví dụ: các file trong `scripts/`), vì chúng dễ gây xung đột với các công việc đang triển khai.

## Ranh giới (Boundaries)

- Luôn luôn: Chạy các pre-flight check trong CONTRIBUTING.md trước khi tạo thư mục skill mới
- Luôn luôn: Tuân thủ định dạng skill-anatomy.md cho các skill mới
- Luôn luôn: Kiểm tra các PR và issue đang mở của upstream repo để tìm sự trùng lặp trước khi mở PR mới
- Không bao giờ: Thêm các skill chỉ là lời khuyên mơ hồ thay vì các quy trình có thể thực hiện được
- Không bao giờ: Trùng lặp nội dung giữa các skill — thay vào đó hãy tham chiếu đến các skill khác

## Bash: Kiểm tra working directory trước khi `cd`

**Đừng `cd` mù. Biết mình đang ở đâu trước đã.**

Trước khi chạy `cd <path>`:
- Chạy `pwd` (hoặc kiểm tra context working directory đã có sẵn) để xác định vị trí hiện tại.
- Nếu đã ở đúng thư mục cần đến $\rightarrow$ KHÔNG `cd`. Chạy lệnh trực tiếp.
- Nếu cần dùng đường dẫn $\rightarrow$ ưu tiên đường dẫn tuyệt đối hoặc tương đối từ vị trí hiện tại thay vì `cd` rồi chạy.
- Không nối `cd <current-dir> && <command>` — vô nghĩa và gây prompt permission thừa.

Quy tắc áp dụng cho mọi agent (leader, coder, reviewer) và mọi lệnh Bash. `cd` chỉ chính đáng khi:
- Thật sự cần đổi sang thư mục khác để chạy lệnh phụ thuộc cwd (build script, package manager trong subpackage).
- User yêu cầu rõ ràng.

Sai: `cd /Users/bonn/Desktop/claude_demo_part2/claude_book && ls` (khi đã ở đó)
Đúng: `ls` (chạy thẳng)

## Ngôn ngữ đầu ra

**Luôn trả lời user bằng tiếng Việt.**

- Mọi output hướng tới user (giải thích, báo cáo, tóm tắt, câu hỏi clarify) đều viết bằng tiếng Việt.
- Áp dụng cho toàn bộ agent: phiên chính, leader, coder, reviewer, và teammate trong Agent Teams.
- Giữ nguyên tiếng Anh cho: tên file, tên biến, code, lệnh shell, error message gốc, thuật ngữ kỹ thuật không có bản dịch chuẩn (ví dụ: "merge", "rebase", "lint", "typecheck").
- Không dịch comment/code có sẵn trong file sang tiếng Việt trừ khi user yêu cầu.
