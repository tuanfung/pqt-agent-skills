---
name: doubt-driven-development
description: Đưa mọi quyết định không tầm thường qua một đợt đánh giá đối kháng với ngữ cảnh mới (fresh-context adversarial review) trước khi chốt. Sử dụng khi tính chính xác quan trọng hơn tốc độ, khi làm việc trong mã nguồn không quen thuộc, khi rủi ro cao (môi trường production, logic nhạy cảm về bảo mật, các thao tác không thể đảo ngược), hoặc bất cứ khi nào mà việc xác minh một kết quả tự tin ở hiện tại sẽ rẻ hơn so với việc gỡ lỗi (debug) sau này.
---

# Phát triển hướng nghi vấn (Doubt-Driven Development)

## Tổng quan

Một câu trả lời tự tin không đồng nghĩa với một câu trả lời chính xác. Các phiên làm việc kéo dài tích lũy ngữ cảnh, điều này âm thầm biến các giả định thành "sự thật" mà không ai nhận ra. Phát triển hướng nghi vấn (doubt-driven development) là kỷ luật hiện thực hóa một người đánh giá với ngữ cảnh mới — thiên hướng **bác bỏ**, thay vì chấp thuận — trước khi bất kỳ kết quả không tầm thường nào được chấp nhận.

Đây không phải là `/review`. `/review` là một phán quyết về một sản phẩm đã hoàn thành. Đây là một tư thế trong khi thực hiện: các quyết định không tầm thường được chất vấn chéo trong khi việc điều chỉnh hướng đi vẫn còn rẻ.

## Khi nào nên sử dụng

Một quyết định được coi là **không tầm thường (non-trivial)** khi có ít nhất một trong các điều sau là đúng:

- Nó giới thiệu hoặc sửa đổi logic rẽ nhánh (branching logic)
- Nó vượt qua ranh giới của một module hoặc dịch vụ
- Nó khẳng định một đặc tính mà hệ thống kiểu (type system) hoặc trình biên dịch không thể xác minh (thread safety, idempotence, thứ tự, invariants)
- Tính chính xác của nó phụ thuộc vào ngữ cảnh mà người đọc trong tương lai không thể thấy
- Phạm vi ảnh hưởng (blast radius) của nó là không thể đảo ngược (triển khai production, di cư dữ liệu, thay đổi API công khai)

Áp dụng skill khi:

- Sắp đưa ra một quyết định kiến trúc trong tình trạng không chắc chắn
- Sắp commit mã nguồn không tầm thường
- Sắp khẳng định một sự thật không hiển nhiên ("điều này là an toàn", "điều này có khả năng mở rộng", "điều này khớp với spec")
- Làm việc trong mã nguồn mà bạn không hiểu hết

**Khi KHÔNG nên sử dụng:**

- Các thao tác cơ học (đổi tên, định dạng, di chuyển file)
- Tuân theo một hướng dẫn rõ ràng, không gây nhầm lẫn của người dùng
- Đọc hoặc tóm tắt mã nguồn hiện có
- Các thay đổi một dòng với tính chính xác hiển nhiên
- Các thao tác thuần túy về công cụ (chạy test, liệt kê file)
- Người dùng đã yêu cầu rõ ràng ưu tiên tốc độ hơn là xác minh

Nếu bạn nghi ngờ mọi phím bấm, bạn sẽ không bao giờ bàn giao được gì. Skill này chỉ áp dụng cho các quyết định không tầm thường như định nghĩa ở trên.

## Ràng buộc nạp (Loading Constraints)

Skill này được thiết kế cho **bộ điều phối phiên chính (main-session orchestrator)**, nơi Bước 3 (DOUBT, chi tiết bên dưới) có thể khởi tạo một người đánh giá với ngữ cảnh mới.

- **KHÔNG thêm skill này vào frontmatter `skills:` của một persona.** Một persona tuân theo Bước 3 sẽ khởi tạo một persona khác — đây là anti-pattern về điều phối bị cấm rõ ràng trong `references/orchestration-patterns.md` ("personas không được gọi các personas khác").
- **Nếu bạn thấy mình đang áp dụng skill này từ bên trong ngữ cảnh của một subagent** (nơi Claude Code ngăn chặn việc khởi tạo subagent lồng nhau): cách tốt nhất là thông báo cho người dùng rằng doubt-driven không thể chạy lồng nhau và để phiên chính xử lý. Như một phương án cuối cùng, tồn tại một phương án dự phòng tự vấn bị suy giảm — viết lại ARTIFACT + CONTRACT như một self-prompt mới với một vách ngăn tinh thần cứng nhắc với lập luận trước đó, và thực hiện các Bước 1–5. Đây **không phải là đánh giá ngữ cảnh mới (fresh-context review)** (vì bạn mang theo ngữ cảnh của chính mình), vì vậy hãy đánh dấu kết quả là bị suy giảm và ưu tiên leo thang (escalation) bất cứ khi nào có thể liên lạc với người dùng.

## Quy trình

Sao chép checklist này khi áp dụng skill:

```
Chu kỳ nghi vấn (Doubt cycle):
- [ ] Bước 1: CLAIM — viết khẳng định + lý do tại sao nó quan trọng
- [ ] Bước 2: EXTRACT — tách biệt artifact + contract, loại bỏ lập luận
- [ ] Bước 3: DOUBT — gọi người đánh giá ngữ cảnh mới với prompt đối kháng
- [ ] Bước 4: RECONCILE — phân loại mọi phát hiện so với văn bản artifact
- [ ] Bước 5: STOP — đạt điều kiện dừng (phát hiện tầm thường, 3 chu kỳ, hoặc người dùng ghi đè)
```

### Bước 1: CLAIM — Đưa ra điều cần xác nhận

Nêu tên quyết định trong hai hoặc ba dòng:

```
CLAIM: "Lớp caching mới là thread-safe trong điều kiện
        khối lượng đọc lớn như mô tả trong spec."
WHY THIS MATTERS: một race condition ở đây sẽ làm hỏng dữ liệu người dùng 
                  và rất khó phát hiện trong QA.
```

Nếu bạn không thể viết khẳng định một cách súc tích như vậy, bạn chỉ đang có một "cảm giác" chứ không phải một quyết định. Hãy làm rõ nó trước khi xem xét kỹ lưỡng.

### Bước 2: EXTRACT — Đơn vị đánh giá nhỏ nhất

Một người đánh giá ngữ cảnh mới cần **artifact** và **contract**, không cần hành trình đi đến đó.

- Code: phần diff hoặc hàm — không phải toàn bộ file
- Quyết định: đề xuất trong 3–5 câu cộng với các ràng buộc cần thỏa mãn
- Khẳng định: lời khẳng định cộng với bằng chứng được cho là hỗ trợ nó (giữ tách biệt với khối CLAIM ở Bước 1, vốn là giả thuyết của bộ điều phối đang bị xem xét)

Loại bỏ lập luận của bạn. Nếu bạn đưa ra các kết luận, bạn sẽ nhận lại sự xác nhận cho các kết luận đó. Đơn vị phải đủ nhỏ để người đánh giá có thể nắm bắt trong một lần đọc — nếu đó là một PR 500 dòng, hãy phân rã trước.

### Bước 3: DOUBT — Gọi người đánh giá ngữ cảnh mới

Prompt của người đánh giá **phải mang tính đối kháng**. Cách đặt vấn đề quyết định câu trả lời.

```
Đánh giá đối kháng. Hãy tìm điểm sai trong artifact này.
Giả định rằng tác giả đang quá tự tin. Hãy tìm kiếm:
- Các giả định không được nêu rõ
- Các trường hợp biên (edge cases) chưa được xử lý
- Sự phụ thuộc ngầm (hidden coupling) hoặc trạng thái chia sẻ (shared state)
- Các cách mà contract có thể bị vi phạm
- Các quy ước hiện có mà điều này có thể phá vỡ
- Các chế độ thất bại (failure modes) dưới đầu vào không mong đợi

KHÔNG xác nhận. KHÔNG tóm tắt. Hãy tìm vấn đề, hoặc nêu rõ 
rằng bạn không thể tìm thấy bất kỳ vấn đề nào sau khi xem xét kỹ lưỡng.

ARTIFACT: <dán artifact>
CONTRACT: <dán contract>
```

**Chỉ truyền ARTIFACT + CONTRACT. KHÔNG truyền CLAIM.** Việc đưa kết luận cho người đánh giá sẽ khiến họ thiên hướng đồng ý. Người đánh giá phải độc lập xác định xem artifact có thỏa mãn contract hay không.

Trong Claude Code, các người đánh giá dựa trên vai trò trong `agents/` được thiết kế để bắt đầu với ngữ cảnh cô lập và có thể sử dụng ở đây — xem `agents/` để biết danh sách và mức độ phù hợp theo miền.

**Prompt đối kháng ở trên có ưu tiên cao hơn hình thái phản hồi mặc định của persona.** Các persona như `code-reviewer` được viết để đưa ra phán quyết cân bằng với cả điểm mạnh và điểm yếu; doubt-driven chỉ cần kết quả là các vấn đề. Hãy dán nguyên văn prompt đối kháng vào lời gọi để nó ghi đè mặc định của persona. Nếu hình thái phản hồi của persona không thể ghi đè một cách sạch sẽ, hãy chuyển sang một subagent chung với prompt đối kháng.

#### Leo thang đa mô hình (Cross-model escalation)

Một người đánh giá đơn mô hình sẽ chia sẻ cùng các điểm mù với tác giả gốc — một mô hình khác với kiến trúc khác, "lạnh" hơn sẽ phát hiện ra chúng. Doubt-driven đã là tùy chọn cho các quyết định không tầm thường, vì vậy trong phạm vi đó, việc cung cấp đánh giá đa mô hình là một phần giá trị của skill, không phải là sự cản trở tùy chọn.

**Các phiên tương tác: luôn đề xuất. Không bao giờ lặng lẽ bỏ qua.**

**Bước 1: Hỏi người dùng**

Sau khi đánh giá đơn mô hình trong Bước 3 ở trên, nhưng trước khi RECONCILE, hãy tạm dừng và hỏi:

> *"Đã hoàn thành đánh giá đơn mô hình. Bạn có muốn một ý kiến thứ hai từ mô hình khác không? Các lựa chọn: Gemini CLI, Codex CLI, đánh giá thủ công bên ngoài (bạn tự dán vào nơi khác), hoặc bỏ qua."*

Câu hỏi này là bắt buộc trong mọi chu kỳ nghi vấn tương tác — ngay cả với các artifact có vẻ ít rủi ro. Người dùng — chứ không phải agent — quyết định liệu chi phí có xứng đáng hay không. Việc của agent là đưa ra lựa chọn.

**Bước 2: Nếu người dùng chọn CLI — xác minh, sau đó gọi**

1. Kiểm tra công cụ có trong PATH không (`which gemini`, `which codex`).
2. Kiểm tra xem nó có hoạt động không (`gemini --version` hoặc tương đương) trước khi truyền toàn bộ prompt — một binary cũ hoặc hỏng có thể vượt qua `which` nhưng thất bại khi chạy thực tế.
3. Xác nhận lệnh gọi chính xác với người dùng, bao gồm các cờ (flags) cần thiết, xác thực và biến môi trường (ví dụ: API keys). Các bản triển khai khác nhau; đừng bao giờ giả định.
4. Chỉ truyền ARTIFACT + CONTRACT + prompt đối kháng. Không có ngữ cảnh phiên, không có CLAIM.
5. Chú ý escape shell. Nếu artifact chứa dấu ngoặc kép, `$(...)`, hoặc backticks, hãy ưu tiên dùng stdin (`echo … | gemini`) hoặc heredoc thay vì inline `-p "…"`. Khi nghi ngờ, hãy yêu cầu người dùng xác nhận lệnh gọi trước khi chạy.
6. Đưa kết quả vào Bước 4 (RECONCILE).

**Không bao giờ nội suy artifact vào một đối số được bao quanh bởi dấu ngoặc kép của shell.** Code, markdown và review prompt thường xuyên chứa backticks, `$(...)` và các ký tự ngoặc kép có thể làm cắt ngắn prompt hoặc thực thi shell nhúng. Hãy viết toàn bộ prompt vào một file và pipe nó qua stdin.

Ví dụ về hình thái (xác minh các cờ với công cụ đã cài đặt của bạn — cú pháp khác nhau tùy theo bản triển khai và phiên bản):

```bash
# Viết prompt đối kháng + ARTIFACT + CONTRACT vào file tạm trước.
# Sau đó pipe qua stdin để các ký tự đặc biệt của shell trong artifact giữ nguyên.

# Codex (sandbox read-only ngăn CLI ghi vào workspace của bạn):
codex exec --sandbox read-only -C <repo-path> - < /tmp/doubt-prompt.md

# Gemini ('--approval-mode plan' là read-only; '-p ""' kích hoạt chế độ 
# không tương tác và prompt được đọc từ stdin):
gemini --approval-mode plan -p "" < /tmp/doubt-prompt.md
```

Một sandbox read-only là chi tiết then chốt: một doubt artifact có thể chính nó chứa các hướng dẫn (cố ý hoặc vô tình prompt injection) mà CLI đa mô hình nếu không có sandbox sẽ thực thi đối với workspace của bạn.

**Bước 3: Nếu CLI không khả dụng hoặc thất bại**

Thông báo thất bại rõ ràng. Đề xuất: chạy thủ công, thử công cụ khác, hoặc bỏ qua. Không được lặng lẽ quay lại đơn mô hình — người dùng cần biết đa mô hình đã không diễn ra.

**Bước 4: Nếu người dùng bỏ qua**

Xác nhận việc bỏ qua trong kết quả (*"Tiếp tục với các phát hiện từ đơn mô hình"*) và chuyển sang RECONCILE. Bỏ qua là bình thường; lặng lẽ bỏ qua thì không.

**Ngữ cảnh không tương tác** (CI, `/loop`, autonomous-loop, các lượt chạy theo lịch):

- Đa mô hình bị **bỏ qua**, và việc bỏ qua phải được **thông báo** trong kết quả: *"Đã bỏ qua đa mô hình: ngữ cảnh không tương tác."*
- **Không bao giờ gọi một CLI bên ngoài mà không có sự cho phép rõ ràng của người dùng** — đây là thuộc tính an toàn then chốt.

Đa mô hình làm tăng chi phí, độ trễ và sự mong manh của công cụ. Agent đưa ra lựa chọn trong mỗi chu kỳ; người dùng quyết định liệu artifact này có xứng đáng hay không.

### Bước 4: RECONCILE — Hợp nhất các phát hiện

Kết quả của người đánh giá là dữ liệu, không phải phán quyết. **Bạn vẫn là bộ điều phối.** Hãy đọc lại văn bản artifact đối chiếu với từng phát hiện trước khi phân loại — phê duyệt hời hợt (rubber-stamping) kết quả của người đánh giá cũng là một kiểu thất bại tương tự như việc phớt lờ nó.

Với mỗi phát hiện, hãy phân loại theo **thứ tự ưu tiên** này (lớp khớp đầu tiên sẽ thắng):

1. **Hiểu sai Contract (Contract misread)** — người đánh giá gắn cờ điều gì đó cụ thể vì CONTRACT bạn cung cấp không rõ ràng hoặc không đầy đủ. Sửa contract trước, phân loại lại trong chu kỳ tiếp theo.
2. **Hợp lệ + có thể xử lý (Valid + actionable)** — vấn đề thực sự yêu cầu thay đổi artifact. Thay đổi nó, lặp lại chu kỳ.
3. **Đánh đổi hợp lệ (Valid trade-off)** — vấn đề là thực, nhưng chi phí sửa chữa vượt quá chi phí chấp nhận. Tài liệu hóa sự đánh đổi một cách rõ ràng để người dùng thấy.
4. **Nhiễu (Noise)** — người đánh giá gắn cờ điều gì đó thực chất là chính xác trong ngữ cảnh mà người đánh giá không có. Ghi chú lại, bỏ qua, và hỏi: liệu việc thêm ngữ cảnh đó vào contract có ngăn chặn được cảnh báo sai này không?

Một người đánh giá mới có thể sai vì thiếu ngữ cảnh. Đừng phó mặc chỉ vì họ là người "mới".

### Bước 5: STOP — Vòng lặp có giới hạn, không phải đệ quy

Dừng khi:

- Lượt lặp tiếp theo chỉ trả về các phát hiện tầm thường hoặc đã được xem xét, **hoặc**
- Đã hoàn thành 3 chu kỳ (leo thang lên người dùng, đừng tự mình xoáy sâu vào lượt thứ tư), **hoặc**
- Người dùng nói rõ "ship it" (triển khai đi)

Nếu sau 3 chu kỳ, người đánh giá vẫn đưa ra các vấn đề thực chất, artifact có thể chưa sẵn sàng. Hãy thông báo điều này cho người dùng — ba chu kỳ không giải quyết được là thông tin về artifact, không phải là lý do để tiếp tục lặp.

Nếu 3 chu kỳ là "rõ ràng không đủ" vì artifact quá lớn: artifact quá to — quay lại Bước 2 và phân rã. Không được nới lỏng giới hạn này.

## Các lý lẽ biện hộ phổ biến

| Lý lẽ biện hộ | Thực tế |
|---|---|
| "Tôi tự tin, bỏ qua bước nghi vấn đi" | Sự tự tin tương quan kém với tính chính xác trong các vấn đề mới. Những khoảnh khắc chắc chắn chính là lúc các điểm mù ẩn nấp. |
| "Khởi tạo người đánh giá rất tốn kém" | Gỡ lỗi một commit sai trên production còn tốn kém hơn. Việc kiểm tra có giới hạn; lỗi thì không. |
| "Người đánh giá sẽ chỉ bắt bẻ vụn vặt" | Chỉ xảy ra nếu không giới hạn phạm vi. Hãy giới hạn prompt vào "các vấn đề khiến điều này thất bại theo contract." |
| "Tôi sẽ làm doubt ở cuối cùng với `/review`" | `/review` là cánh cửa cuối cùng. Doubt-driven bắt được các hướng đi sai sớm khi việc điều chỉnh còn rẻ. Đến lúc PR thì đã quá muộn. |
| "Nếu tôi nghi ngờ mọi bước, tôi sẽ không bao giờ ship được gì" | Skill này áp dụng cho các quyết định không tầm thường, không phải mọi phím bấm. Hãy đọc lại "Khi KHÔNG nên sử dụng." |
| "Hai ý kiến luôn tốt hơn một" | Không phải khi ý kiến thứ hai có ít ngữ cảnh hơn và tạo ra nhiễu. Hãy đối chiếu (reconcile), đừng phó mặc. |
| "Người đánh giá không đồng ý nên tôi đã sai" | Người đánh giá thiếu ngữ cảnh của bạn — sự bất đồng là thông tin, không phải phán quyết. Đọc lại artifact, phân loại, sau đó quyết định. |
| "Đa mô hình luôn tốt hơn" | Đa mô hình bắt được các điểm mù mà một mô hình chia sẻ với chính nó, nhưng nó làm tăng chi phí và sự mong manh của công cụ. Hãy đề xuất trong mỗi chu kỳ tương tác — người dùng quyết định liệu artifact có xứng đáng hay không. Việc của agent là đưa ra lựa chọn, không phải là ngăn chặn nó. |
| "Người dùng đã đồng ý một lần, nên tôi có thể tiếp tục gọi CLI" | Mỗi lần gọi là một lần cho phép riêng. Artifact, prompt và các cờ thay đổi giữa các lần gọi — hãy xác nhận lại lệnh gọi chính xác với người dùng trước mỗi lần chạy. |

## Dấu hiệu cảnh báo (Red Flags)

- Khởi tạo người đánh giá ngữ cảnh mới cho một thay đổi đổi tên một dòng hoặc định dạng
- Coi kết quả của người đánh giá là có thẩm quyền mà không đọc lại văn bản artifact
- Lặp >3 chu kỳ mà không leo thang lên người dùng
- Prompt người đánh giá bằng câu "cái này có tốt không?" thay vì "tìm vấn đề"
- Bỏ qua nghi vấn dưới áp lực thời gian đối với một quyết định rủi ro cao
- Khởi tạo lại ngữ cảnh mới cho một artifact không thay đổi (bạn sẽ nhận được cùng các phát hiện; bạn đang kéo dài thời gian)
- **Vở kịch nghi vấn (dấu hiệu có thể kiểm tra)**: qua 2 hoặc nhiều chu kỳ mà người đánh giá đưa ra các phát hiện thực chất, nhưng không có phát hiện nào được phân loại là có thể xử lý (actionable). Bạn đang xác nhận, không phải nghi vấn. Hãy dừng lại và leo thang.
- Chỉ nghi vấn sau khi commit — đó là `/review`, không phải phát triển hướng nghi vấn
- Hardcode lệnh gọi CLI bên ngoài mà không xác nhận với người dùng rằng công cụ tồn tại, đã được cấu hình và chấp nhận đúng cú pháp đó
- **Lặng lẽ bỏ qua đa mô hình trong một chu kỳ nghi vấn tương tác.** Ngay cả khi không khuyến khích, lời đề nghị phải hiển hiện. Bỏ qua là bình thường; lặng lẽ bỏ qua thì không.
- Lặng lẽ quay lại khi CLI bên ngoài báo lỗi hoặc bị thiếu — hãy thông báo thất bại và để người dùng điều hướng
- Loại bỏ contract khỏi đầu vào của người đánh giá
- Truyền CLAIM cho người đánh giá (gây thiên kiến đồng ý)

## Tương tác với các Skill khác

- **`code-review-and-quality` / `/review`**: bổ trợ cho nhau. `/review` là phán quyết PR sau sự việc; doubt-driven là theo từng quyết định trong khi thực hiện. Hãy sử dụng cả hai.
- **`source-driven-development`**: SDD xác minh *các sự thật về framework* đối với tài liệu chính thức. Doubt-driven xác minh *lập luận của bạn về artifact*. SDD kiểm tra xem API có tồn tại hay không; doubt-driven kiểm tra xem bạn có sử dụng nó đúng theo contract hay không.
- **`test-driven-development`**: bước RED của TDD là sự nghi vấn được cụ thể hóa — một bài test thất bại là một nỗ lực bác bỏ. Khi TDD được áp dụng, bài test thất bại đó *chính là* bước nghi vấn cho các khẳng định về hành vi.
- **`debugging-and-error-recovery`**: khi người đánh giá phát hiện một chế độ thất bại thực sự, hãy chuyển sang skill debugging để định vị và sửa chữa.
- **Quy tắc điều phối repo** (`references/orchestration-patterns.md`): skill này điều phối từ phiên chính. Một persona gọi một persona khác là anti-pattern B — xem Ràng buộc nạp ở trên.

## Xác minh

Sau khi áp dụng phát triển hướng nghi vấn:

- [ ] Mọi quyết định không tầm thường (theo định nghĩa trên) đã được nêu rõ ràng là một CLAIM trước khi chốt
- [ ] Ít nhất một đợt đánh giá ngữ cảnh mới cho mỗi artifact không tầm thường (một bài test thất bại do bước RED của TDD tạo ra sẽ thỏa mãn điều này cho các khẳng định hành vi, theo phần Tương tác với các Skill khác)
- [ ] Người đánh giá đã nhận được ARTIFACT + CONTRACT — KHÔNG PHẢI CLAIM, KHÔNG PHẢI lập luận của bạn
- [ ] Prompt của người đánh giá là đối kháng ("tìm vấn đề"), không phải xác nhận ("nó có tốt không")
- [ ] Các phát hiện được phân loại đối chiếu với văn bản artifact (không phê duyệt hời hợt) theo thứ tự ưu tiên: hiểu sai contract / có thể xử lý / đánh đổi / nhiễu
- [ ] Một điều kiện dừng đã được đáp ứng (phát hiện tầm thường, 3 chu kỳ, hoặc người dùng ghi đè)
- [ ] Trong chế độ tương tác, đa mô hình đã được **đề xuất rõ ràng** cho người dùng (bất kể mức độ rủi ro của artifact) và phản hồi đã được ghi nhận trong kết quả
- [ ] Trong chế độ không tương tác, đa mô hình đã bị bỏ qua và việc bỏ qua đã được thông báo
- [ ] Mọi lệnh gọi CLI bên ngoài đều được thực hiện sau khi kiểm tra PATH, kiểm tra binary hoạt động, xác nhận cú pháp với người dùng và có sự cho phép rõ ràng để chạy
