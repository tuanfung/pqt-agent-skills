# Đóng góp cho Agent Skills

Cảm ơn bạn đã quan tâm đến việc đóng góp! Dự án này là một bộ sưu tập các kỹ năng kỹ thuật cấp độ production dành cho các AI coding agent.

## Thêm một Skill mới

### Trước khi đề xuất một skill mới

Bộ kỹ năng này đã bao quát hầu hết vòng đời phát triển, và nhiều đề xuất thường trùng lặp với một skill hiện có hoặc một PR đang mở khác. Trước khi mở PR, hãy thực hiện các kiểm tra sau để người review không phải phân loại các nội dung trùng lặp:

1. **Tìm kiếm trong danh mục.** Duyệt [danh sách skill trong README](README.md) và xem qua `skills/` để tìm một skill hiện có bao quát ý tưởng của bạn, toàn bộ hoặc một phần.
2. **Kiểm tra các PR đang mở.** Chạy `gh pr list --state open` (hoặc duyệt tab PRs) và tìm kiếm các đề xuất về cùng chủ đề. Đã có những cụm skill gần như trùng lặp; đừng thêm vào những cụm này.
3. **Đọc về cấu trúc.** Xác nhận ý tưởng của bạn phù hợp với định dạng trong [docs/skill-anatomy.md](docs/skill-anatomy.md), là một workflow có thể thực hiện được kèm theo xác minh, chứ không phải những lời khuyên mơ hồ.
4. **Giải thích khoảng trống trong mô tả PR.** Nêu rõ lý do tại sao nội dung này không được bao quát bởi một skill hiện có hoặc PR đang mở. Nếu có sự trùng lặp, hãy đề xuất mở rộng skill hiện có thay vì thêm một skill mới.

Nếu ý tưởng của bạn là sự cải thiện cho một skill hiện có, hãy ưu tiên chỉnh sửa tập trung vào skill đó thay vì tạo một thư mục mới.

### Tạo skill

1. Tạo một thư mục trong `skills/` với tên theo định dạng kebab-case
2. Thêm file `SKILL.md` theo định dạng trong [docs/skill-anatomy.md](docs/skill-anatomy.md)
3. Bao gồm YAML frontmatter với các trường `name` và `description`
4. Đảm bảo `description` bắt đầu bằng việc skill đó làm gì (ngôi thứ ba), sau đó bao gồm một hoặc nhiều điều kiện kích hoạt `Use when`

### Tiêu chuẩn chất lượng Skill

Các skill cần phải:

- **Cụ thể** — Các bước có thể thực hiện được, không phải lời khuyên mơ hồ
- **Có thể xác minh** — Tiêu chí kết thúc rõ ràng với các yêu cầu về minh chứng
- **Đã qua thực chiến** — Dựa trên các workflow kỹ thuật thực tế, không phải các lý tưởng lý thuyết
- **Tối giản** — Chỉ bao gồm nội dung cần thiết để hướng dẫn agent một cách chính xác

### Cấu trúc

Mọi skill mới phải có:

- `SKILL.md` trong thư mục skill
- YAML frontmatter với `name` và `description` hợp lệ

Các skill mới nhìn chung nên tuân theo cấu trúc tiêu chuẩn:

- **Overview** — Skill này làm gì và tại sao nó quan trọng
- **When to Use** — Các điều kiện kích hoạt
- **Process** — Workflow từng bước một
- **Common Rationalizations** — Những lý do agent thường dùng để bỏ qua các bước, kèm theo lời phản biện
- **Red Flags** — Các dấu hiệu cảnh báo rằng skill đang bị áp dụng sai
- **Verification** — Cách xác nhận skill đã được áp dụng chính xác

Các trường frontmatter trên là bắt buộc. Cấu trúc các phần là một mô hình khuyến nghị: các tiêu đề tương đương như `How It Works`, `Workflow`, hoặc `Core Process` đều được chấp nhận nếu chúng giữ nguyên mục đích và giúp skill dễ theo dõi.

### Những điều không nên làm

- Không trùng lặp nội dung giữa các skill — hãy tham chiếu đến các skill khác
- Không thêm các skill chỉ đưa ra lời khuyên mơ hồ thay vì các quy trình có thể thực hiện được
- Không tạo các file hỗ trợ trừ khi nội dung vượt quá 100 dòng
- Không tạo thư mục `scripts/` trống chỉ để cho giống với skill khác — chỉ thêm `scripts/` khi skill bao gồm các helper có thể chạy được
- Không đặt tài liệu tham khảo bên trong các thư mục skill — hãy sử dụng `references/`

## Chỉnh sửa các Skill hiện có

- Giữ các thay đổi tập trung và tối giản
- Giữ nguyên cấu trúc và văn phong hiện có
- Kiểm tra để đảm bảo YAML frontmatter vẫn hợp lệ sau khi chỉnh sửa

## Kiểm tra Hooks

Session-start hook (`hooks/session-start.sh`) nạp meta-skill `using-agent-skills` vào mỗi phiên Claude Code mới. Một regression test tại `hooks/session-start-test.sh` sẽ xác thực JSON payload của hook — cả khi có `jq` và khi không có.

Hãy chạy nó trước khi mở bất kỳ PR nào tác động đến:

- `hooks/session-start.sh`
- `skills/using-agent-skills/SKILL.md` (nội dung meta-skill được nhúng bởi hook)

```bash
bash hooks/session-start-test.sh
```

Kết quả mong đợi: `session-start JSON payload OK`. Script sẽ thoát với mã khác 0 nếu có bất kỳ lỗi assertion nào.

### Tái hiện trường hợp không có jq (fallback)

Hook sẽ tự động chuyển sang payload ưu tiên `INFO` khi `jq` không nằm trong `PATH`. Để kiểm tra nhánh này tại local, hãy loại bỏ thư mục của `jq` khỏi `PATH` khi gọi test:

```bash
JQ_DIR=$(dirname "$(command -v jq)")
PATH=$(echo "$PATH" | tr ':' '\n' | grep -v "^${JQ_DIR}$" | tr '\n' ':' | sed 's/:$//') \
  bash hooks/session-start-test.sh
```

Cách này hoạt động tốt khi `jq` nằm trong thư mục riêng (ví dụ: `/opt/homebrew/bin` từ Homebrew, `/usr/local/bin` từ cài đặt thủ công). Nếu `jq` dùng chung thư mục bin hệ thống với các công cụ khác mà test phụ thuộc vào (như `mktemp` trong `/usr/bin`), cách đơn giản hơn là cài đặt `jq` qua một trình quản lý gói riêng để nó có thư mục bin riêng, sau đó chạy lại.

Kiểm tra `command -v jq` của hook sẽ thất bại khi `PATH` bị loại bỏ, fallback ưu tiên `INFO` sẽ chạy, và test sẽ xác thực thông báo hướng dẫn `jq is required` thay vì payload thông thường.

## Báo cáo vấn đề

Hãy mở một issue nếu bạn phát hiện:

- Một skill đưa ra hướng dẫn sai hoặc lỗi thời
- Thiếu bao quát cho một workflow kỹ thuật phổ biến
- Sự mâu thuẫn giữa các skill

## License

Bằng cách đóng góp, bạn đồng ý rằng các đóng góp của mình sẽ được cấp phép theo MIT License.
