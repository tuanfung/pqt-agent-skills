# Danh sách kiểm tra bảo mật (Security Checklist)

Tài liệu tham khảo nhanh về bảo mật ứng dụng web. Sử dụng cùng với skill `security-and-hardening`.

## Mục lục

- [Mô hình hóa mối đe dọa (Bắt đầu từ đây)](#threat-modeling-start-here)
- [Kiểm tra trước khi commit](#pre-commit-checks)
- [Xác thực (Authentication)](#authentication)
- [Phân quyền (Authorization)](#authorization)
- [Xác thực đầu vào (Input Validation)](#input-validation)
- [Security Headers](#security-headers)
- [Cấu hình CORS](#cors-configuration)
- [Bảo vệ dữ liệu](#data-protection)
- [Bảo mật phụ thuộc (Dependency Security)](#dependency-security)
- [Bảo mật AI / LLM](#ai--llm-security)
- [Xử lý lỗi](#error-handling)
- [Tham khảo nhanh OWASP Top 10](#owasp-top-10-quick-reference)
- [Tham khảo nhanh OWASP Top 10 cho LLM](#owasp-top-10-for-llms-quick-reference)

## Mô hình hóa mối đe dọa (Bắt đầu từ đây)

Trước khi áp dụng các biện pháp kiểm soát, hãy dành năm phút để suy nghĩ như một kẻ tấn công:

- [ ] Đã lập bản đồ các ranh giới tin cậy (trust boundaries) (requests, uploads, webhooks, third-party APIs, đầu ra của LLM)
- [ ] Đã xác định các tài sản (assets) (credentials, PII, dữ liệu thanh toán, hành động của admin, luân chuyển tiền tệ)
- [ ] Đã thực hiện phân tích STRIDE cho mỗi ranh giới (Spoofing, Tampering, Repudiation, Info disclosure, DoS, Elevation)
- [ ] Đã viết các trường hợp lạm dụng (abuse cases) bên cạnh các trường hợp sử dụng (use cases) ("tôi có thể lợi dụng điều này như thế nào?")

## Kiểm tra trước khi commit

- [ ] Không có bí mật (secrets) trong code (`git diff --cached | grep -i "password\|secret\|api_key\|token"`)
- [ ] `.gitignore` bao gồm: `.env`, `.env.local`, `*.pem`, `*.key`
- [ ] `.env.example` sử dụng các giá trị giữ chỗ (không phải bí mật thật)

## Xác thực (Authentication)

- [ ] Mật khẩu được hash bằng bcrypt (≥12 rounds), scrypt, hoặc argon2
- [ ] Session cookies: `httpOnly`, `secure`, `sameSite: 'lax'`
- [ ] Đã cấu hình thời hạn hết hạn của session (max-age hợp lý)
- [ ] Rate limiting tại endpoint đăng nhập (≤10 lần thử trong 15 phút)
- [ ] Password reset tokens: giới hạn thời gian (≤1 giờ), sử dụng một lần
- [ ] Khóa tài khoản sau nhiều lần thất bại (tùy chọn, có thông báo)
- [ ] Hỗ trợ MFA cho các thao tác nhạy cảm (tùy chọn nhưng khuyến khích)

## Phân quyền (Authorization)

- [ ] Mọi endpoint được bảo vệ đều kiểm tra xác thực (authentication)
- [ ] Mọi truy cập tài nguyên đều kiểm tra quyền sở hữu/vai trò (ngăn chặn IDOR)
- [ ] Các endpoint dành cho admin yêu cầu xác minh vai trò admin
- [ ] API keys được giới hạn ở mức quyền tối thiểu cần thiết
- [ ] JWT tokens được xác thực (signature, expiration, issuer)

## Xác thực đầu vào (Input Validation)

- [ ] Tất cả đầu vào của người dùng được xác thực tại các ranh giới hệ thống (API routes, form handlers)
- [ ] Việc xác thực sử dụng allowlists (không dùng denylists)
- [ ] Độ dài chuỗi bị giới hạn (min/max)
- [ ] Các dải giá trị số được xác thực
- [ ] Định dạng Email, URL và ngày tháng được xác thực bằng các thư viện phù hợp
- [ ] Upload file: giới hạn loại file, giới hạn kích thước, xác minh nội dung
- [ ] Các truy vấn SQL được tham số hóa (parameterized) (không nối chuỗi)
- [ ] Đầu ra HTML được mã hóa (sử dụng auto-escaping của framework)
- [ ] URL được xác thực trước khi redirect (ngăn chặn open redirect)
- [ ] Việc fetch URL ở phía server được đưa vào allowlist; chặn các IP riêng tư/dự phòng (ngăn chặn SSRF)

## Security Headers

```
Content-Security-Policy: default-src 'self'; script-src 'self'
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 0  (disabled, rely on CSP)
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
```

## Cấu hình CORS

```typescript
// Hạn chế (khuyến nghị)
cors({
  origin: ['https://yourdomain.com', 'https://app.yourdomain.com'],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
})

// KHÔNG BAO GIỜ sử dụng trong môi trường production:
cors({ origin: '*' })  // Cho phép mọi origin
```

## Bảo vệ dữ liệu

- [ ] Các trường nhạy cảm được loại bỏ khỏi phản hồi API (`passwordHash`, `resetToken`, v.v.)
- [ ] Dữ liệu nhạy cảm không được ghi log (mật khẩu, token, số thẻ tín dụng đầy đủ)
- [ ] PII được mã hóa khi lưu trữ (nếu quy định yêu cầu)
- [ ] Sử dụng HTTPS cho mọi giao tiếp bên ngoài
- [ ] Các bản sao lưu cơ sở dữ liệu được mã hóa

## Bảo mật phụ thuộc (Dependency Security)

```bash
# Kiểm tra phụ thuộc
npm audit

# Tự động sửa nếu có thể
npm audit fix

# Kiểm tra các lỗ hổng nghiêm trọng
npm audit --audit-level=critical

# Giữ cho các phụ thuộc luôn được cập nhật
npx npm-check-updates
```

**Vệ sinh chuỗi cung ứng** (`npm audit` sẽ không phát hiện được các gói độc hại):
- [ ] Lockfile đã được commit; CI cài đặt bằng `npm ci` (không dùng `npm install`)
- [ ] Các phụ thuộc mới được xem xét (bảo trì, lượt tải, `postinstall` scripts)
- [ ] Không có typosquats (`cross-env` so với `crossenv`, `react-dom` so với `reactdom`)

## Bảo mật AI / LLM

Đối với bất kỳ tính năng nào gọi LLM (chatbot, trình tóm tắt, agent, RAG):

- [ ] Đầu ra của mô hình được coi là không đáng tin cậy — không bao giờ đưa trực tiếp vào `eval`/SQL/shell/`innerHTML`/đường dẫn file
- [ ] Giả định có prompt injection; quyền hạn được thực thi trong code, không phải trong system prompt
- [ ] Bí mật, dữ liệu chéo tenant và toàn bộ system prompt được giữ ngoài context window
- [ ] Quyền hạn của tool/agent được giới hạn; các hành động gây phá hủy hoặc không thể hoàn tác yêu cầu xác nhận
- [ ] Đã thiết lập giới hạn token, rate và đệ quy/vòng lặp (kiểm soát mức tiêu thụ)

## Xử lý lỗi

```typescript
// Production: lỗi chung, không hiển thị chi tiết nội bộ
res.status(500).json({
  error: { code: 'INTERNAL_ERROR', message: 'Something went wrong' }
});

// KHÔNG BAO GIỜ dùng trong production:
res.status(500).json({
  error: err.message,
  stack: err.stack,         // Lộ chi tiết nội bộ
  query: err.sql,           // Lộ chi tiết cơ sở dữ liệu
});
```

## Tham khảo nhanh OWASP Top 10

| # | Lỗ hổng | Phòng ngừa |
|---|---|---|
| 1 | Kiểm soát truy cập bị hỏng | Kiểm tra xác thực tại mọi endpoint, xác minh quyền sở hữu |
| 2 | Lỗi mã hóa | HTTPS, hash mạnh, không để bí mật trong code |
| 3 | Tiêm mã (Injection) | Truy vấn tham số hóa, xác thực đầu vào |
| 4 | Thiết kế không an toàn | Mô hình hóa mối đe dọa, phát triển hướng đặc tả (spec-driven development) |
| 5 | Cấu hình bảo mật sai | Security headers, quyền tối thiểu, kiểm tra phụ thuộc |
| 6 | Thành phần dễ bị tổn thương | `npm audit`, cập nhật phụ thuộc, sử dụng tối thiểu các phụ thuộc |
| 7 | Lỗi xác thực | Mật khẩu mạnh, rate limiting, quản lý session |
| 8 | Lỗi toàn vẹn dữ liệu | Xác minh cập nhật/phụ thuộc, các artifact có chữ ký |
| 9 | Lỗi ghi log | Ghi log các sự kiện bảo mật, không ghi log bí mật |
| 10 | SSRF | Xác thực/allowlist URL, hạn chế các yêu cầu gửi ra ngoài |

## Tham khảo nhanh OWASP Top 10 cho LLM

Đối với các ứng dụng có tính năng LLM. Xem [OWASP GenAI Security Project](https://genai.owasp.org/llm-top-10/).

| ID | Rủi ro | Phòng ngừa |
|---|---|---|
| LLM01 | Prompt Injection | Không tin tưởng system prompt như một ranh giới; thực thi quyền hạn trong code |
| LLM02 | Tiết lộ thông tin nhạy cảm | Giữ bí mật/PII ngoài prompt; lọc đầu ra |
| LLM03 | Chuỗi cung ứng | Kiểm tra kỹ các mô hình, tập dữ liệu và plugin như bất kỳ phụ thuộc nào khác |
| LLM04 | Đầu độc dữ liệu và mô hình | Sử dụng nguồn mô hình tin cậy, xác minh tính toàn vẹn; kiểm tra dữ liệu fine-tuning và RAG |
| LLM05 | Xử lý đầu ra không đúng cách | Coi đầu ra mô hình là không đáng tin cậy; xác thực, tham số hóa, mã hóa |
| LLM06 | Quyền hạn quá mức | Giới hạn quyền hạn của tool; xác nhận các hành động gây phá hủy |
| LLM07 | Lộ System Prompt | Giả định system prompt có thể bị lộ; không để bí mật trong đó |
| LLM08 | Điểm yếu của Vector và Embedding | Phân vùng RAG embeddings theo tenant; xác thực tài liệu trước khi indexing |
| LLM09 | Thông tin sai lệch | Cung cấp căn cứ cho câu trả lời bằng các trích dẫn; xác thực các tuyên bố quan trọng; luôn có con người kiểm soát (human-in-the-loop) |
| LLM10 | Tiêu thụ không giới hạn | Giới hạn token, tốc độ yêu cầu và độ sâu của vòng lặp/đệ quy |
