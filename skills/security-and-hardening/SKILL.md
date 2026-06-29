---
name: security-and-hardening
description: Gia cố mã nguồn chống lại các lỗ hổng. Sử dụng khi xử lý đầu vào của người dùng, xác thực, lưu trữ dữ liệu hoặc tích hợp bên ngoài. Sử dụng khi xây dựng bất kỳ tính năng nào chấp nhận dữ liệu không đáng tin cậy, quản lý phiên làm việc của người dùng hoặc tương tác với các dịch vụ bên thứ ba.
---

# Bảo mật và Gia cố (Security and Hardening)

## Tổng quan

Các phương pháp phát triển ưu tiên bảo mật cho ứng dụng web. Coi mọi đầu vào bên ngoài là độc hại, mọi thông tin bí mật là thiêng liêng và mọi kiểm tra ủy quyền là bắt buộc. Bảo mật không phải là một giai đoạn — đó là một ràng buộc đối với mọi dòng mã chạm vào dữ liệu người dùng, xác thực hoặc các hệ thống bên ngoài.

## Khi nào nên sử dụng

- Xây dựng bất kỳ tính năng nào chấp nhận đầu vào từ người dùng
- Triển khai xác thực (authentication) hoặc ủy quyền (authorization)
- Lưu trữ hoặc truyền tải dữ liệu nhạy cảm
- Tích hợp với các API hoặc dịch vụ bên ngoài
- Thêm tính năng tải tệp lên, webhooks hoặc callbacks
- Xử lý dữ liệu thanh toán hoặc PII (thông tin nhận dạng cá nhân)

## Quy trình: Mô hình đe dọa trước tiên (Threat Model First)

Các biện pháp kiểm soát được gắn vào mà không có mô hình đe dọa chỉ là phỏng đoán. Trước khi gia cố, hãy dành năm phút suy nghĩ như một kẻ tấn công:

1. **Xác định các biên tin cậy (trust boundaries).** Dữ liệu không đáng tin cậy xâm nhập vào hệ thống của bạn từ đâu? Yêu cầu HTTP, các trường biểu mẫu, tải tệp lên, webhooks, API bên thứ ba, hàng đợi tin nhắn và **đầu ra của LLM**. Mỗi biên là một bề mặt tấn công.
2. **Xác định các tài sản (assets).** Điều gì đáng bị đánh cắp hoặc phá hoại? Thông tin xác thực, PII, dữ liệu thanh toán, các hành động quản trị, luân chuyển tiền tệ.
3. **Áp dụng STRIDE cho mỗi biên** — một lăng kính nhanh, không phải là một nghi lễ:

| Đe dọa | Câu hỏi | Biện pháp giảm thiểu điển hình |
|---|---|---|
| **S**poofing (Giả mạo) | Ai đó có thể mạo danh người dùng/dịch vụ không? | Xác thực, xác minh chữ ký |
| **T**ampering (Can thiệp) | Dữ liệu có thể bị thay đổi trong quá trình truyền tải hoặc khi lưu trữ không? | Kiểm tra tính toàn vẹn, truy vấn tham số hóa (parameterized queries), HTTPS |
| **R**epudiation (Chối bỏ) | Một hành động có thể bị phủ nhận sau đó không? | Ghi nhật ký kiểm tra (audit logging) các sự kiện bảo mật |
| **I**nformation disclosure (Tiết lộ thông tin) | Dữ liệu có thể bị rò rỉ không? | Mã hóa, danh sách cho phép (allowlists) các trường, thông báo lỗi chung chung |
| **D**enial of service (Từ chối dịch vụ) | Hệ thống có thể bị làm quá tải không? | Giới hạn tốc độ (rate limiting), giới hạn kích thước đầu vào, timeouts |
| **E**levation of privilege (Leo thang đặc quyền) | Người dùng có thể giành được quyền hạn mà họ không nên có không? | Kiểm tra ủy quyền, quyền tối thiểu (least privilege) |

4. **Viết các trường hợp lạm dụng (abuse cases) bên cạnh các trường hợp sử dụng (use cases).** Với mỗi tính năng, hãy hỏi "tôi sẽ lạm dụng điều này như thế nào?" — sau đó biến điều đó thành bài kiểm tra đầu tiên của bạn.

Nếu bạn không thể xác định các biên tin cậy cho một tính năng, bạn chưa sẵn sàng để bảo mật nó. Đây chính là OWASP **A04: Insecure Design** — hầu hết các vụ vi phạm bắt đầu từ thiết kế, không phải từ mã nguồn.

## Hệ thống biên ba tầng

### Luôn thực hiện (Không ngoại lệ)

- **Xác thực mọi đầu vào bên ngoài** tại biên hệ thống (API routes, trình xử lý biểu mẫu)
- **Tham số hóa tất cả các truy vấn cơ sở dữ liệu** — không bao giờ nối chuỗi đầu vào của người dùng vào SQL
- **Mã hóa đầu ra (encode output)** để ngăn chặn XSS (sử dụng tính năng tự động thoát chuỗi của framework, không bỏ qua nó)
- **Sử dụng HTTPS** cho tất cả các giao tiếp bên ngoài
- **Băm (hash) mật khẩu** bằng bcrypt/scrypt/argon2 (không bao giờ lưu văn bản thuần túy)
- **Thiết lập các header bảo mật** (CSP, HSTS, X-Frame-Options, X-Content-Type-Options)
- **Sử dụng cookie httpOnly, secure, sameSite** cho phiên làm việc (sessions)
- **Chạy `npm audit`** (hoặc tương đương) trước mỗi lần phát hành

### Hỏi trước (Yêu cầu con người phê duyệt)

- Thêm luồng xác thực mới hoặc thay đổi logic xác thực
- Lưu trữ các danh mục dữ liệu nhạy cảm mới (PII, thông tin thanh toán)
- Thêm tích hợp dịch vụ bên ngoài mới
- Thay đổi cấu hình CORS
- Thêm trình xử lý tải tệp lên
- Sửa đổi giới hạn tốc độ (rate limiting) hoặc điều tiết (throttling)
- Cấp quyền hoặc vai trò nâng cao

### Không bao giờ làm

- **Không bao giờ commit thông tin bí mật (secrets)** vào hệ thống quản lý phiên bản (API keys, mật khẩu, tokens)
- **Không bao giờ ghi nhật ký dữ liệu nhạy cảm** (mật khẩu, tokens, số thẻ tín dụng đầy đủ)
- **Không bao giờ tin tưởng xác thực phía client** như một biên bảo mật
- **Không bao giờ tắt các header bảo mật** để cho tiện
- **Không bao giờ sử dụng `eval()` hoặc `innerHTML`** với dữ liệu do người dùng cung cấp
- **Không bao giờ lưu trữ phiên làm việc trong bộ nhớ mà client có thể truy cập** (localStorage cho auth tokens)
- **Không bao giờ để lộ stack traces** hoặc chi tiết lỗi nội bộ cho người dùng

## Các mẫu ngăn chặn OWASP Top 10

Đây là các mẫu ngăn chặn, không phải là bảng xếp hạng. Để xem thứ tự năm 2021, hãy xem bảng tham khảo nhanh trong `references/security-checklist.md`.

### Injection (SQL, NoSQL, OS Command)

```typescript
// XẤU: SQL injection thông qua nối chuỗi
const query = `SELECT * FROM users WHERE id = '${userId}'`;

// TỐT: Truy vấn tham số hóa
const user = await db.query('SELECT * FROM users WHERE id = $1', [userId]);

// TỐT: ORM với đầu vào được tham số hóa
const user = await prisma.user.findUnique({ where: { id: userId } });
```

### Broken Authentication (Xác thực bị lỗi)

```typescript
// Băm mật khẩu
import { hash, compare } from 'bcrypt';

const SALT_ROUNDS = 12;
const hashedPassword = await hash(plaintext, SALT_ROUNDS);
const isValid = await compare(plaintext, hashedPassword);

// Quản lý phiên làm việc
app.use(session({
  secret: process.env.SESSION_SECRET,  // Lấy từ môi trường, không ghi trong mã
  resave: false,
  saveUninitialized: false,
  cookie: {
    httpOnly: true,     // Không thể truy cập qua JavaScript
    secure: true,       // Chỉ HTTPS
    sameSite: 'lax',    // Bảo vệ CSRF
    maxAge: 24 * 60 * 60 * 1000,  // 24 giờ
  },
}));
```

### Cross-Site Scripting (XSS)

```typescript
// XẤU: Hiển thị đầu vào của người dùng dưới dạng HTML
element.innerHTML = userInput;

// TỐT: Sử dụng tính năng tự động thoát chuỗi của framework (React mặc định làm điều này)
return <div>{userInput}</div>;

// Nếu BẮT BUỘC phải hiển thị HTML, hãy làm sạch (sanitize) trước
import DOMPurify from 'dompurify';
const clean = DOMPurify.sanitize(userInput);
```

### Broken Access Control (Kiểm soát truy cập bị lỗi)

```typescript
// Luôn kiểm tra ủy quyền (authorization), không chỉ xác thực (authentication)
app.patch('/api/tasks/:id', authenticate, async (req, res) => {
  const task = await taskService.findById(req.params.id);

  // Kiểm tra xem người dùng đã xác thực có sở hữu tài nguyên này không
  if (task.ownerId !== req.user.id) {
    return res.status(403).json({
      error: { code: 'FORBIDDEN', message: 'Không có quyền sửa đổi tác vụ này' }
    });
  }

  // Tiếp tục cập nhật
  const updated = await taskService.update(req.params.id, req.body);
  return res.json(updated);
});
```

### Security Misconfiguration (Cấu hình bảo mật sai)

```typescript
// Security headers (sử dụng helmet cho Express)
import helmet from 'helmet';
app.use(helmet());

// Content Security Policy
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'"],
    styleSrc: ["'self'", "'unsafe-inline'"],  // Thắt chặt nếu có thể
    imgSrc: ["'self'", 'data:', 'https:'],
    connectSrc: ["'self'"],
  },
}));

// CORS — giới hạn cho các nguồn đã biết
app.use(cors({
  origin: process.env.ALLOWED_ORIGINS?.split(',') || 'http://localhost:3000',
  credentials: true,
}));
```

### Sensitive Data Exposure (Lộ dữ liệu nhạy cảm)

```typescript
// Không bao giờ trả về các trường nhạy cảm trong phản hồi API
function sanitizeUser(user: UserRecord): PublicUser {
  const { passwordHash, resetToken, ...publicFields } = user;
  return publicFields;
}

// Sử dụng biến môi trường cho các thông tin bí mật
const API_KEY = process.env.STRIPE_API_KEY;
if (!API_KEY) throw new Error('STRIPE_API_KEY chưa được cấu hình');
```

### Server-Side Request Forgery (SSRF)

Bất cứ khi nào máy chủ truy xuất một URL do người dùng tác động — webhooks, "nhập từ URL", proxy hình ảnh, xem trước liên kết — kẻ tấn công có thể hướng nó đến các dịch vụ nội bộ (cloud metadata, `localhost`, IP riêng).

```typescript
// XẤU: truy xuất bất cứ thứ gì người dùng cung cấp
await fetch(req.body.webhookUrl);

// TỐT: danh sách cho phép scheme + host, từ chối nếu BẤT KỲ IP nào được phân giải là IP riêng, cấm chuyển hướng (redirects)
import { lookup } from 'node:dns/promises';
import ipaddr from 'ipaddr.js';

const ALLOWED_HOSTS = new Set(['hooks.example.com']);

async function assertSafeUrl(raw: string): Promise<URL> {
  const url = new URL(raw);
  if (url.protocol !== 'https:') throw new Error('chỉ chấp nhận https');
  if (!ALLOWED_HOSTS.has(url.hostname)) throw new Error('host không được cho phép');
  // Phân giải TẤT CẢ các bản ghi; một địa chỉ riêng/dự phòng duy nhất sẽ làm thất bại kiểm tra.
  const addrs = await lookup(url.hostname, { all: true });
  if (addrs.some((a) => ipaddr.parse(a.address).range() !== 'unicast')) {
    throw new Error('IP riêng/dự phòng');
  }
  return url;
}

await fetch(await assertSafeUrl(req.body.webhookUrl), { redirect: 'error' });
```

Kiểm tra `range() !== 'unicast'` bao gồm loopback, link-local `169.254.169.254` (cloud metadata, mục tiêu SSRF số 1), các dải IP riêng và unique-local trên cả IPv4 và IPv6.

**Lưu ý — điều này vẫn có khoảng trống TOCTOU.** `fetch` sẽ phân giải DNS một lần nữa sau khi kiểm tra, vì vậy kẻ tấn công sử dụng bản ghi có TTL ngắn có thể rebind sang một IP nội bộ giữa lúc xác thực và kết nối. Đối với các bề mặt rủi ro cao, hãy phân giải một lần và kết nối tới IP đã ghim, hoặc đặt một agent lọc phía trước (`request-filtering-agent` / `ssrf-req-filter`).

## Mẫu xác thực đầu vào (Input Validation Patterns)

### Xác thực Schema tại các biên

```typescript
import { z } from 'zod';

const CreateTaskSchema = z.object({
  title: z.string().min(1).max(200).trim(),
  description: z.string().max(2000).optional(),
  priority: z.enum(['low', 'medium', 'high']).default('medium'),
  dueDate: z.string().datetime().optional(),
});

// Xác thực tại trình xử lý route
app.post('/api/tasks', async (req, res) => {
  const result = CreateTaskSchema.safeParse(req.body);
  if (!result.success) {
    return res.status(422).json({
      error: {
        code: 'VALIDATION_ERROR',
        message: 'Đầu vào không hợp lệ',
        details: result.error.flatten(),
      },
    });
  }
  // result.data hiện đã được định kiểu và xác thực
  const task = await taskService.create(result.data);
  return res.status(201).json(task);
});
```

### An toàn khi tải tệp lên

```typescript
// Giới hạn loại tệp và kích thước
const ALLOWED_TYPES = ['image/jpeg', 'image/png', 'image/webp'];
const MAX_SIZE = 5 * 1024 * 1024; // 5MB

function validateUpload(file: UploadedFile) {
  if (!ALLOWED_TYPES.includes(file.mimetype)) {
    throw new ValidationError('Loại tệp không được cho phép');
  }
  if (file.size > MAX_SIZE) {
    throw new ValidationError('Tệp quá lớn (tối đa 5MB)');
  }
  // Đừng tin tưởng phần mở rộng của tệp — kiểm tra magic bytes nếu quan trọng
}
```

## Phân loại kết quả npm audit

Không phải tất cả các phát hiện audit đều yêu cầu hành động ngay lập tức. Hãy sử dụng cây quyết định này:

```
npm audit báo cáo một lỗ hổng
├── Mức độ: critical (nghiêm trọng) hoặc high (cao)
│   ├── Mã bị lỗi có thể truy cập được trong ứng dụng của bạn không?
│   │   ├── CÓ --> Sửa ngay lập tức (cập nhật, vá hoặc thay thế dependency)
│   │   └── KHÔNG (dep chỉ cho phát triển, đường dẫn mã không sử dụng) --> Sửa sớm, nhưng không phải là rào cản (blocker)
│   └── Có bản vá không?
│       ├── CÓ --> Cập nhật lên phiên bản đã vá
│       └── KHÔNG --> Kiểm tra các cách khắc phục tạm thời, xem xét thay thế dependency hoặc thêm vào allowlist với ngày xem xét lại
├── Mức độ: moderate (trung bình)
│   ├── Có thể truy cập trong môi trường production? --> Sửa trong chu kỳ phát hành tiếp theo
│   └── Chỉ dành cho phát triển? --> Sửa khi thuận tiện, theo dõi trong backlog
└── Mức độ: low (thấp)
    └── Theo dõi và sửa trong các đợt cập nhật dependency định kỳ
```

**Các câu hỏi chính:**
- Hàm bị lỗi có thực sự được gọi trong luồng mã của bạn không?
- Dependency này là dependency chạy runtime hay chỉ cho phát triển (dev-only)?
- Lỗ hổng có thể khai thác được trong ngữ cảnh triển khai của bạn không (ví dụ: lỗ hổng phía máy chủ trong một ứng dụng chỉ chạy phía client)?

Khi bạn trì hoãn một bản sửa lỗi, hãy ghi lại lý do và đặt ngày xem xét lại.

### Vệ sinh chuỗi cung ứng (Supply-Chain Hygiene)

`npm audit` bắt các CVE đã biết; nó sẽ không bắt được một gói độc hại hoặc bị lỗi đánh máy (typosquatted). Ngoài ra:

- **Commit lockfile** và cài đặt bằng `npm ci` (không phải `npm install`) trong CI — đảm bảo bản build có thể tái lập, không bị trôi phiên bản ngầm.
- **Xem xét các dependency mới trước khi thêm chúng** — mức độ bảo trì, số lượt tải xuống và liệu chúng có thực sự cần thiết. Mỗi dependency là một bề mặt tấn công (OWASP **A06: Vulnerable Components**, **LLM03: Supply Chain**).
- **Cẩn thận với các script `postinstall`** trong các gói không quen thuộc — chúng chạy mã tùy ý tại thời điểm cài đặt.
- **Cảnh giác với lỗi đánh máy (typosquats)** — `cross-env` so với `crossenv`, `react-dom` so với `reactdom`.

## Giới hạn tốc độ (Rate Limiting)

```typescript
import rateLimit from 'express-rate-limit';

// Giới hạn tốc độ API chung
app.use('/api/', rateLimit({
  windowMs: 15 * 60 * 1000, // 15 phút
  max: 100,                   // 100 yêu cầu mỗi cửa sổ
  standardHeaders: true,
  legacyHeaders: false,
}));

// Giới hạn nghiêm ngặt hơn cho các endpoint xác thực
app.use('/api/auth/', rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 10,  // 10 lần thử mỗi 15 phút
}));
```

## Quản lý thông tin bí mật (Secrets Management)

```
tệp .env:
  ├── .env.example  → Được commit (mẫu với các giá trị giữ chỗ)
  ├── .env          → KHÔNG được commit (chứa thông tin bí mật thật)
  └── .env.local    → KHÔNG được commit (ghi đè cục bộ)

.gitignore phải bao gồm:
  .env
  .env.local
  .env.*.local
  *.pem
  *.key
```

**Luôn kiểm tra trước khi commit:**
```bash
# Kiểm tra các thông tin bí mật vô tình bị stage
git diff --cached | grep -i "password\|secret\|api_key\|token"
```

**Nếu một thông tin bí mật từng bị commit, hãy thay đổi (rotate) nó.** Xóa dòng đó hoặc viết lại lịch sử là không đủ — hãy coi như nó đã bị lộ ngay khi nó chạm tới remote. Thu hồi và cấp lại khóa trước, sau đó mới xóa nó khỏi lịch sử.

## Bảo mật tính năng AI / LLM

Nếu ứng dụng của bạn gọi một LLM — chatbot, tóm tắt, agent, RAG — nó sẽ thừa hưởng một bề mặt tấn công mới. Đối chiếu với [OWASP Top 10 cho Ứng dụng LLM (2025)](https://genai.owasp.org/llm-top-10/):

- **Coi mọi đầu ra của mô hình là đầu vào không đáng tin cậy (LLM05: Improper Output Handling).** Không bao giờ truyền trực tiếp đầu ra của LLM vào `eval`, SQL, shell, `innerHTML`, hoặc đường dẫn tệp. Xác thực và mã hóa nó chính xác như cách bạn làm với đầu vào thô của người dùng.
- **Giả định rằng các câu lệnh (prompts) có thể bị đánh chiếm (LLM01: Prompt Injection).** Văn bản không đáng tin cậy trong cửa sổ ngữ cảnh — một tin nhắn người dùng, một trang web được truy xuất, một tệp PDF — có thể chứa các hướng dẫn. System prompt không phải là biên bảo mật; hãy thực thi quyền hạn trong mã, không phải trong prompt.
- **Giữ bí mật và dữ liệu của người dùng khác ra khỏi prompts (LLM02 / LLM07).** Bất cứ thứ gì trong ngữ cảnh đều có thể bị lặp lại. Đừng đặt API keys, dữ liệu chéo tenant, hoặc toàn bộ system prompt ở nơi mà mô hình có thể lặp lại.
- **Hạn chế quyền hạn của tool và agent (LLM06: Excessive Agency).** Giới hạn phạm vi của các tool ở mức tối thiểu, yêu cầu xác nhận cho các hành động hủy hoại hoặc không thể đảo ngược, và xác thực mọi đối số của tool.
- **Giới hạn mức tiêu thụ (LLM10: Unbounded Consumption).** Giới hạn tokens, tốc độ yêu cầu và độ sâu của vòng lặp/đệ quy để một đầu vào được thiết kế khéo léo không thể làm tăng chi phí hoặc treo hệ thống.
- **Cô lập dữ liệu truy xuất (LLM08: Vector and Embedding Weaknesses).** Trong RAG, coi vector store là một biên tin cậy: phân vùng embeddings theo từng tenant để một người dùng không thể truy xuất dữ liệu của người khác, và xác thực tài liệu trước khi lập chỉ mục để nội dung bị độc không thể điều hướng câu trả lời.

```typescript
// XẤU: tin tưởng đầu ra mô hình như một câu lệnh hoặc markup
const sql = await llm.generate(`Write SQL for: ${userQuestion}`);
await db.query(sql);                                   // thực thi truy vấn tùy ý
container.innerHTML = await llm.reply(userMessage);   // stored XSS, thông qua mô hình

// TỐT: đầu ra mô hình là dữ liệu — phân tích một cách phòng thủ, sau đó xác thực, rồi mã hóa
let intent;
try {
  intent = CommandSchema.parse(JSON.parse(await llm.replyJson(userMessage)));
} catch {
  throw new ValidationError('đầu ra mô hình không mong muốn'); // JSON.parse hoặc schema thất bại
}
await runAllowlistedAction(intent.action, intent.params);
container.textContent = await llm.reply(userMessage);
```

## Danh sách kiểm tra xem xét bảo mật (Security Review Checklist)

```markdown
### Xác thực (Authentication)
- [ ] Mật khẩu được băm bằng bcrypt/scrypt/argon2 (salt rounds ≥ 12)
- [ ] Session tokens là httpOnly, secure, sameSite
- [ ] Đăng nhập có giới hạn tốc độ (rate limiting)
- [ ] Tokens đặt lại mật khẩu có thời hạn hết hạn

### Ủy quyền (Authorization)
- [ ] Mọi endpoint đều kiểm tra quyền hạn người dùng
- [ ] Người dùng chỉ có thể truy cập tài nguyên của chính họ
- [ ] Các hành động quản trị yêu cầu xác minh vai trò quản trị

### Đầu vào (Input)
- [ ] Mọi đầu vào người dùng được xác thực tại biên
- [ ] Các truy vấn SQL được tham số hóa
- [ la ] Đầu ra HTML được mã hóa/thoát chuỗi (encoded/escaped)
- [ ] Các yêu cầu URL phía máy chủ nằm trong danh sách cho phép (không SSRF tới dịch vụ nội bộ)

### Dữ liệu (Data)
- [ ] Không có thông tin bí mật trong mã hoặc hệ thống quản lý phiên bản
- [ ] Các trường nhạy cảm bị loại bỏ khỏi phản hồi API
- [ ] PII được mã hóa khi lưu trữ (nếu áp dụng)

### Cơ sở hạ tầng (Infrastructure)
- [ ] Cấu hình các header bảo mật (CSP, HSTS, v.v.)
- [ ] CORS bị giới hạn cho các nguồn đã biết
- [ ] Các dependency được kiểm tra lỗ hổng
- [ la ] Thông báo lỗi không làm lộ chi tiết nội bộ

### Chuỗi cung ứng (Supply Chain)
- [ ] Lockfile được commit; CI cài đặt bằng `npm ci`
- [ ] Các dependency mới được xem xét (bảo trì, lượt tải, script postinstall)

### AI / LLM (nếu sử dụng)
- [ ] Đầu ra mô hình được coi là không đáng tin cậy (không eval/SQL/innerHTML/shell)
- [ ] Thông tin bí mật và dữ liệu người dùng khác được giữ ngoài prompts
- [ la ] Quyền hạn tool/agent được giới hạn; các hành động hủy hoại yêu cầu xác nhận
```

## Xem thêm

Để biết chi tiết các danh sách kiểm tra bảo mật và các bước xác minh trước khi commit, hãy xem `references/security-checklist.md`.

## Các lý lẽ biện minh phổ biến

| Lý lẽ biện minh | Thực tế |
|---|---|
| "Đây là công cụ nội bộ, bảo mật không quan trọng" | Các công cụ nội bộ vẫn bị xâm nhập. Kẻ tấn công nhắm vào mắt xích yếu nhất. |
| "Chúng tôi sẽ thêm bảo mật sau" | Việc bổ sung bảo mật sau này khó hơn gấp 10 lần so với việc xây dựng ngay từ đầu. Hãy thêm nó ngay bây giờ. |
| "Không ai cố gắng khai thác điều này" | Các trình quét tự động sẽ tìm thấy nó. Bảo mật bằng cách che giấu (security by obscurity) không phải là bảo mật. |
| "Framework đã xử lý bảo mật rồi" | Framework cung cấp công cụ, không cung cấp sự đảm bảo. Bạn vẫn cần sử dụng chúng một cách chính xác. |
| "Nó chỉ là bản mẫu (prototype)" | Bản mẫu sẽ trở thành sản phẩm chính thức. Hãy xây dựng thói quen bảo mật ngay từ ngày đầu tiên. |
| "Mô hình hóa đe dọa là quá mức ở đây" | Năm phút "tôi sẽ tấn công điều này như thế nào?" ngăn chặn những lỗi thiết kế mà không một biện pháp kiểm soát nào có thể vá được sau này. |
| "Nó chỉ là đầu ra của LLM, chỉ là văn bản thôi" | "Văn bản" đó có thể là một câu lệnh SQL, một thẻ script hoặc một câu lệnh shell. Hãy coi nó như bất kỳ đầu vào không đáng tin cậy nào. |

## Các dấu hiệu nguy hiểm (Red Flags)

- Đầu vào người dùng được truyền trực tiếp vào truy vấn cơ sở dữ liệu, câu lệnh shell hoặc hiển thị HTML
- Thông tin bí mật trong mã nguồn hoặc lịch sử commit
- API endpoints không có kiểm tra xác thực hoặc ủy quyền
- Thiếu cấu hình CORS hoặc sử dụng nguồn wildcard (`*`)
- Không có giới hạn tốc độ cho các endpoint xác thực
- Stack traces hoặc lỗi nội bộ bị lộ cho người dùng
- Dependencies có lỗ hổng nghiêm trọng đã biết
- Máy chủ truy xuất URL do người dùng cung cấp mà không có danh sách cho phép (SSRF)
- Đầu ra LLM/mô hình được truyền vào truy vấn, DOM, shell hoặc `eval`
- Thông tin bí mật, PII hoặc toàn bộ system prompt được đặt trong cửa sổ ngữ cảnh LLM

## Xác minh (Verification)

Sau khi triển khai mã liên quan đến bảo mật:

- [ ] `npm audit` không cho thấy lỗ hổng critical hoặc high
- [ ] Không có thông tin bí mật trong mã nguồn hoặc lịch sử git
- [ ] Mọi đầu vào người dùng được xác thực tại các biên hệ thống
- [ ] Xác thực và ủy quyền được kiểm tra trên mọi endpoint được bảo vệ
- [ ] Các header bảo mật hiện diện trong phản hồi (kiểm tra bằng DevTools trình duyệt)
- [ ] Phản hồi lỗi không làm lộ chi tiết nội bộ
- [ ] Giới hạn tốc độ đang hoạt động trên các endpoint xác thực
- [ ] Các yêu cầu URL phía máy chủ được xác thực với danh sách cho phép (không SSRF)
- [ ] Đầu ra LLM/mô hình được xác thực và mã hóa trước khi sử dụng (nếu có tính năng AI)
