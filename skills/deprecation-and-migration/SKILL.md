---
name: deprecation-and-migration
description: Quản lý việc loại bỏ (deprecation) và di chuyển (migration). Sử dụng khi xóa bỏ các hệ thống, API hoặc tính năng cũ. Sử dụng khi di chuyển người dùng từ triển khai này sang triển khai khác. Sử dụng khi quyết định xem nên duy trì hay ngừng hỗ trợ (sunset) mã nguồn hiện có.
---

# Loại bỏ và Di chuyển (Deprecation and Migration)

## Tổng quan

Mã nguồn là một khoản nợ (liability), không phải là tài sản. Mỗi dòng mã đều có chi phí bảo trì liên tục — các lỗi cần sửa, các dependency cần cập nhật, các bản vá bảo mật cần áp dụng và các kỹ sư mới cần được hướng dẫn (onboard). Deprecation là kỷ luật loại bỏ những đoạn mã không còn mang lại giá trị tương xứng, và migration là quá trình chuyển đổi người dùng một cách an toàn từ cái cũ sang cái mới.

Hầu hết các tổ chức kỹ thuật đều giỏi trong việc xây dựng. Rất ít tổ chức giỏi trong việc loại bỏ. Skill này giải quyết khoảng trống đó.

## Khi nào nên sử dụng

- Thay thế một hệ thống, API hoặc library cũ bằng một cái mới
- Ngừng hỗ trợ (sunsetting) một tính năng không còn cần thiết
- Hợp nhất các triển khai trùng lặp
- Loại bỏ mã chết (dead code) mà không ai sở hữu nhưng mọi người đều phụ thuộc vào
- Lập kế hoạch cho vòng đời của một hệ thống mới (việc lập kế hoạch deprecation bắt đầu ngay từ giai đoạn thiết kế)
- Quyết định xem nên duy trì một hệ thống cũ (legacy system) hay đầu tư vào migration

## Các nguyên tắc cốt lõi

### Mã nguồn là một khoản nợ

Mỗi dòng mã đều có chi phí duy trì: nó cần các bản test, tài liệu, bản vá bảo mật, cập nhật dependency và gây tốn công sức tư duy cho bất kỳ ai làm việc xung quanh. Giá trị của mã nguồn nằm ở chức năng mà nó cung cấp, chứ không phải bản thân đoạn mã đó. Khi cùng một chức năng có thể được cung cấp với ít mã hơn, ít phức tạp hơn hoặc thông qua các abstraction tốt hơn — thì đoạn mã cũ nên được loại bỏ.

### Định luật Hyrum khiến việc loại bỏ trở nên khó khăn

Với đủ số lượng người dùng, mọi hành vi có thể quan sát được đều trở thành điều mà họ phụ thuộc vào — bao gồm cả các lỗi, những điểm bất thường về thời gian (timing quirks) và các tác dụng phụ không được ghi chép trong tài liệu. Đây là lý do tại sao deprecation đòi hỏi migration chủ động, chứ không chỉ là thông báo. Người dùng không thể "chỉ cần chuyển sang" khi họ phụ thuộc vào những hành vi mà bản thay thế không tái lập được.

### Lập kế hoạch Deprecation bắt đầu từ lúc Thiết kế

Khi xây dựng một thứ gì đó mới, hãy tự hỏi: "Chúng ta sẽ loại bỏ thứ này như thế nào trong 3 năm tới?" Các hệ thống được thiết kế với interface sạch, feature flags và diện tích bề mặt (surface area) tối thiểu sẽ dễ dàng deprecate hơn so với các hệ thống làm rò rỉ các chi tiết triển khai (implementation details) ở khắp mọi nơi.

## Quyết định Deprecation

Trước khi deprecate bất cứ điều gì, hãy trả lời các câu hỏi sau:

```
1. Hệ thống này còn cung cấp giá trị duy nhất nào không?
   → Nếu có, hãy duy trì. Nếu không, tiếp tục.

2. Có bao nhiêu người dùng/consumer phụ thuộc vào nó?
   → Định lượng phạm vi migration.

3. Đã có bản thay thế chưa?
   → Nếu chưa, hãy xây dựng bản thay thế trước. Đừng deprecate mà không có phương án thay thế.

4. Chi phí migration cho mỗi consumer là bao nhiêu?
   → Nếu có thể tự động hóa dễ dàng, hãy thực hiện. Nếu phải làm thủ công và tốn nhiều công sức, hãy cân nhắc so với chi phí bảo trì.

5. Chi phí bảo trì liên tục nếu KHÔNG deprecate là bao nhiêu?
   → Rủi ro bảo mật, thời gian của kỹ sư, chi phí cơ hội do sự phức tạp gây ra.
```

## Deprecation Bắt buộc và Khuyến nghị

| Loại | Khi nào sử dụng | Cơ chế |
|------|-------------|-----------|
| **Khuyến nghị (Advisory)** | Migration là tùy chọn, hệ thống cũ ổn định | Cảnh báo, tài liệu, nhắc nhở. Người dùng migrate theo lộ trình riêng của họ. |
| **Bắt buộc (Compulsory)** | Hệ thống cũ có vấn đề bảo mật, cản trở tiến độ hoặc chi phí bảo trì không còn bền vững | Hạn chót cố định. Hệ thống cũ sẽ bị xóa vào ngày X. Cung cấp công cụ hỗ trợ migration. |

**Mặc định là khuyến nghị.** Chỉ sử dụng bắt buộc khi chi phí bảo trì hoặc rủi ro đủ lớn để buộc phải migration. Deprecation bắt buộc yêu cầu cung cấp công cụ hỗ trợ migration, tài liệu và hỗ trợ — bạn không thể chỉ thông báo một hạn chót.

## Quy trình Migration

### Bước 1: Xây dựng bản thay thế

Đừng deprecate nếu không có phương án thay thế hoạt động tốt. Bản thay thế phải:

- Đáp ứng tất cả các use case quan trọng của hệ thống cũ
- Có tài liệu và hướng dẫn migration
- Được chứng minh hiệu quả trong môi trường production (không chỉ là "về lý thuyết là tốt hơn")

### Bước 2: Thông báo và Lập tài liệu

```markdown
## Thông báo Deprecation: OldService

**Trạng thái:** Đã bị deprecate từ 2025-03-01
**Thay thế:** NewService (xem hướng dẫn migration bên dưới)
**Ngày xóa:** Khuyến nghị — chưa có hạn chót cố định
**Lý do:** OldService yêu cầu scale thủ công và thiếu khả năng observability.
            NewService xử lý cả hai một cách tự động.

### Hướng dẫn Migration
1. Thay thế `import { client } from 'old-service'` bằng `import { client } from 'new-service'`
2. Cập nhật cấu hình (xem ví dụ bên dưới)
3. Chạy script xác minh migration: `npx migrate-check`
```

### Bước 3: Migration lũy tiến (Incrementally)

Migrate từng consumer một, không làm tất cả cùng lúc. Với mỗi consumer:

```
1. Xác định tất cả các điểm tiếp xúc (touchpoints) với hệ thống bị deprecate
2. Cập nhật để sử dụng bản thay thế
3. Xác minh hành vi tương khớp (tests, kiểm tra tích hợp)
4. Xóa bỏ các tham chiếu đến hệ thống cũ
5. Xác nhận không có lỗi hồi quy (regressions)
```

**Quy tắc Churn (The Churn Rule):** Nếu bạn sở hữu cơ sở hạ tầng bị deprecate, bạn có trách nhiệm migration người dùng của mình — hoặc cung cấp các bản cập nhật tương thích ngược (backward-compatible) mà không yêu cầu migration. Đừng thông báo deprecation rồi để người dùng tự tìm cách giải quyết.

### Bước 4: Loại bỏ hệ thống cũ

Chỉ sau khi tất cả các consumer đã migrate:

```
1. Xác minh không còn sử dụng hoạt động (metrics, logs, phân tích dependency)
2. Xóa mã nguồn
3. Xóa các bản test, tài liệu và cấu hình liên quan
4. Xóa các thông báo deprecation
5. Ăn mừng — loại bỏ mã nguồn là một thành tựu
```

## Các mô hình Migration (Migration Patterns)

### Strangler Pattern

Chạy hệ thống cũ và mới song song. Điều hướng traffic dần dần từ cũ sang mới. Khi hệ thống cũ xử lý 0% traffic, hãy loại bỏ nó.

```
Giai đoạn 1: Hệ thống mới xử lý 0%, cũ xử lý 100%
Giai đoạn 2: Hệ thống mới xử lý 10% (canary)
Giai đoạn 3: Hệ thống mới xử lý 50%
Giai đoạn 4: Hệ thống mới xử lý 100%, hệ thống cũ ở trạng thái chờ (idle)
Giai đoạn 5: Loại bỏ hệ thống cũ
```

### Adapter Pattern

Tạo một adapter để chuyển đổi các lời gọi từ interface cũ sang triển khai mới. Các consumer tiếp tục sử dụng interface cũ trong khi bạn migration phần backend.

```typescript
// Adapter: old interface, new implementation
class LegacyTaskService implements OldTaskAPI {
  constructor(private newService: NewTaskService) {}

  // Old method signature, delegates to new implementation
  getTask(id: number): OldTask {
    const task = this.newService.findById(String(id));
    return this.toOldFormat(task);
  }
}
```

### Migration bằng Feature Flag

Sử dụng feature flags để chuyển đổi từng consumer từ hệ thống cũ sang hệ thống mới:

```typescript
function getTaskService(userId: string): TaskService {
  if (featureFlags.isEnabled('new-task-service', { userId })) {
    return new NewTaskService();
  }
  return new LegacyTaskService();
}
```

## Mã nguồn Zombie (Zombie Code)

Mã nguồn Zombie là mã mà không ai sở hữu nhưng mọi người đều phụ thuộc vào. Nó không được bảo trì tích cực, không có chủ sở hữu rõ ràng, và tích tụ các lỗ hổng bảo mật cũng như các vấn đề tương thích. Các dấu hiệu:

- Không có commit nào trong hơn 6 tháng nhưng vẫn có consumer đang hoạt động
- Không có maintainer hoặc team được chỉ định
- Các bản test bị fail mà không ai sửa
- Các dependency có lỗ hổng bảo mật đã biết nhưng không ai cập nhật
- Tài liệu tham chiếu đến các hệ thống không còn tồn tại

**Giải pháp:** Hoặc là chỉ định một chủ sở hữu và bảo trì đúng cách, hoặc deprecate nó với một kế hoạch migration cụ thể. Mã nguồn Zombie không thể cứ nằm trong trạng thái lửng lơ — nó hoặc là được đầu tư, hoặc là bị loại bỏ.

## Các lý lẽ biện minh phổ biến

| Lý lẽ | Thực tế |
|---|---|
| "Nó vẫn hoạt động mà, tại sao phải xóa?" | Mã hoạt động nhưng không ai bảo trì sẽ tích tụ nợ bảo mật và sự phức tạp. Chi phí bảo trì tăng lên một cách âm thầm. |
| "Có thể sau này ai đó sẽ cần đến" | Nếu sau này cần, có thể xây dựng lại. Việc giữ lại mã không sử dụng 'để phòng hờ' tốn kém hơn là xây dựng lại từ đầu. |
| "Chi phí migration quá đắt" | Hãy so sánh chi phí migration với chi phí bảo trì liên tục trong 2-3 năm. Migration thường rẻ hơn về lâu dài. |
| "Chúng tôi sẽ deprecate sau khi hoàn thành hệ thống mới" | Lập kế hoạch deprecation bắt đầu từ lúc thiết kế. Khi hệ thống mới hoàn tất, bạn sẽ có những ưu tiên mới. Hãy lập kế hoạch ngay bây giờ. |
| "Người dùng sẽ tự migration" | Họ sẽ không làm vậy. Hãy cung cấp công cụ, tài liệu và các động lực — hoặc tự thực hiện migration (Quy tắc Churn). |
| "Chúng tôi có thể duy trì cả hai hệ thống vô thời hạn" | Hai hệ thống làm cùng một việc nghĩa là chi phí bảo trì, kiểm thử, tài liệu và hướng dẫn (onboarding) tăng gấp đôi. |

## Các dấu hiệu cảnh báo (Red Flags)

- Các hệ thống bị deprecate nhưng không có bản thay thế
- Thông báo deprecation nhưng không có công cụ hỗ trợ migration hoặc tài liệu
- Deprecation 'mềm' (khuyến nghị) đã kéo dài nhiều năm mà không có tiến triển
- Mã nguồn Zombie không có chủ sở hữu nhưng vẫn có consumer hoạt động
- Thêm tính năng mới vào một hệ thống đã bị deprecate (thay vào đó hãy đầu tư vào bản thay thế)
- Deprecation mà không đo lường mức độ sử dụng hiện tại
- Xóa mã nguồn mà không xác minh rằng không còn consumer nào đang hoạt động

## Xác minh

Sau khi hoàn tất một quá trình deprecation:

- [ ] Bản thay thế đã được chứng minh trong production và đáp ứng tất cả các use case quan trọng
- [ ] Có hướng dẫn migration với các bước và ví dụ cụ thể
- [ ] Tất cả các consumer đang hoạt động đã được migrate (được xác minh qua metrics/logs)
- [ ] Mã cũ, tests, tài liệu và cấu hình đã được loại bỏ hoàn toàn
- [ ] Không còn tham chiếu nào đến hệ thống bị deprecate trong codebase
- [ ] Các thông báo deprecation đã được xóa bỏ (vì chúng đã hoàn thành mục đích)
