---
name: shipping-and-launch
description: Chuẩn bị triển khai sản phẩm (production launch). Sử dụng khi chuẩn bị triển khai lên môi trường production. Sử dụng khi bạn cần một danh sách kiểm tra trước khi ra mắt (pre-launch checklist), khi thiết lập giám sát (monitoring), khi lập kế hoạch triển khai theo giai đoạn (staged rollout), hoặc khi bạn cần một chiến lược hoàn tác (rollback strategy).
---

# Shipping and Launch

## Tổng quan

Triển khai với sự tự tin. Mục tiêu không chỉ là triển khai — mà là triển khai một cách an toàn, với hệ thống giám sát đã sẵn sàng, kế hoạch hoàn tác đã chuẩn bị và hiểu rõ thế nào là thành công. Mọi lần ra mắt đều phải có khả năng hoàn tác (reversible), có thể quan sát (observable) và thực hiện theo từng bước (incremental).

## Khi nào sử dụng

- Triển khai một tính năng lên production lần đầu tiên
- Phát hành một thay đổi lớn cho người dùng
- Di chuyển dữ liệu hoặc hạ tầng (migrating data or infrastructure)
- Mở chương trình beta hoặc truy cập sớm (early access)
- Bất kỳ lần triển khai nào tiềm ẩn rủi ro (tất cả mọi lần triển khai)

## Danh sách kiểm tra trước khi ra mắt (Pre-Launch Checklist)

### Chất lượng mã nguồn (Code Quality)

- [ ] Tất cả các bài kiểm tra đều vượt qua (unit, integration, e2e)
- [ ] Build thành công và không có cảnh báo (warnings)
- [ ] Lint và kiểm tra kiểu (type checking) vượt qua
- [ ] Mã nguồn đã được review và phê duyệt
- [ ] Không còn các ghi chú TODO cần được giải quyết trước khi ra mắt
- [ ] Không còn các câu lệnh gỡ lỗi `console.log` trong mã nguồn production
- [ ] Xử lý lỗi (error handling) bao quát các trường hợp thất bại dự kiến

### Bảo mật (Security)

- [ ] Không có bí mật (secrets) trong mã nguồn hoặc quản lý phiên bản (version control)
- [ ] `npm audit` không cho thấy các lỗ hổng nghiêm trọng (critical) hoặc mức độ cao (high)
- [ ] Kiểm tra đầu vào (input validation) trên tất cả các endpoint hướng tới người dùng
- [ ] Các kiểm tra xác thực (authentication) và phân quyền (authorization) đã được thiết lập
- [ ] Cấu hình các header bảo mật (CSP, HSTS, v.v.)
- [ ] Giới hạn tốc độ (rate limiting) trên các endpoint xác thực
- [ ] CORS được cấu hình cho các nguồn (origins) cụ thể (không dùng ký tự đại diện wildcard)

### Hiệu năng (Performance)

- [ ] Core Web Vitals nằm trong ngưỡng "Tốt" (Good)
- [ ] Không có truy vấn N+1 trong các luồng quan trọng
- [ ] Hình ảnh đã được tối ưu hóa (nén, kích thước đáp ứng, lazy loading)
- [ ] Kích thước bundle nằm trong ngân sách cho phép
- [ ] Các truy vấn cơ sở dữ liệu có chỉ mục (index) phù hợp
- [ ] Cấu hình lưu bộ nhớ đệm (caching) cho các tài sản tĩnh và các truy vấn lặp lại

### Khả năng tiếp cận (Accessibility)

- [ ] Điều hướng bằng bàn phím hoạt động cho tất cả các thành phần tương tác
- [ ] Trình đọc màn hình (screen reader) có thể truyền tải nội dung và cấu trúc trang
- [ ] Độ tương phản màu sắc đáp ứng chuẩn WCAG 2.1 AA (4.5:1 cho văn bản)
- [ ] Quản lý tiêu điểm (focus management) chính xác cho các modal và nội dung động
- [ ] Các thông báo lỗi mang tính mô tả và được liên kết với các trường trong form
- [ ] Không có cảnh báo về khả năng tiếp cận trong axe-core hoặc Lighthouse

### Hạ tầng (Infrastructure)

- [ ] Các biến môi trường (environment variables) đã được thiết lập trong production
- [ ] Các di chuyển cơ sở dữ liệu (database migrations) đã được áp dụng (hoặc sẵn sàng để áp dụng)
- [ ] DNS và SSL đã được cấu hình
- [ ] CDN đã được cấu hình cho các tài sản tĩnh
- [ ] Hệ thống ghi log và báo cáo lỗi đã được cấu hình
- [ ] Điểm kiểm tra sức khỏe (health check endpoint) tồn tại và phản hồi

### Tài liệu (Documentation)

- [ ] README đã được cập nhật với bất kỳ yêu cầu thiết lập mới nào
- [ ] Tài liệu API hiện tại
- [ ] Các ADR đã được viết cho bất kỳ quyết định kiến trúc nào
- [ ] Changelog đã được cập nhật
- [ ] Tài liệu hướng dẫn người dùng đã được cập nhật (nếu áp dụng)

## Chiến lược Feature Flag

Triển khai phía sau các feature flag để tách biệt việc triển khai (deployment) khỏi việc phát hành (release):

```typescript
// Kiểm tra feature flag
const flags = await getFeatureFlags(userId);

if (flags.taskSharing) {
  // Tính năng mới: chia sẻ tác vụ
  return <TaskSharingPanel task={task} />;
}

// Mặc định: hành vi hiện tại
return null;
```

**Vòng đời của feature flag:**

```
1. TRIỂN KHAI với cờ TẮT     → Mã nguồn nằm trong production nhưng không hoạt động
2. BẬT cho nhóm/beta         → Kiểm thử nội bộ trong môi trường production
3. TRIỂN KHAI DẦN            → 5% → 25% → 50% → 100% người dùng
4. GIÁM SÁT tại mỗi giai đoạn → Theo dõi tỷ lệ lỗi, hiệu năng, phản hồi của người dùng
5. DỌN DẸP                  → Xóa flag và các nhánh mã nguồn không còn sử dụng sau khi triển khai toàn bộ
```

**Quy tắc:**
- Mọi feature flag đều phải có người sở hữu và ngày hết hạn
- Dọn dẹp các flag trong vòng 2 tuần sau khi triển khai toàn bộ
- Không lồng các feature flag (tạo ra các tổ hợp mũ số lượng lớn)
- Kiểm thử cả hai trạng thái của flag (bật và tắt) trong CI

## Triển khai theo giai đoạn (Staged Rollout)

### Trình tự triển khai

```
1. TRIỂN KHAI lên staging
   └── Toàn bộ bộ kiểm thử chạy trong môi trường staging
   └── Smoke test thủ công các luồng quan trọng

2. TRIỂN KHAI lên production (feature flag TẮT)
   └── Xác minh triển khai thành công (health check)
   └── Kiểm tra giám sát lỗi (không có lỗi mới)

3. BẬT cho nhóm (flag BẬT cho người dùng nội bộ)
   └── Nhóm sử dụng tính năng trong production
   └── Cửa sổ giám sát 24 giờ

4. Triển khai CANARY (flag BẬT cho 5% người dùng)
   └── Giám sát tỷ lệ lỗi, độ trễ, hành vi người dùng
   └── So sánh các chỉ số: canary với mốc cơ sở (baseline)
   └── Cửa sổ giám sát 24-48 giờ
   └── Chỉ tiếp tục nếu tất cả các ngưỡng đều vượt qua (xem bảng dưới đây)

5. Tăng DẦN (25% -> 50% -> 100%)
   └── Tương tự giám sát ở mỗi bước
   └── Có khả năng hoàn tác về tỷ lệ phần trăm trước đó tại bất kỳ thời điểm nào

6. Triển khai TOÀN BỘ (flag BẬT cho tất cả người dùng)
   └── Giám sát trong 1 tuần
   └── Dọn dẹp feature flag
```

### Ngưỡng quyết định triển khai

Sử dụng các ngưỡng này để quyết định nên tiếp tục, tạm dừng hoặc hoàn tác tại mỗi giai đoạn:

| Chỉ số | Tiếp tục (xanh) | Tạm dừng và điều tra (vàng) | Hoàn tác (đỏ) |
|--------|-----------------|-------------------------------|-----------------|
| Tỷ lệ lỗi | Trong khoảng 10% so với mốc cơ sở | Cao hơn mốc cơ sở từ 10-100% | > 2 lần mốc cơ sở |
| Độ trễ P95 | Trong khoảng 20% so với mốc cơ sở | Cao hơn mốc cơ sở từ 20-50% | > 50% so với mốc cơ sở |
| Lỗi JS phía client | Không có loại lỗi mới | Lỗi mới xuất hiện ở < 0.1% số phiên | Lỗi mới xuất hiện ở > 0.1% số phiên |
| Chỉ số kinh doanh | Trung lập hoặc tích cực | Giảm < 5% (có thể là nhiễu) | Giảm > 5% |

### Khi nào cần Hoàn tác (Roll Back)

Hoàn tác ngay lập tức nếu:
- Tỷ lệ lỗi tăng hơn 2 lần so với mốc cơ sở
- Độ trễ P95 tăng hơn 50%
- Các sự cố do người dùng báo cáo tăng đột biến
- Phát hiện các vấn đề về toàn vẹn dữ liệu
- Phát hiện lỗ hổng bảo mật

## Giám sát và Khả năng quan sát (Monitoring and Observability)

### Những gì cần giám sát

```
Chỉ số ứng dụng:
├── Tỷ lệ lỗi (tổng thể và theo từng endpoint)
├── Thời gian phản hồi (p50, p95, p99)
├── Khối lượng yêu cầu (request volume)
├── Người dùng hoạt động
└── Các chỉ số kinh doanh chính (chuyển đổi, tương tác)

Chỉ số hạ tầng:
├── Sử dụng CPU và bộ nhớ
├── Sử dụng pool kết nối cơ sở dữ liệu
├── Không gian đĩa
├── Độ trễ mạng
└── Độ sâu của hàng đợi (nếu áp dụng)

Chỉ số phía client:
├── Core Web Vitals (LCP, INP, CLS)
├── Lỗi JavaScript
├── Tỷ lệ lỗi API từ góc nhìn của client
└── Thời gian tải trang
```

### Báo cáo lỗi

```typescript
// Thiết lập error boundary với báo cáo lỗi
class ErrorBoundary extends React.Component {
  componentDidCatch(error: Error, info: React.ErrorInfo) {
    // Báo cáo tới dịch vụ theo dõi lỗi
    reportError(error, {
      componentStack: info.componentStack,
      userId: getCurrentUser()?.id,
      page: window.location.pathname,
    });
  }

  render() {
    if (this.state.hasError) {
      return <ErrorFallback onRetry={() => this.setState({ hasError: false })} />;
    }
    return this.props.children;
  }
}

// Báo cáo lỗi phía server
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  reportError(err, {
    method: req.method,
    url: req.url,
    userId: req.user?.id,
  });

  // Không để lộ thông tin nội bộ cho người dùng
  res.status(500).json({
    error: { code: 'INTERNAL_ERROR', message: 'Something went wrong' },
  });
});
```

### Xác minh sau triển khai

Trong giờ đầu tiên sau khi triển khai:

```
1. Kiểm tra health endpoint trả về 200
2. Kiểm tra bảng điều khiển giám sát lỗi (không có loại lỗi mới)
3. Kiểm tra bảng điều khiển độ trễ (không có sự suy giảm hiệu năng)
4. Kiểm thử thủ công luồng người dùng quan trọng
5. Xác minh logs đang truyền về và có thể đọc được
6. Xác nhận cơ chế hoàn tác hoạt động (chạy thử nếu có thể)
```

## Chiến lược Hoàn tác (Rollback Strategy)

Mọi lần triển khai đều cần một kế hoạch hoàn tác trước khi thực hiện:

```markdown
## Kế hoạch Hoàn tác cho [Tính năng/Phiên bản]

### Điều kiện kích hoạt
- Tỷ lệ lỗi > 2 lần mốc cơ sở
- Độ trễ P95 > [X]ms
- Người dùng báo cáo về [vấn đề cụ thể]

### Các bước hoàn tác
1. Tắt feature flag (nếu áp dụng)
   HOẶC
1. Triển khai phiên bản trước đó: `git revert <commit> && git push`
2. Xác minh hoàn tác: health check, giám sát lỗi
3. Thông báo: thông báo cho nhóm về việc hoàn tác

### Lưu ý về Cơ sở dữ liệu
- Migration [X] có phương án hoàn tác: `npx prisma migrate rollback`
- Dữ liệu được chèn bởi tính năng mới: [được giữ lại / được dọn dẹp]

### Thời gian hoàn tác
- Feature flag: < 1 phút
- Triển khai lại phiên bản trước: < 5 phút
- Hoàn tác cơ sở dữ liệu: < 15 phút
```

## Xem thêm

- Về Definition of Done áp dụng cho toàn bộ dự án mà mọi thay đổi phải vượt qua trước checklist này, xem tại `references/definition-of-done.md`
- Về các kiểm tra bảo mật trước khi ra mắt, xem tại `references/security-checklist.md`
- Về checklist hiệu năng trước khi ra mắt, xem tại `references/performance-checklist.md`
- Về xác minh khả năng tiếp cận trước khi ra mắt, xem tại `references/accessibility-checklist.md`

## Những lời bào chữa phổ biến

| Lời bào chữa | Thực tế |
|---|---|
| "Nó chạy tốt trên staging, chắc chắn sẽ chạy tốt trên production" | Production có dữ liệu, mô hình lưu lượng và các trường hợp biên khác nhau. Hãy giám sát sau khi triển khai. |
| "Chúng ta không cần feature flags cho cái này" | Mọi tính năng đều được hưởng lợi từ một "công tắc ngắt" (kill switch). Ngay cả những thay đổi "đơn giản" cũng có thể gây ra lỗi. |
| "Giám sát (monitoring) là quá tải/tốn kém" | Không có giám sát nghĩa là bạn phát hiện ra vấn đề từ lời phàn nàn của người dùng thay vì từ bảng điều khiển. |
| "Chúng ta sẽ thêm giám sát sau" | Hãy thêm nó trước khi ra mắt. Bạn không thể gỡ lỗi thứ mà bạn không nhìn thấy. |
| "Hoàn tác là thừa nhận thất bại" | Hoàn tác là kỹ thuật có trách nhiệm. Triển khai một tính năng bị hỏng mới là thất bại. |

## Dấu hiệu cảnh báo (Red Flags)

- Triển khai mà không có kế hoạch hoàn tác
- Không có giám sát hoặc báo cáo lỗi trong production
- Phát hành kiểu "big-bang" (mọi thứ cùng lúc, không qua staging)
- Feature flags không có ngày hết hạn hoặc người sở hữu
- Không có ai giám sát lần triển khai trong giờ đầu tiên
- Cấu hình môi trường production được thực hiện bằng trí nhớ, không phải bằng mã nguồn
- "Chiều thứ Sáu rồi, triển khai luôn đi"

## Xác minh

Trước khi triển khai:

- [ ] Hoàn thành danh sách kiểm tra trước khi ra mắt (tất cả các mục đều xanh)
- [ ] Cấu hình feature flag (nếu áp dụng)
- [ ] Tài liệu hóa kế hoạch hoàn tác
- [ ] Thiết lập các bảng điều khiển giám sát (monitoring dashboards)
- [ ] Thông báo cho nhóm về việc triển khai

Sau khi triển khai:

- [ ] Health check trả về 200
- [ ] Tỷ lệ lỗi bình thường
- [ ] Độ trễ bình thường
- [ ] Luồng người dùng quan trọng hoạt động tốt
- [ ] Logs đang truyền về
- [ ] Cơ chế hoàn tác đã được kiểm thử hoặc xác minh sẵn sàng
