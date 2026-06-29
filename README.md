# Agent Skills

**Các kỹ năng kỹ thuật cấp độ production dành cho các AI coding agent.**

Các skill mã hóa quy trình làm việc, các cổng chất lượng (quality gates) và các thực hành tốt nhất (best practices) mà các kỹ sư cấp cao sử dụng khi xây dựng phần mềm. Những skill này được đóng gói để các AI agent tuân thủ một cách nhất quán trong mọi giai đoạn phát triển.

<a href="https://trendshift.io/repositories/25200" target="_blank"><img src="https://trendshift.io/api/badge/repositories/25200" alt="addyosmani%2Fagent-skills | Trendshift" style="width: 250px; height: 55px;" width="250" height="55"/></a>

![Addy's Agent Skills](https://addyosmani.com/assets/images/addys-agent-skills.jpg)

```
  DEFINE          PLAN           BUILD          VERIFY         REVIEW          SHIP
 ┌──────┐      ┌──────┐      ┌──────┐      ┌──────┐      ┌──────┐      ┌──────┐
 │ Idea │ ───▶ │ Spec │ ───▶ │ Code │ ───▶ │ Test │ ───▶ │  QA  │ ───▶ │  Go  │
 │Refine│      │  PRD │      │ Impl │      │Debug │      │ Gate │      │ Live │
 └──────┘      └──────┘      └──────┘      └──────┘      └──────┘      └──────┘
  /spec          /plan          /build        /test         /review       /ship
```

---

## Câu lệnh

8 câu lệnh slash tương ứng với vòng đời phát triển. Mỗi câu lệnh sẽ tự động kích hoạt các skill phù hợp.

| Công việc đang làm | Câu lệnh | Nguyên tắc chính |
|-------------------|---------|---------------|
| Xác định những gì cần xây dựng | `/spec` | Spec trước khi viết code |
| Lập kế hoạch xây dựng | `/plan` | Các task nhỏ, nguyên tử |
| Xây dựng tăng dần | `/build` | Từng lát cắt một |
| Chứng minh nó hoạt động | `/test` | Test là bằng chứng |
| Review trước khi merge | `/review` | Cải thiện sức khỏe của code |
| Kiểm tra hiệu suất web | `/webperf` | Đo lường trước khi tối ưu |
| Đơn giản hóa code | `/code-simplify` | Sự rõ ràng quan trọng hơn sự khéo léo |
| Triển khai lên production | `/ship` | Nhanh hơn là an toàn hơn |

Muốn ít bước thủ công hơn sau khi đã có spec? **`/build auto`** sẽ tạo kế hoạch và thực hiện mọi task trong một lượt phê duyệt duy nhất — bạn phê duyệt kế hoạch một lần, sau đó nó sẽ chạy tự động. Nó loại bỏ việc con người phải can thiệp *giữa* các task, nhưng không loại bỏ việc xác minh: mọi task vẫn được phát triển theo hướng test-driven và commit riêng biệt, đồng thời nó sẽ tạm dừng khi gặp lỗi hoặc các bước rủi ro.

Các skill cũng tự động kích hoạt dựa trên những gì bạn đang làm — thiết kế API sẽ kích hoạt `api-and-interface-design`, xây dựng UI sẽ kích hoạt `frontend-ui-engineering`, v.v.

---

## Bắt đầu nhanh

<details>
<summary><b>Claude Code (khuyến nghị)</b></summary>

**Cài đặt từ Marketplace:**

```
/plugin marketplace add addyosmani/agent-skills
/plugin install agent-skills@addy-agent-skills
```

> **Lỗi SSH?** Marketplace clone các repo qua SSH. Nếu bạn chưa thiết lập SSH key trên GitHub, hãy thêm SSH key của bạn hoặc sử dụng toàn bộ URL HTTPS để buộc clone qua HTTPS:
> ```bash
> /plugin marketplace add https://github.com/addyosmani/agent-skills.git
> /plugin install agent-skills@addy-agent-skills
> ```

**Local / Phát triển:**

```bash
git clone https://github.com/addyosmani/agent-skills.git
claude --plugin-dir /path/to/agent-skills
```

</details>

<details>
<summary><b>Cursor</b></summary>

Sao chép bất kỳ file `SKILL.md` nào vào `.cursor/rules/`, hoặc tham chiếu đến toàn bộ thư mục `skills/`. Xem [docs/cursor-setup.md](docs/cursor-setup.md).

</details>

<details>
<summary><b>Antigravity CLI</b></summary>

Cài đặt dưới dạng plugin gốc cho các skill, subagent và slash command. Xem [docs/antigravity-setup.md](docs/antigravity-setup.md).

**Cài đặt từ repo:**

```bash
agy plugin install https://github.com/addyosmani/agent-skills.git
```

**Cài đặt từ bản clone local:**

```bash
git clone https://github.com/addyosmani/agent-skills.git
agy plugin install ./agent-skills
```

</details>

<details>
<summary><b>Gemini CLI</b></summary>

Cài đặt dưới dạng các skill gốc để tự động phát hiện, hoặc thêm vào `GEMINI.md` để có context bền vững. Xem [docs/gemini-cli-setup.md](docs/gemini-cli-setup.md).

**Cài đặt từ repo:**

```bash
gemini skills install https://github.com/addyosmani/agent-skills.git --path skills
```

**Cài đặt từ bản clone local:**

```bash
gemini skills install ./agent-skills/skills/
```

</details>

<details>
<summary><b>Windsurf</b></summary>

Thêm nội dung skill vào cấu hình rules của Windsurf. Xem [docs/windsurf-setup.md](docs/windsurf-setup.md).

</details>

<details>
<summary><b>OpenCode</b></summary>

Sử dụng thực thi skill do agent điều khiển thông qua `AGENTS.md` và công cụ `skill`.

Xem [docs/opencode-setup.md](docs/opencode-setup.md).

</details>

<details>
<summary><b>GitHub Copilot</b></summary>

Sử dụng các định nghĩa agent từ `agents/` làm persona cho Copilot và nội dung skill trong `.github/copilot-instructions.md`. Xem [docs/copilot-setup.md](docs/copilot-setup.md).

</details>

<details>
  <summary><b>Kiro IDE & CLI </b></summary>
  Các skill cho Kiro nằm trong \".kiro/skills/\" và có thể được lưu trữ ở cấp độ Project hoặc Global. Kiro cũng hỗ trợ `Agents.md`. Xem tài liệu Kiro tại https://kiro.dev/docs/skills/
</details>

<details>
<summary><b>Codex / Các Agent khác</b></summary>

Các skill là Markdown thuần túy - chúng hoạt động với bất kỳ agent nào chấp nhận system prompt hoặc các file hướng dẫn. Xem [docs/getting-started.md](docs/getting-started.md).

</details>

---

## Tất cả 24 Skill

Các câu lệnh trên là điểm bắt đầu. Bộ công cụ bao gồm tổng cộng 24 skill — 23 skill vòng đời cộng với meta-skill `using-agent-skills`. Mỗi skill là một quy trình làm việc có cấu trúc với các bước, cổng xác minh (verification gates) và bảng chống biện hộ (anti-rationalization tables). Bạn cũng có thể tham chiếu trực tiếp đến bất kỳ skill nào.

### Meta - Khám phá skill nào phù hợp

| Skill | Chức năng | Sử dụng khi |
|-------|-------------|----------|
| [using-agent-skills](skills/using-agent-skills/SKILL.md) | Ánh xạ công việc đầu vào với quy trình skill phù hợp và định nghĩa các quy tắc vận hành chung | Bắt đầu một phiên làm việc hoặc quyết định skill nào phù hợp |

### Define - Làm rõ những gì cần xây dựng

| Skill | Chức năng | Sử dụng khi |
|-------|-------------|----------|
| [interview-me](skills/interview-me/SKILL.md) | Phỏng vấn từng câu hỏi một để trích xuất những gì người dùng thực sự muốn thay vì những gì họ nghĩ mình nên muốn, cho đến khi đạt độ tin cậy ~95% | Yêu cầu chưa rõ ràng, hoặc người dùng gọi \"interview me\" / \"grill me\" |
| [idea-refine](skills/idea-refine/SKILL.md) | Tư duy phân kỳ/hội tụ có cấu trúc để biến các ý tưởng mơ hồ thành các đề xuất cụ thể | Bạn có một khái niệm sơ bộ cần khám phá |
| [spec-driven-development](skills/spec-driven-development/SKILL.md) | Viết PRD bao gồm các mục tiêu, câu lệnh, cấu trúc, phong cách code, kiểm thử và ranh giới trước khi viết bất kỳ đoạn code nào | Bắt đầu một dự án, tính năng hoặc thay đổi quan trọng |

### Plan - Chia nhỏ

| Skill | Chức năng | Sử dụng khi |
|-------|-------------|----------|
| [planning-and-task-breakdown](skills/planning-and-task-breakdown/SKILL.md) | Chia nhỏ spec thành các task nhỏ, có thể xác minh với tiêu chí chấp nhận (acceptance criteria) và thứ tự phụ thuộc | Bạn đã có spec và cần các đơn vị có thể thực thi được |

### Build - Viết code

| Skill | Chức năng | Sử dụng khi |
|-------|-------------|----------|
| [incremental-implementation](skills/incremental-implementation/SKILL.md) | Các lát cắt dọc mỏng (thin vertical slices) - implement, test, xác minh, commit. Sử dụng feature flags, các giá trị mặc định an toàn, các thay đổi dễ rollback | Bất kỳ thay đổi nào chạm đến nhiều hơn một file |
| [test-driven-development](skills/test-driven-development/SKILL.md) | Red-Green-Refactor, kim tự tháp test (80/15/5), kích thước test, DAMP thay vì DRY, Quy tắc Beyonce, browser testing | Triển khai logic, sửa lỗi hoặc thay đổi hành vi |
| [context-engineering](skills/context-engineering/SKILL.md) | Cung cấp cho agent thông tin đúng vào đúng thời điểm - các file quy tắc, đóng gói context, tích hợp MCP | Bắt đầu một phiên, chuyển đổi task, hoặc khi chất lượng đầu ra giảm sút |
| [source-driven-development](skills/source-driven-development/SKILL.md) | Căn cứ mọi quyết định về framework vào tài liệu chính thức - xác minh, trích dẫn nguồn, đánh dấu những gì chưa được xác minh | Bạn muốn code có căn cứ, trích dẫn nguồn cho bất kỳ framework hoặc thư viện nào |
| [doubt-driven-development](skills/doubt-driven-development/SKILL.md) | Review đối kháng (adversarial review) với context mới cho mọi quyết định không tầm thường đang thực hiện - CLAIM → EXTRACT → DOUBT → RECONCILE → STOP, với tùy chọn leo thang cross-model khi được người dùng cho phép | Rủi ro cao (production, bảo mật, không thể đảo ngược), làm việc với code lạ, hoặc một đầu ra tự tin thì xác minh bây giờ sẽ rẻ hơn là debug sau này |
| [frontend-ui-engineering](skills/frontend-ui-engineering/SKILL.md) | Kiến trúc component, design system, quản lý state, responsive design, khả năng tiếp cận WCAG 2.1 AA | Xây dựng hoặc chỉnh sửa các giao diện hướng người dùng |
| [api-and-interface-design](skills/api-and-interface-design/SKILL.md) | Thiết kế contract-first, Định luật Hyrum, Quy tắc Một Phiên Bản (One-Version Rule), ngữ nghĩa lỗi, xác thực ranh giới | Thiết kế API, ranh giới module hoặc các interface công khai |

### Verify - Chứng minh nó hoạt động

| Skill | Chức năng | Sử dụng khi |
|-------|-------------|----------|
| [browser-testing-with-devtools](skills/browser-testing-with-devtools/SKILL.md) | Chrome DevTools MCP cho dữ liệu runtime trực tiếp - kiểm tra DOM, console logs, network traces, profiling hiệu suất | Xây dựng hoặc debug bất cứ thứ gì chạy trong trình duyệt |
| [debugging-and-error-recovery](skills/debugging-and-error-recovery/SKILL.md) | Phân loại 5 bước: tái hiện, khoanh vùng, thu hẹp, sửa, bảo vệ. Quy tắc stop-the-line, fallback an toàn | Test thất bại, build lỗi, hoặc hành vi không mong muốn |

### Review - Cổng chất lượng trước khi merge

| Skill | Chức năng | Sử dụng khi |
|-------|-------------|----------|
| [code-review-and-quality](skills/code-review-and-quality/SKILL.md) | Review 5 trục, kích thước thay đổi (~100 dòng), nhãn mức độ nghiêm trọng (Nit/Optional/FYI), quy chuẩn tốc độ review, chiến lược chia nhỏ | Trước khi merge bất kỳ thay đổi nào |
| [code-simplification](skills/code-simplification/SKILL.md) | Hàng rào Chesterton (Chesterton's Fence), Quy tắc 500, giảm độ phức tạp trong khi vẫn bảo toàn chính xác hành vi | Code hoạt động nhưng khó đọc hoặc khó bảo trì hơn mức cần thiết |
| [security-and-hardening](skills/security-and-hardening/SKILL.md) | Ngăn chặn OWASP Top 10, mẫu xác thực (auth patterns), quản lý bí mật (secrets management), kiểm tra phụ thuộc, hệ thống ranh giới ba tầng | Xử lý đầu vào người dùng, xác thực, lưu trữ dữ liệu hoặc tích hợp bên ngoài |
| [performance-optimization](skills/performance-optimization/SKILL.md) | Tiếp cận đo lường trước - mục tiêu Core Web Vitals, quy trình profiling, phân tích bundle, phát hiện anti-pattern | Có yêu cầu về hiệu suất hoặc bạn nghi ngờ có sự sụt giảm hiệu suất (regression) |

### Ship - Triển khai tự tin

| Skill | Chức năng | Sử dụng khi |
|-------|-------------|----------|
| [git-workflow-and-versioning](skills/git-workflow-and-versioning/SKILL.md) | Trunk-based development, atomic commits, kích thước thay đổi (~100 dòng), mẫu commit-as-save-point | Thực hiện bất kỳ thay đổi code nào (luôn luôn) |
| [ci-cd-and-automation](skills/ci-cd-and-automation/SKILL.md) | Shift Left, Nhanh hơn là An toàn hơn, feature flags, pipeline cổng chất lượng, vòng lặp phản hồi lỗi | Thiết lập hoặc sửa đổi pipeline build và deploy |
| [deprecation-and-migration](skills/deprecation-and-migration/SKILL.md) | Tư duy 'code là nợ' (code-as-liability), deprecation bắt buộc và khuyến cáo, mẫu migration, loại bỏ zombie code | Loại bỏ hệ thống cũ, migration người dùng hoặc ngừng cung cấp tính năng |
| [documentation-and-adrs](skills/documentation-and-adrs/SKILL.md) | Hồ sơ quyết định kiến trúc (ADR), tài liệu API, tiêu chuẩn tài liệu inline - ghi chép lại lý do *tại sao* | Đưa ra các quyết định kiến trúc, thay đổi API hoặc triển khai tính năng |
| [observability-and-instrumentation](skills/observability-and-instrumentation/SKILL.md) | Logging có cấu trúc, RED metrics, OpenTelemetry tracing, cảnh báo dựa trên triệu chứng - cài đặt telemetry khi xây dựng | Thêm telemetry, hoặc triển khai bất cứ thứ gì chạy trên production |
| [shipping-and-launch](skills/shipping-and-launch/SKILL.md) | Checklist trước khi launch, vòng đời feature flag, rollout theo giai đoạn, quy trình rollback, thiết lập giám sát | Chuẩn bị triển khai lên production |

---

## Persona của Agent

Các persona chuyên gia được cấu hình sẵn cho các lượt review mục tiêu:

| Agent | Vai trò | Góc nhìn |
|-------|------|-------------|
| [code-reviewer](agents/code-reviewer.md) | Kỹ sư Senior Staff | Review code 5 trục với tiêu chuẩn \"liệu một kỹ sư staff có phê duyệt điều này không?\" |
| [test-engineer](agents/test-engineer.md) | Chuyên gia QA | Chiến lược test, phân tích độ bao phủ (coverage analysis) và mẫu Prove-It |
| [security-auditor](agents/security-auditor.md) | Kỹ sư bảo mật | Phát hiện lỗ hổng, mô hình hóa mối đe dọa (threat modeling), đánh giá OWASP |
| [web-performance-auditor](agents/web-performance-auditor.md) | Kỹ sư hiệu suất web | Audit Core Web Vitals với chế độ Quick/Deep và quy tắc trung thực về số liệu; chạy qua `/webperf` |

Xem [docs/agents.md](docs/agents.md) để biết ma trận quyết định, quy tắc orchestration và cách các persona kết hợp với các skill và slash command.

---

## Checklist tham chiếu

Tài liệu tham khảo nhanh mà các skill sẽ lấy ra khi cần thiết:

| Tham chiếu | Nội dung |
|-----------|--------|
| [definition-of-done.md](references/definition-of-done.md) | Tiêu chuẩn chung cho toàn dự án mà mọi thay đổi phải đạt được, phân biệt với tiêu chí chấp nhận cho từng task |
| [testing-patterns.md](references/testing-patterns.md) | Cấu trúc test, đặt tên, mocking, ví dụ về React/API/E2E, anti-patterns |
| [security-checklist.md](references/security-checklist.md) | Kiểm tra trước khi commit, xác thực, xác thực đầu vào, headers, CORS, OWASP Top 10 |
| [performance-checklist.md](references/performance-checklist.md) | Mục tiêu Core Web Vitals, checklist frontend/backend, câu lệnh đo lường |
| [accessibility-checklist.md](references/accessibility-checklist.md) | Điều hướng bàn phím, trình đọc màn hình, thiết kế trực quan, ARIA, công cụ kiểm thử |
| [observability-checklist.md](references/observability-checklist.md) | Câu hỏi on-call, logging có cấu trúc, RED/USE metrics, tracing, cảnh báo dựa trên triệu chứng, cổng trước khi launch |
| [orchestration-patterns.md](references/orchestration-patterns.md) | Các mẫu orchestration đa persona được khuyến khích, anti-patterns và quy tắc \"persona không gọi persona\" |

---

## Cách thức hoạt động của Skill

Mỗi skill tuân theo một cấu trúc nhất quán:

```
┌─────────────────────────────────────────────────┐
│  SKILL.md                                       │
│                                                 │
│  ┌─ Frontmatter ─────────────────────────────┐  │
│  │ name: tên-viết-thường-có-gạch-ngang               │  │
│  │ description: Hướng dẫn agent thực hiện [task].│  │
│  │              Sử dụng khi…                    │  │
│  └───────────────────────────────────────────┘  │                                                                                                
│  Tổng quan         → Skill này làm gì        │
│  Khi nào sử dụng      → Điều kiện kích hoạt       │
│  Quy trình          → Luồng công việc từng bước       │
│  Biện hộ            → Lý do bào chữa + phản biện         │
│  Dấu hiệu cảnh báo    → Dấu hiệu có gì đó sai     │
│  Xác minh           → Yêu cầu về bằng chứng       │
└─────────────────────────────────────────────────┘
```

**Các lựa chọn thiết kế chính:**

- **Quy trình, không phải văn bản.** Các skill là những luồng công việc mà agent tuân theo, không phải tài liệu tham khảo để đọc. Mỗi skill có các bước, điểm kiểm tra và tiêu chí kết thúc.
- **Chống biện hộ (Anti-rationalization).** Mỗi skill bao gồm một bảng các lý do bào chữa phổ biến mà agent sử dụng để bỏ qua các bước (ví dụ: \"Tôi sẽ thêm test sau\") kèm theo các đối luận đã được ghi chép.
- **Xác minh là điều bắt buộc.** Mỗi skill kết thúc bằng các yêu cầu về bằng chứng - test pass, kết quả build, dữ liệu runtime. \"Có vẻ đúng\" không bao giờ là đủ.
- **Tiết lộ dần (Progressive disclosure).** `SKILL.md` là điểm bắt đầu. Các tham chiếu hỗ trợ chỉ được tải khi cần thiết, giúp tối thiểu hóa việc sử dụng token.

---

## Cấu trúc dự án

```
agent-skills/
├── skills/                            # 24 skill (23 vòng đời + 1 meta)
│   ├── interview-me/                  #   Define
│   ├── idea-refine/                   #   Define
│   ├── spec-driven-development/       #   Define
│   ├── planning-and-task-breakdown/   #   Plan
│   ├── incremental-implementation/    #   Build
│   ├── context-engineering/           #   Build
│   ├── source-driven-development/     #   Build
│   ├── doubt-driven-development/      #   Build
│   ├── frontend-ui-engineering/       #   Build
│   ├── test-driven-development/       #   Build
│   ├── api-and-interface-design/      #   Build
│   ├── browser-testing-with-devtools/ #   Verify
│   ├── debugging-and-error-recovery/  #   Verify
│   ├── code-review-and-quality/       #   Review
│   ├── code-simplification/          #   Review
│   ├── security-and-hardening/        #   Review
│   ├── performance-optimization/      #   Review
│   ├── git-workflow-and-versioning/   #   Ship
│   ├── ci-cd-and-automation/          #   Ship
│   ├── deprecation-and-migration/     #   Ship
│   ├── documentation-and-adrs/        #   Ship
│   ├── observability-and-instrumentation/ # Ship
│   ├── shipping-and-launch/           #   Ship
│   └── using-agent-skills/            #   Meta: cách sử dụng bộ công cụ này
├── agents/                            # 4 persona chuyên gia
├── references/                        # 5 checklist bổ trợ
├── hooks/                             # Các hook vòng đời phiên làm việc
├── .claude/commands/                  # 8 câu lệnh slash (Claude Code)
├── .gemini/commands/                  # 8 câu lệnh slash (Gemini CLI)
├── commands/                          # 8 câu lệnh slash (Antigravity CLI)
├── plugin.json                        # Manifest plugin Antigravity
└── docs/                              # Hướng dẫn thiết lập cho từng công cụ
```

---

## Tại sao cần Agent Skills?

Các AI coding agent thường mặc định chọn đường ngắn nhất - điều này thường đồng nghĩa với việc bỏ qua spec, test, review bảo mật và các thực hành giúp phần mềm đáng tin cậy. Agent Skills cung cấp cho agent các quy trình làm việc có cấu trúc, áp đặt kỷ luật tương tự như cách các kỹ sư cấp cao triển khai code production.

Mỗi skill mã hóa những phán đoán kỹ thuật đắt giá: *khi nào* nên viết spec, *cái gì* cần test, *cách* review và *khi nào* ship. Đây không phải là những prompt chung chung - mà là những quy trình làm việc định hướng, có quan điểm rõ ràng, giúp phân biệt giữa chất lượng production và chất lượng prototype.

Các skill tích hợp các thực hành tốt nhất từ văn hóa kỹ thuật của Google — bao gồm các khái niệm từ [Software Engineering at Google](https://abseil.io/resources/swe-book) và [hướng dẫn thực hành kỹ thuật của Google](https://google.github.io/eng-practices/). Bạn sẽ tìm thấy Định luật Hyrum trong thiết kế API, Quy tắc Beyonce và kim tự tháp test trong kiểm thử, chuẩn kích thước thay đổi và tốc độ review trong review code, Hàng rào Chesterton trong đơn giản hóa, trunk-based development trong quy trình git, Shift Left và feature flags trong CI/CD, và một skill deprecation chuyên biệt coi code là nợ. Đây không phải là những nguyên tắc trừu tượng — chúng được nhúng trực tiếp vào các quy trình từng bước mà agent tuân theo.

---

## So sánh

Bạn thắc mắc điều này so với [Superpowers](https://github.com/obra/superpowers) hoặc [các skill của Matt Pocock](https://github.com/mattpocock/skills) như thế nào? Xem **[docs/comparison.md](docs/comparison.md)** để có cái nhìn trung thực, đối chiếu trực tiếp về cách cả ba được xây dựng khác nhau như thế nào và khi nào nên dùng cái nào — bao gồm cả link đến một thử nghiệm đối đầu có kiểm soát.

---

## Đóng góp

Các skill cần phải **cụ thể** (các bước có thể thực hiện, không phải lời khuyên mơ hồ), **có thể xác minh** (tiêu chí kết thúc rõ ràng với yêu cầu bằng chứng), **đã qua thử lửa** (dựa trên các quy trình thực tế), và **tối giản** (chỉ bao gồm những gì cần thiết để hướng dẫn agent).

Xem [docs/skill-anatomy.md](docs/skill-anatomy.md) để biết chi tiết định dạng và [CONTRIBUTING.md](CONTRIBUTING.md) để biết các hướng dẫn.

---

## Giấy phép

MIT - hãy sử dụng các skill này trong các dự án, đội ngũ và công cụ của bạn.
