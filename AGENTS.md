# AGENTS.md

File này cung cấp hướng dẫn cho các AI coding agents (Claude Code, Cursor, Copilot, Antigravity, v.v.) khi làm việc với mã nguồn trong repository này.

## Tổng quan Repository

Một tập hợp các skills dành cho Claude.ai và Claude Code cho các kỹ sư phần mềm cấp cao. Skills là các tập hợp hướng dẫn và script được đóng gói nhằm mở rộng khả năng của Claude và các coding agents của bạn.

## Tích hợp OpenCode

OpenCode sử dụng **mô hình thực thi hướng skill** (skill-driven execution model) được vận hành bởi công cụ `skill` và thư mục `/skills` của repository này.

### Các quy tắc cốt lõi

- Nếu một nhiệm vụ khớp với một skill, bạn PHẢI gọi skill đó.
- Các skills nằm tại `skills/<skill-name>/SKILL.md`.
- Không bao giờ triển khai trực tiếp nếu có một skill tương ứng.
- Luôn tuân thủ chính xác các hướng dẫn của skill (không áp dụng một phần).

### Ánh xạ Ý định $\rightarrow$ Skill

Agent nên tự động ánh xạ ý định của người dùng sang các skills:

- Tính năng / chức năng mới $\rightarrow$ `spec-driven-development`, sau đó là `incremental-implementation`, `test-driven-development`.
- Lập kế hoạch / chia nhỏ nhiệm vụ $\rightarrow$ `planning-and-task-breakdown`.
- Lỗi / thất bại / hành vi không mong muốn $\rightarrow$ `debugging-and-error-recovery`.
- Review mã nguồn $\rightarrow$ `code-review-and-quality`.
- Tái cấu trúc / đơn giản hóa $\rightarrow$ `code-simplification`.
- Thiết kế API hoặc giao diện $\rightarrow$ `api-and-interface-design`.
- Công việc UI $\rightarrow$ `frontend-ui-engineering`.

### Ánh xạ Vòng đời (Các lệnh ngầm định)

OpenCode không hỗ trợ các slash commands như `/spec` hoặc `/plan`.

Thay vào đó, agent phải tuân theo vòng đời nội bộ sau:

- ĐỊNH NGHĨA (DEFINE) $\rightarrow$ `spec-driven-development`
- LẬP KẾ HOẠCH (PLAN) $\rightarrow$ `planning-and-task-breakdown`
- XÂY DỰNG (BUILD) $\rightarrow$ `incremental-implementation` + `test-driven-development`
- XÁC MINH (VERIFY) $\rightarrow$ `debugging-and-error-recovery`
- REVIEW $\rightarrow$ `code-review-and-quality`
- TRIỂN KHAI (SHIP) $\rightarrow$ `shipping-and-launch`

### Mô hình thực thi

Đối với mọi yêu cầu:

1. Xác định xem có bất kỳ skill nào áp dụng được không (ngay cả khi chỉ có 1% khả năng).
2. Gọi skill phù hợp bằng công cụ `skill`.
3. Tuân thủ nghiêm ngặt quy trình làm việc (workflow) của skill.
4. Chỉ tiến hành triển khai sau khi các bước bắt buộc (spec, plan, v.v.) đã hoàn thành.

### Chống lý lý lẽ hóa (Anti-Rationalization)

Những suy nghĩ sau đây là sai và phải bị bỏ qua:

- "Việc này quá nhỏ để dùng skill"
- "Tôi có thể triển khai nhanh việc này"
- "Tôi sẽ thu thập ngữ cảnh trước"

Hành vi đúng:

- Luôn kiểm tra và sử dụng các skills trước tiên.

Điều này đảm bảo OpenCode hoạt động tương tự như Claude Code với việc thực thi đầy đủ quy trình làm việc.

## Điều phối (Orchestration): Personas, Skills và Commands

Repo này có ba lớp có thể kết hợp. Chúng có các nhiệm vụ khác nhau và không nên bị nhầm lẫn:

- **Skills** (`skills/<name>/SKILL.md`) — các quy trình làm việc với các bước và tiêu chí kết thúc. Là câu trả lời cho câu hỏi *làm thế nào (how)*. Đây là các bước bắt buộc khi ý định khớp với skill.
- **Personas** (`agents/<role>.md`) — các vai trò với một góc nhìn và định dạng đầu ra. Là câu trả lời cho câu hỏi *ai (who)*.
- **Slash commands** (`.claude/commands/*.md`) — các điểm khởi đầu cho người dùng. Là câu trả lời cho câu hỏi *khi nào (when)*. Đây là lớp điều phối (orchestration layer).

Quy tắc kết hợp: **người dùng (hoặc một slash command) là người điều phối. Personas không gọi các personas khác.** Một persona có thể gọi các skills.

### Mô hình Team Orchestration (Khi dùng `/team`)

Khi lệnh `/team` được kích hoạt, hệ thống chuyển sang chế độ điều phối nhóm. Thay vì một agent đơn lẻ, công việc được phân bổ cho 5 vai trò chuyên biệt theo chu trình khép kín:

- **Leader**: Định hướng $\rightarrow$ Plan.
- **Coder**: Triển khai (Build).
- **Tester**: Xác minh (Verify).
- **Reviewer**: Kiểm soát chất lượng (Review).
- **Shipper**: Đóng gói & Phát hành (Ship).

Mọi trao đổi giữa các vai trò này PHẢI thông qua `SendMessage` trong tmux.

### Mô hình Parallel Fan-out

Mô hình điều phối đa persona duy nhất mà repo này chấp nhận ngoài Team Mode là **phân tán song song với một bước hợp nhất (parallel fan-out with a merge step)** — được sử dụng bởi `/ship` để chạy `code-reviewer`, `security-auditor`, và `test-engineer` đồng thời và tổng hợp các báo cáo của họ.

Xem [docs/agents.md](docs/agents.md) để biết ma trận quyết định và [references/orchestration-patterns.md](references/orchestration-patterns.md) để biết danh mục đầy đủ các mô hình.

**Khả năng tương tác Claude Code:** các personas trong `agents/` hoạt động như các subagents của Claude Code và như những thành viên trong Agent Teams. Subagents không thể khởi tạo subagents khác, và các teams không thể lồng nhau.

## Tạo một Skill Mới

> **Trước khi bắt đầu:** hãy chạy các kiểm tra chuẩn bị trong [CONTRIBUTING.md](CONTRIBUTING.md#before-proposing-a-new-skill), tìm kiếm trong danh mục, kiểm tra các PR đang mở (`gh pr list --state open`), xác nhận ý tưởng phù hợp với [docs/skill-anatomy.md](docs/skill-anatomy.md), và giải trình khoảng trống trong mô tả PR của bạn. Hầu hết các ý tưởng skill mới đều trùng lặp với một skill hiện có hoặc một PR đang mở; hãy ưu tiên mở rộng một skill hiện có thay vì thêm một skill gần như trùng lặp. CONTRIBUTING.md là nguồn sự thật duy nhất (single source of truth) cho quy trình này.

Các skills trong repo này ưu tiên sử dụng markdown: mỗi skill nằm tại `skills/<kebab-case-name>/SKILL.md` với YAML frontmatter (`name`, `description`) và tuân theo cấu trúc các phần (Overview, When to Use, Process, Common Rationalizations, Red Flags, Verification). Chỉ thêm thư mục `scripts/` khi skill cung cấp các helper có thể chạy được; hầu hết các skills chỉ gồm markdown, và không có các gói zip riêng cho từng skill.

Để biết đầy đủ định dạng, quy ước đặt tên, quy tắc frontmatter, ngưỡng file hỗ trợ và các nguyên tắc viết, hãy xem [docs/skill-anatomy.md](docs/skill-anatomy.md), nguồn sự thật duy nhất cho cấu trúc skill. Không nhắc lại các hướng dẫn đó ở đây, hãy dẫn link tới đó.

## Pull Requests

Các PR nhắm tới branch mặc định của upstream repository.

- Trước khi mở một PR, hãy tìm kiếm các PR và issue đang mở của upstream repository để xem có công việc nào tác động đến cùng các file hoặc quy tắc hay không.
- Ưu tiên các PR nhỏ, tập trung thay vì thực hiện refactor lớn trên các file được chia sẻ rộng rãi.

## Ranh giới (Boundaries)

- Luôn luôn: Chạy các pre-flight check trong CONTRIBUTING.md trước khi tạo thư mục skill mới
- Luôn luôn: Tuân thủ định dạng skill-anatomy.md cho các skill mới
- Luôn luôn: Kiểm tra các PR và issue đang mở của upstream repo để tìm sự trùng lặp trước khi mở PR mới
- Không bao giờ: Thêm các skill chỉ là lời khuyên mơ hồ thay vì các quy trình có thể thực hiện được
- Không bao giờ: Trùng lặp nội dung giữa các skill — thay vào đó hãy tham chiếu đến các skill khác
