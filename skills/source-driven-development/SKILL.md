---
name: source-driven-development
description: Căn cứ mọi quyết định triển khai vào tài liệu chính thức. Sử dụng khi bạn muốn mã nguồn có thẩm quyền, được trích dẫn nguồn và không chứa các pattern cũ. Sử dụng khi xây dựng với bất kỳ framework hoặc thư viện nào mà sự chính xác là quan trọng.
---

# Source-Driven Development

## Tổng quan

Mọi quyết định mã nguồn cụ thể cho framework phải được hỗ trợ bởi tài liệu chính thức. Đừng triển khai dựa trên trí nhớ — hãy xác minh, trích dẫn và để người dùng thấy nguồn của bạn. Dữ liệu huấn luyện sẽ bị lỗi thời, API bị loại bỏ (deprecated), các best practice sẽ thay đổi. Skill này đảm bảo người dùng nhận được mã nguồn mà họ có thể tin tưởng vì mọi pattern đều truy xuất ngược lại một nguồn có thẩm quyền mà họ có thể kiểm tra.

## Khi nào nên sử dụng

- Người dùng muốn mã nguồn tuân theo các best practice hiện tại cho một framework nhất định
- Xây dựng boilerplate, mã khởi tạo (starter code), hoặc các pattern sẽ được sao chép trong toàn bộ dự án
- Người dùng yêu cầu rõ ràng việc triển khai đã được lập tài liệu, xác minh hoặc "chính xác"
- Triển khai các tính năng mà cách tiếp cận khuyến nghị của framework là quan trọng (forms, routing, data fetching, state management, auth)
- Xem xét hoặc cải thiện mã nguồn sử dụng các pattern đặc thù của framework
- Bất cứ khi nào bạn định viết mã đặc thù cho framework dựa trên trí nhớ

**Khi KHÔNG nên sử dụng:**

- Sự chính xác không phụ thuộc vào một phiên bản cụ thể (đổi tên biến, sửa lỗi đánh máy, di chuyển file)
- Logic thuần túy hoạt động giống nhau trên tất cả các phiên bản (vòng lặp, câu lệnh điều kiện, cấu trúc dữ liệu)
- Người dùng muốn tốc độ hơn là xác minh ("cứ làm nhanh đi")

## Quy trình

```
DETECT ──→ FETCH ──→ IMPLEMENT ──→ CITE
  │          │           │            │
  ▼          ▼           ▼            ▼
 Stack?     Lấy các    Tuân theo    Hiển thị
 nào?       docs tương  pattern được  nguồn
            ứng        lập tài liệu
```

### Bước 1: Phát hiện Stack và Phiên bản

Đọc file phụ thuộc của dự án để xác định phiên bản chính xác:

```
package.json    → Node/React/Vue/Angular/Svelte
composer.json   → PHP/Symfony/Laravel
requirements.txt / pyproject.toml → Python/Django/Flask
go.mod          → Go
Cargo.toml      → Rust
Gemfile         → Ruby/Rails
```

Nêu rõ những gì bạn tìm thấy:

```
STACK DETECTED:
- React 19.1.0 (từ package.json)
- Vite 6.2.0
- Tailwind CSS 4.0.3
→ Đang lấy docs chính thức cho các pattern tương ứng.
```

Nếu phiên bản bị thiếu hoặc không rõ ràng, hãy **hỏi người dùng**. Đừng đoán — phiên bản quyết định pattern nào là chính xác.

### Bước 2: Lấy Tài liệu Chính thức

Lấy trang tài liệu cụ thể cho tính năng bạn đang triển khai. Không lấy trang chủ, không lấy toàn bộ docs — hãy lấy trang liên quan.

**Hệ thống phân cấp nguồn (theo thứ tự ưu tiên):**

| Ưu tiên | Nguồn | Ví dụ |
|----------|--------|---------|
| 1 | Tài liệu chính thức | react.dev, docs.djangoproject.com, symfony.com/doc |
| 2 | Blog / changelog chính thức | react.dev/blog, nextjs.org/blog |
| 3 | Tham chiếu tiêu chuẩn web | MDN, web.dev, html.spec.whatwg.org |
| 4 | Tương thích trình duyệt/runtime | caniuse.com, node.green |

**Không có thẩm quyền — tuyệt đối không trích dẫn làm nguồn chính:**

- Câu trả lời trên Stack Overflow
- Bài viết blog hoặc hướng dẫn (ngay cả những bài phổ biến)
- Tài liệu hoặc tóm tắt do AI tạo ra
- Dữ liệu huấn luyện của chính bạn (đó chính là mục đích — hãy xác minh nó)

**Hãy chính xác với những gì bạn lấy:**

```
SAI:  Lấy trang chủ React
ĐÚNG: Lấy react.dev/reference/react/useActionState

SAI:  Tìm kiếm "django authentication best practices"
ĐÚNG: Lấy docs.djangoproject.com/en/6.0/topics/auth/
```

Sau khi lấy, hãy trích xuất các pattern chính và ghi chú bất kỳ cảnh báo deprecation hoặc hướng dẫn migration nào.

Khi các nguồn chính thức xung đột với nhau (ví dụ: hướng dẫn migration mâu thuẫn với tham chiếu API), hãy nêu sự sai lệch này cho người dùng và xác minh pattern nào thực sự hoạt động với phiên bản đã phát hiện.

### Bước 3: Triển khai theo các Pattern đã được lập tài liệu

Viết mã khớp với những gì tài liệu hiển thị:

- Sử dụng API signatures từ docs, không dùng từ trí nhớ
- Nếu docs hiển thị một cách mới để làm điều gì đó, hãy sử dụng cách mới
- Nếu docs deprecate một pattern, đừng sử dụng phiên bản bị deprecated đó
- Nếu docs không đề cập đến điều gì đó, hãy đánh dấu là chưa được xác minh (unverified)

**Khi docs xung đột với mã nguồn hiện có của dự án:**

```
CONFLICT DETECTED:
Mã nguồn hiện tại sử dụng useState cho trạng thái loading của form,
nhưng docs của React 19 khuyến nghị sử dụng useActionState cho pattern này.
(Nguồn: react.dev/reference/react/useActionState)

Tùy chọn:
A) Sử dụng pattern hiện đại (useActionState) — nhất quán với docs hiện tại
B) Khớp với mã nguồn hiện có (useState) — nhất quán với codebase
→ Bạn ưu tiên cách tiếp cận nào?
```

Hãy nêu rõ xung đột. Đừng tự ý chọn một cái.

### Bước 4: Trích dẫn Nguồn

Mọi pattern đặc thù cho framework đều phải có trích dẫn. Người dùng phải có thể xác minh mọi quyết định.

**Trong comment của mã nguồn:**

```typescript
// React 19 form handling với useActionState
// Nguồn: https://react.dev/reference/react/useActionState#usage
const [state, formAction, isPending] = useActionState(submitOrder, initialState);
```

**Trong cuộc hội thoại:**

```
Tôi đang sử dụng useActionState thay vì useState thủ công cho
trạng thái gửi form. React 19 đã thay thế pattern
isPending/setIsPending thủ công bằng hook này.

Nguồn: https://react.dev/blog/2024/12/05/react-19#actions
"useTransition hiện hỗ trợ các hàm async [...] để xử lý
các trạng thái pending một cách tự động"
```

**Quy tắc trích dẫn:**

- Sử dụng URL đầy đủ, không dùng URL rút ngắn
- Ưu tiên deep links với anchors nếu có thể (ví dụ: `/useActionState#usage` thay vì `/useActionState`) — anchors tồn tại tốt hơn khi cấu trúc docs thay đổi so với các trang cấp cao nhất
- Trích dẫn đoạn văn liên quan khi nó hỗ trợ một quyết định không hiển nhiên
- Bao gồm dữ liệu hỗ trợ trình duyệt/runtime khi đề xuất các tính năng của nền tảng
- Nếu bạn không thể tìm thấy tài liệu cho một pattern, hãy nêu rõ:

```
UNVERIFIED: Tôi không thể tìm thấy tài liệu chính thức cho
pattern này. Điều này dựa trên dữ liệu huấn luyện và có thể đã lỗi thời.
Hãy xác minh trước khi sử dụng trong production.
```

Sự trung thực về những gì bạn không thể xác minh có giá trị hơn sự tự tin giả tạo.

## Các lý lẽ biện minh phổ biến

| Lý lẽ biện minh | Thực tế |
|---|---|
| "Tôi tự tin về API này" | Sự tự tin không phải là bằng chứng. Dữ liệu huấn luyện chứa các pattern lỗi thời trông có vẻ đúng nhưng sẽ bị lỗi với các phiên bản hiện tại. Hãy xác minh. |
| "Lấy docs làm lãng phí token" | Hallucinating (ảo giác) một API còn lãng phí hơn. Người dùng sẽ gỡ lỗi trong một giờ, sau đó phát hiện ra signature của hàm đã thay đổi. Một lần fetch ngăn chặn hàng giờ làm lại. |
| "Docs sẽ không có thứ tôi cần" | Nếu docs không đề cập đến, đó là thông tin giá trị — pattern đó có thể không được khuyến nghị chính thức. |
| "Tôi sẽ chỉ đề cập rằng nó có thể đã lỗi thời" | Một lời miễn trừ trách nhiệm không giúp ích gì. Hoặc là xác minh và trích dẫn, hoặc đánh dấu rõ ràng là chưa xác minh. Lấp lửng là lựa chọn tệ nhất. |
| "Đây là một tác vụ đơn giản, không cần kiểm tra" | Các tác vụ đơn giản với pattern sai sẽ trở thành template. Người dùng sao chép trình xử lý form bị deprecated của bạn vào mười component trước khi phát hiện ra cách tiếp cận hiện đại tồn tại. |

## Các dấu hiệu cảnh báo (Red Flags)

- Viết mã đặc thù cho framework mà không kiểm tra docs cho phiên bản đó
- Sử dụng "Tôi tin rằng" hoặc "Tôi nghĩ" về một API thay vì trích dẫn nguồn
- Triển khai một pattern mà không biết nó áp dụng cho phiên bản nào
- Trích dẫn Stack Overflow hoặc các bài blog thay vì tài liệu chính thức
- Sử dụng các API bị deprecated vì chúng xuất hiện trong dữ liệu huấn luyện
- Không đọc `package.json` / các file phụ thuộc trước khi triển khai
- Chuyển giao mã nguồn mà không có trích dẫn nguồn cho các quyết định đặc thù của framework
- Lấy toàn bộ trang docs khi chỉ có một trang là liên quan

## Xác minh

Sau khi triển khai với source-driven development:

- [ ] Các phiên bản framework và thư viện đã được xác định từ file phụ thuộc
- [ ] Tài liệu chính thức đã được lấy cho các pattern đặc thù của framework
- [ ] Tất cả các nguồn là tài liệu chính thức, không phải bài blog hay dữ liệu huấn luyện
- [ ] Mã nguồn tuân theo các pattern được hiển thị trong tài liệu của phiên bản hiện tại
- [ ] Các quyết định không hiển nhiên bao gồm trích dẫn nguồn với URL đầy đủ
- [ ] Không sử dụng API bị deprecated (đã kiểm tra với hướng dẫn migration)
- [ ] Các xung đột giữa docs và mã nguồn hiện có đã được nêu ra cho người dùng
- [ ] Bất cứ điều gì không thể xác minh đều được đánh dấu rõ ràng là chưa xác minh
