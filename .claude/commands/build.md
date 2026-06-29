---
description: Triển khai các tác vụ một cách tăng dần — xây dựng, kiểm thử, xác minh, commit. Thêm "auto" để chạy toàn bộ kế hoạch trong một lượt phê duyệt duy nhất.
---

Kích hoạt skill agent-skills:incremental-implementation cùng với agent-skills:test-driven-development.

## Chế độ

- **`/build`** — triển khai tác vụ chờ *tiếp theo*, sau đó dừng lại (cẩn thận, thực hiện từng phần một).
- **`/build auto`** — tạo kế hoạch nếu cần, nhận một lần phê duyệt duy nhất, sau đó triển khai *mọi* tác vụ mà không dừng lại giữa chừng.

`$ARGUMENTS` dùng để chọn chế độ. Coi `auto` (chuẩn) hoặc `all` là chế độ tự trị (autonomous mode); bất kỳ giá trị nào khác (hoặc để trống) sẽ là chế độ một tác vụ mặc định. Lưu ý: chế độ tự trị không nhanh hơn *trên mỗi tác vụ* — nó vẫn chạy cùng một vòng lặp test-driven — nó chỉ loại bỏ việc con người phải xác nhận giữa các tác vụ.

## Mặc định: một tác vụ

Chọn tác vụ chờ tiếp theo từ kế hoạch. Sau đó:

1. Đọc tiêu chí chấp nhận (acceptance criteria) của tác vụ
2. Tải ngữ cảnh liên quan (mã nguồn hiện có, patterns, types)
3. Viết một test case thất bại cho hành vi mong đợi (RED)
4. Triển khai lượng mã tối thiểu để vượt qua test (GREEN)
5. Chạy toàn bộ bộ test để kiểm tra regression
6. Chạy build để xác minh biên dịch
7. Commit với thông báo mô tả chi tiết
8. Đánh dấu tác vụ đã hoàn thành và dừng lại

## Tự trị: toàn bộ kế hoạch (`/build auto`)

Sử dụng chế độ này khi đã có spec và bạn muốn gộp kế hoạch + build vào một lượt chạy. Nó loại bỏ việc xác nhận thủ công giữa các tác vụ — nhưng **không** loại bỏ việc xác minh. Mỗi tác vụ vẫn phải vượt qua test và có commit riêng.

1. **Yêu cầu phải có spec.** Chỉ tìm spec tại các đường dẫn đã biết: `SPEC.md` ở thư mục gốc của repo, `docs/SPEC.md`, hoặc một file trong `spec/`. File README hoặc tài liệu tùy ý không được tính. Nếu không có, hãy dừng lại và bảo người dùng chạy `/spec` trước — không tự ý tạo ra các yêu cầu.
2. **Thiết lập một baseline sạch.** Chạy `git status --porcelain`. Nếu có các thay đổi chưa commit nằm ngoài các artifact lập kế hoạch dự kiến (`SPEC.md`, `docs/SPEC.md`, `spec/*`, `tasks/plan.md`, `tasks/todo.md`), hãy dừng lại và yêu cầu người dùng commit, stash, hoặc xác nhận cách xử lý. Các commit tự trị cho từng tác vụ không được bao gồm các công việc cục bộ không liên quan, nếu không cam kết rollback sạch sẽ bị phá vỡ.
3. **Lập kế hoạch nếu cần.** Nếu chưa có `tasks/plan.md`, kích hoạt agent-skills:planning-and-task-breakdown để tạo.
4. **Điểm kiểm tra duy nhất.** Hiển thị toàn bộ kế hoạch và chờ một câu khẳng định rõ ràng (ví dụ: "approve", "go", "yes"). Coi các câu trả lời mơ hồ ("looks reasonable", "I guess") là **không** được phê duyệt. Đây là cổng kiểm soát duy nhất của con người — sau khi phê duyệt, quá trình sẽ chạy tự trị. Nếu bạn đã tạo `tasks/plan.md`, hãy commit nó thành một commit chuẩn bị riêng lẻ ngay bây giờ để không bị lẫn vào commit của tác vụ đầu tiên.
5. **Thực thi mọi tác vụ theo thứ tự phụ thuộc.** Sử dụng các phụ thuộc đã khai báo của mỗi tác vụ; nếu không rõ ràng, hãy thực thi theo thứ tự trong kế hoạch. Với mỗi tác vụ, chạy toàn bộ vòng lặp mặc định ở trên (RED → GREEN → regression → build → commit → đánh dấu hoàn thành). Chỉ stage những file mà tác vụ đó tác động cùng với cập nhật trạng thái tác vụ — tuyệt đối không dùng `git add -A` một cách mù quáng — và tạo một commit cho mỗi tác vụ để đảm bảo bất kỳ thời điểm nào cũng có thể rollback sạch.
6. **Dừng lại và hỏi người dùng** (không được tự ý tiếp tục) khi:
   - một test không thể vượt qua hoặc build bị hỏng mà không có cách sửa rõ ràng → tuân theo agent-skills:debugging-and-error-recovery
   - spec mơ hồ, hoặc một tác vụ cần một quyết định mà spec không đề cập
   - một tác vụ có rủi ro cao hoặc không thể đảo ngược — thay đổi auth/permission, migration dữ liệu gây mất mát, thanh toán, xóa, deploy, bất cứ thứ gì chạm đến secret, **hoặc bất cứ điều gì bạn không thể hoàn tác bằng `git revert`** → tuân theo agent-skills:doubt-driven-development và nhận xác nhận rõ ràng trước khi tiếp tục

   Sau khi người dùng giải quyết xong vấn đề chặn (blocker), họ sẽ kích hoạt lại `/build auto` — nó sẽ tiếp tục từ tác vụ chờ tiếp theo.
7. **Tóm tắt cuối cùng:** các tác vụ đã hoàn thành, các test đã thêm, các commit đã tạo, và bất cứ điều gì bị bỏ qua, đánh dấu hoặc để lại cho người dùng.

Nếu bất kỳ bước nào thất bại, hãy tuân theo skill agent-skills:debugging-and-error-recovery.
