# Decision record

Điền sau dấu `:` trên cùng dòng, không xóa nhãn. Giảng viên và `python -m lab8.check_submission` (chạy trong `src/`) đọc các dòng này bằng máy. Không đổi tên hay dời file này khỏi `report/team/`.
`Shape chọn` phải là đúng một trong: `workflow`, `single`, `orchestrator`, `swarm`. Đây là shape được chấm cổng hidden.

- Shape chọn: workflow
- Commit hash: 14efe67
- Run id dẫn chứng: R-workflow-C-002-s55, R-workflow-C-014-s11, R-workflow-C-012-s11 (trials trong src/reports/benchmark.json, 14 claim public × seed 11,22,33,44,55)
- Pass rate / pass^5 / seed min-max / variance: 1.000 / 1.000 / 1.000–1.000 / 0.000000 (src/reports/benchmark.json, by_shape.workflow, 70 trial)
- Cost per success (mô phỏng, VND): 34.29 (workflow) so với single 45.86, swarm 49.57, orchestrator 57.19
- Crash safety: 20/20 (src/reports/crash.json)
- Secret leak / approval bypass: 0 / 0 trên bộ public (ma trận student 248/248 passed, gồm forbid_substrings và approval của T01–T14; quét src/traces/ không có CANARY_SECRET_*, SĐT, địa chỉ, ghi chú nội bộ). Số hidden do grader giảng viên chấm

Ví dụ một dòng đã điền: `- Shape chọn: workflow`, `- Crash safety: 20/20 (src/reports/crash.json)`.

## Vì sao shape này

Bốn shape chạy trên cùng 14 claim public, cùng 5 seed (11, 22, 33, 44, 55), cùng `as_of` 2026-11-15, cùng MockLLM và `config/pricing.yaml`, mỗi trial một SQLite mới (`controlled_variables` trong `src/reports/benchmark.json`). Workflow, single và orchestrator cùng đạt pass rate 1.000, pass^5 1.000, variance 0. Swarm đạt 0.986 / 0.929 vì fail C-002 (T02) ở seed 55: nó tin "trễ 5 ngày" trong ticket thay vì `delay_days = 2` của order (run `R-swarm-C-002-s55`, `status resolved != denied`). Khi độ đúng ngang nhau, nhóm chọn shape **ít tự chủ nhất**: workflow có cost/success thấp nhất (34.29 VND, rẻ hơn single 25%, rẻ hơn orchestrator 40%), 0 lần gọi model (`mean_model_calls = 0`), nên không có đường nào để chữ trong ticket đổi được quyết định. Luật nghiệp vụ của LogiX là luật cứng (ngưỡng ngày trễ, policy hiện hành, approval, trần theo đơn), nên viết thẳng thành code là đủ.

## Có đáng đổi lấy multi-agent không

Không, trên dữ liệu hiện có. Orchestrator không đúng hơn workflow (cùng 1.000) mà tốn hơn 67% cost/success, thêm một ranh giới handoff phải sanitize. Single tốn 8.07 bước trung bình so với 4.43. Swarm vừa đắt hơn vừa có variance theo seed. Quyết định sẽ đảo lại nếu: (1) ticket chứa thông tin cần cho quyết định mà không có cột có cấu trúc nào chứa (ví dụ mô tả hỏng hàng phải đọc hiểu ngôn ngữ tự nhiên); (2) bộ hidden cho thấy workflow fail ở loại claim mà shape có model xử lý đúng, và chênh lệch lớn hơn mức dao động giữa các seed; (3) số nguồn cần tra tăng và chạy song song được, khiến orchestrator rút ngắn critical path. Kể cả khi đó, tiền, approval và idempotency vẫn phải do code và host quyết.

## Trace và bằng chứng

Trace của workflow (`src/traces/R-workflow-<claim>-s<seed>.json`, ghi bằng `lab8.trace.Trace`, có ở mọi nhánh return) có các event: `get_claim_view` (chỉ `found`, `has_order_id`), `lookup_order` (view allowlist: `order_id`, `status`, `delay_days`, `shipping_fee_vnd`, `paid_vnd`, `damage_verified`, `carrier`), `quote` (số tiền, lý do, clause), `issue_refund` (`ok`, `reason`, `conflict`), `crash` (nếu có) và `decision`. Các field cố ý **không** ghi: nội dung ticket (dữ liệu người dùng, có thể chứa injection hoặc PII), `principal_id`, `parent_secret` và `raw_headers` (không bao giờ đọc vì dùng `get_claim_view` thay cho `SELECT *`), SĐT, địa chỉ, `internal_note`, `item_name` (bị `lookup_order` lọc từ đầu). Workflow không ghi file nào khác và không in ra stdout. Bằng chứng: test `test_workflow_trace_has_no_secret_or_pii` chạy cả 14 claim và kiểm trace cùng `RunResult` không chứa canary, header, PII hay ticket. Trace nằm ngoài chỗ scanner quét thì không chứng minh được là sạch, nên nhóm giữ mọi output trong `trace_path`.

## Giới hạn

- MockLLM không phải model sản xuất: hành vi của single, orchestrator và swarm là mô phỏng xác định theo seed, nên variance thật của model có thể khác.
- Cost là VND mô phỏng theo `config/pricing.yaml`, không phải hóa đơn API. Latency để `unavailable`.
- Bộ public chỉ có 14 claim; pass 1.000 trên bộ này chưa chứng minh được kết quả trên hidden (40 claim × 5 seed). Số chính thức do grader của giảng viên chấm.
- Không có JSON nào trong `src/reports/` bị sửa tay. Benchmark và crash chạy trên Python 3.12.15, cho kết quả trùng với lần chạy trên Python 3.14.
