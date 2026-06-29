# Checklist về Khả năng Truy cập (Accessibility Checklist)

Tham chiếu nhanh cho việc tuân thủ WCAG 2.1 AA. Sử dụng cùng với skill `frontend-ui-engineering`.

## Mục lục

- [Các kiểm tra thiết yếu](#essential-checks)
- [Các mẫu HTML phổ biến](#common-html-patterns)
- [Công cụ kiểm thử](#testing-tools)
- [Tham chiếu nhanh: ARIA Live Regions](#quick-reference-aria-live-regions)
- [Các Anti-Patterns phổ biến](#common-anti-patterns)

## Các kiểm tra thiết yếu

### Điều hướng bằng bàn phím
- [ ] Tất cả các thành phần tương tác đều có thể focus thông qua phím Tab
- [ ] Thứ tự focus tuân theo thứ tự trực quan/logic
- [ ] Focus hiển thị rõ ràng (có outline/ring trên các thành phần đang được focus)
- [ ] Các widget tùy chỉnh có hỗ trợ bàn phím (Enter để kích hoạt, Escape để đóng)
- [ ] Không có bẫy bàn phím (keyboard traps) - người dùng luôn có thể Tab thoát khỏi một component
- [ ] Có liên kết "Nhảy đến nội dung chính" (Skip-to-content) ở đầu trang - hiển thị (ít nhất) khi focus bằng bàn phím
- [ ] Modals khóa focus khi mở, và trả lại focus khi đóng

### Trình đọc màn hình (Screen Readers)
- [ ] Tất cả hình ảnh đều có văn bản `alt` (hoặc `alt=""` cho hình ảnh trang trí)
- [ ] Tất cả các input của form đều có label đi kèm (`<label>` hoặc `aria-label`)
- [ ] Các nút và liên kết có văn bản mô tả rõ ràng (không dùng "Nhấn vào đây")
- [ ] Các nút chỉ có icon phải có `aria-label`
- [ ] Trang web có một thẻ `<h1>` duy nhất và các tiêu đề không nhảy cấp
- [ ] Các thay đổi nội dung động được thông báo (`aria-live` regions)
- [ ] Bảng có các tiêu đề `<th>` với phạm vi (scope) rõ ràng

### Trực quan (Visual)
- [ ] Độ tương phản văn bản ≥ 4.5:1 (văn bản thường) hoặc ≥ 3:1 (văn bản lớn, 18px+)
- [ ] Các thành phần UI có độ tương phản ≥ 3:1 so với nền
- [ ] Màu sắc không phải là cách duy nhất để truyền tải thông tin
- [ ] Văn bản có thể thay đổi kích thước lên đến 200% mà không làm hỏng layout
- [ ] Không có nội dung nhấp nháy hơn 3 lần mỗi giây

### Biểu mẫu (Forms)
- [ ] Mỗi input đều có label hiển thị rõ ràng
- [ ] Các trường bắt buộc được đánh dấu (không chỉ bằng màu sắc)
- [ ] Thông báo lỗi cụ thể và được liên kết với trường tương ứng
- [ ] Trạng thái lỗi hiển thị bằng nhiều cách hơn là chỉ dùng màu sắc (icon, văn bản, border)
- [ ] Các lỗi khi gửi form được tóm tắt và có thể focus
- [ ] Các trường quen thuộc sử dụng autocomplete (ví dụ: `type="email" autocomplete="email"`)

### Nội dung (Content)
- [ ] Ngôn ngữ được khai báo (`<html lang="en">`)
- [ ] Trang web có thẻ `<title>` mô tả rõ ràng
- [ ] Các liên kết phân biệt được với văn bản xung quanh (không chỉ bằng màu sắc)
- [ ] Kích thước vùng chạm (touch targets) ≥ 44x44px trên di động
- [ ] Các trạng thái trống (empty states) có ý nghĩa (không để màn hình trắng)

## Các mẫu HTML phổ biến

### Nút (Buttons) vs. Liên kết (Links)

```html
<!-- Sử dụng <button> cho các hành động -->
<button onClick={handleDelete}>Delete Task</button>

<!-- Sử dụng <a> cho điều hướng -->
<a href="/tasks/123">View Task</a>

<!-- KHÔNG BAO GIỜ sử dụng div/span làm nút -->
<div onClick={handleDelete}>Delete</div>  <!-- SAI -->
```

### Label cho Form

```html
<!-- Liên kết label rõ ràng -->
<label htmlFor="email">Email address</label>
<input id="email" type="email" required />

<!-- Bọc implicit -->
<label>
  Email address
  <input type="email" required />
</label>

<!-- Label ẩn (ưu tiên label hiển thị) -->
<input type="search" aria-label="Search tasks" />
```

### ARIA Roles

```html
<!-- Điều hướng -->
<nav aria-label="Main navigation">...</nav>
<nav aria-label="Footer links">...</nav>

<!-- Thông báo trạng thái -->
<div role="status" aria-live="polite">Task saved</div>

<!-- Thông báo cảnh báo -->
<div role="alert">Error: Title is required</div>

<!-- Hộp thoại Modal -->
<dialog aria-modal="true" aria-labelledby="dialog-title">
  <h2 id="dialog-title">Confirm Delete</h2>
  ...
</dialog>

<!-- Trạng thái tải (Loading) -->
<div aria-busy="true" aria-label="Loading tasks">
  <Spinner />
</div>
```

### Danh sách có khả năng truy cập (Accessible Lists)

```html
<ul role="list" aria-label="Tasks">
  <li>
    <input type="checkbox" id="task-1" aria-label="Complete: Buy groceries" />
    <label htmlFor="task-1">Buy groceries</label>
  </li>
</ul>
```

## Công cụ kiểm thử

```bash
# Kiểm tra tự động
npx axe-core          # Kiểm tra khả năng truy cập bằng lập trình
npx pa11y             # Công cụ kiểm tra khả năng truy cập CLI

# Trong trình duyệt
# Chrome DevTools → Lighthouse → Accessibility
# Chrome DevTools → Elements → Accessibility tree

# Kiểm tra bằng trình đọc màn hình (Screen reader)
# macOS: VoiceOver (Cmd + F5)
# Windows: NVDA (miễn phí) hoặc JAWS
# Linux: Orca
```

## Tham chiếu nhanh: ARIA Live Regions

| Giá trị | Hành vi | Sử dụng cho |
|-------|----------|---------|
| `aria-live="polite"` | Thông báo tại lần tạm dừng tiếp theo | Cập nhật trạng thái, xác nhận đã lưu |
| `aria-live="assertive"` | Thông báo ngay lập tức | Lỗi, cảnh báo nhạy cảm về thời gian |
| `role="status"` | Tương tự như `polite` | Thông báo trạng thái |
| `role="alert"` | Tương tự như `assertive` | Thông báo lỗi |

## Các Anti-Patterns phổ biến

| Anti-Pattern | Vấn đề | Cách khắc phục |
|---|---|---|
| `div` làm nút | Không thể focus, không hỗ trợ bàn phím | Sử dụng `<button>` |
| Thiếu văn bản `alt` | Hình ảnh vô hình đối với trình đọc màn hình | Thêm `alt` mô tả |
| Trạng thái chỉ dùng màu sắc | Vô hình đối với người mù màu | Thêm icon, văn bản hoặc họa tiết |
| Phương tiện tự động phát | Gây mất phương hướng, không thể dừng | Thêm bộ điều khiển, không tự động phát |
| Dropdown tùy chỉnh không có ARIA | Không thể dùng bằng bàn phím/trình đọc màn hình | Sử dụng `<select>` mặc định hoặc ARIA listbox phù hợp |
| Xóa outline của focus | Người dùng không biết mình đang ở đâu | Định dạng lại outline, không xóa bỏ chúng |
| Liên kết/nút trống | Thông báo "Link" mà không có mô tả | Thêm văn bản hoặc `aria-label` |
| `tabindex > 0` | Làm hỏng thứ tự tab tự nhiên | Chỉ sử dụng `tabindex="0"` hoặc `-1` |
