# Tài liệu tham khảo các mẫu Kiểm thử (Testing Patterns)

Tài liệu tham khảo nhanh về các mẫu kiểm thử phổ biến trong toàn bộ stack. Sử dụng cùng với skill `test-driven-development`.

## Mục lục

- [Cấu trúc Kiểm thử (Arrange-Act-Assert)](#test-structure-arrange-act-assert)
- [Quy ước đặt tên Kiểm thử](#test-naming-conventions)
- [Các khẳng định (Assertions) phổ biến](#common-assertions)
- [Các mẫu Mocking](#mocking-patterns)
- [Kiểm thử React/Component](#reactcomponent-testing)
- [Kiểm thử API / Integration](#api--integration-testing)
- [Kiểm thử E2E (Playwright)](#e2e-testing-playwright)
- [Các mẫu kiểm thử sai lầm (Anti-Patterns)](#test-anti-patterns)

## Cấu trúc Kiểm thử (Arrange-Act-Assert)

```typescript
it('describes expected behavior', () => {
  // Arrange: Thiết lập dữ liệu kiểm thử và các điều kiện tiên quyết
  const input = { title: 'Test Task', priority: 'high' };

  // Act: Thực hiện hành động đang được kiểm thử
  const result = createTask(input);

  // Assert: Xác minh kết quả
  expect(result.title).toBe('Test Task');
  expect(result.priority).toBe('high');
  expect(result.status).toBe('pending');
});
```

## Quy ước đặt tên Kiểm thử

```typescript
// Mẫu: [đơn vị] [hành vi mong đợi] [điều kiện]
describe('TaskService.createTask', () => {
  it('creates a task with default pending status', () => {});
  it('throws ValidationError when title is empty', () => {});
  it('trims whitespace from title', () => {});
  it('generates a unique ID for each task', () => {});
});
```

## Các khẳng định (Assertions) phổ biến

```typescript
// So sánh bằng
expect(result).toBe(expected);           // So sánh bằng nghiêm ngặt (===)
expect(result).toEqual(expected);        // So sánh bằng sâu (đối với object/array)
expect(result).toStrictEqual(expected);  // So sánh bằng sâu + khớp kiểu dữ liệu

// Giá trị đúng/sai (Truthiness)
expect(result).toBeTruthy();
expect(result).toBeFalsy();
expect(result).toBeNull();
expect(result).toBeDefined();
expect(result).toBeUndefined();

// Số
expect(result).toBeGreaterThan(5);
expect(result).toBeLessThanOrEqual(10);
expect(result).toBeCloseTo(0.3, 5);      // Số phẩy động

// Chuỗi
expect(result).toMatch(/pattern/);
expect(result).toContain('substring');

// Mảng / Đối tượng
expect(array).toContain(item);
expect(array).toHaveLength(3);
expect(object).toHaveProperty('key', 'value');

// Lỗi
expect(() => fn()).toThrow();
expect(() => fn()).toThrow(ValidationError);
expect(() => fn()).toThrow('specific message');

// Bất đồng bộ (Async)
await expect(asyncFn()).resolves.toBe(value);
await expect(asyncFn()).rejects.toThrow(Error);
```

## Các mẫu Mocking

### Mock Functions

```typescript
const mockFn = jest.fn();
mockFn.mockReturnValue(42);
mockFn.mockResolvedValue({ data: 'test' });
mockFn.mockImplementation((x) => x * 2);

expect(mockFn).toHaveBeenCalled();
expect(mockFn).toHaveBeenCalledWith('arg1', 'arg2');
expect(mockFn).toHaveBeenCalledTimes(3);
```

### Mock Modules

```typescript
// Mock toàn bộ một module
jest.mock('./database', () => ({
  query: jest.fn().mockResolvedValue([{ id: 1, title: 'Test' }]),
}));

// Mock các export cụ thể
jest.mock('./utils', () => ({
  ...jest.requireActual('./utils'),
  generateId: jest.fn().mockReturnValue('test-id'),
}));
```

### Chỉ Mock tại các ranh giới (Boundaries)

```
Mock những thứ này:                    Đừng mock những thứ này:
├── Các lời gọi Database             ├── Các hàm tiện ích nội bộ
├── Các request HTTP              ├── Logic nghiệp vụ
├── Các thao tác với hệ thống file     ├── Biến đổi dữ liệu
├── Các lời gọi API bên ngoài         ├── Các hàm kiểm tra tính hợp lệ (validation)
└── Thời gian/Ngày tháng (khi cần)    └── Các hàm thuần khiết (pure functions)
```

## Kiểm thử React/Component

```tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';

describe('TaskForm', () => {
  it('submits the form with entered data', async () => {
    const onSubmit = jest.fn();
    render(<TaskForm onSubmit={onSubmit} />);

    // Tìm các element theo role/label truy cập được (không dùng test ID)
    await screen.findByRole('textbox', { name: /title/i });
    fireEvent.change(screen.getByRole('textbox', { name: /title/i }), {
      target: { value: 'New Task' },
    });
    fireEvent.click(screen.getByRole('button', { name: /create/i }));

    await waitFor(() => {
      expect(onSubmit).toHaveBeenCalledWith({ title: 'New Task' });
    });
  });

  it('shows validation error for empty title', async () => {
    render(<TaskForm onSubmit={jest.fn()} />);

    fireEvent.click(screen.getByRole('button', { name: /create/i }));

    expect(await screen.findByText(/title is required/i)).toBeInTheDocument();
  });
});
```

## Kiểm thử API / Integration

```typescript
import request from 'supertest';
import { app } from '../src/app';

describe('POST /api/tasks', () => {
  it('creates a task and returns 201', async () => {
    const response = await request(app)
      .post('/api/tasks')
      .send({ title: 'Test Task' })
      .set('Authorization', `Bearer ${testToken}`)
      .expect(201);

    expect(response.body).toMatchObject({
      id: expect.any(String),
      title: 'Test Task',
      status: 'pending',
    });
  });

  it('returns 422 for invalid input', async () => {
    const response = await request(app)
      .post('/api/tasks')
      .send({ title: '' })
      .set('Authorization', `Bearer ${testToken}`)
      .expect(422);

    expect(response.body.error.code).toBe('VALIDATION_ERROR');
  });

  it('returns 401 without authentication', async () => {
    await request(app)
      .post('/api/tasks')
      .send({ title: 'Test' })
      .expect(401);
  });
});
```

## Kiểm thử E2E (Playwright)

```typescript
import { test, expect } from '@playwright/test';

test('user can create and complete a task', async ({ page }) => {
  // Điều hướng và xác thực
  await page.goto('/');
  await page.fill('[name="email"]', 'test@example.com');
  await page.fill('[name="password"]', 'testpass123');
  await page.click('button:has-text("Log in")');

  // Tạo một task
  await page.click('button:has-text("New Task")');
  await page.fill('[name="title"]', 'Buy groceries');
  await page.click('button:has-text("Create")');

  // Xác minh task xuất hiện
  await expect(page.locator('text=Buy groceries')).toBeVisible();

  // Hoàn thành task
  await page.click('[aria-label="Complete Buy groceries"]');
  await expect(page.locator('text=Buy groceries')).toHaveCSS(
    'text-decoration-line', 'line-through'
  );
});
```

## Các mẫu kiểm thử sai lầm (Anti-Patterns)

| Anti-Pattern | Vấn đề | Cách tiếp cận tốt hơn |
|---|---|---|
| Kiểm thử chi tiết triển khai | Bị hỏng khi refactor | Kiểm thử đầu vào/đầu ra |
| Snapshot mọi thứ | Không ai xem xét sự khác biệt của snapshot | Khẳng định các giá trị cụ thể |
| Trạng thái thay đổi dùng chung | Các bài kiểm thử làm nhiễu lẫn nhau | Setup/teardown cho mỗi bài kiểm thử |
| Kiểm thử mã nguồn bên thứ ba | Lãng phí thời gian, không phải lỗi của bạn | Mock ranh giới |
| Bỏ qua kiểm thử để vượt qua CI | Che giấu các lỗi thực tế | Sửa hoặc xóa bài kiểm thử |
| Sử dụng `test.skip` vĩnh viễn | Mã chết (dead code) | Xóa hoặc sửa nó |
| Khẳng định quá rộng | Không bắt được các lỗi hồi quy (regressions) | Hãy cụ thể |
| Không xử lý lỗi bất đồng bộ | Lỗi bị nuốt, báo thành công giả | Luôn `await` các bài kiểm thử bất đồng bộ |
