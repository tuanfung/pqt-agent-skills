---
name: frontend-ui-engineering
description: Xây dựng các giao diện người dùng (UI) chất lượng sản phẩm thực tế. Sử dụng khi xây dựng hoặc chỉnh sửa các giao diện hướng người dùng. Sử dụng khi tạo các component, triển khai layout, quản lý state, hoặc khi kết quả đầu ra cần có giao diện và cảm giác đạt chất lượng sản phẩm thực tế thay vì trông như do AI tạo ra.
---

# Frontend UI Engineering

## Tổng quan

Xây dựng các giao diện người dùng chất lượng sản phẩm thực tế, đảm bảo khả năng tiếp cận (accessible), hiệu suất cao (performant) và trau chuốt về mặt hình ảnh. Mục tiêu là tạo ra UI trông như được xây dựng bởi một kỹ sư am hiểu về thiết kế tại một công ty hàng đầu — không phải như được tạo ra bởi AI. Điều này có nghĩa là tuân thủ thực sự hệ thống thiết kế (design system), đảm bảo khả năng tiếp cận đúng chuẩn, các mô hình tương tác (interaction patterns) được tính toán kỹ lưỡng và không có "thẩm mỹ kiểu AI" chung chung.

## Khi nào nên sử dụng

- Xây dựng các component UI hoặc trang mới
- Chỉnh sửa các giao diện hướng người dùng hiện có
- Triển khai các layout đáp ứng (responsive layouts)
- Thêm tính tương tác hoặc quản lý state
- Sửa các lỗi về hình ảnh hoặc UX

## Kiến trúc Component

### Cấu trúc file

Đặt mọi thứ liên quan đến một component ở cùng một nơi (colocate):

```
src/components/
  TaskList/
    TaskList.tsx          # Triển khai component
    TaskList.test.tsx     # Tests
    TaskList.stories.tsx  # Storybook stories (nếu sử dụng)
    use-task-list.ts      # Custom hook (nếu state phức tạp)
    types.ts              # Các kiểu dữ liệu đặc thù cho component (nếu cần)
```

### Mô hình Component (Component Patterns)

**Ưu tiên kết hợp (composition) hơn là cấu hình (configuration):**

```tsx
// Tốt: Có thể kết hợp (Composable)
<Card>
  <CardHeader>
    <CardTitle>Tasks</CardTitle>
  </CardHeader>
  <CardBody>
    <TaskList tasks={tasks} />
  </CardBody>
</Card>

// Nên tránh: Quá nhiều cấu hình (Over-configured)
<Card
  title="Tasks"
  headerVariant="large"
  bodyPadding="md"
  content={<TaskList tasks={tasks} />}
/>
```

**Giữ cho các component tập trung:**

```tsx
// Tốt: Chỉ làm một việc
export function TaskItem({ task, onToggle, onDelete }: TaskItemProps) {
  return (
    <li className="flex items-center gap-3 p-3">
      <Checkbox checked={task.done} onChange={() => onToggle(task.id)} />
      <span className={task.done ? 'line-through text-muted' : ''}>{task.title}</span>
      <Button variant="ghost" size="sm" onClick={() => onDelete(task.id)}>
        <TrashIcon />
      </Button>
    </li>
  );
}
```

**Tách biệt việc lấy dữ liệu (data fetching) khỏi phần trình diễn (presentation):**

```tsx
// Container: xử lý dữ liệu
export function TaskListContainer() {
  const { tasks, isLoading, error } = useTasks();

  if (isLoading) return <TaskListSkeleton />;
  if (error) return <ErrorState message="Failed to load tasks" retry={refetch} />;
  if (tasks.length === 0) return <EmptyState message="No tasks yet" />;

  return <TaskList tasks={tasks} />;
}

// Presentation: xử lý hiển thị
export function TaskList({ tasks }: { tasks: Task[] }) {
  return (
    <ul role="list" className="divide-y">
      {tasks.map(task => <TaskItem key={task.id} task={task} />)}
    </ul>
  );
}
```

## Quản lý State

**Chọn phương pháp đơn giản nhất mà vẫn hiệu quả:**

```
Local state (useState)           → State UI đặc thù cho component
Lifted state                     → Chia sẻ giữa 2-3 component anh em
Context                          → Theme, auth, locale (đọc nhiều, ghi ít)
URL state (searchParams)         → Bộ lọc, phân trang, state UI có thể chia sẻ được
Server state (React Query, SWR)  → Dữ liệu từ xa có caching
Global store (Zustand, Redux)    → State client phức tạp chia sẻ toàn ứng dụng
```

**Tránh truyền prop qua nhiều cấp (prop drilling) sâu hơn 3 cấp.** Nếu bạn đang truyền props qua các component không sử dụng chúng, hãy cân nhắc sử dụng context hoặc tái cấu trúc cây component.

## Tuân thủ Hệ thống Thiết kế (Design System)

### Tránh "Thẩm mỹ kiểu AI" (AI Aesthetic)

UI do AI tạo ra thường có những mô hình dễ nhận biết. Hãy tránh tất cả những điều sau:

| Mặc định của AI | Tại sao là vấn đề | Chất lượng Sản phẩm Thực tế |
|---|---|---|
| Mọi thứ đều màu tím/indigo | Các model thường chọn bảng màu "an toàn", khiến mọi ứng dụng trông giống hệt nhau | Sử dụng bảng màu thực tế của dự án |
| Lạm dụng gradient | Gradient tạo ra nhiễu thị giác và xung đột với hầu hết các design system | Sử dụng màu phẳng hoặc gradient tinh tế phù hợp với design system |
| Bo góc tối đa (rounded-2xl) | Bo góc quá mức tạo cảm giác "thân thiện" nhưng bỏ qua phân cấp bán kính góc trong các thiết kế thực tế | Sử dụng border-radius nhất quán từ design system |
| Các section hero chung chung | Layout theo mẫu, không liên kết với nội dung thực tế hoặc nhu cầu người dùng | Layout ưu tiên nội dung (content-first) |
| Nội dung kiểu Lorem ipsum | Văn bản giả che giấu các vấn đề layout mà nội dung thực tế sẽ làm lộ ra (độ dài, ngắt dòng, tràn nội dung) | Nội dung giả lập thực tế |
| Padding quá lớn ở mọi nơi | Padding hào phóng một cách đồng đều làm mất phân cấp thị giác và lãng phí không gian màn hình | Thang đo khoảng cách (spacing scale) nhất quán |
| Lưới thẻ (card grids) rập khuôn | Lưới đồng nhất là lối tắt layout, bỏ qua thứ tự ưu tiên thông tin và mô hình quét mắt | Layout phục vụ mục đích cụ thể |
| Thiết kế lạm dụng đổ bóng (shadow) | Đổ bóng nhiều lớp tạo ra độ sâu gây nhiễu nội dung và làm chậm tốc độ render trên thiết bị cấu hình thấp | Đổ bóng tinh tế hoặc không dùng trừ khi design system yêu cầu |

### Khoảng cách và Layout

Sử dụng thang đo khoảng cách nhất quán. Không tự ý chế ra các giá trị:

```css
/* Sử dụng thang đo: bước nhảy 0.25rem (hoặc bất cứ điều gì dự án sử dụng) */
/* Tốt */  padding: 1rem;      /* 16px */
/* Tốt */  gap: 0.75rem;       /* 12px */
/* Xấu */   padding: 13px;      /* Không nằm trong thang đo */
/* Xấu */   margin-top: 2.3rem; /* Không nằm trong thang đo */
```

### Hệ thống Kiểu chữ (Typography)

Tôn trọng phân cấp kiểu chữ:

```
h1 → Tiêu đề trang (một tiêu đề mỗi trang)
h2 → Tiêu đề section
h3 → Tiêu đề subsection
body → Văn bản mặc định
small → Văn bản phụ/hỗ trợ
```

Không bỏ qua các cấp độ tiêu đề. Không sử dụng style tiêu đề cho các nội dung không phải là tiêu đề.

### Màu sắc

- Sử dụng các token màu ngữ nghĩa: `text-primary`, `bg-surface`, `border-default` — không dùng giá trị hex thô.
- Đảm bảo độ tương phản đủ (4.5:1 cho văn bản thường, 3:1 cho văn bản lớn).
- Không chỉ dựa vào màu sắc để truyền tải thông tin (hãy sử dụng thêm biểu tượng, văn bản hoặc mô hình).

## Khả năng tiếp cận (WCAG 2.1 AA)

Mọi component phải đáp ứng các tiêu chuẩn sau:

### Điều hướng bằng bàn phím

```tsx
// Mọi thành phần tương tác phải có thể tiếp cận bằng bàn phím
<button onClick={handleClick}>Click me</button>        // ✓ Có thể focus mặc định
<div onClick={handleClick}>Click me</div>               // ✗ Không thể focus
<div role="button" tabIndex={0} onClick={handleClick}    // ✓ Nhưng nên ưu tiên dùng <button>
     onKeyDown={e => {
       if (e.key === 'Enter') handleClick();
       if (e.key === ' ') e.preventDefault();
     }}
     onKeyUp={e => {
       if (e.key === ' ') handleClick();
     }}>
  Click me
</div>
```

### Nhãn ARIA (ARIA Labels)

```tsx
// Gán nhãn cho các thành phần tương tác thiếu văn bản hiển thị
<button aria-label="Close dialog"><XIcon /></button>

// Gán nhãn cho các input của form
<label htmlFor="email">Email</label>
<input id="email" type="email" />

// Hoặc sử dụng aria-label khi không có nhãn hiển thị
<input aria-label="Search tasks" type="search" />
```

### Quản lý Tiêu điểm (Focus Management)

```tsx
// Di chuyển focus khi nội dung thay đổi
function Dialog({ isOpen, onClose }: DialogProps) {
  const closeRef = useRef<HTMLButtonElement>(null);

  useEffect(() => {
    if (isOpen) closeRef.current?.focus();
  }, [isOpen]);

  // Giữ focus bên trong dialog khi mở
  return (
    <dialog open={isOpen}>
      <button ref={closeRef} onClick={onClose}>Close</button>
      {/* nội dung dialog */}
    </dialog>
  );
}
```

### Trạng thái Trống và Lỗi có ý nghĩa

```tsx
// Không hiển thị màn hình trống
function TaskList({ tasks }: { tasks: Task[] }) {
  if (tasks.length === 0) {
    return (
      <div role="status" className="text-center py-12">
        <TasksEmptyIcon className="mx-auto h-12 w-12 text-muted" />
        <h3 className="mt-2 text-sm font-medium">No tasks</h3>
        <p className="mt-1 text-sm text-muted">Get started by creating a new task.</p>
        <Button className="mt-4" onClick={onCreateTask}>Create Task</Button>
      </div>
    );
  }

  return <ul role="list">...</ul>;
}
```

## Thiết kế Đáp ứng (Responsive Design)

Thiết kế cho thiết bị di động trước, sau đó mở rộng dần:

```tsx
// Tailwind: responsive mobile-first
<div className="
  grid grid-cols-1      /* Mobile: một cột */
  sm:grid-cols-2        /* Small: 2 cột */
  lg:grid-cols-3        /* Large: 3 cột */
  gap-4
">
```

Kiểm tra tại các điểm breakpoint: 320px, 768px, 1024px, 1440px.

## Loading và Chuyển cảnh (Transitions)

```tsx
// Skeleton loading (không dùng spinner cho nội dung)
function TaskListSkeleton() {
  return (
    <div className="space-y-3" aria-busy="true" aria-label="Loading tasks">
      {Array.from({ length: 3 }).map((_, i) => (
        <div key={i} className="h-12 bg-muted animate-pulse rounded" />
      ))}
    </div>
  );
}

// Cập nhật lạc quan (optimistic updates) để tăng cảm giác tốc độ
function useToggleTask() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: toggleTask,
    onMutate: async (taskId) => {
      await queryClient.cancelQueries({ queryKey: ['tasks'] });
      const previous = queryClient.getQueryData(['tasks']);

      queryClient.setQueryData(['tasks'], (old: Task[]) =>
        old.map(t => t.id === taskId ? { ...t, done: !t.done } : t)
      );

      return { previous };
    },
    onError: (_err, _taskId, context) => {
      queryClient.setQueryData(['tasks'], context?.previous);
    },
  });
}
```

## Xem thêm

Để biết yêu cầu chi tiết về khả năng tiếp cận và các công cụ kiểm tra, xem tại `references/accessibility-checklist.md`.

## Những lý do biện minh phổ biến (Common Rationalizations)

| Lý do biện minh | Thực tế |
|---|---|
| "Khả năng tiếp cận (Accessibility) chỉ là có thì tốt" | Đó là yêu cầu pháp lý ở nhiều khu vực và là tiêu chuẩn chất lượng kỹ thuật. |
| "Chúng ta sẽ làm responsive sau" | Việc sửa lại responsive sau này khó gấp 3 lần so với việc xây dựng ngay từ đầu. |
| "Thiết kế chưa chốt nên tôi bỏ qua styling" | Hãy sử dụng các mặc định của design system. UI không có style tạo ấn tượng xấu ban đầu cho người review. |
| "Đây chỉ là bản prototype" | Các bản prototype thường trở thành mã nguồn sản phẩm. Hãy xây dựng nền móng đúng ngay từ đầu. |
| "Thẩm mỹ kiểu AI hiện tại là ổn rồi" | Nó báo hiệu chất lượng thấp. Hãy sử dụng đúng design system của dự án ngay từ đầu. |

## Dấu hiệu cảnh báo (Red Flags)

- Component dài hơn 200 dòng (hãy chia nhỏ chúng)
- Sử dụng style inline hoặc các giá trị pixel tùy tiện
- Thiếu các trạng thái lỗi (error states), trạng thái tải (loading states), hoặc trạng thái trống (empty states)
- Không kiểm tra điều hướng bằng bàn phím
- Chỉ dùng màu sắc làm dấu hiệu duy nhất cho trạng thái (đỏ/xanh mà không có văn bản hoặc biểu tượng)
- Giao diện "kiểu AI" chung chung (gradient màu tím, thẻ quá khổ, layout mẫu)

## Xác minh (Verification)

Sau khi xây dựng UI:

- [ ] Component render mà không có lỗi console
- [ ] Mọi thành phần tương tác đều có thể tiếp cận bằng bàn phím (Tab qua trang)
- [ ] Trình đọc màn hình (screen reader) có thể truyền tải nội dung và cấu trúc trang
- [ ] Responsive: hoạt động tốt tại 320px, 768px, 1024px, 1440px
- [ ] Tất cả trạng thái loading, error, và empty đều được xử lý
- [ ] Tuân thủ design system của dự án (khoảng cách, màu sắc, kiểu chữ)
- [ ] Không có cảnh báo accessibility trong dev tools hoặc axe-core
