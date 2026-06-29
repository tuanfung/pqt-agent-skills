---
name: code-review-and-quality
description: Thực hiện đánh giá mã nguồn đa chiều. Sử dụng trước khi merge bất kỳ thay đổi nào. Sử dụng khi đánh giá mã nguồn do chính bạn, một agent khác, hoặc con người viết. Sử dụng khi bạn cần đánh giá chất lượng mã nguồn trên nhiều phương diện trước khi đưa vào nhánh chính.
---

# Đánh giá và Chất lượng Mã nguồn (Code Review and Quality)

## Tổng quan

Đánh giá mã nguồn đa chiều với các chốt kiểm soát chất lượng (quality gates). Mọi thay đổi đều phải được đánh giá trước khi merge — không có ngoại lệ. Quá trình đánh giá bao gồm năm trục: tính chính xác, khả năng đọc, kiến trúc, bảo mật và hiệu năng.

**Tiêu chuẩn phê duyệt:** Phê duyệt một thay đổi khi nó chắc chắn cải thiện sức khỏe tổng thể của mã nguồn, ngay cả khi nó chưa hoàn hảo. Mã nguồn hoàn hảo không tồn tại — mục tiêu là cải tiến liên tục. Đừng chặn một thay đổi chỉ vì nó không giống hệt cách bạn sẽ viết. Nếu nó cải thiện codebase và tuân thủ các quy ước của dự án, hãy phê duyệt nó.

## Khi nào sử dụng

- Trước khi merge bất kỳ PR hoặc thay đổi nào
- Sau khi hoàn thành triển khai một tính năng
- Khi một agent hoặc model khác tạo ra mã nguồn mà bạn cần đánh giá
- Khi tái cấu trúc (refactoring) mã nguồn hiện có
- Sau bất kỳ lần sửa lỗi nào (đánh giá cả bản sửa lỗi và bài kiểm tra hồi quy - regression test)

## Đánh giá Năm Trục (The Five-Axis Review)

Mỗi lần đánh giá sẽ xem xét mã nguồn theo các phương diện sau:

### 1. Tính chính xác (Correctness)

Mã nguồn có thực hiện đúng những gì nó tuyên bố không?

- Nó có khớp với đặc tả (spec) hoặc yêu cầu nhiệm vụ không?
- Các trường hợp biên (edge cases) có được xử lý không (null, rỗng, giá trị biên)?
- Các luồng lỗi có được xử lý không (không chỉ là happy path)?
- Nó có vượt qua tất cả các bài kiểm tra không? Các bài kiểm tra có thực sự kiểm tra đúng thứ cần kiểm tra không?
- Có lỗi sai lệch một đơn vị (off-by-one errors), tình trạng tranh giành (race conditions), hoặc không nhất quán về trạng thái không?

### 2. Khả năng đọc & Sự đơn giản (Readability & Simplicity)

Một kỹ sư khác (hoặc agent khác) có thể hiểu mã nguồn này mà không cần tác giả giải thích không?

- Tên gọi có mang tính mô tả và nhất quán với quy ước của dự án không? (Không dùng `temp`, `data`, `result` mà không có ngữ cảnh)
- Luồng điều khiển có dễ hiểu không (tránh các toán tử ba ngôi lồng nhau, callback quá sâu)?
- Mã nguồn có được tổ chức logic không (các mã liên quan được nhóm lại, ranh giới module rõ ràng)?
- Có bất kỳ thủ thuật "thông minh" nào cần được đơn giản hóa không?
- **Việc này có thể thực hiện với ít dòng code hơn không?** (1000 dòng trong khi 100 dòng là đủ là một thất bại)
- **Các lớp trừu tượng (abstractions) có xứng đáng với độ phức tạp của chúng không?** (Đừng tổng quát hóa cho đến khi có trường hợp sử dụng thứ ba)
- Liệu các chú thích có giúp làm rõ ý định không rõ ràng không? (Nhưng đừng chú thích mã nguồn hiển nhiên.)
- Có các thành phần code chết (dead code) không: các biến không sử dụng (`_unused`), shim tương thích ngược, hoặc các chú thích `// removed`?
- **Một điều kiện mới có bị "gắn" vào một luồng không liên quan không?** Đó là một "mùi" thiết kế (design smell), không phải là lỗi nhỏ (nit) — hãy đẩy logic đó vào một helper, state, hoặc policy riêng thay vì làm rối luồng hiện có.
- **Các điều kiện lặp lại trên cùng một hình thái có xuất hiện không?** Chúng báo hiệu sự thiếu hụt của một model hoặc dispatcher. Một nhánh "tạm thời" thường trở thành nợ kỹ thuật vĩnh viễn.

### 3. Kiến trúc (Architecture)

Thay đổi này có phù hợp với thiết kế của hệ thống không?

- Nó có tuân theo các pattern hiện có hoặc giới thiệu một pattern mới không? Nếu mới, nó có chính đáng không?
- Nó có duy trì ranh giới module sạch sẽ không?
- Có sự trùng lặp mã nguồn nào cần được chia sẻ không?
- Các phụ thuộc (dependencies) có đi đúng hướng không (không có phụ thuộc vòng - circular dependencies)?
- Mức độ trừu tượng có phù hợp không (không quá phức tạp - over-engineered, không quá liên kết chặt - coupled)?
- **Việc tái cấu trúc này làm giảm độ phức tạp hay chỉ chuyển nó sang chỗ khác?** Hãy đếm số khái niệm mà người đọc phải nắm giữ để theo dõi thay đổi. Nếu một phiên bản "sạch hơn" mà số lượng khái niệm không đổi, thì nó không sạch hơn — hãy ưu tiên việc tái cấu trúc khiến toàn bộ các nhánh, chế độ (modes), hoặc lớp (layers) biến mất thay vì chỉ tập trung lại cùng một logic. Ưu tiên xóa bỏ một lớp trừu tượng hơn là đánh bóng nó.
- **Logic đặc thù của tính năng có bị rò rỉ vào một module dùng chung hoặc đa mục đích không?** Giữ logic trong lớp sở hữu nó, tái sử dụng helper chuẩn hiện có thay vì tạo ra một bản gần giống, và đừng bình thường hóa sự sai lệch kiến trúc.
- **Ranh giới kiểu (type boundaries) có tường minh không?** Hãy đặt câu hỏi về việc sử dụng `any`/`unknown`/optional/casts một cách tùy tiện và các fallback ngầm định che lấp một bất biến (invariant) không rõ ràng — việc làm cho ranh giới trở nên tường minh thường khiến luồng điều khiển xung quanh đơn giản hơn.

### 4. Bảo mật (Security)

Để được hướng dẫn chi tiết về bảo mật, hãy xem `security-and-hardening`. Thay đổi này có gây ra lỗ hổng bảo mật không?

- Đầu vào của người dùng có được xác thực và làm sạch (sanitized) không?
- Các thông tin bí mật (secrets) có được giữ ngoài mã nguồn, logs và version control không?
- Việc xác thực/phân quyền (authentication/authorization) có được kiểm tra nơi cần thiết không?
- Các truy vấn SQL có được tham số hóa không (không nối chuỗi)?
- Các đầu ra có được mã hóa để ngăn chặn XSS không?
- Các phụ thuộc có đến từ nguồn đáng tin cậy và không có lỗ hổng đã biết không?
- Dữ liệu từ nguồn bên ngoài (APIs, logs, nội dung người dùng, file cấu hình) có được coi là không đáng tin cậy không?
- Các luồng dữ liệu bên ngoài có được xác thực tại ranh giới hệ thống trước khi sử dụng trong logic hoặc render không?

### 5. Hiệu năng (Performance)

Để profiling và tối ưu hóa chi tiết, hãy xem `performance-optimization`. Thay đổi này có gây ra vấn đề về hiệu năng không?

- Có pattern truy vấn N+1 không?
- Có vòng lặp không giới hạn hoặc truy xuất dữ liệu không kiểm soát không?
- Có thao tác đồng bộ nào nên là async không?
- Có bất kỳ lần re-render không cần thiết nào trong các UI component không?
- Có thiếu phân trang (pagination) trên các endpoint danh sách không?
- Có đối tượng lớn nào được tạo trong các hot paths không?

## Các biện pháp khắc phục cấu trúc (Structural Remedies)

Khi bạn gắn cờ một vấn đề về cấu trúc, hãy đề xuất cách di chuyển — không chỉ nêu vấn đề. Một bài đánh giá chỉ nói "cái này phức tạp" sẽ khiến tác giả phải đoán. Hãy sử dụng một cách tái cấu trúc cụ thể:

- **Thay thế một chuỗi các điều kiện** bằng một typed model hoặc một dispatcher tường minh.
- **Gộp các nhánh trùng lặp** thành một luồng rõ ràng hơn.
- **Tách biệt orchestration khỏi logic nghiệp vụ (business logic)** để mỗi phần có thể đọc độc lập.
- **Di chuyển logic đặc thù của tính năng** ra khỏi module dùng chung vào package sở hữu khái niệm đó.
- **Tái sử dụng helper chuẩn** thay vì tạo ra một bản gần giống.
- **Làm cho ranh giới kiểu trở nên tường minh** để các nhánh phân rẽ phía sau biến mất.
- **Xóa bỏ pass-through wrapper** gây gián tiếp mà không làm rõ API.
- **Tách một helper, hoặc chia nhỏ một file lớn** thành các module tập trung.

Ưu tiên biện pháp loại bỏ các thành phần chuyển động hơn là biện pháp phân tán cùng một độ phức tạp ra xung quanh.

## Kích thước thay đổi (Change Sizing)

Các thay đổi nhỏ, tập trung sẽ dễ đánh giá hơn, merge nhanh hơn và triển khai an toàn hơn. Mục tiêu các kích thước sau:

```
~100 dòng thay đổi   → Tốt. Có thể đánh giá trong một lần ngồi.
~300 dòng thay đổi   → Chấp nhận được nếu đó là một thay đổi logic duy nhất.
~1000 dòng thay đổi  → Quá lớn. Hãy chia nhỏ nó ra.
```

**Chú ý kích thước file, không chỉ kích thước diff.** Một diff nhỏ vẫn có thể đẩy một file vượt quá ranh giới lành mạnh — khoảng 1000 dòng *tổng cộng* trong một file đơn lẻ (khác với ngưỡng ~1000 dòng *thay đổi* ở trên) là một tín hiệu kiểm tra phổ biến, không phải là giới hạn cứng. Khi một thay đổi làm tăng đáng kể kích thước của một file vốn đã lớn, hãy xem xét việc tách helper, subcomponents, hoặc modules *trước*, thay vì tiếp tục dồn thêm vào. Phân rã, sau đó mới thêm.

**Thế nào là "một thay đổi":** Một chỉnh sửa tự vận hành, giải quyết một vấn đề, bao gồm các bài kiểm tra liên quan và giữ cho hệ thống hoạt động sau khi gửi. Một phần của một tính năng — không phải toàn bộ tính năng.

**Chiến lược chia nhỏ khi thay đổi quá lớn:**

| Chiến lược | Cách làm | Khi nào |
|----------|-----|------|
| **Stack** | Gửi một thay đổi nhỏ, bắt đầu thay đổi tiếp theo dựa trên đó | Các phụ thuộc tuần tự |
| **Theo nhóm file** | Tách các thay đổi cho các nhóm cần những người đánh giá khác nhau | Các mối quan tâm cắt ngang (cross-cutting concerns) |
| **Ngang (Horizontal)** | Tạo mã dùng chung/stubs trước, sau đó đến các bên tiêu thụ | Kiến trúc phân lớp |
| **Dọc (Vertical)** | Chia nhỏ thành các lát cắt full-stack nhỏ hơn của tính năng | Công việc phát triển tính năng |

**Khi các thay đổi lớn được chấp nhận:** Xóa toàn bộ file và tái cấu trúc tự động nơi người đánh giá chỉ cần xác nhận ý định, không cần kiểm tra từng dòng.

**Tách biệt tái cấu trúc (refactoring) khỏi công việc phát triển tính năng.** Một thay đổi vừa tái cấu trúc mã nguồn hiện có vừa thêm hành vi mới là hai thay đổi — hãy gửi chúng riêng biệt. Các dọn dẹp nhỏ (đổi tên biến) có thể được bao gồm tùy theo quyết định của người đánh giá.

## Mô tả thay đổi (Change Descriptions)

Mọi thay đổi cần một mô tả có thể đứng độc lập trong lịch sử version control.

**Dòng đầu tiên:** Ngắn gọn, ở thể mệnh lệnh, độc lập. "Delete the FizzBuzz RPC" chứ không phải "Deleting the FizzBuzz RPC." Phải đủ thông tin để ai đó tìm kiếm lịch sử có thể hiểu thay đổi mà không cần đọc diff.

**Thân bài:** Cái gì đang thay đổi và tại sao. Bao gồm ngữ cảnh, các quyết định và lý do không hiển thị trong chính mã nguồn. Liên kết đến số bug, kết quả benchmark, hoặc tài liệu thiết kế nếu liên quan. Thừa nhận những thiếu sót trong cách tiếp cận nếu có.

**Các mẫu sai (Anti-patterns):** "Fix bug", "Fix build", "Add patch", "Moving code from A to B", "Phase 1", "Add convenience functions".

## Quy trình đánh giá (Review Process)

### Bước 1: Hiểu ngữ cảnh

Trước khi xem mã nguồn, hãy hiểu ý định:

```
- Thay đổi này đang cố gắng đạt được điều gì?
- Nó triển khai spec hoặc nhiệm vụ nào?
- Sự thay đổi hành vi dự kiến là gì?
```

### Bước 2: Đánh giá các bài kiểm tra trước

Các bài kiểm tra tiết lộ ý định và độ bao phủ:

```
- Có bài kiểm tra cho thay đổi này không?
- Chúng có kiểm tra hành vi (không phải chi tiết triển khai) không?
- Các trường hợp biên có được bao phủ không?
- Các bài kiểm tra có tên mô tả không?
- Liệu các bài kiểm tra có bắt được lỗi hồi quy nếu mã nguồn thay đổi không?
```

### Bước 3: Đánh giá triển khai

Đi qua mã nguồn với năm trục trong tâm trí:

```
Đối với mỗi file thay đổi:
1. Tính chính xác: Mã này có làm đúng những gì bài kiểm tra yêu cầu không?
2. Khả năng đọc: Tôi có thể hiểu điều này mà không cần giúp đỡ không?
3. Kiến trúc: Điều này có phù hợp với hệ thống không?
4. Bảo mật: Có lỗ hổng nào không?
5. Hiệu năng: Có nút thắt cổ chai nào không?
```

### Bước 4: Phân loại phát hiện

Gắn nhãn cho mọi bình luận với mức độ nghiêm trọng để tác giả biết điều gì là bắt buộc so với tùy chọn:

| Tiền tố | Ý nghĩa | Hành động của tác giả |
|--------|---------|---------------|
| *(không có tiền tố)* | Thay đổi bắt buộc | Phải giải quyết trước khi merge |
| **Critical:** | Chặn merge | Lỗ hổng bảo mật, mất dữ liệu, hỏng chức năng |
| **Nit:** | Nhỏ, tùy chọn | Tác giả có thể bỏ qua — định dạng, sở thích phong cách |
| **Optional:** / **Consider:** | Gợi ý | Đáng cân nhắc nhưng không bắt buộc |
| **FYI** | Chỉ để thông tin | Không cần hành động — ngữ cảnh để tham khảo sau này |

Điều này ngăn tác giả coi mọi phản hồi là bắt buộc và lãng phí thời gian vào các gợi ý tùy chọn.

**Ưu tiên những gì quan trọng.** Sắp xếp các phát hiện theo mức độ ảnh hưởng: tính chính xác và bảo mật trước, sau đó là các thoái lui về cấu trúc và các đơn giản hóa bị bỏ lỡ, sau đó mới đến mọi thứ khác. Đừng chôn vùi một vấn đề thực sự dưới những lỗi nhỏ về thẩm mỹ — một vài bình luận có sức thuyết phục cao sẽ tốt hơn một danh sách dài. Nếu bạn có một vấn đề về cấu trúc và mười lỗi nhỏ, thì vấn đề cấu trúc *chính là* nội dung bài đánh giá.

### Bước 5: Xác minh việc xác minh

Kiểm tra câu chuyện xác minh của tác giả:

```
- Những bài kiểm tra nào đã được chạy?
- Build có vượt qua không?
- Thay đổi có được kiểm tra thủ công không?
- Có ảnh chụp màn hình cho các thay đổi UI không?
- Có sự so sánh trước/sau không?
```

## Mô hình đánh giá đa Model (Multi-Model Review Pattern)

Sử dụng các model khác nhau cho các góc nhìn đánh giá khác nhau:

```
Model A viết mã nguồn
    │
    ▼
Model B đánh giá tính chính xác và kiến trúc
    │
    ▼
Model A giải quyết phản hồi
    │
    ▼
Con người đưa ra quyết định cuối cùng
```

Việc này giúp bắt được các vấn đề mà một model duy nhất có thể bỏ lỡ — các model khác nhau có những điểm mù khác nhau.

**Ví dụ prompt cho một agent đánh giá:**
```
Review this code change for correctness, security, and adherence to
our project conventions. The spec says [X]. The change should [Y].
Flag any issues as Critical, Required, Optional, or Nit.
```

## Vệ sinh mã nguồn chết (Dead Code Hygiene)

Sau bất kỳ lần tái cấu trúc hoặc thay đổi triển khai nào, hãy kiểm tra mã nguồn bị mồ côi:

1. Xác định mã nguồn hiện không thể truy cập hoặc không được sử dụng
2. Liệt kê rõ ràng
3. **Hỏi trước khi xóa:** "Tôi có nên xóa các thành phần hiện không còn sử dụng này không: [danh sách]?"

Đừng để code chết nằm rải rác — nó gây nhầm lẫn cho những người đọc và agent trong tương lai. Nhưng đừng âm thầm xóa những thứ bạn không chắc chắn. Khi nghi ngờ, hãy hỏi.

```
DEAD CODE IDENTIFIED:
- formatLegacyDate() in src/utils/date.ts — replaced by formatDate()
- OldTaskCard component in src/components/ — replaced by TaskCard
- LEGACY_API_URL constant in src/config.ts — no remaining references
→ Safe to remove these?
```

## Tốc độ đánh giá (Review Speed)

Việc đánh giá chậm sẽ làm tắc nghẽn toàn bộ đội ngũ. Chi phí chuyển đổi ngữ cảnh để đánh giá thấp hơn chi phí chờ đợi gây ra cho người khác.

- **Phản hồi trong vòng một ngày làm việc** — đây là mức tối đa, không phải mục tiêu.
- **Nhịp độ lý tưởng:** Phản hồi ngay sau khi yêu cầu đánh giá đến, trừ khi đang trong quá trình lập trình tập trung sâu. Một thay đổi điển hình nên hoàn thành nhiều vòng đánh giá trong một ngày.
- **Ưu tiên các phản hồi cá nhân nhanh chóng** hơn là sự phê duyệt cuối cùng nhanh chóng. Phản hồi nhanh giúp giảm sự ức chế ngay cả khi cần nhiều vòng đánh giá.
- **Thay đổi lớn:** Yêu cầu tác giả chia nhỏ chúng thay vì đánh giá một tập hợp thay đổi khổng lồ.

## Xử lý bất đồng

Khi giải quyết các tranh chấp trong đánh giá, hãy áp dụng phân cấp sau:

1. **Sự thật kỹ thuật và dữ liệu** ghi đè lên ý kiến và sở thích.
2. **Hướng dẫn phong cách (style guides)** là cơ quan có thẩm quyền tuyệt đối về các vấn đề phong cách.
3. **Thiết kế phần mềm** phải được đánh giá dựa trên các nguyên tắc kỹ thuật, không phải sở thích cá nhân.
4. **Sự nhất quán của codebase** là chấp nhận được nếu nó không làm giảm sức khỏe tổng thể.

**Đừng chấp nhận câu "Tôi sẽ dọn dẹp nó sau".** Kinh nghiệm cho thấy việc dọn dẹp trì hoãn hiếm khi xảy ra. Yêu cầu dọn dẹp trước khi gửi trừ khi đó là trường hợp khẩn cấp thực sự. Nếu các vấn đề xung quanh không thể được giải quyết trong thay đổi này, yêu cầu tạo một bug và tự giao cho mình xử lý.

## Sự trung thực trong đánh giá

Khi đánh giá mã nguồn — cho dù do bạn, một agent khác, hoặc con người viết:

- **Đừng phê duyệt hời hợt (rubber-stamp).** "LGTM" mà không có bằng chứng về việc đánh giá sẽ không giúp ích cho ai.
- **Đừng làm nhẹ các vấn đề thực sự.** "Đây có thể là một mối quan tâm nhỏ" khi đó là một bug sẽ ảnh hưởng đến môi trường production là thiếu trung thực.
- **Định lượng các vấn đề khi có thể.** "Truy vấn N+1 này sẽ thêm ~50ms cho mỗi mục trong danh sách" tốt hơn là "cái này có thể chậm".
- **Phản đối các cách tiếp cận có vấn đề rõ ràng.** Sự vâng lời quá mức (sycophancy) là một thất bại trong đánh giá. Nếu việc triển khai có vấn đề, hãy nói trực tiếp và đề xuất các phương án thay thế.
- **Chấp nhận sự ghi đè một cách lịch sự.** Nếu tác giả có đầy đủ ngữ cảnh và không đồng ý, hãy tôn trọng phán quyết của họ. Bình luận về mã nguồn, không bình luận về con người — chuyển đổi các lời phê bình cá nhân thành tập trung vào chính mã nguồn.

## Kỷ luật về phụ thuộc (Dependency Discipline)

Một phần của đánh giá mã nguồn là đánh giá phụ thuộc:

**Trước khi thêm bất kỳ phụ thuộc nào:**
1. Stack hiện tại có giải quyết được vấn đề này không? (Thường là có.)
2. Phụ thuộc này lớn cỡ nào? (Kiểm tra tác động đến bundle.)
3. Nó có được bảo trì tích cực không? (Kiểm tra commit cuối cùng, các issue đang mở.)
4. Nó có lỗ hổng bảo mật đã biết không? (`npm audit`)
5. Giấy phép (license) là gì? (Phải tương thích với dự án.)

**Quy tắc:** Ưu tiên thư viện chuẩn và các tiện ích hiện có hơn là các phụ thuộc mới. Mỗi phụ thuộc là một gánh nặng tiềm ẩn.

## Danh sách kiểm tra đánh giá (The Review Checklist)

```markdown
## Review: [Tiêu đề PR/Thay đổi]

### Ngữ cảnh
- [ ] Tôi hiểu thay đổi này làm gì và tại sao

### Tính chính xác
- [ ] Thay đổi khớp với yêu cầu spec/nhiệm vụ
- [ ] Các trường hợp biên đã được xử lý
- [ ] Các luồng lỗi đã được xử lý
- [ ] Các bài kiểm tra bao phủ thay đổi đầy đủ

### Khả năng đọc
- [ ] Tên gọi rõ ràng và nhất quán
- [ ] Logic dễ hiểu
- [ ] Không có độ phức tạp không cần thiết

### Kiến trúc
- [ ] Tuân theo các pattern hiện có
- [ ] Không có sự liên kết chặt hoặc phụ thuộc không cần thiết
- [ ] Mức độ trừu tượng phù hợp
- [ ] Tái cấu trúc làm giảm độ phức tạp thay vì di chuyển nó
- [ ] Không có logic tính năng trong module dùng chung; file giữ kích thước lành mạnh

### Bảo mật
- [ ] Không có thông tin bí mật trong mã nguồn
- [ ] Đầu vào được xác thực tại các ranh giới
- [ ] Không có lỗ hổng injection
- [ ] Các kiểm tra xác thực đã được thiết lập
- [ ] Các nguồn dữ liệu bên ngoài được coi là không đáng tin cậy

### Hiệu năng
- [ ] Không có pattern N+1
- [ ] Không có thao tác không giới hạn
- [ ] Có phân trang trên các endpoint danh sách

### Xác minh
- [ ] Các bài kiểm tra vượt qua
- [ ] Build thành công
- [ ] Đã xác minh thủ công (nếu có thể)

### Kết luận
- [ ] **Approve** — Sẵn sàng để merge
- [ ] **Request changes** — Các vấn đề phải được giải quyết
```

## Xem thêm

- Để được hướng dẫn đánh giá bảo mật chi tiết, hãy xem `references/security-checklist.md`
- Để kiểm tra đánh giá hiệu năng, hãy xem `references/performance-checklist.md`

## Các lý lẽ biện minh phổ biến (Common Rationalizations)

| Lý lẽ biện minh | Thực tế |
|---|---|
| "Nó chạy được, thế là đủ rồi" | Mã nguồn chạy được nhưng không thể đọc, không bảo mật hoặc sai kiến trúc sẽ tạo ra nợ kỹ thuật tích tụ. |
| "Tôi viết nó, nên tôi biết nó đúng" | Tác giả thường mù quáng trước các giả định của chính họ. Mọi thay đổi đều có lợi khi được một người khác xem xét. |
| "Chúng ta sẽ dọn dẹp nó sau" | "Sau" không bao giờ đến. Đánh giá chính là chốt kiểm soát chất lượng — hãy tận dụng nó. Yêu cầu dọn dẹp trước khi merge, không phải sau đó. |
| "Mã nguồn do AI tạo ra chắc là ổn" | Mã AI cần được xem xét kỹ hơn, chứ không phải ít hơn. Nó tự tin và có vẻ hợp lý, ngay cả khi sai. |
| "Các bài kiểm tra vượt qua, nên nó ổn" | Các bài kiểm tra là cần thiết nhưng không đủ. Chúng không bắt được các vấn đề kiến trúc, bảo mật hoặc quan ngại về khả năng đọc. |
| "Việc tái cấu trúc làm nó sạch hơn" | Di chuyển độ phức tạp không phải là giảm bớt nó. Nếu người đọc vẫn phải nắm giữ cùng một số lượng khái niệm, thì cấu trúc không cải thiện — hãy tìm phiên bản mà các nhánh biến mất. |
| "Đây chỉ là một bổ sung nhỏ cho file này" | Các diff nhỏ vẫn có thể đẩy file vượt quá kích thước lành mạnh và gắn các nhánh vào luồng không liên quan. Hãy đánh giá cấu trúc kết quả, không phải kích thước diff. |

## Cờ đỏ (Red Flags)

- Các PR được merge mà không có bất kỳ sự đánh giá nào
- Đánh giá chỉ kiểm tra xem các bài kiểm tra có vượt qua không (bỏ qua các trục khác)
- "LGTM" mà không có bằng chứng đánh giá thực sự
- Các thay đổi nhạy cảm về bảo mật mà không có đánh giá tập trung vào bảo mật
- Các PR lớn đến mức "quá to để đánh giá tử tế" (hãy chia nhỏ chúng)
- Không có bài kiểm tra hồi quy đi kèm với PR sửa lỗi
- Các bình luận đánh giá không có nhãn mức độ nghiêm trọng — khiến không rõ cái nào là bắt buộc so với tùy chọn
- Chấp nhận câu "Tôi sẽ sửa nó sau" — điều đó không bao giờ xảy ra
- Một đợt tái cấu trúc di chuyển mã nguồn xung quanh mà không làm giảm số lượng khái niệm người đọc phải nắm giữ
- Một thay đổi làm tăng kích thước của một file vốn đã lớn thay vì phân rã nó
- Các điều kiện mới bị rải rác vào các luồng mã không liên quan (thiếu một lớp trừu tượng)
- Một helper tùy chỉnh sao chép một helper chuẩn hiện có, hoặc logic tính năng được đặt trong module dùng chung

## Xác minh (Verification)

Sau khi đánh giá hoàn tất:

- [ ] Tất cả các vấn đề Critical đã được giải quyết
- [ ] Tất cả các thay đổi Required (không tiền tố) đã được giải quyết hoặc được trì hoãn rõ ràng kèm theo lý do chính đáng
- [ ] Các bài kiểm tra vượt qua
- [ ] Build thành công
- [ ] Câu chuyện xác minh đã được ghi lại (cái gì thay đổi, đã được xác minh như thế nào)

**Các điểm chặn giả định:** làm nổi bật và đề xuất thiết kế đơn giản hơn cho mỗi trường hợp sau; chỉ nâng cấp lên mức Required khi thay đổi chủ động làm cấu trúc tệ hơn: một đợt tái cấu trúc di chuyển độ phức tạp thay vì giảm bớt nó; một thay đổi đẩy file vượt quá ranh giới kích thước mà không có sự phân rã; logic tính năng được thêm vào module dùng chung; một bản gần giống của helper chuẩn hiện có; một fallback ngầm định che lấp một bất biến không rõ ràng.
