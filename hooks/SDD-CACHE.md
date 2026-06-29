# sdd-cache hook

Bộ nhớ đệm trích dẫn xuyên phiên cho [`source-driven-development`](../skills/source-driven-development/SKILL.md). Bỏ qua các lệnh gọi `WebFetch` dư thừa mà không làm yếu đi cam kết "xác minh với tài liệu hiện tại" của skill.

## Why

`source-driven-development` truy xuất tài liệu chính thức cho mọi quyết định đặc thù của framework. Khi làm việc trên cùng một dự án qua nhiều phiên, việc này đồng nghĩa với việc phải truy xuất cùng một trang lặp đi lặp lại. Việc lưu nội dung thành bộ nhớ cục bộ (local memory) sẽ mâu thuẫn với mục tiêu của skill — tài liệu thay đổi, và một bộ nhớ đệm cũ sẽ che mất điều đó.

Hook này lưu nội dung đã truy xuất lên đĩa, nhưng **xác thực lại với máy chủ nguồn ở mỗi lần tái sử dụng** thông qua HTTP `If-None-Match` / `If-Modified-Since`. Nội dung chỉ được cung cấp từ bộ nhớ đệm khi máy chủ phản hồi `304 Not Modified`, đây là một sự xác minh mới nhất — không phải là đọc từ bộ nhớ.

## Setup

1. Thêm các hook vào `.claude/settings.json` (hoặc `.claude/settings.local.json` để sử dụng cá nhân):

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "WebFetch",
        "hooks": [
          {
            "type": "command",
            "command": "bash ${CLAUDE_PROJECT_DIR}/hooks/sdd-cache-pre.sh",
            "timeout": 10
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "WebFetch",
        "hooks": [
          {
            "type": "command",
            "command": "bash ${CLAUDE_PROJECT_DIR}/hooks/sdd-cache-post.sh",
            "async": true,
            "timeout": 10
          }
        ]
      }
    ]
  }
}
```

   `${CLAUDE_PROJECT_DIR}` phân giải thành thư mục nơi bạn khởi chạy Claude Code. Đoạn mã trên hoạt động khi các hook nằm trong cùng một dự án. Nếu bạn cài đặt `agent-skills` ở nơi khác (ví dụ: như một plugin dùng chung trong `~/agent-skills`), hãy thay thế `${CLAUDE_PROJECT_DIR}/hooks/...` bằng đường dẫn tuyệt đối đến từng script.

2. Đảm bảo `.claude/sdd-cache/` nằm trong `.gitignore` của bạn (đã được bao gồm trong repo này).

3. Sử dụng `/source-driven-development` (hoặc skill) như bình thường. Không có thay đổi nào đối với skill hay quy trình làm việc của agent — bộ nhớ đệm hoạt động trong suốt (transparent).

## Mental model

Bộ nhớ đệm tài nguyên HTTP với khóa (key) là URL. Việc xác định độ mới (freshness) được ủy quyền cho máy chủ nguồn thông qua `ETag` / `Last-Modified`; không có TTL, không có prompt trong khóa.

Nội dung lưu trữ không phải là HTML thô — `WebFetch` hậu xử lý mỗi phản hồi thông qua một model bằng cách sử dụng prompt của người gọi, vì vậy nội dung chúng ta lưu đệm là kết quả đọc trang của một agent. Khóa chỉ bao gồm URL để việc đọc có thể tái sử dụng xuyên phiên; prompt gốc được giữ dưới dạng metadata và hiển thị trong thông báo hit để agent tiếp theo có thể biết liệu kết quả đọc trước đó có phù hợp hay không.

## How it works

Một mục nhập bộ nhớ đệm cho mỗi URL, được lưu dưới dạng JSON trong `.claude/sdd-cache/<sha>.json`:

| Event | Action |
|---|---|
| `PreToolUse WebFetch` | Nếu một mục nhập tồn tại, gửi yêu cầu `HEAD` với `If-None-Match` / `If-Modified-Since`. Nếu nhận `304`, chặn việc truy xuất và trả về nội dung đệm cho agent qua stderr, với prompt gốc được hiển thị dưới dạng metadata. Ngược lại, cho phép truy xuất. |
| `PostToolUse WebFetch` | Ghi lại phản hồi, gửi yêu cầu `HEAD` để ghi lại `ETag` / `Last-Modified` hiện tại, và lưu trữ `{url, prompt, etag, last_modified, content, fetched_at}`. |

**Freshness rules:**

- Mục nhập chỉ được cung cấp nếu máy chủ nguồn xác nhận `304 Not Modified`.
- Các mục nhập không có header `ETag` hoặc `Last-Modified` sẽ không bao giờ được lưu đệm — nếu không có trình xác thực (validator), hook không thể xác minh độ mới sau này, và việc lưu đệm sẽ đồng nghĩa với việc tin tưởng vào bộ nhớ.
- Khóa bộ nhớ đệm là `sha256(url)`. Cùng một URL nhưng yêu cầu với prompt khác sẽ khớp với cùng một mục nhập; nội dung đệm phản ánh prompt được dùng trong lần truy xuất đầu tiên, và prompt đó được hiển thị kèm theo hit để agent có thể quyết định nên tái sử dụng hay tự truy xuất lại thủ công.

**What the agent sees:**

- Cache hit: `WebFetch` bị chặn bằng exit code 2. Claude Code chuyển payload stderr của hook trả về cho agent dưới dạng lỗi công cụ — đây là tín hiệu dự kiến cho một cache hit, không phải là một lỗi. Payload được tiền tố bằng `[sdd-cache] Cache hit for <url>` và bao bọc nội dung đệm giữa các dấu mốc `----- BEGIN CACHED CONTENT -----` / `----- END CACHED CONTENT -----` để agent có thể sử dụng nó như thể `WebFetch` vừa trả về.
- Cache miss hoặc cũ (stale): `WebFetch` chạy bình thường; kết quả được lưu cho lần sau.

Bản thân skill không thay đổi. Nó tiếp tục tuân theo quy trình `DETECT → FETCH → IMPLEMENT → CITE`. Hook chỉ thay đổi những gì diễn ra ngầm khi `FETCH` chạy.

## Local testing

### 1. Chạy smoke test trực tiếp các script

```bash
# Mô phỏng payload PostToolUse: lưu đệm một trang
echo '{
  "tool_input": {
    "url": "https://react.dev/reference/react/useActionState",
    "prompt": "extract the signature"
  },
  "tool_response": "useActionState(action, initialState) returns [state, formAction, isPending]"
}' | bash hooks/sdd-cache-post.sh

# Kiểm tra mục nhập đã lưu
ls .claude/sdd-cache/
cat .claude/sdd-cache/*.json | jq .

# Mô phỏng lần PreToolUse tiếp theo trên cùng URL + prompt
echo '{
  "tool_input": {
    "url": "https://react.dev/reference/react/useActionState",
    "prompt": "extract the signature"
  }
}' | bash hooks/sdd-cache-pre.sh
echo "exit=$?"
```

Kết quả mong đợi:

- Lệnh đầu tiên tạo một tệp trong `.claude/sdd-cache/` (chỉ khi máy chủ trả về `ETag` hoặc `Last-Modified`).
- Lệnh thứ hai thoát với mã `2` cùng với nội dung đệm trên stderr khi máy chủ nguồn phản hồi `304`, hoặc thoát mã `0` một cách im lặng nếu không.

### 2. Chạy end-to-end trong một phiên thực tế

1. Đăng ký các hook trong `.claude/settings.local.json` như đã trình bày ở trên.
2. Bắt đầu một phiên Claude Code trong repo này.
3. Yêu cầu agent truy xuất một trang tài liệu (ví dụ: "fetch `https://react.dev/reference/react/useActionState` and summarize").
4. Xác minh một tệp xuất hiện trong `.claude/sdd-cache/`.
5. Yêu cầu agent truy xuất lại cùng trang đó với cùng prompt một lần nữa.
6. Xác minh rằng lần `WebFetch` thứ hai bị chặn và nội dung đệm được trả về (hiển thị trong bản ghi phiên dưới dạng lỗi công cụ với tiền tố `[sdd-cache]`).

### 3. Xác minh độ mới (Freshness)

Để xác nhận bộ nhớ đệm bị hủy khi tài liệu thay đổi, hãy ép buộc xảy ra sự sai lệch `ETag`. Chọn một mục nhập cụ thể — dùng `*.json` sẽ không an toàn khi bộ nhớ đệm chứa nhiều hơn một tệp:

```bash
# Chọn mục nhập bạn muốn làm sai lệch (thay bằng tên tệp thực tế)
ENTRY=.claude/sdd-cache/e49c9f378670cfbb1d7d871b6dee16d9.json

# Ghi đè ETag của nó thành một giá trị mà máy chủ nguồn sẽ không nhận diện được
jq '.etag = "W/\"stale-etag-forced\""' "$ENTRY" > "$ENTRY.tmp" && mv "$ENTRY.tmp" "$ENTRY"

# Lần PreToolUse tiếp theo sẽ là một miss (máy chủ trả về 200, không phải 304)
echo '{"tool_input":{"url":"...", "prompt":"..."}}' | bash hooks/sdd-cache-pre.sh
echo "exit=$?"   # mong đợi 0 (cho phép truy xuất)
```

### 4. Gỡ lỗi (Debugging)

Cả hai hook đều ghi các sự kiện có mốc thời gian vào `.claude/sdd-cache/.debug.log` khi chế độ debug được bật. Bật bằng một trong hai cách sau:

```bash
# Tùy chọn A: biến môi trường (theo mỗi phiên)
SDD_CACHE_DEBUG=1 claude

# Tùy chọn B: tệp đánh dấu (persistent)
mkdir -p .claude/sdd-cache && touch .claude/sdd-cache/.debug
# …tắt bằng: rm .claude/sdd-cache/.debug
```

Log ghi lại URL, cấu trúc `tool_response` phát hiện được, trạng thái HEAD, và lý do tại sao mỗi lần gọi là hit hoặc miss. Hữu ích khi một cache miss có vẻ bất thường (thông thường là do máy chủ nguồn ngừng cung cấp trình xác thực).

## Known limitations

- **Nội dung (body) phụ thuộc vào prompt.** Một hit trả về kết quả đọc trang của agent trước đó, với prompt gốc được hiển thị để agent hiện tại có thể quyết định xem nó có áp dụng được không. Nếu không, hãy xóa tệp trong `.claude/sdd-cache/` để buộc truy xuất lại.
- **Mỗi lần ghi vào bộ nhớ đệm tốn thêm một yêu cầu HEAD.** Claude Code không cung cấp các response headers mà `WebFetch` đã nhận được, vì vậy post hook phải truy vấn lại máy chủ nguồn để thu thập `ETag` / `Last-Modified`. Thêm một lượt roundtrip cho mỗi lần miss — đây là cái giá để giữ cho đây là một hook thuần túy mà không cần thay đổi core.
- **Các máy chủ không có `ETag` hoặc `Last-Modified` sẽ không bao giờ được lưu đệm.** Hầu hết các trang tài liệu chính thức (react.dev, docs.djangoproject.com, developer.mozilla.org) đều cung cấp trình xác thực. Các trang không có sẽ luôn được truy xuất lại.
- **Một máy chủ hoạt động không đúng có thể trả về `304` sai.** Đó là lỗi của máy chủ cần chẩn đoán, không phải là một bất biến của bộ nhớ đệm cần phòng tránh; chúng tôi không che đậy nó bằng một TTL. Hãy xóa mục nhập nếu bạn phát hiện một nội dung cũ (stale).
- **Bộ nhớ đệm là cục bộ và theo từng dự án.** Không có bộ nhớ đệm chia sẻ cho toàn đội. Việc thêm bộ nhớ đệm chia sẻ sẽ yêu cầu một lớp lưu trữ định địa chỉ nội dung có chữ ký (signed-content-addressable storage layer), điều này nằm ngoài phạm vi của project.

## Yêu cầu

- `jq`
- `curl`
- `shasum` hoặc `sha256sum` (tự động phát hiện)
- Bash 3.2+
