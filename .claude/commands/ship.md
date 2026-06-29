---
description: Chạy checklist trước khi ra mắt thông qua parallel fan-out tới các persona chuyên gia, sau đó tổng hợp quyết định go/no-go
---

Kích hoạt skill agent-skills:shipping-and-launch.

`/ship` là một **fan-out orchestrator**. Nó chạy ba persona chuyên gia song song đối với thay đổi hiện tại, sau đó hợp nhất các báo cáo của họ thành một quyết định go/no-go duy nhất kèm theo kế hoạch rollback. Các persona hoạt động độc lập — không chia sẻ trạng thái, không có thứ tự — điều này giúp việc thực thi song song trở nên an toàn và hữu ích trong trường hợp này.

## Giai đoạn A — Parallel fan-out

Khởi tạo ba subagent đồng thời bằng công cụ Agent. **Hãy thực hiện cả ba lời gọi công cụ Agent trong một lượt phản hồi duy nhất của assistant để chúng thực thi song song** — các lời gọi tuần tự sẽ làm mất đi mục đích của lệnh này.

Trong Claude Code, mỗi lời gọi truyền `subagent_type` khớp với trường `name` của persona:

1. **`code-reviewer`** — Thực hiện đánh giá năm trục (tính đúng đắn, khả năng đọc, kiến trúc, bảo mật, hiệu năng) trên các staged changes hoặc các commit gần đây. Xuất ra template đánh giá chuẩn.
2. **`security-auditor`** — Thực hiện một lượt kiểm tra lỗ hổng và threat-model. Kiểm tra OWASP Top 10, xử lý secrets, auth/authz, CVEs của dependency. Xuất ra báo cáo kiểm định chuẩn.
3. **`test-engineer`** — Phân tích test coverage cho thay đổi. Xác định các lỗ hổng trong happy path, edge cases, error paths và các kịch bản concurrency. Xuất ra phân tích coverage chuẩn.

Trong các harness khác không có công cụ Agent, hãy kích hoạt system prompt của mỗi persona theo tuần tự và coi kết quả đầu ra của họ như thể được trả về song song — giai đoạn hợp nhất vẫn hoạt động.

Các ràng buộc (từ mô hình subagent của Claude Code):
- Subagent không thể khởi tạo subagent khác — không để một persona ủy quyền cho persona khác.
- Mỗi subagent có một context window riêng và chỉ trả về báo cáo của mình cho session chính này.
- Nếu bạn cần các đồng đội có thể giao tiếp với nhau thay vì chỉ báo cáo lại, hãy sử dụng Claude Code Agent Teams và tham chiếu các persona này như các loại teammate (xem `references/orchestration-patterns.md`).

**Giải quyết Persona.** Nếu bạn đã định nghĩa `code-reviewer`, `security-auditor`, hoặc `test-engineer` của riêng mình trong `.claude/agents/` hoặc `~/.claude/agents/`, những định nghĩa đó sẽ được ưu tiên hơn các phiên bản của plugin này — `/ship` sẽ tự động nhận diện các tùy chỉnh của bạn. Điều này là có ý định: các subagent của plugin nằm ở dưới cùng của bảng ưu tiên scope trong Claude Code, vì vậy các định nghĩa ở cấp độ người dùng sẽ chiến thắng theo thiết kế.

## Giai đoạn B — Hợp nhất trong context chính

Khi cả ba báo cáo đã được trả về, agent chính (không phải sub-persona) sẽ tổng hợp chúng:

1. **Code Quality** — Tổng hợp các phát hiện Critical/Important từ `code-reviewer` và bất kỳ test bị lỗi, lint, hoặc build output nào. Giải quyết các trùng lặp giữa các reviewer.
2. **Security** — Đưa bất kỳ phát hiện Critical/High nào từ `security-auditor` thành launch blockers. Đối chiếu với trục bảo mật của `code-reviewer`.
3. **Performance** — Lấy từ trục hiệu năng của `code-reviewer`; đối chiếu với Core Web Vitals nếu áp dụng.
4. **Accessibility** — Xác minh điều hướng bàn phím, hỗ trợ trình đọc màn hình, độ tương phản (không được bao phủ bởi ba persona — xử lý trực tiếp tại đây, hoặc kích hoạt accessibility checklist).
5. **Infrastructure** — Env vars, migrations, monitoring, feature flags. Xác minh trực tiếp.
6. **Documentation** — README, ADRs, changelog. Xác minh trực tiếp.

## Giai đoạn C — Quyết định và rollback

Tạo ra một đầu ra duy nhất:

```markdown
## Quyết định Ship: GO | NO-GO

### Blockers (phải sửa trước khi ship)
- [Persona nguồn: Phát hiện Critical + file:line]

### Các bản sửa khuyến nghị (nên sửa trước khi ship)
- [Persona nguồn: Phát hiện Important + file:line]

### Các rủi ro đã xác nhận (vẫn ship)
- [Rủi ro + giảm thiểu]

### Kế hoạch Rollback
- Điều kiện kích hoạt: [tín hiệu nào sẽ yêu cầu rollback]
- Quy trình Rollback: [các bước chính xác]
- Mục tiêu thời gian khôi phục (RTO): [mục tiêu]

### Báo cáo chuyên gia (chi tiết)
- [báo cáo code-reviewer]
- [báo cáo security-auditor]
- [báo cáo test-engineer]
```

## Quy tắc

1. Ba persona ở Giai đoạn A chạy song song — không bao giờ chạy tuần tự.
2. Các persona không gọi nhau. Agent chính hợp nhất trong Giai đoạn B.
3. Kế hoạch rollback là bắt buộc trước bất kỳ quyết định GO nào.
4. Nếu bất kỳ persona nào trả về phát hiện Critical, phán quyết mặc định là NO-GO trừ khi người dùng chấp nhận rủi ro một cách rõ ràng.
5. **Chỉ bỏ qua fan-out nếu tất cả các điều sau là đúng:** thay đổi ảnh hưởng đến 2 file hoặc ít hơn, diff dưới 50 dòng, và không chạm vào auth, payments, truy cập dữ liệu, hoặc config/env. Nếu không, mặc định là fan-out. `/ship` được thiết kế cho các thay đổi hướng tới production — khi blast radius là không nhỏ, hãy chạy đánh giá song song ngay cả khi diff trông có vẻ nhỏ.
