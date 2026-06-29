---
name: interview-me
description: Trích xuất điều người dùng thực sự muốn thay vì điều họ nghĩ rằng mình nên muốn. Đạt được điều này thông qua việc phỏng vấn theo kiểu mỗi lần một câu hỏi cho đến khi đạt được độ tin cậy khoảng 95% về ý định cốt lõi. Sử dụng khi một yêu cầu không được mô tả chi tiết ("build me X" mà không nói "cho ai" hoặc "tại sao bây giờ"), khi người dùng yêu cầu trực tiếp ("interview me", "grill me", "are we sure?", "stress-test my thinking"), hoặc khi bạn nhận thấy mình đang âm thầm tự điền các yêu cầu mơ hồ trước khi có bất kỳ kế hoạch, đặc tả hoặc mã nguồn nào.
---

# Interview Me

## Tổng quan

Điều mọi người yêu cầu và điều họ thực sự muốn thường là hai thứ khác nhau. Họ yêu cầu "một bảng điều khiển" (dashboard) vì đó là thứ mà người ta thường yêu cầu, chứ không phải vì một dashboard giải quyết được vấn đề của họ. Họ nói "làm cho nó nhanh hơn" mà không đưa ra một con số cụ thể để đạt được.

Thời điểm rẻ nhất để tìm ra khoảng cách này là trước khi có bất kỳ kế hoạch, đặc tả hoặc mã nguồn nào. Một khi bạn đã bắt đầu xây dựng, chi phí chuyển đổi là có thật, và người dùng sẽ hợp lý hóa thứ sai lầm thành một thứ "đủ tốt". Sự không phù hợp sẽ bị khóa chặt.

Skill này thu hẹp khoảng cách đó trước khi nó gây tốn kém. Các skill trong giai đoạn Define khác giả định rằng bạn đã biết sơ bộ mình muốn gì: `idea-refine` tạo ra các biến thể từ một ý tưởng, `spec-driven-development` viết ra các yêu cầu, `doubt-driven-development` kiểm tra áp lực (stress-test) một kế hoạch sau khi bạn đã phác thảo. interview-me là phần trước tất cả những điều đó, nơi bạn đặt từng câu hỏi một, kèm theo dự đoán tốt nhất của mình, cho đến khi bạn có thể dự đoán điều người dùng sắp nói trước khi họ nói ra.

## Khi nào nên sử dụng

Áp dụng skill này khi:

- Yêu cầu thiếu ít nhất một trong các thông tin: **ai** là người dùng, **tại sao** họ muốn nó, **thế nào là thành công**, hoặc **ràng buộc** chính là gì.
- Yêu cầu mang tính rập khuôn hơn là cụ thể ("build me X", "make it faster") và bạn không thể phân tích rập khuôn đó mà không phải đoán.
- Bạn bị cám dỗ bắt đầu với những giả định mà bạn chưa đưa ra thảo luận.
- Người dùng chưa nói rõ họ đang tối ưu hóa giá trị nào khi có hai giá trị hợp lý đang xung đột (sự đơn giản vs. sự linh hoạt, chi phí vs. tốc độ).
- Người dùng yêu cầu trực tiếp: "interview me", "grill me", "before we start, are we sure?", "stress-test my thinking".

**Khi KHÔNG nên sử dụng:**

- Yêu cầu rõ ràng và độc lập ("rename this variable", "fix this typo").
- Người dùng yêu cầu rõ ràng ưu tiên tốc độ hơn là xác minh.
- Yêu cầu thuần túy về thông tin ("how does X work?", "what does this code do?").
- Các thao tác cơ học (đổi tên, định dạng, di chuyển file).
- Bạn đã có độ tin cậy ≥95%; hãy đọc lại điều kiện dừng bên dưới trước khi giả định rằng bạn chưa đạt được.

## Ràng buộc vận hành

Skill này cần một người dùng đang tương tác trực tiếp và phản hồi nhanh. **Không kích hoạt trong các ngữ cảnh không tương tác** như CI pipelines, các lần chạy theo lịch trình, `/loop`, hoặc autonomous-loop. Nếu bạn đang ở trong những ngữ cảnh đó và yêu cầu không được mô tả chi tiết, hãy gắn cờ đó là một vật cản (blocker) cho người dùng thay vì tự đoán.

## Quy trình

### Bước 1: Đưa ra giả thuyết, kèm theo con số tin cậy

Trước khi hỏi bất cứ điều gì, hãy viết ra cách hiểu tốt nhất hiện tại của bạn về điều người dùng muốn trong **một câu**, cộng với một con số tin cậy trung thực (0–100%):

```
HYPOTHESIS: Bạn muốn một cách để trả lời câu hỏi "chúng ta đang hoạt động thế nào?" trong buổi standup, và "dashboard" là từ thông thường nảy ra trong đầu.
CONFIDENCE: ~30% — thiếu: đối tượng sử dụng, ý nghĩa của "metrics" trong ngữ cảnh này, và thế nào là thành công.
```

Con số này buộc bạn phải trung thực. Nếu bạn viết một con số cao nhưng thực tế không thể dự đoán phản ứng của người dùng đối với ba câu hỏi tiếp theo mà bạn định hỏi, thì con số đó là sai. Hãy bắt đầu ở mức độ tin cậy mà bạn có thể bảo vệ được.

Khi độ tin cậy dưới ~70%, hãy viết thêm một lý do ngắn gọn trên cùng một dòng — điều gì vẫn chưa được giải quyết hoặc còn thiếu. Điều này cho người dùng biết chính xác cuộc phỏng vấn cần làm rõ điều gì, và ngăn con số trở thành một tín hiệu mơ hồ.

### Bước 2: Hỏi từng câu một, mỗi câu đính kèm một phỏng đoán

Định dạng:

```
Q: <một câu hỏi tập trung>
GUESS: <giả thuyết của bạn cho câu trả lời, kèm theo lập luận tạo ra nó>
```

Đợi người dùng phản hồi trước khi hỏi câu tiếp theo.

**Tại sao lại hỏi từng câu một, không hỏi theo nhóm:**

- Người dùng không thể phản ứng với các giả thuyết của bạn nếu bạn vùi chúng trong một danh sách.
- Việc hỏi theo nhóm khuyến khích đọc lướt và đưa ra các câu trả lời hời hợt.
- Câu hỏi thứ ba thường phụ thuộc vào câu trả lời của câu thứ nhất; hỏi tất cả cùng lúc sẽ khóa chặt khung tư duy sai.
- Năng lượng để suy nghĩ kỹ của người dùng là có hạn; hãy sử dụng nó cho từng câu hỏi một.

**Tại sao đính kèm phỏng đoán:**

- Người dùng phản ứng nhanh hơn với một phỏng đoán sai hơn là tự tạo ra một câu trả lời từ đầu.
- Nó gắn bạn với một giả thuyết mà bạn có thể bị sai một cách rõ ràng, điều này giúp bạn trung thực hơn.
- Nó làm lộ ra các giả định của *bạn*, vốn là điều mà cuộc phỏng vấn nhằm mục đích phơi bày.

Rủi ro ở đây là một người dùng lịch sự sẽ đồng ý với phỏng đoán của bạn chỉ để dễ chịu. Hãy giảm thiểu điều này bằng cách thể hiện sự sẵn lòng bị sai, và thỉnh thoảng phỏng đoán theo hướng mà bạn dự đoán người dùng sẽ phản đối.

### Bước 3: Lắng nghe sự khác biệt giữa "muốn" và "nên muốn"

Những câu trả lời nguy hiểm nhất là những câu mà người dùng nói ra điều mà một câu trả lời thấu đáo *nên nghe như thế nào* thay vì điều họ thực sự muốn. Hãy cảnh giác với:

- Các câu trả lời khớp với các mô típ về "best-practice" ("Tôi muốn nó có khả năng mở rộng - scalable", "clean architecture") mà không có chi tiết cụ thể.
- Các câu trả lời dựa vào quy ước ("theo cách mà hầu hết các ứng dụng làm", "cách tiếp cận tiêu chuẩn").
- Các cụm từ như "Tôi có lẽ nên...", "Tôi nghĩ tôi phải...", "thực hành kỹ thuật tốt nói rằng...".
- Sử dụng buzzwords làm mục tiêu — khi "hiện đại", "scalable", "robust" là câu trả lời thay vì một kết quả cụ thể.

Khi bạn nghe thấy những điều này, câu hỏi cần đặt ra là:

> *"Nếu bạn không phải giải trình điều này với bất kỳ ai, bạn thực sự muốn điều gì?"*

Một câu hỏi duy nhất đó thường mang lại hiệu quả hơn năm câu hỏi trước đó.

### Bước 4: Phát biểu lại ý định bằng ngôn ngữ của người dùng

Khi độ tin cậy của bạn cao, hãy viết lại điều bạn nghĩ người dùng muốn. Giữ cho nó ngắn gọn (5–8 dòng), sử dụng ngôn ngữ của họ nếu có thể, và cấu trúc sao cho người dùng có thể xác nhận hoặc chỉnh sửa theo từng dòng:

```
Đây là những gì tôi hiểu bạn muốn:

- Outcome (Kết quả):      <một dòng>
- User (Người dùng):         <một dòng — ai là người hưởng lợi>
- Why now (Tại sao bây giờ):      <một dòng — điều gì đã thay đổi>
- Success (Thành công):      <một dòng — làm sao để biết nó đã hoạt động>
- Constraint (Ràng buộc):   <một dòng — giới hạn ràng buộc>
- Out of scope (Ngoài phạm vi): <một dòng — điều chúng ta rõ ràng sẽ không làm>

Đúng / không / cần điều chỉnh?
```

Việc bao gồm "Out of scope" là không thể thương lượng. Một nửa sự không khớp là do sự không đồng thuận ngầm về những gì *không* được xây dựng.

### Bước 5: Xác nhận — phải là "có" rõ ràng, không phải "sao cũng được"

Cánh cửa thông qua là một từ "có" rõ ràng. Những điều sau đây **không phải** là "có":

- "Sao cũng được, tùy bạn nghĩ cái nào tốt nhất." → Người dùng đang giao phó, nghĩa là họ cũng không có độ tin cậy 95%. Hãy hỏi lại với hai lựa chọn cụ thể dưới dạng một sự lựa chọn.
- "Nghe có vẻ ổn." → Mơ hồ. Hãy hỏi: "Có điều gì bạn muốn điều chỉnh không?" Sự im lặng không phải là xác nhận.
- "Chắc rồi, triển thôi." → Thường là một lời chào lịch sự để kết thúc, không phải là sự tán thành. Sử dụng cách theo dõi tương tự.
- Im lặng sau đó là "ok bắt đầu đi." → Người dùng đã bỏ cuộc với cuộc phỏng vấn, chứ không phải đã đạt được sự thống nhất. Hãy dừng lại và hỏi liệu bạn có bỏ lỡ điều gì không.

Nếu họ chỉnh sửa bạn, hãy tiếp nhận chỉnh sửa đó và phát biểu lại. Lặp lại cho đến khi bạn nhận được một câu trả lời "có" rõ ràng.

### Điểm dừng Tự tin 95%

Bạn hoàn thành khi có thể trả lời "có" cho câu hỏi này:

> *Tôi có thể dự đoán phản ứng của người dùng đối với ba câu hỏi tiếp theo mà tôi sẽ hỏi không?*

Nếu có, bạn đã có sự hiểu biết chung. Dừng phỏng vấn và đưa ra bản phát biểu lại. Nếu không, bạn chưa xong; hãy hỏi câu tiếp theo.

Đây là một bài kiểm tra có thể kiểm chứng, không phải là một cảm giác. Nó cũng có một mức sàn: nếu bạn đã trải qua nhiều vòng mà vẫn không thể dự đoán, đó là thông tin về yêu cầu, không phải lý do để tiếp tục đào sâu. Hãy dừng lại và nói với người dùng: "Tôi đã hỏi X câu hỏi và tôi vẫn không thể dự đoán phản ứng của bạn. Có điều gì đó cơ bản đang bị thiếu. Bạn có muốn lùi lại một bước không?"

## Kết quả đầu ra

Kết quả của skill này là một **phát biểu ý định đã được xác nhận**: bản phát biểu lại từ Bước 4, với một câu trả lời "có" rõ ràng từ Bước 5. Đó là sản phẩm bàn giao. Specs, plans và task lists là các bước tiếp theo; chúng tiêu thụ ý định mà skill này tạo ra.

Nếu người dùng muốn ý định này được lưu giữ (một dự án nhiều phiên làm việc, bàn giao cho một cộng tác viên khác), hãy đề nghị lưu vào `docs/intent/[topic].md`. Chỉ lưu nếu họ xác nhận.

## Ví dụ

Một so sánh ngắn trước và sau.

**Khi không có `interview-me`:**

```
User:  build me a dashboard for our metrics
Agent: <bắt đầu đề xuất các thư viện biểu đồ và bố cục>
```

Agent đã giả định: đối tượng sử dụng, metrics nào, "cho team chúng tôi" nghĩa là gì, thế nào là thành công. Không điều nào trong số này được nêu ra. Mỗi giả định là một nơi mà sau này người dùng có thể nói "đó không phải là ý tôi".

**Khi có `interview-me`:**

```
User: build me a dashboard for our metrics

Agent:
HYPOTHESIS: Bạn muốn một cách để trả lời câu hỏi "chúng ta đang hoạt động thế nào?" trong buổi standup, và "dashboard" nảy ra trong đầu vì đó là câu trả lời thông thường.
CONFIDENCE: ~30% — thiếu: đối tượng sử dụng, metrics là gì, và thế nào là thành công.

Q:     Khi bạn nói "chúng ta đang hoạt động thế nào?", ai là người hỏi — chỉ mình bạn, đội ngũ kỹ thuật trong buổi standup, hay cấp cao hơn (ví dụ: quản lý, điều hành, hội đồng quản trị)?
GUESS: đội ngũ kỹ thuật trong buổi standup, vì từ "chúng ta" thường được hiểu theo hướng đó và standup là nơi câu hỏi này thường được đặt ra. Nếu dành cho ban điều hành, các metrics và khung tư duy sẽ thay đổi rất nhiều.

User: Thực ra là cho tôi. Tôi thường xuyên bị mất dấu xem những thí nghiệm nào đang chạy và tín hiệu sớm của chúng trông như thế nào.

Agent:
Updated read: không phải là dashboard cho team. Đây là một trình theo dõi thí nghiệm cá nhân. Confidence: ~60% — vẫn thiếu: "tín hiệu sớm" nghĩa là gì và thế nào là hoàn thành.

Q:     Vấn đề là bạn không biết những thí nghiệm nào đang tồn tại, hay bạn không thể xem kết quả của chúng ở một nơi?
GUESS: cái thứ hai. Bạn có một danh sách các thí nghiệm ở đâu đó, nhưng kết quả của chúng nằm ở năm công cụ khác nhau, và việc đối chiếu chúng bằng tay là điều bạn đang mệt mỏi.

User: Cái thứ nhất thực ra. Tôi thực sự không có một danh sách nào cả. Chúng nằm rải rác trong các tài liệu khác nhau.

Agent: <tiếp tục>
```

Chỉ sau hai câu hỏi, agent đã phát hiện ra yêu cầu thực sự không phải là "một dashboard." Mà là "một danh sách." Sản phẩm khác, phạm vi khác, công việc khác. Một dashboard sẽ là sai lầm.

## Tương tác với các Skill khác

- **`idea-refine`**: hạ nguồn. Nếu ý định đã xác nhận là "Tôi muốn X nhưng tôi không biết cách xác định phạm vi", hãy bàn giao cho `idea-refine` để tạo ra các biến thể dựa trên ý định hiện đã rõ ràng.
- **`spec-driven-development`**: hạ nguồn. Nếu ý định đã xác nhận là cụ thể ("Tôi muốn X cho người dùng Y với tiêu chí thành công Z"), hãy bàn giao cho `spec-driven-development` để viết ra.
- **`planning-and-task-breakdown`**: cách hai bước hạ nguồn của skill này (sau bản spec).
- **`doubt-driven-development`**: ở đầu ngược lại của dòng thời gian. interview-me là trích xuất ý định trước khi quyết định; doubt-driven là xem xét sản phẩm sau khi quyết định. Cả hai đều phát hiện sự sai lệch, nhưng ở những thời điểm khác nhau.
- **`source-driven-development`**: vuông góc. interview-me làm rõ điều người dùng muốn; SDD xác minh các sự thật về khung làm việc (framework). Chúng không cạnh tranh với nhau.

## Các lập luận phổ biến (Sai lầm)

| Lập luận | Thực tế |
|---|---|
| "Yêu cầu đã đủ rõ ràng rồi" | Nếu bạn không thể viết kết quả mong muốn của người dùng trong một câu ngay lúc này, thì yêu cầu đó chưa rõ ràng. Hãy chạy Bước 1 trước khi quyết định. |
| "Hỏi quá nhiều câu hỏi làm lãng phí thời gian của họ" | Thời gian lãng phí cho 4–6 câu hỏi tập trung là rất nhỏ. Thời gian lãng phí khi xây dựng sai thứ là khổng lồ, và người dùng là người gánh chịu chi phí đó. |
| "Tôi sẽ tìm ra trong khi xây dựng" | Chi phí chuyển đổi sau khi đã có mã nguồn cao gấp 10 lần hiện tại. Khám phá trong khi triển khai chính là làm lại (rework). |
| "Họ nói 'sao cũng được', nên tôi cứ quyết định thôi" | "Sao cũng được" là giao phó, không phải quyết định. Hãy hỏi lại với hai lựa chọn cụ thể. |
| "Tôi nên đưa ra cho họ vài lựa chọn để chọn" | Các lựa chọn hiệu quả khi người dùng biết họ muốn gì và đang chọn giữa các sự đánh đổi. Họ hiện chưa biết mình muốn gì. Liệt kê các lựa chọn làm rộng vùng tìm kiếm; đặt câu hỏi làm hẹp nó lại. |
| "Nếu tôi đính kèm phỏng đoán, tôi đang dẫn dắt họ" | Dẫn dắt chính là mục đích. Phản ứng sẽ nhanh hơn là tạo ra từ đầu. Rủi ro là tâm lý đồng thuận (sycophancy), không phải dẫn dắt; giảm thiểu bằng cách thể hiện sự sẵn lòng bị sai. |
| "Chúng tôi đã nói đủ rồi, tôi hiểu rồi" | Hãy kiểm tra: bạn có thể dự đoán phản ứng của họ đối với ba câu hỏi tiếp theo không? Nếu không, bạn vẫn chưa hiểu. |
| "Người dùng đã nói có, chúng tôi xong rồi" | Nếu lời "có" theo sau một bản phát biểu lại mơ hồ hoặc một câu "nghe có vẻ ổn" mở, thì lời "có" đó là sáo rỗng. Hãy phát biểu lại một cách cụ thể và xác nhận lại. |

## Các dấu hiệu cảnh báo (Red Flags)

- Ba câu hỏi hoặc nhiều hơn trong một tin nhắn: đó là gom nhóm (batching), không phải phỏng vấn.
- Một câu hỏi không có giả thuyết đính kèm: đó là khảo sát, không phải cam kết.
- Chấp nhận "sao cũng được" như một câu trả lời cuối cùng.
- Tạo ra một bản spec, kế hoạch hoặc danh sách công việc trước khi người dùng xác nhận rõ ràng bản phát biểu lại của bạn.
- Câu hỏi được đặt ra dưới dạng "thế nào là best practice?" thay vì "bạn thực sự muốn điều gì?".
- Người dùng đưa ra một câu trả lời mang tính phô diễn ("scalable", "clean", "modern") và bạn chấp nhận nó mà không thăm dò xem đó có phải là điều họ thực sự muốn hay không.
- Ba vòng hoặc nhiều hơn mà độ tin cậy của bạn không tăng lên rõ rệt: bạn đang hỏi sai câu hỏi, hãy lùi lại và định khung lại.
- Một con số tin cậy dưới ~70% mà không có lý do đính kèm: người dùng không thể giúp thu hẹp khoảng cách nếu họ không biết điều gì còn thiếu.
- Lưu tài liệu ý định trước khi người dùng xác nhận (bản thân tài liệu ngầm định một lời "có" mà người dùng chưa đưa ra).
- Bỏ qua dòng "Out of scope" trong bản phát biểu lại (sự không đồng thuận ngầm về các mục tiêu không thực hiện chiếm một nửa nguyên nhân gây sai lệch).

## Xác minh

Sau khi áp dụng interview-me:

- [ ] Một giả thuyết rõ ràng với con số tin cậy đã được nêu ra trong lượt đầu tiên.
- [ ] Mọi con số tin cậy dưới ~70% đều đi kèm với một lý do một dòng (điều gì vẫn chưa được giải quyết hoặc còn thiếu).
- [ ] Các câu hỏi được đặt ra từng câu một, mỗi câu đều đính kèm phỏng đoán của agent.
- [ ] Ít nhất một câu hỏi thăm dò "bạn thực sự muốn gì nếu không phải giải trình?" đã được thực hiện khi người dùng đưa ra câu trả lời mang tính phô diễn hoặc rập khuôn.
- [ ] Một bản phát biểu lại cụ thể (Outcome / User / Why now / Success / Constraint / Out of scope) đã được gửi lại cho người dùng.
- [ ] Người dùng đã xác nhận bản phát biểu lại bằng một từ "có" rõ ràng (không phải "sao cũng được", không phải "nghe có vẻ ổn", không phải im lặng).
- [ ] Tại điểm dừng, agent có thể dự đoán phản ứng đối với ba câu hỏi tiếp theo mà nó sẽ hỏi.
- [ ] Bất kỳ sự bàn giao nào cho skill hạ nguồn (`idea-refine`, `spec-driven-development`) đều được khung theo ý định đã xác nhận, không phải theo yêu cầu thiếu thông tin ban đầu.
