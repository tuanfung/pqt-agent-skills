# Các mẫu Orchestration

Danh mục tham khảo về các mẫu orchestration của agent được repo này ủng hộ, cùng với các anti-pattern cần tránh. Hãy đọc tài liệu này trước khi thêm một slash command mới để điều phối nhiều persona, hoặc trước khi giới thiệu một persona mới "bao bọc" các persona hiện có.

Quy tắc cốt lõi: **người dùng (hoặc một slash command) là người điều phối (orchestrator). Các persona không gọi các persona khác.** Các skill là những bước bắt buộc trong workflow của một persona.

---

## Các mẫu được ủng hộ

### 1. Gọi trực tiếp (không orchestration)

Một persona, một góc nhìn, một artifact. Đây là lựa chọn mặc định và tiết kiệm nhất.

```
user → code-reviewer → report → user
```

**Sử dụng khi:** công việc là một góc nhìn duy nhất trên một artifact và bạn có thể mô tả nó trong một câu.

**Ví dụ:**
- "Review this PR" → `code-reviewer`
- "Find security issues in `auth.ts`" → `security-auditor`
- "What tests are missing for the checkout flow?" → `test-engineer`

**Chi phí:** một lượt tương tác (round trip). Đây là mức cơ sở mà bạn nên luôn dùng để so sánh với các mẫu orchestration.

---

### 2. Slash command đơn persona

Một slash command bao bọc một persona cùng với các skill của dự án. Giúp người dùng không phải giải thích lại workflow mỗi lần sử dụng.

```
/review → code-reviewer (với skill code-review-and-quality) → report
```

**Sử dụng khi:** việc gọi một persona duy nhất lặp đi lặp lại với cùng một thiết lập.

**Ví dụ trong repo này:** `/review`, `/test`, `/code-simplify`.

**Chi phí:** tương đương với gọi trực tiếp. Slash command chỉ đơn giản là một prompt đã được lưu.

**Dấu hiệu sai (Anti-signal):** nếu nội dung của slash command chủ yếu là "quyết định gọi persona nào", hãy xóa nó và để người dùng gọi persona trực tiếp.

---

### 3. Fan-out song song kèm merge

Nhiều persona cùng hoạt động trên một đầu vào đồng thời, mỗi persona tạo ra một báo cáo độc lập. Một bước merge (trong context của agent chính) sẽ tổng hợp chúng thành một quyết định duy nhất.

```
                    ┌─→ code-reviewer    ─┐
/ship → fan out  ───┼─→ security-auditor ─┤→ merge → go/no-go + rollback
                    └─→ test-engineer    ─┘
```

**Sử dụng khi:**
- Các sub-task thực sự độc lập (không chia sẻ trạng thái có thể thay đổi, không phụ thuộc thứ tự)
- Mỗi sub-agent được hưởng lợi từ cửa sổ context riêng
- Bước merge đủ nhỏ để nằm trong context chính
- Thời gian phản hồi thực tế (wall-clock latency) là quan trọng

**Ví dụ trong repo này:** `/ship`.

**Chi phí:** N context sub-agent song song + một lượt merge. Cao hơn gọi trực tiếp, nhưng thời gian thực nhanh hơn và tạo ra báo cáo tốt hơn vì mỗi sub-agent tập trung vào một góc nhìn duy nhất.

**Danh sách kiểm tra (checklist) trước khi áp dụng mẫu này:**
- [ ] Tôi có thể chạy tất cả các sub-agent cùng lúc mà không gặp vấn đề về thứ tự không?
- [ ] Mỗi persona có tạo ra một loại kết quả khác nhau, chứ không chỉ là cùng một kết quả từ một góc nhìn khác không?
- [ ] Bước merge có vừa với context còn lại của agent chính không?
- [ ] Thời gian chờ của người dùng có đủ lâu để việc chạy song song thực sự mang lại hiệu quả rõ rệt không?

Nếu bất kỳ câu trả lời nào là "không", hãy quay lại gọi trực tiếp hoặc sử dụng command đơn persona.

---

### 4. Pipeline tuần tự dưới dạng các slash command do người dùng điều khiển

Người dùng chạy các slash command theo một thứ tự xác định, truyền context (hoặc lịch sử commit) giữa chúng. Không có agent điều phối — chính người dùng LÀ người điều phối (orchestrator).

```
user chạy:  /spec  →  /plan  →  /build  →  /test  →  /review  →  /ship
```

**Sử dụng khi:** workflow có các phụ thuộc (mỗi bước cần đầu ra của bước trước) và sự đánh giá của con người giữa các bước mang lại giá trị.

**Ví dụ trong repo này:** toàn bộ vòng đời DEFINE → PLAN → BUILD → VERIFY → REVIEW → SHIP.

**Chi phí:** một context sub-agent cho mỗi bước. Miễn phí cho lớp điều phối vì không có agent điều phối.

**Tại sao không tự động hóa:** một 'lifecycle orchestrator' bằng LLM sẽ (a) làm mất đi những sắc thái giữa các bước vì phải tóm tắt để bàn giao, (b) bỏ qua các điểm kiểm tra của con người giúp phát hiện sớm các hướng đi sai, và (c) làm tăng gấp đôi chi phí token thông qua các lượt diễn giải lại (paraphrasing).

---

### 5. Cô lập nghiên cứu (bảo toàn context)

Khi một tác vụ yêu cầu đọc một lượng lớn tài liệu mà không nên làm nhiễu context chính, hãy tạo một sub-agent nghiên cứu chỉ trả về một bản tóm tắt (digest).

```
main agent → research sub-agent (đọc 50 files) → digest → main agent tiếp tục
```

**Sử dụng khi:**
- Session chính cần tập trung vào một tác vụ tiếp theo
- Kết quả điều tra nhỏ hơn nhiều so với đầu vào mà nó tiêu thụ
- Chất lượng quyết định tốt hơn khi agent chính có không gian để suy nghĩ sau đó

**Ví dụ:** "Tìm mọi vị trí gọi của API đã bị deprecated này trong toàn bộ monorepo," "Tóm tắt nội dung của 30 ADR này về caching."

**Chi phí:** một context sub-agent cô lập. Luôn xứng đáng khi phương án thay thế là nạp hàng trăm file vào context chính.

**Trên Claude Code, hãy sử dụng sub-agent `Explore` tích hợp sẵn** thay vì định nghĩa một persona nghiên cứu tùy chỉnh. `Explore` chạy trên Haiku, bị từ chối các công cụ write/edit, và được xây dựng chuyên biệt cho mẫu này. Chỉ định nghĩa sub-agent nghiên cứu tùy chỉnh khi `Explore` không phù hợp (ví dụ: bạn cần một system prompt đặc thù cho domain mà mô hình không thể tự suy luận).

---

## Khả năng tương thích với Claude Code

Danh mục này không phụ thuộc vào harness, nhưng hầu hết người đọc sẽ chạy nó trên Claude Code. Dưới đây là cách mỗi mẫu ánh xạ vào các thành phần cơ bản của Claude Code — và nơi nền tảng thực thi các quy tắc giúp chúng ta.

### Nơi lưu trữ persona

Các sub-agent của plugin nằm trong `agents/` tại root của plugin. Repo này là một plugin (`.claude-plugin/plugin.json`), vì vậy `agents/code-reviewer.md`, `agents/security-auditor.md`, và `agents/test-engineer.md` sẽ tự động được phát hiện khi plugin được kích hoạt. Không cần cấu hình đường dẫn.

### Sub-agents và Agent Teams

Claude Code có hai thành phần thực hiện song song. Mẫu 3 (fan-out song song kèm merge) ánh xạ tới **sub-agents**. Nếu bạn cần những đồng đội có thể trò chuyện với nhau, hãy sử dụng **Agent Teams**.

| | Sub-agents | Agent Teams |
|--|-----------|-------------|
| Điều phối | Agent chính fan-out, sub-agents chỉ báo cáo lại | Các đồng đội nhắn tin cho nhau, chia sẻ một danh sách tác vụ |
| Context | Mỗi sub-agent có cửa sổ context riêng | Mỗi đồng đội có cửa sổ context riêng |
| Khi nào sử dụng | Các tác vụ độc lập tạo ra báo cáo | Công việc cộng tác cần thảo luận |
| Trạng thái | Ổn định | Thử nghiệm — yêu cầu `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` |
| Chi phí | Thấp hơn | Cao hơn — mỗi đồng đội là một instance Claude riêng biệt |

**Các persona trong repo này hoạt động được ở cả hai chế độ.** Khi được tạo dưới dạng sub-agents (ví dụ: bởi `/ship`), chúng báo cáo kết quả cho session chính. Khi được tạo dưới dạng đồng đội (`Spawn a teammate using the security-auditor agent type…`), chúng có thể trực tiếp phản biện các kết quả của nhau. Định nghĩa persona là như nhau; chỉ có context khởi tạo là thay đổi.

Một điểm tinh tế: các trường frontmatter `skills` và `mcpServers` trong một persona được tuân thủ khi nó chạy như một sub-agent nhưng bị **bỏ qua khi nó chạy như một đồng đội** — các đồng đội tải skills và MCP servers từ cài đặt dự án và người dùng, giống như một session thông thường. Nếu một persona phụ thuộc vào một skill hoặc MCP server cụ thể, hãy cấu hình nó ở cấp độ session để có sẵn trong cả hai chế độ.

### Các quy tắc do nền tảng thực thi

Hai quy tắc trong danh mục này không chỉ là quy ước — Claude Code thực thi chúng:

- **"Sub-agents không thể tạo ra các sub-agents khác"** (trích nguyên văn từ tài liệu). Anti-pattern B (persona gọi persona) và Anti-pattern D (cây persona sâu) không thể tồn tại trên Claude Code theo thiết kế.
- **"Không có team lồng nhau"** — các đồng đội không thể tạo ra các team riêng của họ. Các anti-pattern tương tự bị chặn ở cấp độ team.

Điều này có nghĩa là bạn có thể áp dụng các mẫu trong danh mục này mà không lo lắng về việc những người đóng góp vô tình xây dựng các anti-pattern. Chúng sẽ đơn giản là không thể tải được.

### Các sub-agent tích hợp sẵn cần biết

Trước khi định nghĩa một sub-agent tùy chỉnh, hãy kiểm tra xem một trong những cái này có đáp ứng được vai trò đó không:

| Tích hợp sẵn | Mục đích |
|----------|---------|
| `Explore` | Tìm kiếm và phân tích codebase chỉ đọc. Sử dụng cho Mẫu 5 (cô lập nghiên cứu). |
| `Plan` | Nghiên cứu chỉ đọc trong chế độ plan. |
| `general-purpose` | Các tác vụ nhiều bước cần cả khám phá và sửa đổi. |

Đừng định nghĩa lại những cái này. Hãy xây dựng các persona chuyên gia (code-reviewer, security-auditor, test-engineer) chồng lên trên chúng.

### Hạn chế frontmatter cho các agent của plugin

Các sub-agent của plugin **không** hỗ trợ các trường frontmatter `hooks`, `mcpServers`, hoặc `permissionMode` — những trường này sẽ bị bỏ qua trong im lặng. Nếu một persona trong tương lai cần bất kỳ điều nào trong số đó, người dùng phải copy file vào `.claude/agents/` hoặc `~/.claude/agents/`.

Các trường CÓ hoạt động trong plugin agents là: `name`, `description`, `tools`, `disallowedTools`, `model`, `maxTurns`, `skills`, `memory`, `background`, `effort`, `isolation`, `color`, `initialPrompt`. Sử dụng `model` cho từng persona nếu bạn muốn tối ưu chi phí (ví dụ: Haiku cho quét coverage của `test-engineer`, Sonnet cho `code-reviewer`, Opus cho `security-auditor`).

### Khởi tạo nhiều sub-agent song song

Trong Claude Code, fan-out song song (Mẫu 3) yêu cầu thực hiện **nhiều cuộc gọi Agent tool trong một lượt phản hồi của assistant**. Các lượt tuần tự sẽ làm tuần tự hóa việc thực thi. `/ship` nêu rõ điều này. Bất kỳ command điều phối mới nào cũng nên làm tương tự.

---

## Ví dụ thực tế: Agent Teams để gỡ lỗi bằng giả thuyết đối lập

Ví dụ này cho thấy khi nào nên sử dụng **Agent Teams** thay vì fan-out sub-agent của `/ship`. Hai mẫu này nhìn qua thì có vẻ giống nhau — cả hai đều khởi tạo cùng ba persona — nhưng giá trị mang lại đến từ những điểm khác nhau.

### Kịch bản

> *Checkout thỉnh thoảng bị treo trong khoảng 30 giây trước khi hoàn tất. Việc này xảy ra khoảng 1 lần sau mỗi 50 session. Không có lỗi trong log. Bắt đầu xuất hiện sau bản release tuần trước.*

Các nguyên nhân gốc rễ tiềm năng (loại trừ lẫn nhau, tất cả đều khớp với triệu chứng):

1. Một race condition trong luồng xác nhận thanh toán mới
2. Một bước kiểm tra auth thỉnh thoảng bị rơi vào một cuộc gọi mạng đồng bộ chậm
3. Thiếu index cho một query có quy mô tăng theo kích thước giỏ hàng
4. Một API bên thứ ba không ổn định, nơi SDK thử lại trong im lặng trước khi hết thời gian chờ (timeout)

Một agent đơn lẻ sẽ chọn lý thuyết khả thi đầu tiên và ngừng điều tra. Một fan-out sub-agent kiểu `/ship` sẽ khiến mỗi persona báo cáo độc lập — nhưng các báo cáo của họ không bao giờ gặp nhau, vì vậy không có gì loại bỏ được các lý thuyết sai.

Đây chính là trường hợp mà tài liệu Agent Teams mô tả: *"Với nhiều điều tra viên độc lập tích cực cố gắng bác bỏ lẫn nhau, lý thuyết nào tồn tại sau cùng sẽ có khả năng cao hơn là nguyên nhân gốc rễ thực sự."*

### Tại sao đây không phải là việc của `/ship`

| | `/ship` (sub-agents) | Agent Teams |
|--|--------------------|-------------|
| Sub-agents nhìn thấy | Cùng một diff, các lăng kính khác nhau | Một danh sách tác vụ chung, tin nhắn của nhau |
| Đầu ra | Ba báo cáo độc lập → một bước merge | Tranh luận đối lập → đồng thuận về nguyên nhân gốc rễ |
| Đúng khi | Bạn muốn một kết luận về một artifact đã biết | Bạn muốn tìm artifact trong số các giả thuyết |

`/ship` là đưa ra phán quyết; Agent Teams là một cuộc điều tra.

### Thiết lập (một lần, cho mỗi môi trường)

Agent Teams đang trong giai đoạn thử nghiệm. Trong `~/.claude/settings.json`:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

Yêu cầu Claude Code v2.1.32 trở lên. Các persona trong repo này sẽ được tự động nhận diện — không cần viết file config team bằng tay.

### Prompt kích hoạt

Nhập vào session chính bằng ngôn ngữ tự nhiên:

```
Users report checkout hangs for ~30 seconds intermittently after last
week's release. No errors in logs.

Create an agent team to debug this with competing hypotheses. Spawn
three teammates using the existing agent types:

  - code-reviewer  — investigate race conditions and blocking calls
                     in the checkout code path
  - security-auditor — investigate auth checks, session handling,
                       and any synchronous network calls added recently
  - test-engineer  — propose tests that would distinguish between the
                     hypotheses and check coverage gaps in checkout

Have them message each other directly to challenge each other's
theories. Update findings as consensus emerges. Only converge when
two teammates agree they can disprove the others'.
```

Lead tạo ra ba đồng đội tham chiếu đến tên các persona hiện có. Nội dung persona được **nối thêm** vào system prompt của mỗi đồng đội như những hướng dẫn bổ sung (bên cạnh các hướng dẫn phối hợp team mà lead cài đặt); prompt kích hoạt ở trên sẽ trở thành tác vụ của họ.

### Điều gì xảy ra

1. Mỗi đồng đội chạy trong cửa sổ context riêng, khám phá codebase từ góc nhìn của chính mình.
2. Các đồng đội sử dụng `message` để gửi kết quả cho nhau trực tiếp. Lead không cần phải chuyển tiếp.
3. Danh sách tác vụ chung hiển thị ai đang điều tra cái gì — có thể xem bất kỳ lúc nào bằng `Ctrl+T` (chế độ in-process) hoặc trong một tmux pane (chế độ split).
4. Khi `code-reviewer` tìm thấy một `Promise.all` lẽ ra phải là tuần tự, nó nhắn tin cho `security-auditor` để xác nhận cuộc gọi auth không phải là một phần của race condition. `security-auditor` kiểm tra và trả lời — hoặc xác nhận race condition là vấn đề thực sự, hoặc đưa ra bằng chứng ngược lại.
5. `test-engineer` đề xuất một bài test tích hợp tập trung cho lý thuyết nào đang thắng thế, và team sẽ sử dụng nó để xác minh trước khi tuyên bố đồng thuận.
6. Lead tổng hợp kết quả cuối cùng và trình bày với bạn.

Bạn có thể ngắt lời bất kỳ đồng đội nào bằng cách chuyển đổi bằng `Shift+Down` và nhập văn bản — hữu ích để định hướng lại một điều tra viên đang đi sai đường.

### Khi nào cần dọn dẹp

Khi cuộc điều tra tìm ra nguyên nhân gốc rễ, hãy nói với lead:

```
Clean up the team
```

Luôn dọn dẹp thông qua lead, không phải qua đồng đội (theo tài liệu: các đồng đội thiếu context đầy đủ của team để dọn dẹp).

### Kỳ vọng chi phí

Ba đồng đội Sonnet chạy trong khoảng 10-15 phút điều tra sẽ tốn kém hơn đáng kể so với ba persona tương tự được tạo dưới dạng sub-agents bởi `/ship`. Lý do là vì *chất lượng kết luận* — đối với việc gỡ lỗi production nơi một bản sửa sai gây tốn kém, việc chi thêm token là một món hời. Đối với việc review PR thông thường, hãy trung thành với `/ship`.

### Anti-pattern trong kịch bản này

Đừng xây dựng lại điều này như một slash command `/debug` thực hiện fan-out sub-agents. Các sub-agent không thể nhắn tin cho nhau — bạn sẽ làm mất đi cuộc tranh luận đối lập tạo nên hiệu quả của mẫu này. Nếu một workflow thường xuyên xuất hiện, hãy ghi lại prompt kích hoạt ở trên như một snippet thay vì gói nó trong một slash command sử dụng sai sub-agents.

### Khi nào KHÔNG nên sử dụng Agent Teams

- Kết luận cho một diff đã biết để đưa lên production → sử dụng `/ship` (sub-agents).
- Một góc nhìn chuyên gia trên một artifact → gọi persona trực tiếp.
- Lifecycle tuần tự (spec → plan → build) → các slash command do người dùng điều khiển (Mẫu 4).
- Nghiên cứu nặng về đọc với một bản tóm tắt nhỏ → sub-agent `Explore` tích hợp sẵn.

Chỉ sử dụng Agent Teams khi các đồng đội **cần** phản biện lẫn nhau để đưa ra câu trả lời đúng.

---

## Các anti-pattern

### A. Persona điều hướng ('meta-orchestrator')

Một persona có nhiệm vụ quyết định sẽ gọi persona nào khác.

```
/work → router-persona → "this needs a review" → code-reviewer → router (paraphrases) → user
```

**Tại sao nó thất bại:**
- Lớp điều hướng thuần túy không mang lại giá trị chuyên môn
- Thêm hai bước diễn giải lại (paraphrasing hops) → mất thông tin + chi phí token tăng khoảng 2 lần
- Người dùng vốn đã biết họ muốn review; họ có thể gọi `/review` trực tiếp
- Lặp lại công việc mà các slash command và mapping ý định trong `AGENTS.md` đã thực hiện

**Thay vào đó:** thêm hoặc tinh chỉnh các slash command. Ghi lại mapping ý định → command trong `AGENTS.md`.

---

### B. Persona gọi persona khác

Một `code-reviewer` tự gọi `security-auditor` khi thấy code auth.

**Tại sao nó thất bại:**
- Các persona được thiết kế để tạo ra một góc nhìn duy nhất; việc nối chuỗi chúng làm mất đi điều đó
- Bản tóm tắt mà persona gọi truyền đi làm mất context mà persona được gọi cần
- Các kịch bản thất bại tăng lên (định dạng đầu ra của persona nào sẽ thắng? quy tắc của ai được áp dụng?)
- Che giấu chi phí đối với người dùng

**Thay vào đó:** hãy để persona gọi *đề xuất* một cuộc kiểm tra tiếp theo trong báo cáo của nó. Người dùng hoặc một slash command sẽ thực hiện lượt chạy thứ hai.

---

### C. Orchestrator tuần tự thực hiện diễn giải

Một agent thay mặt người dùng gọi `/spec`, sau đó là `/plan`, rồi `/build`, v.v.

**Tại sao nó thất bại:**
- Mất đi các điểm kiểm tra của con người giúp phát hiện sớm các hướng đi sai
- Mỗi lần bàn giao đều tóm tắt context — gây ra sự sai lệch tích lũy qua một pipeline dài
- Gấp đôi chi phí token: một lượt orchestrator + một lượt sub-agent cho mỗi bước
- Loại bỏ quyền kiểm soát của người dùng chính tại những điểm mà sự đánh giá là quan trọng nhất

**Thay vào đó:** hãy giữ người dùng là người điều phối. Ghi lại trình tự khuyến nghị trong `README.md` và để người dùng tự thực hiện.

---

### D. Cây persona sâu

`/ship` gọi một `pre-ship-coordinator`, cái này gọi một `quality-coordinator`, và cái đó gọi `code-reviewer`.

**Tại sao nó thất bại:**
- Mỗi lớp thêm vào độ trễ và token mà không mang lại giá trị quyết định
- Việc gỡ lỗi trở thành một cuộc điều tra nhiều cấp
- Các persona ở cấp lá bị mất context do nhiều bước tóm tắt

**Thay vào đó:** giữ độ sâu điều phối tối đa là 1 (slash command → persona). Việc merge diễn ra ở agent chính.

---

## Luồng quyết định

Khi cân nhắc một workflow điều phối mới, hãy đi theo luồng này:

```
Công việc có phải là một góc nhìn duy nhất trên một artifact không?
├── Có → Gọi trực tiếp. Dừng lại.
└── Không  → Cấu hình tương tự có lặp lại không?
         ├── Không  → Gọi trực tiếp, tùy hứng. Dừng lại.
         └── Có → Các sub-task có độc lập không?
                  ├── Không  → Các slash command tuần tự do người dùng chạy (Mẫu 4).
                  └── Có → Fan-out song song kèm merge (Mẫu 3).
                           Kiểm tra đối chiếu với checklist ở trên.
                           Nếu bất kỳ bước kiểm tra nào thất bại → quay lại command đơn persona (Mẫu 2).
```

---

## Khi nào nên thêm một mẫu mới vào danh mục này

Chỉ thêm một mục mới sau khi:

1. Bạn đã sử dụng mẫu này ít nhất hai lần trong công việc thực tế
2. Bạn có thể chỉ ra một artifact cụ thể trong repo này minh chứng cho nó
3. Bạn có thể giải thích tại sao một mẫu hiện có sẽ không hiệu quả
4. Bạn có thể mô tả anti-pattern tương ứng (điều mà mọi người sẽ dễ dàng xây dựng sai thay thế)

Các mục trong danh mục được thêm vào quá sớm sẽ trở thành tài liệu mang tính lý tưởng mà không ai tuân theo.
