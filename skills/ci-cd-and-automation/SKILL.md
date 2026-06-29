---
name: ci-cd-and-automation
description: Tự động hóa thiết lập pipeline CI/CD. Sử dụng khi thiết lập hoặc sửa đổi pipeline build và deployment. Sử dụng khi bạn cần tự động hóa quality gates, cấu hình test runners trong CI, hoặc thiết lập các chiến lược deployment.
---

# CI/CD và Tự động hóa

## Tổng quan

Tự động hóa quality gates để đảm bảo không có thay đổi nào đến được production mà không vượt qua các bài test, lint, type checking, và build. CI/CD là cơ chế thực thi cho mọi skill khác — nó phát hiện những điều mà con người và agent bỏ lỡ, và thực hiện điều đó một cách nhất quán trên mọi thay đổi.

**Shift Left:** Phát hiện vấn đề càng sớm càng tốt trong pipeline. Một bug được phát hiện khi linting chỉ tốn vài phút; cùng bug đó nếu phát hiện ở production sẽ tốn hàng giờ. Di chuyển các bước kiểm tra lên thượng nguồn — static analysis trước test, test trước staging, staging trước production.

**Nhanh hơn là An toàn hơn:** Các batch nhỏ hơn và phát hành thường xuyên hơn làm giảm rủi ro, chứ không tăng thêm. Một bản deployment với 3 thay đổi sẽ dễ debug hơn bản có 30 thay đổi. Phát hành thường xuyên giúp xây dựng niềm tin vào chính quy trình release.

## Khi nào nên sử dụng

- Thiết lập pipeline CI cho dự án mới
- Thêm hoặc sửa đổi các bước kiểm tra tự động
- Cấu hình pipeline deployment
- Khi một thay đổi cần kích hoạt xác minh tự động
- Debug các lỗi CI

## Pipeline Quality Gate

Mọi thay đổi đều phải đi qua các gate này trước khi merge:

```
Pull Request Opened
    │
    ▼
┌─────────────────┐
│   KIỂM TRA LINT  │  eslint, prettier
│   ↓ pass         │
│   KIỂM TRA TYPE  │  tsc --noEmit
│   ↓ pass         │
│   UNIT TEST      │  jest/vitest
│   ↓ pass         │
│   BUILD          │  npm run build
│   ↓ pass         │
│   INTEGRATION    │  API/DB tests
│   ↓ pass         │
│   E2E (tùy chọn)  │  Playwright/Cypress
│   ↓ pass         │
│   KIỂM TRA BẢO MẬT│  npm audit
│   ↓ pass         │
│   KIỂM TRA BUNDLE SIZE
└─────────────────┘
    │
    ▼
  Sẵn sàng để review
```

**Không được bỏ qua bất kỳ gate nào.** Nếu lint thất bại, hãy sửa lint — đừng tắt rule. Nếu test thất bại, hãy sửa code — đừng bỏ qua test.

## Cấu hình GitHub Actions

### Pipeline CI Cơ bản

```yaml
# .github/workflows/ci.yml
name: CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Type check
        run: npx tsc --noEmit

      - name: Test
        run: npm test -- --coverage

      - name: Build
        run: npm run build

      - name: Security audit
        run: npm audit --audit-level=high
```

### Với Integration Test cho Cơ sở dữ liệu

```yaml
  integration:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_DB: testdb
          POSTGRES_USER: ci_user
          POSTGRES_PASSWORD: ${{ secrets.CI_DB_PASSWORD }}
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'
      - run: npm ci
      - name: Run migrations
        run: npx prisma migrate deploy
        env:
          DATABASE_URL: postgresql://ci_user:${{ secrets.CI_DB_PASSWORD }}@localhost:5432/testdb
      - name: Integration tests
        run: npm run test:integration
        env:
          DATABASE_URL: postgresql://ci_user:${{ secrets.CI_DB_PASSWORD }}@localhost:5432/testdb
```

> **Lưu ý:** Ngay cả đối với các cơ sở dữ liệu test chỉ dùng cho CI, hãy sử dụng GitHub Secrets cho thông tin xác thực thay vì hardcode giá trị. Điều này xây dựng thói quen tốt và ngăn chặn việc vô tình tái sử dụng thông tin xác thực của môi trường test trong các ngữ cảnh khác.

### E2E Test

```yaml
  e2e:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'
      - run: npm ci
      - name: Install Playwright
        run: npx playwright install --with-deps chromium
      - name: Build
        run: npm run build
      - name: Run E2E tests
        run: npx playwright test
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/
```

## Phản hồi lỗi CI cho Agent

Sức mạnh của CI khi kết hợp với AI agent là vòng lặp phản hồi. Khi CI thất bại:

```
CI thất bại
    │
    ▼
Sao chép đầu ra lỗi
    │
    ▼
Gửi nó cho agent:
"The CI pipeline failed with this error:
[paste specific error]
Fix the issue and verify locally before pushing again."
    │
    ▼
Agent sửa → push → CI chạy lại
```

**Các pattern chính:**

```
Lỗi lint → Agent chạy `npm run lint --fix` và commit
Lỗi type  → Agent đọc vị trí lỗi và sửa type
Lỗi test → Agent tuân theo skill debugging-and-error-recovery
Lỗi build → Agent kiểm tra config và dependencies
```

## Chiến lược Deployment

### Preview Deployment

Mỗi PR sẽ có một bản preview deployment để test thủ công:

```yaml
# Deploy preview on PR (Vercel/Netlify/etc.)
deploy-preview:
  runs-on: ubuntu-latest
  if: github.event_name == 'pull_request'
  steps:
    - uses: actions/checkout@v4
    - name: Deploy preview
      run: npx vercel --token=${{ secrets.VERCEL_TOKEN }}
```

### Feature Flags

Feature flags tách biệt deployment khỏi release. Deploy các tính năng chưa hoàn thiện hoặc rủi ro đằng sau các flag để bạn có thể:

- **Ship code mà không cần kích hoạt.** Merge vào main sớm, kích hoạt khi sẵn sàng.
- **Roll back mà không cần redeploy.** Tắt flag thay vì revert code.
- **Canary các tính năng mới.** Kích hoạt cho 1% người dùng, sau đó 10%, rồi 100%.
- **Chạy A/B test.** So sánh hành vi khi có và không có tính năng.

```typescript
// Simple feature flag pattern
if (featureFlags.isEnabled('new-checkout-flow', { userId })) {
  return renderNewCheckout();
}
return renderLegacyCheckout();
```

**Vòng đời của flag:** Tạo → Kích hoạt để test → Canary → Rollout toàn bộ → Xóa flag và code thừa. Những flag tồn tại vĩnh viễn sẽ trở thành nợ kỹ thuật (technical debt) — hãy đặt ngày dọn dẹp khi bạn tạo chúng.

### Staged Rollouts

```
PR đã merge vào main
    │
    ▼
  Staging deployment (tự động)
    │ Xác minh thủ công
    ▼
  Production deployment (kích hoạt thủ công hoặc tự động sau staging)
    │
    ▼
  Theo dõi lỗi (cửa sổ 15 phút)
    │
    ├── Phát hiện lỗi → Rollback
    └── Sạch → Hoàn tất
```

### Kế hoạch Rollback

Mọi bản deployment đều phải có khả năng đảo ngược (reversible):

```yaml
# Workflow rollback thủ công
name: Rollback
on:
  workflow_dispatch:
    inputs:
      version:
        description: 'Version to rollback to'
        required: true

jobs:
  rollback:
    runs-on: ubuntu-latest
    steps:
      - name: Rollback deployment
        run: |
          # Deploy phiên bản trước đó được chỉ định
          npx vercel rollback ${{ inputs.version }}
```

## Quản lý Môi trường

```
.env.example       → Được commit (template cho lập trình viên)
.env                → KHÔNG commit (phát triển cục bộ)
.env.test           → Được commit (môi trường test, không chứa secret thật)
CI secrets          → Lưu trong GitHub Secrets / vault
Production secrets  → Lưu trong nền tảng deployment / vault
```

CI không bao giờ được phép có production secrets. Hãy sử dụng các secret riêng biệt cho việc test CI.

## Tự động hóa ngoài CI

### Dependabot / Renovate

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: npm
    directory: /
    schedule:
      interval: weekly
    open-pull-requests-limit: 5
```

### Vai trò Build Cop

Chỉ định một người chịu trách nhiệm giữ cho CI luôn 'xanh'. Khi build bị hỏng, nhiệm vụ của Build Cop là sửa hoặc revert — không phải người gây ra lỗi. Điều này ngăn chặn việc tích tụ các bản build hỏng trong khi mọi người đều cho rằng người khác sẽ sửa.

### Kiểm tra PR

- **Required reviews:** Ít nhất 1 approval trước khi merge
- **Required status checks:** CI phải pass trước khi merge
- **Branch protection:** Không cho phép force-pushes vào main
- **Auto-merge:** Nếu tất cả các kiểm tra pass và được approve, tự động merge

## Tối ưu hóa CI

Khi pipeline vượt quá 10 phút, hãy áp dụng các chiến lược sau theo thứ tự mức độ ảnh hưởng:

```
Pipeline CI chậm?
├── Cache dependencies
│   └── Sử dụng actions/cache hoặc tùy chọn cache của setup-node cho node_modules
├── Chạy các job song song
│   └── Chia lint, typecheck, test, build thành các job song song riêng biệt
├── Chỉ chạy những gì thay đổi
│   └── Sử dụng path filters để bỏ qua các job không liên quan (ví dụ: bỏ qua e2e cho các PR chỉ thay đổi docs)
├── Sử dụng matrix builds
│   └── Shard các bộ test trên nhiều runner
├── Tối ưu hóa bộ test
│   └── Loại bỏ các test chậm khỏi critical path, thay vào đó hãy chạy chúng theo lịch trình
└── Sử dụng runner lớn hơn
    └── Sử dụng larger runners do GitHub cung cấp hoặc self-hosted cho các bản build nặng về CPU
```

**Ví dụ: caching và song song hóa**
```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '22', cache: 'npm' }
      - run: npm ci
      - run: npm run lint

  typecheck:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '22', cache: 'npm' }
      - run: npm ci
      - run: npx tsc --noEmit

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '22', cache: 'npm' }
      - run: npm ci
      - run: npm test -- --coverage
```

## Các lập luận phổ biến

| Lập luận | Thực tế |
|---|---|
| "CI quá chậm" | Hãy tối ưu hóa pipeline (xem phần Tối ưu hóa CI bên dưới), đừng bỏ qua nó. Một pipeline 5 phút ngăn chặn hàng giờ debug. |
| "Thay đổi này là tầm thường, bỏ qua CI đi" | Những thay đổi tầm thường vẫn có thể làm hỏng build. Dù sao thì CI cũng chạy nhanh với những thay đổi nhỏ. |
| "Test bị flaky, cứ chạy lại là được" | Flaky tests che giấu các bug thật và lãng phí thời gian của mọi người. Hãy sửa lỗi flaky. |
| "Chúng ta sẽ thêm CI sau" | Các dự án không có CI sẽ tích tụ các trạng thái hỏng. Hãy thiết lập nó ngay từ ngày đầu tiên. |
| "Test thủ công là đủ rồi" | Test thủ công không có khả năng mở rộng và không thể lặp lại. Hãy tự động hóa những gì có thể. |

## Dấu hiệu cảnh báo (Red Flags)

- Không có pipeline CI trong dự án
- Lỗi CI bị phớt lờ hoặc tắt thông báo
- Tắt các test trong CI để pipeline pass
- Deploy production mà không có xác minh staging
- Không có cơ chế rollback
- Secrets được lưu trong code hoặc file cấu hình CI (không phải trong secrets manager)
- Thời gian CI dài mà không có nỗ lực tối ưu hóa

## Xác minh

Sau khi thiết lập hoặc sửa đổi CI:

- [ ] Tất cả các quality gates đều hiện diện (lint, types, tests, build, audit)
- [ ] Pipeline chạy trên mọi PR và push vào main
- [ ] Lỗi block việc merge (đã cấu hình branch protection)
- [ ] Kết quả CI được phản hồi ngược lại vào vòng lặp phát triển
- [ ] Secrets được lưu trong secrets manager, không phải trong code
- [ ] Deployment có cơ chế rollback
- [ ] Pipeline chạy trong dưới 10 phút cho bộ test
