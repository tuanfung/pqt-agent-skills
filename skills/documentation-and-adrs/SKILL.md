---
name: documentation-and-adrs
description: Ghi lại các quyết định và tài liệu. Sử dụng khi đưa ra các quyết định về kiến trúc, thay đổi public API, triển khai tính năng, hoặc khi bạn cần ghi lại ngữ cảnh mà các kỹ sư và agent trong tương lai cần để hiểu codebase.
---

# Tài liệu và ADR

## Tổng quan

Tài liệu hóa các quyết định, không chỉ là mã nguồn. Những tài liệu giá trị nhất là những tài liệu nắm bắt được lý do *tại sao* — ngữ cảnh, các ràng buộc và sự đánh đổi dẫn đến một quyết định. Mã nguồn cho thấy *cái gì* đã được xây dựng; tài liệu giải thích *tại sao nó được xây dựng theo cách này* và *những phương án thay thế nào đã được xem xét*. Ngữ cảnh này là thiết yếu cho con người và các agent làm việc trong codebase trong tương lai.

## Khi nào sử dụng

- Đưa ra quyết định quan trọng về kiến trúc
- Lựa chọn giữa các phương pháp tiếp cận đối lập
- Thêm hoặc thay đổi một public API
- Triển khai một tính năng làm thay đổi hành vi đối với người dùng
- Onboarding thành viên mới (hoặc agent) vào dự án
- Khi bạn thấy mình phải giải thích cùng một điều lặp đi lặp lại

**Khi KHÔNG nên sử dụng:** Đừng tài liệu hóa những mã nguồn hiển nhiên. Đừng thêm các comment diễn đạt lại những gì mã nguồn đã thể hiện. Đừng viết tài liệu cho các prototype dùng một lần.

## Bản ghi Quyết định Kiến trúc (ADRs)

ADRs ghi lại lập luận đằng sau các quyết định kỹ thuật quan trọng. Chúng là loại tài liệu có giá trị cao nhất mà bạn có thể viết.

### Khi nào cần viết ADR

- Lựa chọn framework, thư viện, hoặc dependency chính
- Thiết kế mô hình dữ liệu hoặc database schema
- Lựa chọn chiến lược authentication
- Quyết định về kiến trúc API (REST vs. GraphQL vs. tRPC)
- Lựa chọn giữa các build tool, hosting platform, hoặc infrastructure
- Bất kỳ quyết định nào mà việc thay đổi sau này sẽ rất tốn kém

### Template ADR

Lưu trữ ADRs trong `docs/decisions/` với số thứ tự tăng dần:

```markdown
# ADR-001: Sử dụng PostgreSQL cho database chính

## Trạng thái
Đã chấp nhận | Được thay thế bởi ADR-XXX | Đã lỗi thời

## Ngày
2025-01-15

## Ngữ cảnh
Chúng ta cần một database chính cho ứng dụng quản lý công việc. Các yêu cầu chính:
- Mô hình dữ liệu quan hệ (users, tasks, teams với các mối quan hệ)
- ACID transactions cho các thay đổi trạng thái task
- Hỗ trợ full-text search cho nội dung task
- Có sẵn managed hosting (cho đội ngũ nhỏ, năng lực vận hành hạn chế)

## Quyết định
Sử dụng PostgreSQL với Prisma ORM.

## Các phương án thay thế đã xem xét

### MongoDB
- Ưu điểm: Schema linh hoạt, dễ dàng bắt đầu
- Nhược điểm: Dữ liệu của chúng ta vốn có tính quan hệ; sẽ cần quản lý các mối quan hệ thủ công
- Bị loại: Dữ liệu quan hệ trong một document store dẫn đến các phép join phức tạp hoặc trùng lặp dữ liệu

### SQLite
- Ưu điểm: Không cần cấu hình, nhúng, đọc nhanh
- Nhược điểm: Hỗ trợ ghi đồng thời hạn chế, không có managed hosting cho production
- Bị loại: Không phù hợp cho ứng dụng web đa người dùng trong production

### MySQL
- Ưu điểm: Chín chắn, được hỗ trợ rộng rãi
- Nhược điểm: PostgreSQL hỗ trợ JSON, full-text search và hệ sinh thái tooling tốt hơn
- Bị loại: PostgreSQL phù hợp hơn với các yêu cầu tính năng của chúng ta

## Hệ quả
- Prisma cung cấp truy cập database type-safe và quản lý migration
- Chúng ta có thể sử dụng full-text search của PostgreSQL thay vì thêm Elasticsearch
- Đội ngũ cần kiến thức về PostgreSQL (kỹ năng tiêu chuẩn, rủi ro thấp)
- Hosting trên managed service (Supabase, Neon, hoặc RDS)
```

### Vòng đời ADR

```
ĐỀ XUẤT → ĐÃ CHẤP NHẬN → (ĐƯỢC THAY THẾ hoặc LỖI THỜI)
```

- **Đừng xóa các ADR cũ.** Chúng lưu giữ ngữ cảnh lịch sử.
- Khi một quyết định thay đổi, hãy viết một ADR mới tham chiếu và thay thế ADR cũ.

## Tài liệu Inline

### Khi nào nên Comment

Comment lý do *tại sao*, không phải cái *gì*:

```typescript
// TỆ: Diễn đạt lại mã nguồn
// Tăng counter thêm 1
counter += 1;

// TỐT: Giải thích ý định không hiển nhiên
// Rate limit sử dụng một sliding window — reset counter tại ranh giới window,
// không theo lịch cố định, để ngăn chặn các cuộc tấn công burst tại các cạnh của window
if (now - windowStart > WINDOW_SIZE_MS) {
  counter = 0;
  windowStart = now;
}
```

### Khi KHÔNG nên Comment

```typescript
// Đừng comment mã nguồn tự giải thích
function calculateTotal(items: CartItem[]): number {
  return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
}

// Đừng để lại comment TODO cho những việc bạn nên làm ngay bây giờ
// TODO: thêm xử lý lỗi  ← Cứ thêm vào luôn

// Đừng để lại mã nguồn bị comment
// const oldImplementation = () => { ... }  ← Xóa nó đi, git đã có history
```

### Tài liệu hóa các Gotcha đã biết

```typescript
/**
 * QUAN TRỌNG: Hàm này phải được gọi trước lần render đầu tiên.
 * Nếu gọi sau hydration, nó sẽ gây ra hiện tượng flash of unstyled content
 * vì theme context không có sẵn trong quá trình SSR.
 *
 * Xem ADR-003 để biết chi tiết lý do thiết kế.
 */
export function initializeTheme(theme: Theme): void {
  // ...
}
```

## Tài liệu API

Đối với các public API (REST, GraphQL, interface thư viện):

### Inline với Type (Ưu tiên cho TypeScript)

```typescript
/**
 * Tạo một task mới.
 *
 * @param input - Dữ liệu tạo task (yêu cầu title, description tùy chọn)
 * @returns Task đã tạo với ID và timestamp do server tạo
 * @throws {ValidationError} Nếu title trống hoặc vượt quá 200 ký tự
 * @throws {AuthenticationError} Nếu người dùng chưa được xác thực
 *
 * @example
 * const task = await createTask({ title: 'Buy groceries' });
 * console.log(task.id); // "task_abc123"
 */
export async function createTask(input: CreateTaskInput): Promise<Task> {
  // ...
}
```

### OpenAPI / Swagger cho REST API

```yaml
paths:
  /api/tasks:
    post:
      summary: Tạo một task
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateTaskInput'
      responses:
        '201':
          description: Task đã được tạo
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Task'
        '422':
          description: Lỗi xác thực
```

## Cấu trúc README

Mọi dự án nên có một README bao gồm:

```markdown
# Tên Dự Án

Mô tả ngắn gọn một đoạn về chức năng của dự án này.

## Bắt đầu nhanh
1. Clone repo
2. Cài đặt dependencies: `npm install`
3. Thiết lập môi trường: `cp .env.example .env`
4. Chạy dev server: `npm run dev`

## Các lệnh
| Lệnh | Mô tả |
|---------|-------------|
| `npm run dev` | Chạy server phát triển |
| `npm test` | Chạy test |
| `npm run build` | Build production |
| `npm run lint` | Chạy linter |

## Kiến trúc
Tổng quan ngắn gọn về cấu trúc dự án và các quyết định thiết kế chính.
Liên kết đến các ADR để biết chi tiết.

## Đóng góp
Cách đóng góp, tiêu chuẩn lập trình, quy trình PR.
```

## Duy trì Changelog

Đối với các tính năng đã triển khai:

```markdown
# Changelog

## [1.2.0] - 2025-01-20
### Thêm mới
- Chia sẻ task: người dùng có thể chia sẻ task với thành viên trong nhóm (#123)
- Thông báo email cho việc phân công task (#124)

### Sửa lỗi
- Các task bị trùng lặp xuất hiện khi nhấn nhanh nút create (#125)

### Thay đổi
- Danh sách task hiện tải 50 mục mỗi trang (trước là 20) để cải thiện UX (#126)
```

## Tài liệu dành cho Agent

Các lưu ý đặc biệt cho ngữ cảnh của AI agent:

- **CLAUDE.md / các file rules** — Tài liệu hóa các quy ước dự án để agent tuân thủ
- **Các file spec** — Giữ cho spec luôn cập nhật để agent xây dựng đúng thứ cần thiết
- **ADRs** — Giúp agent hiểu tại sao các quyết định trong quá khứ được đưa ra (ngăn chặn việc quyết định lại)
- **Inline gotchas** — Ngăn agent rơi vào các bẫy đã biết

## Các lý lẽ phổ biến

| Lý lẽ | Thực tế |
|---|---|
| "Mã nguồn tự giải thích" | Mã nguồn cho thấy cái gì. Nó không cho thấy tại sao, phương án thay thế nào bị loại bỏ, hoặc các ràng buộc nào được áp dụng. |
| "Chúng ta sẽ viết tài liệu khi API ổn định" | API ổn định nhanh hơn khi bạn tài liệu hóa chúng. Tài liệu là bài kiểm tra đầu tiên cho thiết kế. |
| "Không ai đọc tài liệu" | Agent đọc. Các kỹ sư tương lai đọc. Chính bạn của 3 tháng sau sẽ đọc. |
| "ADRs gây tốn công" | Một ADR viết trong 10 phút ngăn chặn một cuộc tranh luận kéo dài 2 giờ về cùng một quyết định sáu tháng sau đó. |
| "Comment sẽ bị lỗi thời" | Comment về lý do *tại sao* thì ổn định. Comment về cái *gì* sẽ bị lỗi thời — đó là lý do tại sao bạn chỉ nên viết vế trước. |

## Dấu hiệu cảnh báo

- Các quyết định kiến trúc không có lập luận bằng văn bản
- Public API không có tài liệu hoặc type
- README không giải thích cách chạy dự án
- Mã nguồn bị comment thay vì xóa bỏ
- Các comment TODO đã tồn tại trong nhiều tuần
- Không có ADR trong dự án có những lựa chọn kiến trúc quan trọng
- Tài liệu diễn đạt lại mã nguồn thay vì giải thích ý định

## Xác minh

Sau khi tài liệu hóa:

- [ ] ADR tồn tại cho tất cả các quyết định kiến trúc quan trọng
- [ ] README bao gồm bắt đầu nhanh, các lệnh và tổng quan kiến trúc
- [ ] Các hàm API có tài liệu về tham số và kiểu trả về
- [ ] Các gotcha đã biết được tài liệu hóa inline tại những nơi cần thiết
- [ ] Không còn mã nguồn bị comment
- [ ] Các file rules (CLAUDE.md v.v.) hiện đang cập nhật và chính xác
