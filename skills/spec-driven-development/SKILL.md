---
name: spec-driven-development
description: Tạo các bản đặc tả (specs) trước khi lập trình. Sử dụng khi bắt đầu một dự án, tính năng mới hoặc thay đổi quan trọng mà chưa có bản đặc tả. Sử dụng khi các yêu cầu không rõ ràng, mơ hồ hoặc chỉ tồn tại dưới dạng ý tưởng sơ bộ.
---

# Phát triển hướng đặc tả (Spec-Driven Development)

## Tổng quan

Viết một bản đặc tả có cấu trúc trước khi viết bất kỳ đoạn mã nào. Bản đặc tả là nguồn sự thật chung (shared source of truth) giữa bạn và kỹ sư con người — nó xác định chúng ta đang xây dựng cái gì, tại sao, và làm sao để biết khi nào nó hoàn thành. Lập trình mà không có đặc tả là đang phỏng đoán.

## Khi nào nên sử dụng

- Bắt đầu một dự án hoặc tính năng mới
- Các yêu cầu mơ hồ hoặc chưa hoàn chỉnh
- Thay đổi ảnh hưởng đến nhiều file hoặc module
- Bạn sắp đưa ra một quyết định về kiến trúc
- Tác vụ mất hơn 30 phút để thực hiện

**Khi KHÔNG nên sử dụng:** Các bản sửa lỗi một dòng, sửa lỗi đánh máy, hoặc những thay đổi mà yêu cầu đã rõ ràng và độc lập.

## Quy trình làm việc có chốt chặn (Gated Workflow)

Phát triển hướng đặc tả có bốn giai đoạn. Không chuyển sang giai đoạn tiếp theo cho đến khi giai đoạn hiện tại được xác nhận.

```
SPECIFY ──→ PLAN ──→ TASKS ──→ IMPLEMENT
   │          │        │          │
   ▼          ▼        ▼          ▼
 Human      Human    Human      Human
 reviews    reviews  reviews    reviews
```

### Giai đoạn 1: Đặc tả (Specify)

Bắt đầu với một tầm nhìn tổng quát. Đặt các câu hỏi làm rõ với con người cho đến khi các yêu cầu trở nên cụ thể.

**Phơi bày các giả định ngay lập tức.** Trước khi viết nội dung đặc tả, hãy liệt kê những gì bạn đang giả định:

```
CÁC GIẢ ĐỊNH TÔI ĐANG ĐẶT RA:
1. Đây là một ứng dụng web (không phải ứng dụng di động native)
2. Xác thực sử dụng session-based cookies (không phải JWT)
3. Cơ sở dữ liệu là PostgreSQL (dựa trên Prisma schema hiện có)
4. Chúng ta chỉ nhắm đến các trình duyệt hiện đại (không hỗ trợ IE11)
→ Hãy chỉnh sửa cho tôi ngay bây giờ hoặc tôi sẽ tiếp tục với các giả định này.
```

Đừng tự ý điền vào các yêu cầu mơ hồ. Mục đích chính của bản đặc tả là làm lộ ra những hiểu lầm *trước khi* mã được viết — các giả định là hình thức hiểu lầm nguy hiểm nhất.

**Viết một tài liệu đặc tả bao gồm sáu lĩnh vực cốt lõi sau:**

1. **Mục tiêu (Objective)** — Chúng ta đang xây dựng cái gì và tại sao? Ai là người dùng? Thành công trông như thế nào?

2. **Câu lệnh (Commands)** — Các câu lệnh có thể thực thi đầy đủ với các flag, không chỉ là tên công cụ.
   ```
   Build: npm run build
   Test: npm test -- --coverage
   Lint: npm run lint --fix
   Dev: npm run dev
   ```

3. **Cấu trúc dự án (Project Structure)** — Nơi chứa mã nguồn, nơi đặt các bản kiểm thử (tests), nơi đặt tài liệu.
   ```
   src/           → Application source code
   src/components → React components
   src/lib        → Shared utilities
   tests/         → Unit and integration tests
   e2e/           → End-to-end tests
   docs/          → Documentation
   ```

4. **Phong cách mã (Code Style)** — Một đoạn mã thực tế thể hiện phong cách của bạn có giá trị hơn ba đoạn văn mô tả nó. Bao gồm các quy ước đặt tên, quy tắc định dạng và ví dụ về kết quả đầu ra tốt.

5. **Chiến lược kiểm thử (Testing Strategy)** — Framework nào, nơi đặt tests, kỳ vọng về độ bao phủ (coverage), cấp độ kiểm thử nào cho các vấn đề nào.

6. **Ranh giới (Boundaries)** — Hệ thống ba cấp độ:
   - **Luôn luôn làm:** Chạy tests trước khi commit, tuân thủ quy ước đặt tên, xác thực đầu vào (validate inputs)
   - **Hỏi trước khi làm:** Thay đổi database schema, thêm dependencies, thay đổi cấu hình CI
   - **Không bao giờ làm:** Commit các thông tin bảo mật (secrets), chỉnh sửa các thư mục vendor, xóa các tests bị lỗi mà không có sự chấp thuận

**Mẫu đặc tả (Spec template):**

```markdown
# Spec: [Tên Dự án/Tính năng]

## Mục tiêu
[Chúng ta đang xây dựng cái gì và tại sao. User stories hoặc tiêu chí chấp nhận (acceptance criteria).]

## Tech Stack
[Framework, ngôn ngữ, các dependencies chính kèm phiên bản]

## Câu lệnh
[Build, test, lint, dev — câu lệnh đầy đủ]

## Cấu trúc dự án
[Sơ đồ thư mục kèm mô tả]

## Phong cách mã
[Đoạn mã ví dụ + các quy ước chính]

## Chiến lược kiểm thử
[Framework, vị trí tests, yêu cầu về coverage, cấp độ kiểm thử]

## Ranh giới
- Luôn luôn: [...]
- Hỏi trước: [...]
- Không bao giờ: [...]

## Tiêu chí thành công
[Cách chúng ta biết việc này đã hoàn thành — các điều kiện cụ thể, có thể kiểm thử được]

## Câu hỏi mở
[Bất cứ điều gì chưa được giải quyết cần con người cung cấp thông tin]
```

**Định nghĩa lại các hướng dẫn thành tiêu chí thành công.** Khi nhận được các yêu cầu mơ hồ, hãy chuyển đổi chúng thành các điều kiện cụ thể:

```
YÊU CẦU: "Làm cho dashboard nhanh hơn"

TIÊU CHÍ THÀNH CÔNG ĐÃ ĐỊNH NGHĨA LẠI:
- Dashboard LCP < 2.5s trên kết nối 4G
- Việc tải dữ liệu ban đầu hoàn tất trong < 500ms
- Không có hiện tượng thay đổi bố cục khi tải (CLS < 0.1)
→ Đây có phải là các mục tiêu đúng không?
```

Điều này cho phép bạn lặp lại, thử lại và giải quyết vấn đề hướng tới một mục tiêu rõ ràng thay vì phỏng đoán ý nghĩa của từ "nhanh hơn".

### Giai đoạn 2: Lập kế hoạch (Plan)

Với bản đặc tả đã được xác nhận, hãy tạo một kế hoạch triển khai kỹ thuật:

1. Xác định các thành phần chính và các dependency của chúng
2. Xác định thứ tự triển khai (cái gì phải được xây dựng trước)
3. Ghi chú các rủi ro và chiến lược giảm thiểu
4. Xác định những gì có thể xây dựng song song và những gì phải tuần tự
5. Xác định các điểm kiểm tra xác nhận (verification checkpoints) giữa các giai đoạn

> Tuân theo `planning-and-task-breakdown` để biết cách lập bản đồ đồ thị phụ thuộc (dependency-graph mapping) và cơ chế cắt lát dọc (vertical-slicing) đằng sau các bước này; đó là nguồn chuẩn. Các đầu dòng ở trên là bản tóm tắt rút gọn; nếu có sự khác biệt, `planning-and-task-breakdown` sẽ được ưu tiên.

Kế hoạch cần phải có thể xem xét được: con người phải có thể đọc và nói "có, đó là cách tiếp cận đúng" hoặc "không, hãy thay đổi X".

### Giai đoạn 3: Tác vụ (Tasks)

Chia kế hoạch thành các tác vụ riêng biệt, có thể triển khai:

- Mỗi tác vụ nên có thể hoàn thành trong một phiên làm việc tập trung duy nhất
- Mỗi tác vụ có tiêu chí chấp nhận (acceptance criteria) rõ ràng
- Mỗi tác vụ bao gồm một bước xác nhận (test, build, kiểm tra thủ công)
- Các tác vụ được sắp xếp theo dependency, không phải theo tầm quan trọng cảm nhận được
- Không tác vụ nào nên yêu cầu thay đổi quá ~5 file

> Tuân theo `planning-and-task-breakdown` để biết đầy đủ cơ chế định cỡ tác vụ (task-sizing) và sắp xếp dependency; đó là nguồn chuẩn. Mẫu dưới đây là một dạng rút gọn; nếu có sự khác biệt, `planning-and-task-breakdown` sẽ được ưu tiên.

**Mẫu tác vụ (Task template):**
```markdown
- [ ] Task: [Mô tả]
  - Acceptance: [Điều gì phải đúng khi hoàn thành]
  - Verify: [Cách xác nhận — câu lệnh test, build, kiểm tra thủ công]
  - Files: [Những file nào sẽ bị ảnh hưởng]
```

### Giai đoạn 4: Triển khai (Implement)

Thực hiện các tác vụ từng cái một theo `skills/incremental-implementation/SKILL.md` (`incremental-implementation`) và `skills/test-driven-development/SKILL.md` (`test-driven-development`). Sử dụng `skills/context-engineering/SKILL.md` (`context-engineering`) để tải các phần đặc tả và file nguồn phù hợp ở mỗi bước thay vì làm ngập agent với toàn bộ bản đặc tả.

## Duy trì sự sống cho bản đặc tả

Bản đặc tả là một tài liệu sống, không phải là một sản phẩm làm một lần rồi bỏ:

- **Cập nhật khi quyết định thay đổi** — Nếu bạn phát hiện mô hình dữ liệu cần thay đổi, hãy cập nhật bản đặc tả trước, sau đó mới triển khai.
- **Cập nhật khi phạm vi thay đổi** — Các tính năng được thêm vào hoặc cắt bỏ cần được phản ánh trong bản đặc tả.
- **Commit bản đặc tả** — Bản đặc tả cần nằm trong hệ thống quản lý phiên bản cùng với mã nguồn.
- **Tham chiếu bản đặc tả trong PR** — Liên kết ngược lại phần đặc tả mà mỗi PR triển khai.

## Những lý do biện minh phổ biến

| Lý do biện minh | Thực tế |
|---|---|
| "Việc này đơn giản, tôi không cần spec" | Các tác vụ đơn giản không cần spec *dài*, nhưng chúng vẫn cần tiêu chí chấp nhận. Một bản spec hai dòng là ổn. |
| "Tôi sẽ viết spec sau khi lập trình xong" | Đó là viết tài liệu, không phải đặc tả. Giá trị của spec là buộc phải rõ ràng *trước khi* viết mã. |
| "Spec sẽ làm chúng ta chậm lại" | Một bản spec 15 phút ngăn chặn hàng giờ làm lại. "Waterfall" trong 15 phút tốt hơn là debug trong 15 giờ. |
| "Yêu cầu kiểu gì cũng sẽ thay đổi" | Đó là lý do tại sao spec là một tài liệu sống. Một bản spec lỗi thời vẫn tốt hơn là không có spec. |
| "Người dùng biết họ muốn gì" | Ngay cả những yêu cầu rõ ràng cũng có những giả định ngầm. Spec phơi bày những giả định đó. |

## Các dấu hiệu cảnh báo (Red Flags)

- Bắt đầu viết mã mà không có bất kỳ yêu cầu nào được viết ra
- Hỏi "tôi có nên bắt đầu xây dựng luôn không?" trước khi làm rõ "hoàn thành" nghĩa là gì
- Triển khai các tính năng không được đề cập trong bất kỳ spec hay danh sách tác vụ nào
- Đưa ra các quyết định kiến trúc mà không ghi chép lại
- Bỏ qua spec vì "quá rõ ràng là cần xây dựng cái gì"

## Xác nhận (Verification)

Trước khi tiến hành triển khai, hãy xác nhận:

- [ ] Bản đặc tả bao quát cả sáu lĩnh vực cốt lõi
- [ ] Con người đã xem xét và phê duyệt bản đặc tả
- [ ] Tiêu chí thành công cụ thể và có thể kiểm thử được
- [ ] Các ranh giới (Luôn luôn/Hỏi trước/Không bao giờ) đã được xác định
- [ ] Bản đặc tả được lưu vào một file trong repository
