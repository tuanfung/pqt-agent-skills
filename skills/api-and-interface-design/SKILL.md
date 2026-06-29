---
name: api-and-interface-design
description: Hướng dẫn thiết kế API và giao diện ổn định. Sử dụng khi thiết kế API, ranh giới module, hoặc bất kỳ giao diện công khai nào. Sử dụng khi tạo các REST hoặc GraphQL endpoint, định nghĩa type contracts giữa các module, hoặc thiết lập ranh giới giữa frontend và backend.
---

# Thiết kế API và Giao diện

## Tổng quan

Thiết kế các giao diện ổn định, được tài liệu hóa tốt và khó bị sử dụng sai. Các giao diện tốt giúp việc thực hiện đúng trở nên dễ dàng và việc làm sai trở nên khó khăn. Điều này áp dụng cho REST API, GraphQL schema, ranh giới module, component prop, và bất kỳ bề mặt nào mà một đoạn mã giao tiếp với đoạn mã khác.

## Khi nào sử dụng

- Thiết kế các API endpoint mới
- Định nghĩa ranh giới module hoặc contract giữa các nhóm
- Tạo giao diện component prop
- Thiết lập database schema định hình cấu trúc API
- Thay đổi các giao diện công khai hiện có

## Các nguyên tắc cốt lõi

### Định luật Hyrum (Hyrum's Law)

> Với một số lượng người dùng API đủ lớn, mọi hành vi có thể quan sát được của hệ thống sẽ được ai đó phụ thuộc vào, bất kể bạn hứa hẹn điều gì trong contract.

Điều này có nghĩa là: mọi hành vi công khai — bao gồm cả những đặc điểm không được ghi chép, nội dung thông báo lỗi, thời gian phản hồi và thứ tự — đều trở thành một contract thực tế một khi người dùng phụ thuộc vào nó. Hệ quả đối với thiết kế:

- **Hãy có ý định rõ ràng về những gì bạn công khai.** Mọi hành vi có thể quan sát được đều là một cam kết tiềm năng.
- **Không làm rò rỉ chi tiết thực hiện (implementation details).** Nếu người dùng có thể quan sát thấy, họ sẽ phụ thuộc vào nó.
- **Lập kế hoạch cho việc deprecation ngay từ lúc thiết kế.** Xem `deprecation-and-migration` để biết cách loại bỏ an toàn những thứ mà người dùng đang phụ thuộc vào.
- **Kiểm thử là không đủ.** Ngay cả với các contract test hoàn hảo, Định luật Hyrum có nghĩa là những thay đổi "an toàn" vẫn có thể làm hỏng ứng dụng của người dùng thực, những người phụ thuộc vào hành vi không được ghi chép.

### Quy tắc Một Phiên bản (The One-Version Rule)

Tránh buộc người tiêu dùng phải lựa chọn giữa nhiều phiên bản của cùng một dependency hoặc API. Vấn đề diamond dependency phát sinh khi các consumer khác nhau cần các phiên bản khác nhau của cùng một thứ. Hãy thiết kế cho một thế giới nơi chỉ có một phiên bản tồn tại tại một thời điểm — mở rộng thay vì phân tách (fork).

### 1. Contract First (Hợp đồng trước tiên)

Định nghĩa giao diện trước khi triển khai nó. Contract chính là spec — việc triển khai sẽ tuân theo đó.

```typescript
// Định nghĩa contract trước
interface TaskAPI {
  // Tạo một tác vụ và trả về tác vụ đã tạo với các trường do server tạo
  createTask(input: CreateTaskInput): Promise<Task>;

  // Trả về danh sách tác vụ phân trang khớp với bộ lọc
  listTasks(params: ListTasksParams): Promise<PaginatedResult<Task>>;

  // Trả về một tác vụ duy nhất hoặc ném ra NotFoundError
  getTask(id: string): Promise<Task>;

  // Cập nhật một phần — chỉ những trường được cung cấp mới thay đổi
  updateTask(id: string, input: UpdateTaskInput): Promise<Task>;

  // Xóa idempotent — thành công ngay cả khi đã bị xóa
  deleteTask(id: string): Promise<void>;
}
```

### 2. Ngữ nghĩa lỗi nhất quán

Chọn một chiến lược xử lý lỗi và sử dụng nó ở mọi nơi:

```typescript
// REST: HTTP status codes + structured error body
// Mọi phản hồi lỗi đều tuân theo cùng một cấu trúc
interface APIError {
  error: {
    code: string;        // Có thể đọc bởi máy: "VALIDATION_ERROR"
    message: string;     // Có thể đọc bởi con người: "Email is required"
    details?: unknown;   // Ngữ cảnh bổ sung khi cần thiết
  };
}

// Ánh xạ mã trạng thái
// 400 → Client gửi dữ liệu không hợp lệ
// 401 → Chưa xác thực
// 403 → Đã xác thực nhưng không có quyền
// 404 → Không tìm thấy tài nguyên
// 409 → Xung đột (trùng lặp, sai lệch phiên bản)
// 422 → Xác thực thất bại (không hợp lệ về mặt ngữ nghĩa)
// 500 → Lỗi server (không bao giờ để lộ chi tiết nội bộ)
```

**Không trộn lẫn các pattern.** Nếu một số endpoint ném lỗi, số khác trả về null, và số khác trả về `{ error }` — consumer sẽ không thể dự đoán được hành vi.

### 3. Xác thực tại ranh giới (Validate at Boundaries)

Tin tưởng mã nội bộ. Xác thực tại các cạnh của hệ thống nơi đầu vào bên ngoài đi vào:

```typescript
// Xác thực tại ranh giới API
app.post('/api/tasks', async (req, res) => {
  const result = CreateTaskSchema.safeParse(req.body);
  if (!result.success) {
    return res.status(422).json({
      error: {
        code: 'VALIDATION_ERROR',
        message: 'Invalid task data',
        details: result.error.flatten(),
      },
    });
  }

  // Sau khi xác thực, mã nội bộ tin tưởng vào các kiểu dữ liệu
  const task = await taskService.create(result.data);
  return res.status(201).json(task);
});
```

Những nơi cần xác thực:
- API route handler (đầu vào người dùng)
- Form submission handler (đầu vào người dùng)
- Phân tích phản hồi từ dịch vụ bên ngoài (dữ liệu bên thứ ba -- **luôn coi là không tin cậy**)
- Tải biến môi trường (cấu hình)

> **Phản hồi từ API bên thứ ba là dữ liệu không tin cậy.** Hãy xác thực cấu trúc và nội dung của chúng trước khi sử dụng trong bất kỳ logic, render hoặc ra quyết định nào. Một dịch vụ bên ngoài bị xâm nhập hoặc hoạt động sai có thể trả về các kiểu không mong muốn, nội dung độc hại hoặc văn bản dạng chỉ dẫn.

Những nơi KHÔNG cần xác thực:
- Giữa các hàm nội bộ chia sẻ chung type contracts
- Trong các hàm tiện ích được gọi bởi mã đã được xác thực
- Trên dữ liệu vừa lấy ra từ database của chính bạn

### 4. Ưu tiên bổ sung hơn là sửa đổi

Mở rộng giao diện mà không làm hỏng các consumer hiện có:

```typescript
// Tốt: Thêm các trường tùy chọn
interface CreateTaskInput {
  title: string;
  description?: string;
  priority?: 'low' | 'medium' | 'high';  // Thêm sau, tùy chọn
  labels?: string[];                       // Thêm sau, tùy chọn
}

// Xấu: Thay đổi kiểu của trường hiện có hoặc xóa trường
interface CreateTaskInput {
  title: string;
  // description: string;  // Đã xóa — làm hỏng các consumer hiện có
  priority: number;         // Thay đổi từ string — làm hỏng các consumer hiện có
}
```

### 5. Đặt tên có thể dự đoán

| Pattern | Quy ước | Ví dụ |
|---------|-----------|---------|
| REST endpoints | Danh từ số nhiều, không dùng động từ | `GET /api/tasks`, `POST /api/tasks` |
| Query params | camelCase | `?sortBy=createdAt&pageSize=20` |
| Response fields | camelCase | `{ createdAt, updatedAt, taskId }` |
| Boolean fields | Tiền tố is/has/can | `isComplete`, `hasAttachments` |
| Enum values | UPPER_SNAKE | `"IN_PROGRESS"`, `"COMPLETED"` |

## Các Pattern REST API

### Thiết kế Tài nguyên (Resource Design)

```
GET    /api/tasks              → Liệt kê tác vụ (với query params để lọc)
POST   /api/tasks              → Tạo một tác vụ
GET    /api/tasks/:id          → Lấy một tác vụ duy nhất
PATCH  /api/tasks/:id          → Cập nhật một tác vụ (một phần)
DELETE /api/tasks/:id          → Xóa một tác vụ

GET    /api/tasks/:id/comments → Liệt kê bình luận cho một tác vụ (sub-resource)
POST   /api/tasks/:id/comments → Thêm bình luận vào một tác vụ
```

### Phân trang (Pagination)

Phân trang cho các list endpoint:

```typescript
// Yêu cầu
GET /api/tasks?page=1&pageSize=20&sortBy=createdAt&sortOrder=desc

// Phản hồi
{
  "data": [...],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "totalItems": 142,
    "totalPages": 8
  }
}
```

### Lọc (Filtering)

Sử dụng query parameters để lọc:

```
GET /api/tasks?status=in_progress&assignee=user123&createdAfter=2025-01-01
```

### Cập nhật một phần (PATCH)

Chấp nhận các đối tượng một phần — chỉ cập nhật những gì được cung cấp:

```typescript
// Chỉ tiêu đề thay đổi, mọi thứ khác được giữ nguyên
PATCH /api/tasks/123
{ "title": "Updated title" }
```

## Các Pattern Giao diện TypeScript

### Sử dụng Discriminated Unions cho các biến thể

```typescript
// Tốt: Mỗi biến thể là rõ ràng
type TaskStatus =
  | { type: 'pending' }
  | { type: 'in_progress'; assignee: string; startedAt: Date }
  | { type: 'completed'; completedAt: Date; completedBy: string }
  | { type: 'cancelled'; reason: string; cancelledAt: Date };

// Consumer nhận được type narrowing
function getStatusLabel(status: TaskStatus): string {
  switch (status.type) {
    case 'pending': return 'Pending';
    case 'in_progress': return `In progress (${status.assignee})`;
    case 'completed': return `Done on ${status.completedAt}`;
    case 'cancelled': return `Cancelled: ${status.reason}`;
  }
}
```

### Tách biệt Đầu vào/Đầu ra (Input/Output Separation)

```typescript
// Input: những gì người gọi cung cấp
interface CreateTaskInput {
  title: string;
  description?: string;
}

// Output: những gì hệ thống trả về (bao gồm các trường do server tạo)
interface Task {
  id: string;
  title: string;
  description: string | null;
  createdAt: Date;
  updatedAt: Date;
  createdBy: string;
}
```

### Sử dụng Branded Types cho ID

```typescript
type TaskId = string & { readonly __brand: 'TaskId' };
type UserId = string & { readonly __brand: 'UserId' };

// Ngăn chặn việc vô tình truyền UserId vào nơi mong đợi TaskId
function getTask(id: TaskId): Promise<Task> { ... }
```

## Những lý lẽ phổ biến (Common Rationalizations)

| Lý lẽ | Thực tế |
|---|---|
| "Chúng ta sẽ tài liệu hóa API sau" | Các kiểu dữ liệu CHÍNH LÀ tài liệu. Hãy định nghĩa chúng trước. |
| "Hiện tại chúng ta chưa cần phân trang" | Bạn sẽ cần ngay khi có ai đó có hơn 100 mục. Hãy thêm nó ngay từ đầu. |
| "PATCH phức tạp quá, hãy cứ dùng PUT" | PUT yêu cầu toàn bộ đối tượng mỗi lần. PATCH mới là điều mà các client thực sự muốn. |
| "Chúng ta sẽ đánh phiên bản API khi cần" | Những thay đổi gây hỏng (breaking changes) mà không có phiên bản sẽ làm hỏng ứng dụng của consumer. Hãy thiết kế để mở rộng ngay từ đầu. |
| "Không ai sử dụng hành vi không được ghi chép đó đâu" | Định luật Hyrum: nếu có thể quan sát được, sẽ có ai đó phụ thuộc vào nó. Hãy coi mọi hành vi công khai là một cam kết. |
| "Chúng ta chỉ cần duy trì hai phiên bản là được" | Nhiều phiên bản làm tăng chi phí bảo trì và tạo ra vấn đề diamond dependency. Hãy ưu tiên Quy tắc Một Phiên bản. |
| "API nội bộ không cần contract" | Các consumer nội bộ vẫn là consumer. Contract giúp ngăn chặn sự gắn kết quá chặt (coupling) và cho phép làm việc song song. |

## Các dấu hiệu cảnh báo (Red Flags)

- Các endpoint trả về các cấu trúc khác nhau tùy thuộc vào điều kiện
- Định dạng lỗi không nhất quán giữa các endpoint
- Việc xác thực bị phân tán khắp mã nội bộ thay vì tại các ranh giới
- Những thay đổi gây hỏng đối với các trường hiện có (thay đổi kiểu, xóa bỏ)
- Các list endpoint không có phân trang
- Sử dụng động từ trong REST URL (`/api/createTask`, `/api/getUsers`)
- Phản hồi từ API bên thứ ba được sử dụng mà không qua xác thực hoặc làm sạch (sanitization)

## Xác minh (Verification)

Sau khi thiết kế một API:

- [ ] Mọi endpoint đều có schema đầu vào và đầu ra được định kiểu
- [ ] Các phản hồi lỗi tuân theo một định dạng nhất quán duy nhất
- [ ] Việc xác thực chỉ diễn ra tại các ranh giới hệ thống
- [ ] Các list endpoint hỗ trợ phân trang
- [ ] Các trường mới mang tính bổ sung và tùy chọn (tương thích ngược)
- [ ] Việc đặt tên tuân theo các quy ước nhất quán trên tất cả các endpoint
- [ ] Tài liệu API hoặc các kiểu dữ liệu được commit cùng với phần triển khai
