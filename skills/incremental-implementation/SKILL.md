---
name: incremental-implementation
description: Triển khai các thay đổi một cách tăng dần. Sử dụng khi triển khai bất kỳ tính năng hoặc thay đổi nào ảnh hưởng đến nhiều hơn một file. Sử dụng khi bạn sắp viết một lượng lớn mã cùng một lúc, hoặc khi một nhiệm vụ cảm thấy quá lớn để hoàn thành trong một bước.
---

# Incremental Implementation

## Tổng quan

Xây dựng theo các lát cắt dọc mỏng — triển khai một phần, kiểm thử (test), xác minh (verify), sau đó mở rộng. Tránh triển khai toàn bộ một tính năng trong một lần. Mỗi bước tăng dần nên để hệ thống ở trạng thái hoạt động và có thể kiểm thử được. Đây là kỷ luật thực thi giúp các tính năng lớn trở nên dễ quản lý hơn.

## Khi nào nên sử dụng

- Triển khai bất kỳ thay đổi nào ảnh hưởng đến nhiều file
- Xây dựng một tính năng mới từ việc phân rã nhiệm vụ (task breakdown)
- Refactoring mã nguồn hiện có
- Bất cứ khi nào bạn định viết nhiều hơn khoảng 100 dòng mã trước khi kiểm thử

**Khi KHÔNG nên sử dụng:** Các thay đổi đơn lẻ trong một file, một hàm nơi phạm vi đã ở mức tối thiểu.

## Chu trình tăng dần (The Increment Cycle)

```
┌──────────────────────────────────────┐
│                                      │
│   Implement ──→ Test ──→ Verify ──┐  │
│       ▲                           │  │
│       └───── Commit ◄─────────────┘  │
│              │                       │
│              ▼                       │
│          Next slice                  │
│                                      │
└──────────────────────────────────────┘
```

Đối với mỗi lát cắt (slice):

1. **Implement**: Triển khai phần chức năng nhỏ nhất và hoàn chỉnh
2. **Test**: Chạy bộ kiểm thử (hoặc viết test nếu chưa có)
3. **Verify**: Xác nhận lát cắt hoạt động như mong đợi (tests pass, build thành công, kiểm tra thủ công)
4. **Commit**: Lưu tiến trình với một thông điệp mô tả (xem `git-workflow-and-versioning` để được hướng dẫn về atomic commit)
5. **Move to the next slice**: Tiếp tục thực hiện, không bắt đầu lại từ đầu

## Chiến lược phân lát (Slicing Strategies)

### Lát cắt dọc (Vertical Slices - Ưu tiên)

Xây dựng một luồng hoàn chỉnh đi xuyên qua toàn bộ stack:

```
Slice 1: Tạo một task (DB + API + basic UI)
    → Tests pass, người dùng có thể tạo task thông qua UI

Slice 2: Liệt kê các task (query + API + UI)
    → Tests pass, người dùng có thể xem các task của họ

Slice 3: Chỉnh sửa task (update + API + UI)
    → Tests pass, người dùng có thể sửa đổi task

Slice 4: Xóa một task (delete + API + UI + xác nhận)
    → Tests pass, hoàn thành đầy đủ CRUD
```

Mỗi lát cắt cung cấp một chức năng hoạt động hoàn chỉnh từ đầu đến cuối (end-to-end).

### Phân lát theo hợp đồng (Contract-First Slicing)

Khi backend và frontend cần phát triển song song:

```
Slice 0: Định nghĩa hợp đồng API (types, interfaces, OpenAPI spec)
Slice 1a: Triển khai backend theo hợp đồng + API tests
Slice 1b: Triển khai frontend với dữ liệu giả (mock data) khớp với hợp đồng
Slice 2: Tích hợp và kiểm thử end-to-end
```

### Phân lát theo rủi ro (Risk-First Slicing)

Giải quyết phần rủi ro nhất hoặc không chắc chắn nhất trước:

```
Slice 1: Chứng minh kết nối WebSocket hoạt động (rủi ro cao nhất)
Slice 2: Xây dựng cập nhật task thời gian thực trên kết nối đã được chứng minh
Slice 3: Thêm hỗ trợ ngoại tuyến (offline support) và kết nối lại
```

Nếu Slice 1 thất bại, bạn sẽ phát hiện ra trước khi đầu tư vào Slice 2 và 3.

## Quy tắc triển khai

### Quy tắc 0: Đơn giản là trên hết (Simplicity First)

Trước khi viết bất kỳ mã nào, hãy hỏi: "Điều đơn giản nhất có thể hoạt động là gì?"

Sau khi viết mã, hãy xem xét lại theo các tiêu chí sau:
- Việc này có thể thực hiện với ít dòng mã hơn không?
- Các trừu tượng hóa (abstractions) này có xứng đáng với độ phức tạp mà chúng tạo ra không?
- Một kỹ sư dày dạn kinh nghiệm (staff engineer) khi nhìn vào đây sẽ nói "tại sao bạn không chỉ..."?
- Tôi đang xây dựng cho các yêu cầu giả định trong tương lai, hay cho nhiệm vụ hiện tại?

```
KIỂM TRA ĐỘ ĐƠN GIẢN:
✗ Generic EventBus với middleware pipeline cho một thông báo duy nhất
✓ Một lời gọi hàm đơn giản

✗ Abstract factory pattern cho hai component tương tự nhau
✓ Hai component đơn giản với các tiện ích dùng chung

✗ Config-driven form builder cho ba form
✓ Ba form component
```

Ba dòng mã tương tự nhau thì tốt hơn là một sự trừu tượng hóa sớm. Hãy triển khai phiên bản ngây thơ, hiển nhiên là chính xác trước. Chỉ tối ưu hóa sau khi tính chính xác đã được chứng minh bằng tests.

### Quy tắc 0.5: Kỷ luật về phạm vi (Scope Discipline)

Chỉ chạm vào những gì nhiệm vụ yêu cầu.

KHÔNG thực hiện:
- "Dọn dẹp" mã nằm cạnh thay đổi của bạn
- Refactor imports trong các file mà bạn không chỉnh sửa
- Xóa các comment mà bạn không hiểu đầy đủ
- Thêm các tính năng không có trong spec vì chúng "có vẻ hữu ích"
- Hiện đại hóa cú pháp trong các file bạn chỉ đang đọc

Nếu bạn nhận thấy điều gì đó đáng để cải thiện nằm ngoài phạm vi nhiệm vụ, hãy ghi chú lại — đừng sửa nó:

```
NHẬN THẤY NHƯNG KHÔNG CHẠM VÀO:
- src/utils/format.ts có một import không sử dụng (không liên quan đến nhiệm vụ này)
- Auth middleware có thể sử dụng thông báo lỗi tốt hơn (nhiệm vụ riêng biệt)
→ Bạn có muốn tôi tạo nhiệm vụ cho những việc này không?
```

### Quy tắc 1: Mỗi lần một việc (One Thing at a Time)

Mỗi bước tăng dần chỉ thay đổi một điều logic. Đừng trộn lẫn các mối quan tâm:

**Tệ:** Một commit vừa thêm component mới, vừa refactor một component cũ, vừa cập nhật build config.

**Tốt:** Ba commit riêng biệt — mỗi commit cho một thay đổi.

### Quy tắc 2: Giữ cho mã có thể biên dịch được (Keep It Compilable)

Sau mỗi bước tăng dần, dự án phải build được và các test hiện có phải pass. Đừng để codebase ở trạng thái bị hỏng giữa các lát cắt.

### Quy tắc 3: Feature Flags cho các tính năng chưa hoàn tất

Nếu một tính năng chưa sẵn sàng cho người dùng nhưng bạn cần merge các bước tăng dần:

```typescript
// Feature flag cho công việc đang triển khai
const ENABLE_TASK_SHARING = process.env.FEATURE_TASK_SHARING === 'true';

if (ENABLE_TASK_SHARING) {
  // Giao diện sharing mới
}
```

Điều này cho phép bạn merge các bước tăng dần nhỏ vào nhánh chính mà không làm lộ ra những phần việc chưa hoàn chỉnh.

### Quy tắc 4: Mặc định an toàn (Safe Defaults)

Mã mới nên mặc định theo hành vi an toàn, thận trọng:

```typescript
// An toàn: mặc định là tắt, chọn tham gia (opt-in)
export function createTask(data: TaskInput, options?: { notify?: boolean }) {
  const shouldNotify = options?.notify ?? false;
  // ...
}
```

### Quy tắc 5: Thân thiện với việc rollback (Rollback-Friendly)

Mỗi bước tăng dần nên có khả năng hoàn tác độc lập:

- Các thay đổi mang tính bổ sung (file mới, hàm mới) dễ dàng revert
- Các chỉnh sửa đối với mã hiện có nên ở mức tối thiểu và tập trung
- Các database migrations nên có các migration rollback tương ứng
- Tránh xóa một thứ trong một commit và thay thế nó trong cùng commit đó — hãy tách chúng ra

## Làm việc với Agent

Khi hướng dẫn một agent triển khai tăng dần:

```
"Hãy triển khai Nhiệm vụ 3 từ bản kế hoạch.

Bắt đầu với thay đổi schema database và API endpoint.
Đừng chạm vào UI lúc này — chúng ta sẽ làm việc đó trong bước tăng dần tiếp theo.

Sau khi triển khai, hãy chạy `npm test` và `npm run build` để xác minh
không có gì bị hỏng."
```

Hãy rõ ràng về những gì nằm trong phạm vi (in scope) và những gì KHÔNG nằm trong phạm vi cho mỗi bước tăng dần.

## Danh sách kiểm tra tăng dần (Increment Checklist)

Sau mỗi bước tăng dần, hãy xác minh:

- [ ] Thay đổi chỉ làm một việc và làm hoàn chỉnh việc đó
- [ ] Tất cả các test hiện có vẫn pass (`npm test`)
- [ ] Build thành công (`npm run build`)
- [ ] Kiểm tra kiểu (Type checking) thành công (`npx tsc --noEmit`)
- [ ] Linting thành công (`npm run lint`)
- [ ] Chức năng mới hoạt động như mong đợi
- [ ] Thay đổi được commit với thông điệp mô tả

**Lưu ý:** Chạy lệnh xác minh sau mỗi thay đổi có thể ảnh hưởng đến nó. Sau một lần chạy thành công, đừng lặp lại cùng một lệnh trừ khi mã đã thay đổi kể từ lần cuối — chạy lại trên mã không thay đổi không cung cấp thêm thông tin gì.

## Các lý lẽ bào chữa phổ biến (Common Rationalizations)

| Lý lẽ bào chữa | Thực tế |
|---|---|
| "Tôi sẽ test tất cả vào cuối cùng" | Lỗi sẽ tích tụ. Một bug ở Slice 1 khiến Slice 2-5 bị sai. Hãy test từng lát cắt. |
| "Làm tất cả một lúc sẽ nhanh hơn" | Nó *cảm giác* nhanh hơn cho đến khi có thứ gì đó hỏng và bạn không thể tìm thấy dòng nào trong số 500 dòng thay đổi gây ra lỗi. |
| "Những thay đổi này quá nhỏ để commit riêng biệt" | Commit nhỏ là miễn phí. Commit lớn che giấu bug và khiến việc rollback trở nên đau đớn. |
| "Tôi sẽ thêm feature flag sau" | Nếu tính năng chưa hoàn tất, nó không nên hiển thị cho người dùng. Hãy thêm flag ngay bây giờ. |
| "Phần refactor này đủ nhỏ để gộp chung" | Refactor trộn lẫn với tính năng khiến cả hai đều khó review và debug hơn. Hãy tách chúng ra. |
| "Để tôi chạy lại lệnh build một lần nữa cho chắc" | Sau một lần chạy thành công, việc lặp lại cùng một lệnh không mang lại giá trị gì trừ khi mã đã thay đổi. Hãy chạy lại sau các chỉnh sửa tiếp theo, không phải để trấn an. |

## Dấu hiệu cảnh báo (Red Flags)

- Viết hơn 100 dòng mã mà không chạy test
- Nhiều thay đổi không liên quan trong một bước tăng dần duy nhất
- Mở rộng phạm vi theo kiểu "để tôi thêm nhanh cái này nữa"
- Bỏ qua bước test/verify để tiến nhanh hơn
- Build hoặc tests bị hỏng giữa các bước tăng dần
- Tích tụ lượng lớn thay đổi chưa commit
- Xây dựng các trừu tượng hóa trước khi trường hợp sử dụng thứ ba yêu cầu
- Chạm vào các file ngoài phạm vi nhiệm vụ "tiện tay"
- Tạo các file tiện ích (utility) mới cho các thao tác chỉ dùng một lần
- Chạy cùng một lệnh build/test hai lần liên tiếp mà không có bất kỳ thay đổi mã nào ở giữa

## Xác minh (Verification)

Sau khi hoàn thành tất cả các bước tăng dần cho một nhiệm vụ:

- [ ] Mỗi bước tăng dần đã được kiểm thử và commit riêng biệt
- [ ] Toàn bộ bộ test đều pass
- [ ] Build sạch (clean)
- [ ] Tính năng hoạt động end-to-end như mô tả
- [ ] Không còn thay đổi nào chưa commit

## Xem thêm

Xác minh theo từng bước tăng dần là bước kiểm tra cục bộ. Trước khi tuyên bố một nhiệm vụ đã hoàn thành, hãy áp dụng Definition of Done của toàn dự án như một cánh cửa cuối cùng, tiêu chuẩn mà mọi bước tăng dần đều phải vượt qua bất kể nhiệm vụ là gì. Xem `references/definition-of-done.md`.
