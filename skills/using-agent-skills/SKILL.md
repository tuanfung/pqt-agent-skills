---
name: using-agent-skills
description: Khám phá và thực thi các agent skills. Đây là meta-skill quản lý cách tất cả các skill khác được khám phá và thực thi, hỗ trợ cả chế độ vận hành đơn lẻ và điều phối nhóm.
---

# Sử dụng Agent Skills

## Tổng quan

Agent Skills là một tập hợp các skill về quy trình kỹ thuật được tổ chức theo giai đoạn phát triển. Mỗi skill mã hóa một quy trình cụ thể mà các kỹ sư dày dạn kinh nghiệm thường theo đuổi.

### ⚙️ Chế độ Vận hành (Operational Modes)

Tùy thuộc vào yêu cầu của người dùng, meta-skill này vận hành theo một trong hai chế độ:

1. **Single-Agent Mode (Mặc định)**: 
   - **Kích hoạt**: Khi người dùng yêu cầu thực hiện tác vụ hoặc gọi `using-agent-skills` mà không đề cập đến việc tạo team.
   - **Vận hành**: Một agent duy nhất (Claude) tự đóng vai tất cả các vị trí cần thiết, thực hiện chuỗi skill theo trình tự. 
   - **Mục tiêu**: Tốc độ, tinh gọn, phù hợp cho tác vụ nhỏ và trung bình.

2. **Team Orchestration Mode (Chế độ Điều phối)**:
   - **Kích hoạt**: CHỈ khi người dùng ra lệnh rõ ràng (ví dụ: "Tạo team", "Khởi động TeamsCreate", "Sử dụng mô hình team").
   - **Vận hành**: Kích hoạt lệnh `TeamsCreate` để thiết lập nhóm đa-agent (Leader, Coder, Tester, Reviewer, Shipper) theo file `team.md`.
   - **Mục tiêu**: Độ tin cậy tối đa, kiểm soát chất lượng nghiêm ngặt qua các Cổng Chất lượng (Quality Gates), phù hợp cho tác vụ phức tạp, rủi ro cao.

## Khám phá và Phân quyền Skill

Khi một tác vụ xuất hiện, hãy xác định giai đoạn phát triển và skill tương ứng. Trong Team Mode, skill này sẽ được giao cho vai trò chịu trách nhiệm tương ứng.

```
Tác vụ xuất hiện
    │
    ├── Chưa biết mình muốn gì? ──────→ interview-me [Leader]
    ├── Có khái niệm sơ bộ, cần các phương án? → idea-refine [Leader]
    ├── Dự án/tính năng/thay đổi mới? ──→ spec-driven-development [Leader]
    ├── Có spec, cần chia tác vụ? ──────→ planning-and-task-breakdown [Leader]
    ├── Đang triển khai code? ────────────→ incremental-implementation [Coder]
    │   ├── Làm về UI? ─────────────────→ frontend-ui-engineering [Coder]
    │   ├── Làm về API? ────────────────→ api-and-interface-design [Coder]
    │   ├── Cần ngữ cảnh tốt hơn? ─────→ context-engineering [Coder]
    │   ├── Cần code được xác minh qua doc? ───→ source-driven-development [Coder]
    │   └── Rủi ro cao / code lạ? ──→ doubt-driven-development [Coder]
    ├── Viết/chạy test? ────────→ test-driven-development [Tester]
    │   └── Dựa trên trình duyệt? ───────────→ browser-testing-with-devtools [Tester]
    ├── Có lỗi xảy ra? ──────────────→ debugging-and-error-recovery [Tester]
    ├── Xem xét code? ───────────────→ code-review-and-quality [Reviewer]
    │   ├── Quá phức tạp? ─────────────→ code-simplification [Reviewer]
    │   ├── Quan ngại về bảo mật? ───────→ security-and-hardening [Reviewer]
    │   └── Quan ngại về hiệu năng? ────→ performance-optimization [Reviewer]
    ├── Commit/chia nhánh? ─────────→ git-workflow-and-versioning [Shipper]
    ├── Làm việc với CI/CD pipeline? ──────────→ ci-cd-and-automation [Shipper]
    ├── Loại bỏ/di chuyển (deprecating/migrating)? ────────→ deprecation-and-migration [Shipper]
    ├── Viết docs/ADRs? ───────────→ documentation-and-adrs [Shipper]
    ├── Thêm logs/metrics/alerts? ───→ observability-and-instrumentation [Shipper]
    └── Deploy/phát hành? ─────────→ shipping-and-launch [Shipper]
```

## Các hành vi vận hành cốt lõi

Các hành vi này áp dụng trong mọi thời điểm, trên tất cả các skill và cho tất cả các vai trò.

### 1. Nêu rõ các giả định
Trước khi triển khai bất cứ điều gì không đơn giản, hãy nêu rõ các giả định của bạn. Đừng tự ý điền vào những yêu cầu mơ hồ.

### 2. Quản lý sự mơ hồ một cách chủ động
Khi gặp mâu thuẫn hoặc spec không rõ ràng: **DỪNG LẠI** $\rightarrow$ Gọi tên điều gây mơ hồ $\rightarrow$ Đưa ra tradeoff $\rightarrow$ Đợi giải quyết.

### 3. Phản hồi khi cần thiết
Không nịnh bợ. Sự bất đồng kỹ thuật trung thực có giá trị hơn sự đồng ý giả tạo. Chỉ ra vấn đề trực tiếp và đề xuất phương án thay thế.

### 4. Ưu tiên sự đơn giản
Chủ động chống lại xu hướng làm phức tạp hóa. Ưu tiên giải pháp đơn giản, hiển nhiên.

### 5. Tuân thủ kỷ luật về phạm vi
Chỉ chạm vào những gì được yêu cầu. Không "dọn dẹp" code không liên quan.

### 6. Xác minh, không giả định
Một tác vụ chưa hoàn thành cho đến khi có minh chứng thực tế (test pass, build output, dữ liệu runtime).

### 7. Giao tiếp tập trung (Chỉ áp dụng trong Team Mode)
Toàn bộ trao đổi giữa các teammates phải thực hiện qua `SendMessage` trong tmux. Tuyệt đối không tự ý tạo subagent mới để giao tiếp.

### 8. Kỷ luật phân quyền (Chỉ áp dụng trong Team Mode)
Mỗi teammate chỉ được phép vận hành các skill thuộc phạm vi trách nhiệm của mình. Việc vận hành skill ngoài phạm vi phải được sự đồng ý hoặc chỉ định từ Leader.

## Quy tắc về Skill

1. **Kiểm tra skill áp dụng trước khi bắt đầu làm việc.**
2. **Skill là các workflow, không phải là gợi ý.** Thực hiện theo đúng thứ tự.
3. **Nhiều skill có thể cùng áp dụng.**
4. **Khi nghi ngờ, hãy bắt đầu với một spec.**

## Trình tự vòng đời & Bàn giao (Handoff)

Đối với một tính năng hoàn chỉnh, trình tự điển hình là:

**[Define/Plan $\rightarrow$ Leader]** $\xrightarrow{\text{Giao Task}}$ **[Build $\rightarrow$ Coder]** $\xrightarrow{\text{Code xong}}$ **[Verify $\rightarrow$ Tester]** $\xrightarrow{\text{Pass Test}}$ **[Review $\rightarrow$ Reviewer]** $\xrightarrow{\text{Pass Review}}$ **[Plan $\rightarrow$ Leader]** $\xrightarrow{\text{Clear Backlog}}$ **[Ship $\rightarrow$ Shipper]**

## Tham khảo nhanh

| Giai đoạn | Skill | Vai trò | Tóm tắt một dòng |
|-------|-------|-------|-----------------|
| Define | interview-me | Leader | Nêu rõ những gì người dùng thực sự muốn |
| Define | idea-refine | Leader | Tinh chỉnh ý tưởng thông qua tư duy phân kỳ và hội tụ |
| Define | spec-driven-development | Leader | Yêu cầu và tiêu chí chấp nhận trước khi viết code |
| Plan | planning-and-task-breakdown | Leader | Chia nhỏ thành các tác vụ nhỏ, có thể xác minh |
| Build | incremental-implementation | Coder | Chia lát cắt dọc mỏng, test từng cái trước khi mở rộng |
| Build | source-driven-development | Coder | Xác minh với tài liệu chính thức trước khi triển khai |
| Build | doubt-driven-development | Coder | Review đối lập với ngữ cảnh mới cho mọi quyết định |
| Build | context-engineering | Coder | Đúng ngữ cảnh vào đúng thời điểm |
| Build | frontend-ui-engineering | Coder | UI chất lượng production với accessibility |
| Build | api-and-interface-design | Coder | Giao diện ổn định với các contract rõ ràng |
| Verify | test-driven-development | Tester | Viết test fail trước, sau đó mới làm cho nó pass |
| Verify | browser-testing-with-devtools | Tester | Chrome DevTools MCP để xác minh runtime |
| Verify | debugging-and-error-recovery | Tester | Tái hiện $\rightarrow$ khoanh vùng $\rightarrow$ sửa $\rightarrow$ phòng ngừa |
| Review | code-review-and-quality | Reviewer | Review 5 trục với các cổng chất lượng |
| Review | code-simplification | Reviewer | Giữ nguyên hành vi trong khi giảm độ phức tạp |
| Review | security-and-hardening | Reviewer | Ngăn chặn theo OWASP, validate input, quyền hạn tối thiểu |
| Review | performance-optimization | Reviewer | Đo lường trước, chỉ tối ưu những gì thực sự quan trọng |
| Ship | git-workflow-and-versioning | Shipper | Commit nguyên tử, lịch sử sạch |
| Ship | ci-cd-and-automation | Shipper | Cổng chất lượng tự động cho mọi thay đổi |
| Ship | deprecation-and-migration | Shipper | Loại bỏ hệ thống cũ và di chuyển người dùng an toàn |
| Ship | documentation-and-adrs | Shipper | Tài liệu hóa lý do (why), không chỉ là cái gì (what) |
| Ship | observability-and-instrumentation | Shipper | Structured logs, RED metrics, traces, cảnh báo |
| Ship | shipping-and-launch | Shipper | Checklist trước khi launch, giám sát, rollback |
