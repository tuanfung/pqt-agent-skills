# hook simplify-ignore

Bảo vệ cấp độ block cho `/code-simplify`. Đánh dấu những đoạn mã không bao giờ nên được đơn giản hóa — model sẽ không nhìn thấy chúng.

## Cài đặt

1. Chú thích các block bạn muốn bảo vệ:

```js
/* simplify-ignore-start: perf-critical */
// manually unrolled XOR — 3x faster than a loop
result[0] = buf[0] ^ key[0];
result[1] = buf[1] ^ key[1];
result[2] = buf[2] ^ key[2];
result[3] = buf[3] ^ key[3];
/* simplify-ignore-end */
```

2. Thêm các hook vào `.claude/settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Read",
        "hooks": [{ "type": "command", "command": "bash ${CLAUDE_PROJECT_DIR}/hooks/simplify-ignore.sh" }]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [{ "type": "command", "command": "bash ${CLAUDE_PROJECT_DIR}/hooks/simplify-ignore.sh" }]
      }
    ],
    "Stop": [
      {
        "hooks": [{ "type": "command", "command": "bash ${CLAUDE_PROJECT_DIR}/hooks/simplify-ignore.sh" }]
      }
    ]
  }
}
```

3. Chạy `/code-simplify` — các block được bảo vệ sẽ trở thành các placeholder dạng `/* BLOCK_de115a1d: perf-critical */`. Model sẽ suy luận về mã xung quanh mà không nhìn thấy phần triển khai được bảo vệ.

> **Lưu ý:** Hook này lưu các bản sao lưu tạm thời trong `.claude/.simplify-ignore-cache/`. Hãy đảm bảo đường dẫn này nằm trong `.gitignore` của bạn.

## Cách hoạt động

Một script, ba sự kiện hook:

| Sự kiện | Hành động |
|---|---|
| `PreToolUse Read` | Sao lưu file, thay thế các block bằng các placeholder `BLOCK_<hash>` trực tiếp tại chỗ |
| `PostToolUse Edit\|Write` | Mở rộng các placeholder trở lại thành mã thực, lưu các thay đổi của model, và lọc lại |
| `Stop` | Khôi phục tất cả các file từ bản sao lưu khi session kết thúc |

Mỗi block được hash nội dung (8 ký tự hex thông qua `shasum`/`sha1sum`) để quá trình khứ hồi là không mơ hồ ngay cả khi model nhân bản hoặc sắp xếp lại các placeholder. Cache được giới hạn trong phạm vi project để ngăn chặn sự can thiệp giữa các session.

## Cú pháp chú thích

```js
/* simplify-ignore-start */           // cơ bản — ẩn block
/* simplify-ignore-start: reason */   // với lý do — xuất hiện trong placeholder
/* simplify-ignore-end */
```

Mọi kiểu comment đều hoạt động (`//`, `/*`, `#`, `<!--`). Hỗ trợ nhiều block trong mỗi file và các block một dòng. Các placeholder bảo toàn cú pháp comment gốc (ví dụ: `# BLOCK_xxx` cho Python, `<!-- BLOCK_xxx -->` cho HTML).

## Khôi phục khi bị crash

Nếu Claude Code bị crash mà không kích hoạt hook Stop, các file trên đĩa có thể vẫn còn các placeholder `BLOCK_<hash>`. Để khôi phục thủ công:

```bash
echo '{}' | bash hooks/simplify-ignore.sh
```

Các bản sao lưu được lưu trữ trong `.claude/.simplify-ignore-cache/` bên trong thư mục project của bạn.

## Các hạn chế đã biết

- **Các block một dòng sẽ ẩn toàn bộ dòng đó.** Nếu `simplify-ignore-start` và `simplify-ignore-end` xuất hiện trên cùng một dòng với mã khác, toàn bộ dòng đó sẽ bị ẩn đối với model, không chỉ phần được chú thích. Hãy sử dụng các dòng riêng biệt cho các chú thích.
- **Việc phát hiện hậu tố comment chỉ hỗ trợ `*/` và `-->`.** Các template engine có trình đóng comment không tiêu chuẩn (ERB `%>`, Blade `--}}`) có thể tạo ra các placeholder không cân đối. Hãy sử dụng comment kiểu `#` hoặc `//` thay thế.
- **Việc mở rộng dự phòng là lũy tiến, không chính xác tuyệt đối.** Nếu model thay đổi định dạng của placeholder (ví dụ: thay đổi văn bản lý do), hook sẽ thử các khớp nối đơn giản dần: full placeholder → prefix+hash+suffix → chỉ hash. Phương án dự phòng chỉ hash có thể để lại các mảnh vụn về mặt thẩm mỹ (ví dụ: dấu `:` lạc lối hoặc văn bản lý do). Một cảnh báo sẽ được in ra stderr khi điều này xảy ra.
- **Đổi tên file sẽ để lại các placeholder.** Nếu model đổi tên hoặc di chuyển một file thông qua lệnh shell, file mới sẽ vẫn giữ các placeholder `BLOCK_<hash>`. Mã gốc được lưu dưới dạng `<old-filename>.recovered` khi session kết thúc. Bạn phải khôi phục thủ công mã đã cứu vãn vào file mới.

## Yêu cầu

- `jq`, `shasum` hoặc `sha1sum` (tự động phát hiện), Bash 3.2+
