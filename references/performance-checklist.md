# Danh sách kiểm tra hiệu năng (Performance Checklist)

Danh sách kiểm tra tra cứu nhanh cho hiệu năng ứng dụng web. Sử dụng cùng với skill `performance-optimization`.

## Mục lục

- [Mục tiêu Core Web Vitals](#core-web-vitals-targets)
- [Chẩn đoán TTFB](#ttfb-diagnosis)
- [Danh sách kiểm tra Frontend](#frontend-checklist)
- [Danh sách kiểm tra Backend](#backend-checklist)
- [Các lệnh đo lường](#measurement-commands)
- [Các Anti-Pattern phổ biến](#common-anti-patterns)

## Mục tiêu Core Web Vitals

| Chỉ số | Tốt | Cần cải thiện | Kém |
|--------|------|------------|------|
| LCP (Largest Contentful Paint) | ≤ 2.5s | ≤ 4.0s | > 4.0s |
| INP (Interaction to Next Paint) | ≤ 200ms | ≤ 500ms | > 500ms |
| CLS (Cumulative Layout Shift) | ≤ 0.1 | ≤ 0.25 | > 0.25 |

## Chẩn đoán TTFB

Khi TTFB chậm (> 800ms), hãy kiểm tra từng thành phần trong thác nước (waterfall) Network của DevTools:

- [ ] **Phân giải DNS (DNS resolution)** chậm → thêm `<link rel="dns-prefetch">` hoặc `<link rel="preconnect">` cho các origin đã biết
- [ ] **Bắt tay TCP/TLS (TCP/TLS handshake)** chậm → bật HTTP/2, cân nhắc triển khai edge, xác minh keep-alive
- [ ] **Xử lý phía server (Server processing)** chậm → profiling backend, kiểm tra các truy vấn chậm, thêm caching

## Danh sách kiểm tra Frontend

### Hình ảnh
- [ ] Hình ảnh sử dụng các định dạng hiện đại (WebP, AVIF)
- [ ] Hình ảnh có kích thước đáp ứng (`srcset` và `sizes`)
- [ ] Hình ảnh và các phần tử `<source>` có `width` và `height` tường minh (ngăn chặn CLS trong art direction)
- [ ] Hình ảnh dưới nếp gấp (below-the-fold) sử dụng `loading="lazy"` và `decoding="async"`
- [ ] Hình ảnh Hero/LCP sử dụng `fetchpriority="high"` và không sử dụng lazy loading

### JavaScript
- [ ] Kích thước bundle dưới 200KB gzipped (cho lần tải đầu tiên)
- [ ] Code splitting với `import()` động cho các route và tính năng nặng
- [ ] Tree shaking được bật (xác minh dependency cung cấp ESM và đánh dấu `sideEffects: false`)
- [ ] Không có JavaScript chặn (blocking) trong `<head>` (sử dụng `defer` hoặc `async`)
- [ ] Các tính toán nặng được chuyển sang Web Workers (nếu áp dụng)
- [ ] `React.memo()` cho các component đắt đỏ khi re-render với cùng props
- [ ] `useMemo()` / `useCallback()` chỉ dùng khi profiling cho thấy có lợi ích
- [ ] Các tác vụ dài (Long tasks > 50ms) được chia nhỏ để giữ luồng chính (main thread) luôn sẵn sàng — đây là đòn bẩy chính cho INP
- [ ] Sử dụng pattern `yieldToMain` bên trong các vòng lặp chạy dài để các sự kiện input có thể chạy giữa các phân đoạn (chunks)
- [ ] Sử dụng các API lập lịch hiện đại khi có sẵn: `scheduler.yield()` (ưu tiên), `scheduler.postTask()` với các mức ưu tiên, `isInputPending()` để chỉ yield khi cần thiết
- [ ] `requestIdleCallback` cho các công việc có thể trì hoãn, không khẩn cấp (analytics flush, prefetch, warmup)
- [ ] Các công việc không quan trọng được trì hoãn ra khỏi các event handler (ví dụ: analytics, logging) để phản hồi tương tác không bị chậm trễ
- [ ] Các script bên thứ ba được tải với `async` / `defer`, được kiểm tra kích thước và sử dụng facade khi quá nặng (chat widgets, embeds)

### CSS
- [ ] Critical CSS được inline hoặc preload
- [ ] Không có CSS chặn render (render-blocking) cho các style không quan trọng
- [ ] Không tốn chi phí runtime CSS-in-JS trong production (sử dụng extraction)

### Phông chữ (Fonts)
- [ ] Giới hạn trong 2–3 họ phông chữ, mỗi họ 2–3 weight (mỗi weight bổ sung là một request khác)
- [ ] Chỉ sử dụng định dạng WOFF2 (nhỏ nhất, hỗ trợ phổ biến — bỏ qua WOFF/TTF/EOT)
- [ ] Tự host (self-hosted) khi có thể (CDN phông chữ bên thứ ba làm tăng round-trip cho DNS + TCP + TLS)
- [ ] Các phông chữ quan trọng cho LCP được preload: `<link rel="preload" as="font" type="font/woff2" crossorigin>`
- [ ] `font-display: swap` (hoặc `optional` cho phông chữ không quan trọng) để tránh FOIT chặn render
- [ ] Subset bằng `unicode-range` để chỉ gửi những glyph mà mỗi trang cần
- [ ] Cân nhắc sử dụng Variable fonts khi cần nhiều weight/style (một file thay thế cho nhiều file)
- [ ] Điều chỉnh thông số phông chữ dự phòng (fallback font metrics) với `size-adjust`, `ascent-override`, `descent-override` để giảm CLS khi chuyển phông chữ (font swap)
- [ ] Cân nhắc sử dụng system font stack trước khi dùng bất kỳ phông chữ tùy chỉnh nào

### Mạng (Network)
- [ ] Tài sản tĩnh (static assets) được cache với `max-age` dài + content hashing
- [ ] Phản hồi API được cache khi phù hợp (`Cache-Control`)
- [ ] HTTP/2 hoặc HTTP/3 được bật
- [ ] Các tài nguyên được preconnect (`<link rel="preconnect">`) cho các origin đã biết
- [ ] `fetchpriority` được sử dụng cho các tài nguyên quan trọng không phải hình ảnh (ví dụ: `<link rel="preload">` chính, `<script>` trên nếp gấp) — không chỉ dành cho `<img>`
- [ ] Không có các chuyển hướng (redirect) không cần thiết

### Rendering
- [ ] Không gây ra layout thrashing (forced synchronous layouts)
- [ ] Animations sử dụng `transform` và `opacity` (tăng tốc bởi GPU)
- [ ] Các danh sách dài sử dụng ảo hóa (virtualization) (ví dụ: `react-window`)
- [ ] Không re-render toàn trang không cần thiết
- [ ] Các phần ngoài màn hình sử dụng `content-visibility: auto` cùng với `contain-intrinsic-size` để bỏ qua layout/paint cho các khu vực không hiển thị
- [ ] Không sử dụng event handler `unload` và không dùng `Cache-Control: no-store` cho các phản hồi HTML — để giữ khả năng sử dụng back/forward cache (bfcache)

## Danh sách kiểm tra Backend

### Cơ sở dữ liệu (Database)
- [ ] Không có pattern truy vấn N+1 (sử dụng eager loading / joins)
- [ ] Các truy vấn có index phù hợp
- [ ] Các endpoint danh sách được phân trang (không bao giờ dùng `SELECT * FROM table`)
- [ ] Cấu hình connection pooling
- [ ] Bật ghi nhật ký truy vấn chậm (slow query logging)

### API
- [ ] Thời gian phản hồi < 200ms (p95)
- [ ] Không có tính toán nặng đồng bộ trong request handlers
- [ ] Sử dụng thao tác hàng loạt (bulk operations) thay vì vòng lặp các cuộc gọi đơn lẻ
- [ ] Nén phản hồi (gzip/brotli)
- [ ] Caching phù hợp (in-memory, Redis, CDN)

### Cơ sở hạ tầng (Infrastructure)
- [ ] Sử dụng CDN cho tài sản tĩnh (static assets)
- [ ] Server đặt gần người dùng (hoặc triển khai edge)
- [ ] Cấu hình mở rộng theo chiều ngang (horizontal scaling) (nếu cần)
- [ ] Có endpoint kiểm tra sức khỏe (health check) cho bộ cân bằng tải (load balancer)

## Các lệnh đo lường

### Dữ liệu trường (field data) INP và quy trình DevTools

1. **Ưu tiên dữ liệu trường** — kiểm tra [CrUX Vis](https://developer.chrome.com/docs/crux/vis) hoặc công cụ RUM của bạn để xem INP thực tế của người dùng trước khi tối ưu hóa
2. **Xác định các tương tác chậm** — mở DevTools → Performance panel → ghi lại trong khi tương tác; tìm các long tasks được kích hoạt bởi clicks/keystrokes
3. **Kiểm tra trên Android tầm trung** — các vấn đề về INP thường chỉ xuất hiện trên phần cứng chậm hơn; sử dụng thiết bị thật hoặc tính năng CPU throttling của DevTools (chậm lại 4×–6×)

```bash
# Lighthouse CLI
npx lighthouse https://localhost:3000 --output json --output-path ./report.json

# Bundle analysis
npx webpack-bundle-analyzer stats.json
# hoặc đối với Vite:
npx vite-bundle-visualizer

# Kiểm tra kích thước bundle
npx bundlesize

# Web Vitals trong mã nguồn
import { onLCP, onINP, onCLS } from 'web-vitals';
onLCP(console.log);
onINP(console.log);
onCLS(console.log);

# INP với chi tiết mức độ tương tác (attribution build)
import { onINP } from 'web-vitals/attribution';
onINP(({ value, attribution }) => {
  const { interactionTarget, inputDelay, processingDuration, presentationDelay } = attribution;
  console.log({ value, interactionTarget, inputDelay, processingDuration, presentationDelay });
});
```

## Các Anti-Pattern phổ biến

| Anti-Pattern | Tác động | Cách khắc phục |
|---|---|---|
| N+1 queries | Tải DB tăng tuyến tính | Sử dụng joins, includes, hoặc batch loading |
| Unbounded queries | Cạn kiệt bộ nhớ, timeout | Luôn phân trang, thêm LIMIT |
| Missing indexes | Đọc chậm khi dữ liệu tăng | Thêm index cho các cột được lọc/sắp xếp |
| Layout thrashing | Giật (jank), rớt khung hình | Gom nhóm các thao tác đọc DOM, sau đó gom nhóm các thao tác ghi |
| Unoptimized images | LCP chậm, lãng phí băng thông | Sử dụng WebP, kích thước đáp ứng, lazy load |
| Large bundles | Thời gian tương tác (TTI) chậm | Code split, tree shake, kiểm tra deps |
| Blocking main thread | INP kém, UI không phản hồi | Chia nhỏ long tasks với `scheduler.yield()` / `yieldToMain`, chuyển sang Web Workers |
| Memory leaks | Bộ nhớ tăng dần, cuối cùng gây crash | Dọn dẹp listeners, intervals, refs |
