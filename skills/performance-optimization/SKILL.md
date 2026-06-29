---
name: performance-optimization
description: Tối ưu hóa hiệu suất ứng dụng. Sử dụng khi có các yêu cầu về hiệu suất, khi bạn nghi ngờ có sự sụt giảm hiệu suất (performance regressions), hoặc khi Core Web Vitals hoặc thời gian tải trang cần được cải thiện. Sử dụng khi profiling cho thấy các điểm nghẽn (bottlenecks) cần được khắc phục.
---

# Tối ưu hóa Hiệu suất (Performance Optimization)

## Tổng quan

Hãy đo lường trước khi tối ưu hóa. Việc tối ưu hóa hiệu suất mà không đo lường chỉ là phỏng đoán — và phỏng đoán dẫn đến tối ưu hóa sớm (premature optimization), làm tăng độ phức tạp mà không cải thiện được những điều quan trọng. Hãy profile trước, xác định điểm nghẽn thực tế, khắc phục nó, và đo lường lại. Chỉ tối ưu hóa những gì kết quả đo lường chứng minh là quan trọng.

## Khi nào nên sử dụng

- Có các yêu cầu về hiệu suất trong bản đặc tả (ngân sách thời gian tải trang, SLA về thời gian phản hồi)
- Người dùng hoặc hệ thống giám sát báo cáo hành vi chậm chạp
- Điểm Core Web Vitals dưới ngưỡng cho phép
- Bạn nghi ngờ một thay đổi đã gây ra sự sụt giảm hiệu suất (regression)
- Xây dựng các tính năng xử lý tập dữ liệu lớn hoặc lưu lượng truy cập cao

**Khi KHÔNG nên sử dụng:** Đừng tối ưu hóa trước khi bạn có bằng chứng về sự cố. Tối ưu hóa sớm làm tăng độ phức tạp, gây tốn kém hơn cả lợi ích về hiệu suất mà nó mang lại.

## Mục tiêu Core Web Vitals

| Chỉ số | Tốt | Cần cải thiện | Kém |
|--------|------|-------------------|------|
| **LCP** (Largest Contentful Paint) | ≤ 2.5s | ≤ 4.0s | > 4.0s |
| **INP** (Interaction to Next Paint) | ≤ 200ms | ≤ 500ms | > 500ms |
| **CLS** (Cumulative Layout Shift) | ≤ 0.1 | ≤ 0.25 | > 0.25 |

## Quy trình Tối ưu hóa

```
1. ĐO LƯỜNG (MEASURE)  → Thiết lập mốc cơ sở (baseline) với dữ liệu thực tế
2. XÁC ĐỊNH (IDENTIFY) → Tìm điểm nghẽn thực tế (không phỏng đoán)
3. KHẮC PHỤC (FIX)      → Giải quyết điểm nghẽn cụ thể
4. XÁC MINH (VERIFY)   → Đo lường lại, xác nhận sự cải thiện
5. BẢO VỆ (GUARD)    → Thêm giám sát hoặc test để ngăn chặn sụt giảm hiệu suất (regression)
```

### Bước 1: Đo lường

Có hai phương pháp bổ trợ cho nhau — hãy sử dụng cả hai:

- **Synthetic (Lighthouse, tab Performance trong DevTools):** Điều kiện kiểm soát, có thể tái lập. Tốt nhất để phát hiện regression trong CI và cô lập các sự cố cụ thể.
- **RUM (thư viện web-vitals, CrUX):** Dữ liệu người dùng thực trong điều kiện thực tế. Cần thiết để xác nhận rằng một bản sửa lỗi thực sự cải thiện trải nghiệm người dùng.

**Frontend:**
```bash
# Synthetic: Lighthouse trong Chrome DevTools (hoặc CI)
# Chrome DevTools → Performance tab → Record
# Chrome DevTools MCP → Performance trace

# RUM: Thư viện Web Vitals trong mã nguồn
import { onLCP, onINP, onCLS } from 'web-vitals';

onLCP(console.log);
onINP(console.log);
onCLS(console.log);
```

**Backend:**
```bash
# Ghi log thời gian phản hồi
# Giám sát hiệu suất ứng dụng (APM - Application Performance Monitoring)
# Ghi log truy vấn cơ sở dữ liệu kèm theo thời gian thực hiện

# Đo thời gian đơn giản
console.time('db-query');
const result = await db.query(...);
console.timeEnd('db-query');
```

### Bắt đầu Đo lường từ đâu

Sử dụng triệu chứng để quyết định cần đo lường điều gì trước:

```
Cái gì chậm?
├── Tải trang lần đầu
│   ├── Bundle lớn? --> Đo kích thước bundle, kiểm tra code splitting
│   ├── Phản hồi server chậm? --> Đo TTFB trong Network waterfall của DevTools
│   │   ├── DNS lâu? --> Thêm dns-prefetch / preconnect cho các origin đã biết
│   │   ├── TCP/TLS lâu? --> Bật HTTP/2, kiểm tra triển khai edge, keep-alive
│   │   └── Đang chờ (server) lâu? --> Profile backend, kiểm tra query và caching
│   └── Tài nguyên chặn render (Render-blocking)? --> Kiểm tra Network waterfall cho CSS/JS chặn render
├── Tương tác cảm thấy chậm chạp
│   ├── UI bị treo khi click? --> Profile main thread, tìm các long tasks (>50ms)
│   ├── Lag khi nhập form? --> Kiểm tra re-renders, chi phí của controlled component
│   └── Animation bị giật (jank)? --> Kiểm tra layout thrashing, forced reflows
├── Trang sau khi điều hướng
│   ├── Đang tải dữ liệu? --> Đo thời gian phản hồi API, kiểm tra hiện tượng waterfall
│   └── Client rendering? --> Profile thời gian render component, kiểm tra lỗi N+1 fetches
└── Backend / API
    ├── Một endpoint đơn lẻ chậm? --> Profile truy vấn database, kiểm tra index
    ├── Tất cả endpoint đều chậm? --> Kiểm tra connection pool, bộ nhớ, CPU
    └── Chậm chạp ngắt quãng? --> Kiểm tra xung đột khóa (lock contention), tạm dừng GC, dependencies bên ngoài
```

### Bước 2: Xác định Điểm nghẽn (Bottleneck)

Các điểm nghẽn phổ biến theo danh mục:

**Frontend:**

| Triệu chứng | Nguyên nhân khả thi | Cách điều tra |
|---------|-------------|---------------|
| LCP chậm | Hình ảnh lớn, tài nguyên chặn render, server chậm | Kiểm tra network waterfall, kích thước hình ảnh |
| CLS cao | Hình ảnh không có kích thước, nội dung tải muộn, font bị thay đổi | Kiểm tra layout shift attribution |
| INP kém | JavaScript nặng trên main thread, cập nhật DOM lớn | Kiểm tra long tasks trong Performance trace |
| Tải ban đầu chậm | Bundle lớn, quá nhiều yêu cầu mạng | Kiểm tra kích thước bundle, code splitting |

**Backend:**

| Triệu chứng | Nguyên nhân khả thi | Cách điều tra |
|---------|-------------|---------------|
| API phản hồi chậm | Truy vấn N+1, thiếu index, query chưa tối ưu | Kiểm tra log truy vấn database |
| Bộ nhớ tăng trưởng | Rò rỉ tham chiếu (leaked references), cache không giới hạn, payload lớn | Phân tích Heap snapshot |
| CPU tăng đột biến | Tính toán nặng đồng bộ, regex backtracking | CPU profiling |
| Độ trễ cao | Thiếu caching, tính toán dư thừa, nhiều chặng mạng (network hops) | Trace yêu cầu qua các tầng stack |

### Bước 3: Khắc phục các Anti-Pattern phổ biến

#### Truy vấn N+1 (Backend)

```typescript
// XẤU: N+1 — một truy vấn cho mỗi task để tìm chủ sở hữu
const tasks = await db.tasks.findMany();
for (const task of tasks) {
  task.owner = await db.users.findUnique({ where: { id: task.ownerId } });
}

// TỐT: Một truy vấn duy nhất với join/include
const tasks = await db.tasks.findMany({
  include: { owner: true },
});
```

#### Lấy dữ liệu không giới hạn

```typescript
// XẤU: Lấy tất cả bản ghi
const allTasks = await db.tasks.findMany();

// TỐT: Phân trang với giới hạn (limits)
const tasks = await db.tasks.findMany({
  take: 20,
  skip: (page - 1) * 20,
  orderBy: { createdAt: 'desc' },
});
```

#### Thiếu tối ưu hóa hình ảnh (Frontend)

```html
<!-- XẤU: Không có kích thước, không tối ưu định dạng -->
<img src="/hero.jpg" />

<!-- TỐT: Hình ảnh Hero / LCP — định hướng nghệ thuật (art direction) + chuyển đổi độ phân giải, ưu tiên cao -->
<!--
  Kết hợp hai kỹ thuật:
  - Art direction (media): cắt/bố cục khác nhau cho mỗi breakpoint
  - Resolution switching (srcset + sizes): kích thước file phù hợp với mật độ màn hình
-->
<picture>
  <!-- Mobile: cắt theo chiều dọc (8:10) -->
  <source
    media="(max-width: 767px)"
    srcset="/hero-mobile-400.avif 400w, /hero-mobile-800.avif 800w"
    sizes="100vw"
    width="800"
    height="1000"
    type="image/avif"
  />
  <source
    media="(max-width: 767px)"
    srcset="/hero-mobile-400.webp 400w, /hero-mobile-800.webp 800w"
    sizes="100vw"
    width="800"
    height="1000"
    type="image/webp"
  />
  <!-- Desktop: cắt theo chiều ngang (2:1) -->
  <source
    srcset="/hero-800.avif 800w, /hero-1200.avif 1200w, /hero-1600.avif 1600w"
    sizes="(max-width: 1200px) 100vw, 1200px"
    width="1200"
    height="600"
    type="image/avif"
  />
  <source
    srcset="/hero-800.webp 800w, /hero-1200.webp 1200w, /hero-1600.webp 1600w"
    sizes="(max-width: 1200px) 100vw, 1200px"
    width="1200"
    height="600"
    type="image/webp"
  />
  <img
    src="/hero-desktop.jpg"
    width="1200"
    height="600"
    fetchpriority="high"
    alt="Mô tả hình ảnh Hero"
  />
</picture>

<!-- TỐT: Hình ảnh dưới nếp gấp (below-the-fold) — tải chậm (lazy load) + giải mã bất đồng bộ (async decoding) -->
<img
  src="/content.webp"
  width="800"
  height="400"
  loading="lazy"
  decoding="async"
  alt="Mô tả hình ảnh nội dung"
/>
```

#### Re-render không cần thiết (React)

```tsx
// XẤU: Tạo đối tượng mới trong mỗi lần render, khiến các component con bị re-render
function TaskList() {
  return <TaskFilters options={{ sortBy: 'date', order: 'desc' }} />;
}

// TỐT: Tham chiếu ổn định
const DEFAULT_OPTIONS = { sortBy: 'date', order: 'desc' } as const;
function TaskList() {
  return <TaskFilters options={DEFAULT_OPTIONS} />;
}

// Sử dụng React.memo cho các component tốn kém
const TaskItem = React.memo(function TaskItem({ task }: Props) {
  return <div>{/* render tốn kém */}</div>;
});

// Sử dụng useMemo cho các tính toán tốn kém
function TaskStats({ tasks }: Props) {
  const stats = useMemo(() => calculateStats(tasks), [tasks]);
  return <div>{stats.completed} / {stats.total}</div>;
}
```

#### Kích thước Bundle lớn

```typescript
// Các bundler hiện đại (Vite, webpack 5+) tự động xử lý named imports với tree-shaking,
// miễn là dependency cung cấp ESM và được đánh dấu `sideEffects: false` trong package.json.
// Hãy profile trước khi thay đổi kiểu import — lợi ích thực sự đến từ việc chia nhỏ (splitting) và lazy loading.

// TỐT: Dynamic import cho các tính năng nặng, ít dùng
const ChartLibrary = lazy(() => import('./ChartLibrary'));

// TỐT: Chia nhỏ mã ở cấp độ route (route-level code splitting) bọc trong Suspense
const SettingsPage = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <SettingsPage />
    </Suspense>
  );
}
```

#### Thiếu Caching (Backend)

```typescript
// Cache dữ liệu được đọc thường xuyên và ít thay đổi
const CACHE_TTL = 5 * 60 * 1000; // 5 phút
let cachedConfig: AppConfig | null = null;
let cacheExpiry = 0;

async function getAppConfig(): Promise<AppConfig> {
  if (cachedConfig && Date.now() < cacheExpiry) {
    return cachedConfig;
  }
  cachedConfig = await db.config.findFirst();
  cacheExpiry = Date.now() + CACHE_TTL;
  return cachedConfig;
}

// HTTP caching headers cho tài nguyên tĩnh
app.use('/static', express.static('public', {
  maxAge: '1y',           // Cache trong 1 năm
  immutable: true,        // Không bao giờ xác thực lại (sử dụng content hashing trong tên file)
}));

// Cache-Control cho phản hồi API
res.set('Cache-Control', 'public, max-age=300'); // 5 phút
```

## Ngân sách Hiệu suất (Performance Budget)

Thiết lập ngân sách và thực thi chúng:

```
JavaScript bundle: < 200KB gzipped (tải lần đầu)
CSS: < 50KB gzipped
Hình ảnh: < 200KB mỗi hình (phần trên nếp gấp - above the fold)
Fonts: < 100KB tổng cộng
Thời gian phản hồi API: < 200ms (p95)
Thời gian tương tác (Time to Interactive): < 3.5s trên 4G
Điểm Lighthouse Performance: ≥ 90
```

**Thực thi trong CI:**
```bash
# Kiểm tra kích thước bundle
npx bundlesize --config bundlesize.config.json

# Lighthouse CI
npx lhci autorun
```

## Xem thêm

Để biết danh sách kiểm tra hiệu suất chi tiết, các lệnh tối ưu hóa và tham chiếu anti-pattern, xem `references/performance-checklist.md`.

## Các lý lẽ bào chữa phổ biến

| Lý lẽ bào chữa | Thực tế |
|---|---|
| "Chúng ta sẽ tối ưu hóa sau" | Nợ hiệu suất sẽ tích tụ. Hãy khắc phục các anti-pattern hiển nhiên ngay bây giờ, trì hoãn các micro-optimizations. |
| "Trên máy tôi thì nhanh" | Máy của bạn không phải là máy của người dùng. Hãy profile trên phần cứng và mạng đại diện. |
| "Việc tối ưu hóa này là hiển nhiên" | Nếu bạn không đo lường, bạn không biết chắc. Hãy profile trước. |
| "Người dùng sẽ không nhận ra 100ms" | Nghiên cứu cho thấy độ trễ 100ms ảnh hưởng đến tỷ lệ chuyển đổi. Người dùng nhận ra nhiều hơn bạn nghĩ. |
| "Framework đã xử lý hiệu suất rồi" | Framework ngăn chặn một số sự cố nhưng không thể sửa lỗi truy vấn N+1 hoặc bundle quá khổ. |

## Dấu hiệu cảnh báo (Red Flags)

- Tối ưu hóa mà không có dữ liệu profiling để chứng minh
- Các mẫu truy vấn N+1 trong quá trình lấy dữ liệu
- Các endpoint dạng danh sách mà không có phân trang (pagination)
- Hình ảnh không có kích thước, không có lazy loading hoặc không có kích thước tương ứng (responsive sizes)
- Kích thước bundle tăng trưởng mà không được xem xét
- Không có giám sát hiệu suất trong môi trường production
- Sử dụng `React.memo` và `useMemo` khắp mọi nơi (lạm dụng cũng tệ như thiếu hụt)

## Xác minh (Verification)

Sau bất kỳ thay đổi nào liên quan đến hiệu suất:

- [ ] Có kết quả đo lường trước và sau (số liệu cụ thể)
- [ ] Điểm nghẽn cụ thể đã được xác định và giải quyết
- [ ] Core Web Vitals nằm trong ngưỡng "Tốt"
- [ ] Kích thước bundle không tăng đáng kể
- [ ] Không có truy vấn N+1 trong mã lấy dữ liệu mới
- [ ] Ngân sách hiệu suất vượt qua trong CI (nếu có cấu hình)
- [ ] Các test hiện có vẫn vượt qua (việc tối ưu hóa không làm hỏng hành vi của ứng dụng)
