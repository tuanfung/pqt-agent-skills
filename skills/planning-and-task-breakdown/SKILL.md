---
name: planning-and-task-breakdown
description: Chia nhỏ công việc thành các task có thứ tự. Sử dụng khi bạn có spec hoặc yêu cầu rõ ràng và cần chia nhỏ công việc thành các task có thể thực thi được. Sử dụng khi một task cảm thấy quá lớn để bắt đầu, khi bạn cần ước tính phạm vi (scope), hoặc khi có thể thực hiện công việc song song.
---

# Lập kế hoạch và Chia nhỏ Task

## Tổng quan

Chia nhỏ công việc thành các task nhỏ, có thể xác minh được với các tiêu chí chấp nhận (acceptance criteria) rõ ràng. Việc chia nhỏ task hiệu quả là điểm khác biệt giữa một agent hoàn thành công việc một cách tin cậy và một agent tạo ra một mớ hỗn độn. Mỗi task nên đủ nhỏ để có thể thực hiện, kiểm thử và xác minh trong một phiên làm việc tập trung duy nhất.

## Khi nào sử dụng

- Bạn có một spec và cần chia nó thành các đơn vị có thể thực thi được
- Một task cảm thấy quá lớn hoặc mơ hồ để bắt đầu
- Công việc cần được song song hóa giữa nhiều agent hoặc nhiều phiên làm việc
- Bạn cần truyền đạt phạm vi (scope) công việc cho con người
- Thứ tự thực hiện không rõ ràng

**Khi KHÔNG nên sử dụng:** Những thay đổi đơn lẻ trong một file với phạm vi hiển nhiên, hoặc khi spec đã chứa các task được định nghĩa rõ ràng.

## Quy trình lập kế hoạch

### Bước 1: Chuyển sang chế độ Lập kế hoạch

Trước khi viết bất kỳ đoạn code nào, hãy hoạt động ở chế độ chỉ đọc (read-only):

- Đọc spec và các phần liên quan trong codebase
- Xác định các pattern và convention hiện có
- Ánh xạ các phụ thuộc (dependencies) giữa các component
- Ghi chú các rủi ro và những điểm chưa biết

**KHÔNG viết code trong quá trình lập kế hoạch.** Kết quả đầu ra là một tài liệu kế hoạch, không phải là phần thực thi.

### Bước 2: Xác định Biểu đồ Phụ thuộc (Dependency Graph)

Ánh xạ cái gì phụ thuộc vào cái gì:

```
Database schema
    │
    ├── API models/types
    │       │
    │       ├── API endpoints
    │       │       │
    │       │       └── Frontend API client
    │       │               │
    │       │               └── UI components
    │       │
    │       └── Validation logic
    │
    └── Seed data / migrations
```

Thứ tự thực hiện sẽ tuân theo biểu đồ phụ thuộc từ dưới lên trên: xây dựng nền móng trước.

### Bước 3: Chia cắt theo chiều dọc (Vertical Slicing)

Thay vì xây dựng toàn bộ database, sau đó toàn bộ API, rồi đến toàn bộ UI — hãy xây dựng một luồng tính năng hoàn chỉnh tại một thời điểm:

**Xấu (chia cắt theo chiều ngang - horizontal slicing):**
```
Task 1: Xây dựng toàn bộ database schema
Task 2: Xây dựng tất cả API endpoints
Task 3: Xây dựng tất cả UI components
Task 4: Kết nối mọi thứ lại với nhau
```

**Tốt (chia cắt theo chiều dọc - vertical slicing):**
```
Task 1: Người dùng có thể tạo tài khoản (schema + API + UI cho đăng ký)
Task 2: Người dùng có thể đăng nhập (auth schema + API + UI cho đăng nhập)
Task 3: Người dùng có thể tạo task (task schema + API + UI cho việc tạo)
Task 4: Người dùng có thể xem danh sách task (query + API + UI cho giao diện danh sách)
```

Mỗi lát cắt dọc sẽ cung cấp một chức năng hoạt động được và có thể kiểm thử.

### Bước 4: Viết Task

Mỗi task tuân theo cấu trúc sau:

```markdown
## Task [N]: [Tiêu đề mô tả ngắn gọn]

**Mô tả:** Một đoạn văn giải thích task này đạt được điều gì.

**Tiêu chí chấp nhận (Acceptance criteria):**
- [ ] [Điều kiện cụ thể, có thể kiểm thử]
- [ ] [Điều kiện cụ thể, có thể kiểm thử]

**Xác minh (Verification):**
- [ ] Tests vượt qua: `npm test -- --grep "feature-name"`
- [ ] Build thành công: `npm run build`
- [ ] Kiểm tra thủ công: [mô tả những gì cần xác minh]

**Phụ thuộc (Dependencies):** [Số thứ tự các task mà task này phụ thuộc vào, hoặc "None"]

**Các file có khả năng bị ảnh hưởng:**
- `src/path/to/file.ts`
- `tests/path/to/test.ts`

**Ước tính phạm vi (Estimated scope):** [Small: 1-2 files | Medium: 3-5 files | Large: 5+ files]
```

### Bước 5: Sắp xếp và Tạo điểm kiểm tra (Checkpoint)

Sắp xếp các task sao cho:

1. Các phụ thuộc được thỏa mãn (xây dựng nền móng trước)
2. Mỗi task để lại hệ thống trong trạng thái hoạt động được
3. Các điểm xác minh (verification checkpoints) xuất hiện sau mỗi 2-3 task
4. Các task rủi ro cao được thực hiện sớm (fail fast)

Thêm các điểm kiểm tra rõ ràng:

```markdown
## Checkpoint: Sau Task 1-3
- [ ] Tất cả tests vượt qua
- [ ] Ứng dụng build không có lỗi
- [ ] Luồng người dùng cốt lõi hoạt động end-to-end
- [ ] Review với con người trước khi tiếp tục
```

## Hướng dẫn về Kích thước Task

| Kích thước | Files | Phạm vi | Ví dụ |
|------|-------|-------|---------|
| **XS** | 1 | Thay đổi một function đơn lẻ hoặc config | Thêm một quy tắc validation |
| **S** | 1-2 | Một component hoặc endpoint | Thêm một API endpoint mới |
| **M** | 3-5 | Một lát cắt tính năng | Luồng đăng ký người dùng |
| **L** | 5-8 | Tính năng đa component | Tìm kiếm với lọc và phân trang |
| **XL** | 8+ | **Quá lớn — cần chia nhỏ thêm** | — |

Nếu một task có kích thước L hoặc lớn hơn, nó nên được chia thành các task nhỏ hơn. Một agent hoạt động tốt nhất với các task S và M.

**Khi nào cần chia nhỏ task hơn nữa:**
- Nó mất nhiều hơn một phiên làm việc tập trung (khoảng 2+ giờ làm việc của agent)
- Bạn không thể mô tả tiêu chí chấp nhận trong 3 gạch đầu dòng trở xuống
- Nó chạm vào hai hoặc nhiều subsystem độc lập (ví dụ: auth và billing)
- Bạn thấy mình viết từ "và" trong tiêu đề task (dấu hiệu đó là hai task)

## Mẫu Tài liệu Kế hoạch

```markdown
# Kế hoạch thực hiện: [Tên Tính năng/Dự án]

## Tổng quan
[Một đoạn văn tóm tắt về những gì chúng ta đang xây dựng]

## Quyết định Kiến trúc (Architecture Decisions)
- [Quyết định chính 1 và lý do]
- [Quyết định chính 2 và lý do]

## Danh sách Task

### Giai đoạn 1: Nền móng (Foundation)
- [ ] Task 1: ...
- [ ] Task 2: ...

### Checkpoint: Nền móng
- [ ] Tests vượt qua, build sạch

### Giai đoạn 2: Các tính năng cốt lõi (Core Features)
- [ ] Task 3: ...
- [ ] Task 4: ...

### Checkpoint: Tính năng cốt lõi
- [ ] Luồng end-to-end hoạt động

### Giai đoạn 3: Hoàn thiện (Polish)
- [ ] Task 5: ...
- [ ] Task 6: ...

### Checkpoint: Hoàn tất
- [ ] Tất cả tiêu chí chấp nhận được đáp ứng
- [ ] Sẵn sàng để review

## Rủi ro và Giảm thiểu
| Rủi ro | Tác động | Giải pháp giảm thiểu |
|------|--------|------------|
| [Rủi ro] | [High/Med/Low] | [Chiến lược] |

## Câu hỏi còn bỏ ngỏ
- [Câu hỏi cần con người trả lời]
```

## Cơ hội Song song hóa

Khi có sẵn nhiều agent hoặc phiên làm việc:

- **An toàn để song song hóa:** Các lát cắt tính năng độc lập, viết test cho các tính năng đã thực hiện, viết tài liệu
- **Phải tuần tự:** Database migrations, thay đổi shared state, chuỗi phụ thuộc (dependency chains)
- **Cần phối hợp:** Các tính năng chia sẻ một API contract (định nghĩa contract trước, sau đó song song hóa)

## Những lời tự trấn an phổ biến

| Lời tự trấn an | Thực tế |
|---|---|
| "Tôi sẽ vừa làm vừa tìm hiểu" | Đó là cách bạn kết thúc với một mớ hỗn độn và phải làm lại. 10 phút lập kế hoạch tiết kiệm được nhiều giờ thực hiện. |
| "Các task quá hiển nhiên" | Vẫn nên viết chúng ra. Các task rõ ràng sẽ làm lộ ra các phụ thuộc tiềm ẩn và các trường hợp biên (edge cases) bị bỏ sót. |
| "Lập kế hoạch là tốn thời gian" | Lập kế hoạch chính là nhiệm vụ. Thực hiện mà không có kế hoạch thì chỉ là gõ phím. |
| "Tôi có thể nhớ hết trong đầu" | Cửa sổ context là hữu hạn. Kế hoạch bằng văn bản sẽ tồn tại qua các ranh giới phiên làm việc và quá trình nén (compaction). |

## Dấu hiệu cảnh báo (Red Flags)

- Bắt đầu thực hiện mà không có danh sách task bằng văn bản
- Các task ghi là "implement the feature" mà không có tiêu chí chấp nhận
- Không có các bước xác minh trong kế hoạch
- Tất cả các task đều có kích thước XL
- Không có checkpoint giữa các task
- Thứ tự phụ thuộc không được xem xét

## Xác minh

Trước khi bắt đầu thực hiện, hãy xác nhận:

- [ ] Mọi task đều có tiêu chí chấp nhận
- [ ] Mọi task đều có bước xác minh
- [ ] Các phụ thuộc của task đã được xác định và sắp xếp đúng thứ tự
- [ ] Không có task nào chạm vào quá ~5 file
- [ ] Có các checkpoint giữa các giai đoạn chính
- [ ] Con người đã xem xét và phê duyệt kế hoạch

## Xem thêm

Tiêu chí chấp nhận là theo từng task và trả lời câu hỏi "liệu chúng ta có xây dựng đúng thứ cần thiết không?". Chúng nằm trên Definition of Done của toàn dự án, tiêu chuẩn chung mà mọi task phải vượt qua trước khi được coi là hoàn thành. Xem `references/definition-of-done.md`.
