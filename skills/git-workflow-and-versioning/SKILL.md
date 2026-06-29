---
name: git-workflow-and-versioning
description: Xây dựng các thực hành quy trình git. Sử dụng khi thực hiện bất kỳ thay đổi mã nguồn nào. Sử dụng khi commit, phân nhánh, giải quyết xung đột, hoặc khi bạn cần tổ chức công việc trên nhiều luồng song song.
---

# Quy trình Git và Quản lý phiên bản

## Tổng quan

Git là lưới an toàn của bạn. Hãy coi các commit như các điểm lưu (save points), các nhánh (branches) như các môi trường thử nghiệm (sandboxes), và lịch sử như tài liệu hướng dẫn. Với việc các agent AI tạo mã nguồn với tốc độ cao, kiểm soát phiên bản kỷ luật là cơ chế giúp các thay đổi có thể quản lý, xem xét và hoàn tác được.

## Khi nào nên sử dụng

Luôn luôn. Mọi thay đổi mã nguồn đều phải đi qua git.

## Các nguyên tắc cốt lõi

### Phát triển dựa trên nhánh chính (Trunk-Based Development - Khuyên dùng)

Luôn giữ cho nhánh `main` có thể triển khai được. Làm việc trên các nhánh tính năng ngắn hạn và merge trở lại trong vòng 1-3 ngày. Các nhánh phát triển tồn tại lâu là những chi phí ẩn — chúng gây phân kỳ, tạo ra xung đột khi merge và làm chậm quá trình tích hợp. Nghiên cứu của DORA cho thấy phát triển dựa trên nhánh chính (trunk-based development) có liên quan mật thiết đến các đội ngũ kỹ thuật hiệu suất cao.

```
main ──●──●──●──●──●──●──●──●──●──  (luôn có thể triển khai)
        ╲      ╱  ╲    ╱
         ●──●─╱    ●──╱    ← các nhánh tính năng ngắn hạn (1-3 ngày)
```

Đây là mặc định được khuyên dùng. Các đội ngũ sử dụng gitflow hoặc các nhánh tồn tại lâu có thể áp dụng các nguyên tắc (atomic commits, thay đổi nhỏ, thông điệp mô tả chi tiết) vào mô hình phân nhánh của họ — kỷ luật commit quan trọng hơn chiến lược phân nhánh cụ thể.

- **Các nhánh phát triển là chi phí.** Mỗi ngày một nhánh tồn tại, nó tích tụ rủi ro khi merge.
- **Các nhánh release là chấp nhận được.** Khi bạn cần ổn định một bản phát hành trong khi nhánh `main` vẫn tiếp tục tiến về phía trước.
- **Feature flags > các nhánh dài hạn.** Ưu tiên triển khai các công việc chưa hoàn thành đằng sau các flags thay vì giữ chúng trên một nhánh trong nhiều tuần.

### 1. Commit sớm, Commit thường xuyên

Mỗi phần tăng trưởng thành công sẽ có một commit riêng. Đừng tích tụ một lượng lớn các thay đổi chưa được commit.

```
Mô hình làm việc:
  Triển khai một phần (slice) → Test → Xác minh (Verify) → Commit → Phần tiếp theo

Không làm thế này:
  Triển khai tất cả mọi thứ → Hy vọng nó hoạt động → Một commit khổng lồ
```

Các commit là các điểm lưu. Nếu thay đổi tiếp theo làm hỏng điều gì đó, bạn có thể hoàn tác (revert) về trạng thái tốt gần nhất ngay lập tức.

### 2. Atomic Commits

Mỗi commit thực hiện một việc logic duy nhất:

```
# Tốt: Mỗi commit độc lập
git log --oneline
a1b2c3d Add task creation endpoint with validation
d4e5f6g Add task creation form component
h7i8j9k Connect form to API and add loading state
m1n2o3p Add task creation tests (unit + integration)

# Tệ: Mọi thứ bị trộn lẫn với nhau
git log --oneline
x1y2z3a Add task feature, fix sidebar, update deps, refactor utils
```

### 3. Thông điệp mô tả chi tiết

Thông điệp commit giải thích lý do *tại sao* (why), chứ không chỉ là *cái gì* (what):

```
# Tốt: Giải thích ý định
feat: add email validation to registration endpoint

Prevents invalid email formats from reaching the database.
Uses Zod schema validation at the route handler level,
consistent with existing validation patterns in auth.ts.

# Tệ: Mô tả những gì đã hiển nhiên từ diff
update auth.ts
```

**Định dạng:**
```
<type>: <short description>

<optional body explaining why, not what>
```

**Các loại (Types):**
- `feat` — Tính năng mới
- `fix` — Sửa lỗi
- `refactor` — Thay đổi mã nguồn không sửa lỗi cũng không thêm tính năng
- `test` — Thêm hoặc cập nhật tests
- `docs` — Chỉ cập nhật tài liệu
- `chore` — Công cụ, phụ thuộc, cấu hình

### 4. Tách biệt các mối quan tâm (Separate Concerns)

Không kết hợp các thay đổi về định dạng với các thay đổi về hành vi. Không kết hợp refactors với các tính năng. Mỗi loại thay đổi nên là một commit riêng biệt — và lý tưởng nhất là một PR riêng biệt:

```
# Tốt: Tách biệt các mối quan tâm
git commit -m "refactor: extract validation logic to shared utility"
git commit -m "feat: add phone number validation to registration"

# Tệ: Trộn lẫn các mối quan tâm
git commit -m "refactor validation and add phone number field"
```

**Tách biệt refactoring khỏi công việc phát triển tính năng.** Một thay đổi refactoring và một thay đổi tính năng là hai thay đổi khác nhau — hãy gửi chúng riêng biệt. Điều này giúp mỗi thay đổi dễ dàng review, revert và dễ hiểu hơn trong lịch sử. Những dọn dẹp nhỏ (như đổi tên biến) có thể được gộp vào commit tính năng tùy theo quyết định của người review.

### 5. Kiểm soát kích thước thay đổi

Mục tiêu khoảng ~100 dòng cho mỗi commit/PR. Các thay đổi trên ~1000 dòng nên được chia nhỏ. Xem các chiến lược chia nhỏ trong `code-review-and-quality` để biết cách chia nhỏ các thay đổi lớn.

```
~100 dòng  → Dễ review, dễ revert
~300 dòng  → Chấp nhận được cho một thay đổi logic duy nhất
~1000 dòng → Chia nhỏ thành các thay đổi ít hơn
```

## Chiến lược phân nhánh

### Nhánh tính năng (Feature Branches)

```
main (luôn có thể triển khai)
  │
  ├── feature/task-creation    ← Một tính năng cho mỗi nhánh
  ├── feature/user-settings    ← Công việc song song
  └── fix/duplicate-tasks      ← Sửa lỗi
```

- Phân nhánh từ `main` (hoặc nhánh mặc định của đội ngũ)
- Giữ các nhánh ngắn hạn (merge trong vòng 1-3 ngày) — các nhánh tồn tại lâu là chi phí ẩn
- Xóa nhánh sau khi merge
- Ưu tiên sử dụng feature flags thay vì các nhánh dài hạn cho các tính năng chưa hoàn tất

### Đặt tên nhánh

```
feature/<short-description>   → feature/task-creation
fix/<short-description>       → fix/duplicate-tasks
chore/<short-description>     → chore/update-deps
refactor/<short-description>  → refactor/auth-module
```

## Làm việc với Worktrees

Đối với công việc song song của các agent AI, hãy sử dụng git worktrees để chạy nhiều nhánh cùng lúc:

```bash
# Tạo một worktree cho nhánh tính năng
git worktree add ../project-feature-a feature/task-creation
git worktree add ../project-feature-b feature/user-settings

# Mỗi worktree là một thư mục riêng biệt với nhánh riêng
# Các agent có thể làm việc song song mà không gây can thiệp lẫn nhau
ls ../
  project/              ← nhánh main
  project-feature-a/    ← nhánh task-creation
  project-feature-b/    ← nhánh user-settings

# Khi hoàn tất, merge và dọn dẹp
git worktree remove ../project-feature-a
```

Lợi ích:
- Nhiều agent có thể làm việc trên các tính năng khác nhau đồng thời
- Không cần chuyển đổi nhánh (mỗi thư mục có nhánh riêng)
- Nếu một thử nghiệm thất bại, chỉ cần xóa worktree — không có gì bị mất
- Các thay đổi được cô lập cho đến khi được merge rõ ràng

## Mô hình Save Point

```
Agent bắt đầu làm việc
    │
    ├── Thực hiện một thay đổi
    │   ├── Test vượt qua? → Commit → Tiếp tục
    │   └── Test thất bại? → Revert về commit cuối cùng → Điều tra
    │
    ├── Thực hiện thay đổi khác
    │   ├── Test vượt qua? → Commit → Tiếp tục
    │   └── Test thất bại? → Revert về commit cuối cùng → Điều tra
    │
    └── Tính năng hoàn tất → Tất cả các commit tạo nên một lịch sử sạch
```

Mô hình này có nghĩa là bạn không bao giờ mất quá một phần công việc. Nếu một agent đi chệch hướng, `git reset --hard HEAD` sẽ đưa bạn trở lại trạng thái thành công cuối cùng.

## Tóm tắt thay đổi

Sau bất kỳ thay đổi nào, hãy cung cấp một bản tóm tắt có cấu trúc. Điều này giúp việc review dễ dàng hơn, ghi chép lại kỷ luật về phạm vi (scope), và làm lộ ra các thay đổi không mong muốn:

```
CÁC THAY ĐỔI ĐÃ THỰC HIỆN:
- src/routes/tasks.ts: Added validation middleware to POST endpoint
- src/lib/validation.ts: Added TaskCreateSchema using Zod

NHỮNG THỨ TÔI KHÔNG CHẠM VÀO (có chủ đích):
- src/routes/auth.ts: Has similar validation gap but out of scope
- src/middleware/error.ts: Error format could be improved (separate task)

CÁC VẤN ĐỀ TIỀM NĂNG:
- The Zod schema is strict — rejects extra fields. Confirm this is desired.
- Added zod as a dependency (72KB gzipped) — already in package.json
```

Mô hình này giúp phát hiện sớm các giả định sai và cung cấp cho người review một bản đồ rõ ràng về thay đổi. Phần "NHỮNG THỨ TÔI KHÔNG CHẠM VÀO" đặc biệt quan trọng — nó cho thấy bạn đã tuân thủ kỷ luật về phạm vi và không tự ý thực hiện những cải tiến không được yêu cầu.

## Vệ sinh trước khi Commit

Trước mỗi commit:

```bash
# 1. Kiểm tra những gì bạn sắp commit
git diff --staged

# 2. Đảm bảo không có bí mật (secrets)
git diff --staged | grep -i "password\|secret\|api_key\|token"

# 3. Chạy tests
npm test

# 4. Chạy linting
npm run lint

# 5. Chạy kiểm tra kiểu (type checking)
npx tsc --noEmit
```

Tự động hóa việc này với các git hooks:

```json
// package.json (sử dụng lint-staged + husky)
{
  "lint-staged": {
    "*.{ts,tsx}": ["eslint --fix", "prettier --write"],
    "*.{json,md}": ["prettier --write"]
  }
}
```

## Xử lý các file được tạo tự động

- **Chỉ commit các file được tạo tự động** nếu dự án yêu cầu (ví dụ: `package-lock.json`, Prisma migrations)
- **Không commit** kết quả build (`dist/`, `.next/`), các file môi trường (`.env`), hoặc cấu hình IDE (`.vscode/settings.json` trừ khi được chia sẻ)
- **Hãy có một file `.gitignore`** bao gồm: `node_modules/`, `dist/`, `.env`, `.env.local`, `*.pem`

## Sử dụng Git để gỡ lỗi (Debugging)

```bash
# Tìm commit nào đã gây ra lỗi
git bisect start
git bisect bad HEAD
git bisect good <known-good-commit>
# Git sẽ checkout các điểm giữa; chạy test tại mỗi điểm để thu hẹp phạm vi

# Xem những thay đổi gần đây
git log --oneline -20
git diff HEAD~5..HEAD -- src/

# Tìm xem ai là người cuối cùng thay đổi một dòng cụ thể
git blame src/services/task.ts

# Tìm kiếm từ khóa trong các thông điệp commit
git log --grep="validation" --oneline
```

## Các lý lẽ phổ biến

| Lý lẽ | Thực tế |
|---|---|
| "Tôi sẽ commit khi tính năng hoàn tất" | Một commit khổng lồ là không thể review, gỡ lỗi hoặc revert. Hãy commit theo từng phần (slice). |
| "Thông điệp không quan trọng" | Thông điệp là tài liệu. Bạn trong tương lai (và các agent tương lai) sẽ cần hiểu cái gì đã thay đổi và tại sao. |
| "Tôi sẽ squash tất cả sau" | Squashing phá hủy trình tự phát triển. Hãy ưu tiên các commit tăng dần sạch sẽ ngay từ đầu. |
| "Phân nhánh gây thêm phiền phức" | Các nhánh ngắn hạn không tốn kém và ngăn chặn các công việc xung đột va chạm nhau. Các nhánh dài hạn mới là vấn đề — hãy merge trong vòng 1-3 ngày. |
| "Tôi sẽ chia nhỏ thay đổi này sau" | Thay đổi lớn khó review hơn, rủi ro hơn khi triển khai và khó revert hơn. Hãy chia nhỏ trước khi gửi, không phải sau đó. |
| "Tôi không cần .gitignore" | Cho đến khi `.env` chứa bí mật production bị commit. Hãy thiết lập nó ngay lập tức. |

## Các dấu hiệu cảnh báo (Red Flags)

- Tích tụ lượng lớn thay đổi chưa được commit
- Thông điệp commit kiểu "fix", "update", "misc"
- Thay đổi định dạng bị trộn lẫn với thay đổi hành vi
- Không có `.gitignore` trong dự án
- Commit `node_modules/`, `.env`, hoặc các sản phẩm build
- Các nhánh tồn tại lâu bị phân kỳ đáng kể so với main
- Force-push vào các nhánh dùng chung

## Xác minh

Đối với mỗi commit:

- [ ] Commit thực hiện một việc logic duy nhất
- [ ] Thông điệp giải thích lý do tại sao, tuân thủ các quy ước về loại (type)
- [ ] Tests vượt qua trước khi commit
- [ ] Không có bí mật (secrets) trong diff
- [ ] Không có thay đổi chỉ về định dạng bị trộn lẫn với thay đổi hành vi
- [ ] `.gitignore` bao gồm các loại trừ tiêu chuẩn
