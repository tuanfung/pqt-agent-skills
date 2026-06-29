# Định nghĩa Hoàn thành (Definition of Done)

Một tiêu chuẩn chung cho toàn bộ dự án mà mọi thay đổi đều phải vượt qua trước khi được coi là hoàn thành. Khác với tiêu chí chấp nhận (acceptance criteria), vốn thay đổi tùy theo từng tác vụ và trả lời cho câu hỏi "chúng ta có xây dựng đúng thứ cần thiết không?", Định nghĩa Hoàn thành là nhất quán trong mọi trường hợp và trả lời cho câu hỏi "liệu thứ này đã hoàn thiện theo tiêu chuẩn của chúng ta chưa?". Hãy sử dụng nó như một chốt chặn cuối cùng trong `planning-and-task-breakdown`, `incremental-implementation`, và `shipping-and-launch`.

## Định nghĩa Hoàn thành vs. Tiêu chí Chấp nhận (Acceptance Criteria)

| | Tiêu chí Chấp nhận (Acceptance Criteria) | Định nghĩa Hoàn thành (Definition of Done) |
|---|---|---|
| Phạm vi | Cụ thể cho một tác vụ hoặc đặc tả | Áp dụng cho mọi lần tăng trưởng (increment) |
| Thay đổi | Khác nhau cho mỗi hạng mục | Cố định và được tái sử dụng |
| Trả lời | "Chúng ta đã xây dựng *thứ này* chưa?" | "Nó đã *sẵn sàng* chưa?" |
| Người sở hữu | Được xác định khi lập kế hoạch tác vụ | Được xác định một lần cho dự án |
| Ví dụ | "Người dùng có thể đặt lại mật khẩu qua liên kết email" | "Các bài test vượt qua, không có hồi quy, tài liệu đã cập nhật" |

Hai khái niệm này bổ trợ cho nhau. Một tác vụ chỉ được coi là hoàn thành khi các tiêu chí chấp nhận của chính nó được đáp ứng **VÀ** Định nghĩa Hoàn thành chung được thỏa mãn. Bỏ qua một trong hai sẽ dẫn đến kết quả là công việc trông có vẻ đã xong nhưng thực chất thì chưa.

## Danh sách Kiểm tra Chung

Áp dụng danh sách này cho mọi thay đổi trước khi tuyên bố hoàn thành.

### Tính đúng đắn (Correctness)
- [ ] Tất cả các tiêu chí chấp nhận cho tác vụ đều được đáp ứng
- [ ] Mã nguồn chạy và hoạt động như dự kiến, được xác minh khi chạy (runtime), không chỉ dừng lại ở việc biên dịch hoặc kiểm tra kiểu (typecheck)
- [ ] Hành vi mới được bao phủ bởi các bài test (có kết quả thất bại nếu không có thay đổi và vượt qua khi có thay đổi)
- [ ] Các bài test hiện có vẫn vượt qua; không gây ra lỗi hồi quy (regressions)
- [ ] Các trường hợp biên (edge cases) và luồng lỗi được xử lý, không chỉ là luồng chạy thành công (happy path)

### Chất lượng (Quality)
- [ ] Mã nguồn thể hiện rõ ý định thông qua cách đặt tên và cấu trúc; không cần ghi chú để giải thích *nó làm gì*
- [ ] Không có logic nghiệp vụ bị trùng lặp
- [ ] Không còn mã chết (dead code), kết quả gỡ lỗi (debug output), hoặc các khối mã bị comment lại
- [ ] Các thay đổi nằm trong phạm vi tác vụ; không có các đợt tái cấu trúc (refactor) không liên quan bị lồng vào
- [ ] Vượt qua kiểm tra linting và định dạng (formatting)

Chi tiết sâu hơn về các mục này nằm trong `code-review-and-quality` (đánh giá năm trục) và `code-simplification` (giảm độ phức tạp mà không thay đổi hành vi).

### Tích hợp (Integration)
- [ ] Thay đổi hoạt động tốt với phần còn lại của hệ thống, không chỉ hoạt động độc lập
- [ ] Các thay đổi về database migrations, cấu hình (config) và feature flags đã được tính toán
- [ ] Xem xét tính tương thích ngược (backward compatibility) cho bất kỳ thay đổi nào đối với interface công khai hoặc API

### Tài liệu (Documentation)
- [ ] Các interface công khai, API và hành vi hướng tới người dùng đã được lập tài liệu
- [ ] Các quyết định kiến trúc cần lưu giữ đã được ghi lại (xem `documentation-and-adrs`)
- [ ] Tài liệu mô tả trạng thái hiện tại bằng ngôn ngữ không phụ thuộc thời gian, không phải lịch sử thay đổi

### Sẵn sàng bàn giao (Ship-readiness)
- [ ] Các ảnh hưởng về bảo mật đã được xem xét đối với bất kỳ đầu vào không tin cậy, xác thực (auth), hoặc xử lý dữ liệu nào (xem `security-and-hardening`)
- [ ] Đã thiết lập khả năng quan sát (observability) cho các luồng quan trọng mới (logs, metrics, traces) (xem `observability-and-instrumentation`)
- [ ] Có phương án hoàn tác (rollback) cho bất kỳ thay đổi rủi ro nào (xem `shipping-and-launch`)
- [ ] Con người đã xem xét và phê duyệt trước khi merge hoặc triển khai (deploy)

## Cách áp dụng

- **Theo từng tác vụ**: xác nhận phần Tính đúng đắn và Chất lượng trước khi đánh dấu hoàn thành tác vụ.
- **Theo từng tính năng**: xác nhận Tích hợp và Tài liệu trước khi coi tính năng đó là hoàn tất.
- **Theo mỗi bản phát hành**: danh sách kiểm tra đầy đủ là mức tối thiểu; `shipping-and-launch` sẽ bổ sung thêm các chốt chặn đặc thù cho việc triển khai (deploy).

Điều chỉnh danh sách này cho phù hợp với dự án một lần, sau đó tái sử dụng mà không thay đổi. Một Định nghĩa Hoàn thành mà phải thương lượng lại sau mỗi sprint thì không còn là Định nghĩa Hoàn thành nữa.

## Các dấu hiệu cảnh báo (Red Flags)

- "Xong rồi, tôi chỉ là chưa chạy thử thôi": công việc chưa được xác minh thì không được coi là hoàn thành.
- "Các bài test vượt qua" được dùng như một từ đồng nghĩa với hoàn thành trong khi tài liệu, lỗi hồi quy hoặc xác minh runtime bị bỏ qua.
- Áp dụng một tiêu chuẩn khác nhau tùy thuộc vào áp lực thời hạn (deadline).
- Tiêu chí chấp nhận được coi là toàn bộ tiêu chuẩn, mà không có mức sàn chất lượng chung.
- Tuyên bố "Hoàn thành" trước khi con người xem xét đối với những thay đổi cần thiết.
