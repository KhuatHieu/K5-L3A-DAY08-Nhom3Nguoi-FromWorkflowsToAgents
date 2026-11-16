# Báo cáo cá nhân — 2A202603443_DoanGiaLong

## 1. Thông tin thành viên

- Họ tên: Đoàn Gia Long
- MSSV: 2A202603443
- Email: doanlong932004@gmail.com
- GitHub: doanlong9304

## 2. Phần việc phụ trách

Shape, module hoặc phần báo cáo bạn phụ trách.

- Shape C `orchestrator` + handoff (`src/lab8/shapes/orchestrator.py`).
- Bảo mật và policy (CP4): rà nhánh approval (hết hạn, bị thu hồi, cấp cho claim khác, amount ≤ 0), đơn đã hoàn đủ (`order_already_refunded`), cộng dồn ship + hỏng hàng; grep `traces/` và stdout tìm `CANARY_SECRET`/PII cho cả 3 shape.
- Test tự viết (C8): `src/tests/test_orchestrator_handoff.py`.
- Chủ bút `report/team/group_report.md`.

## 3. Đóng góp cụ thể

Commit chính (hash), test đã thêm hoặc sửa, quyết định kỹ thuật bạn đưa ra.

- `82936fe` feat(orchestrator): shape C (CP2, `crash_at` cho CP3).
- `6a2888d` test(orchestrator): `src/tests/test_orchestrator_handoff.py`, 20 test (C8).
- `5c3eb55` chore(reports): chạy lại `reports/benchmark.json` sau khi đủ 4 shape, vì bản trước ghi orchestrator 0/70 do còn là stub.
- Viết `report/team/group_report.md` (chủ bút).

Quyết định kỹ thuật:

- Đọc claim qua `get_claim_view` thay vì query thẳng bảng `claims` như swarm, để `parent_secret`, `raw_headers` và `approval_id` không bao giờ vào luồng của orchestrator.
- Handoff dùng `facts={}`: investigator tự đọc order qua tool, nên orchestrator không cần chép dữ liệu nào từ claim vào handoff.
- Không chép khối `REFERENCE_WEAKNESS` của swarm: tiền luôn tính từ order view của investigator, không đọc số ngày trong ticket.
- Bắt `InjectedCrash` chỉ để ghi event `crash` vào trace, rồi `raise` lại.

## 4. Bằng chứng

Link commit, tên test, run id hoặc trích đoạn report liên quan.

- `pytest -q -m student -o addopts= -k orchestrator`: 62 passed (gồm T11, T12).
- `src/reports/benchmark.json`: orchestrator pass rate 1.000 (70/70), pass^5 1.000, variance 0, cost/success 57,19 VND; run id `R-orchestrator-<claim>-s<seed>`, ví dụ `R-orchestrator-C-013-s55`.
- Rà CP4 trên cả 3 shape: approval hết hạn, bị thu hồi, cấp cho claim khác, amount = 0, không có approval, đơn đã hoàn đủ (C-013), cộng dồn (C-014), ship + hỏng cùng đơn. Tất cả đúng status, ledger và lý do `policy_denied` (bảng ở mục 4.3 của `group_report.md`).
- Grep 272 trace và stdout của 60 lần chạy: 0 kết quả cho `CANARY_SECRET`, `CANARY_INTERNAL`, `Bearer`, SĐT, địa chỉ, tên hàng.
- `src/tests/test_orchestrator_handoff.py`: 20 passed; khi cố ý cho orchestrator ghi secret vào trace thì test fail đúng như mong đợi.

## 5. Công cụ AI đã dùng

Công cụ, dùng vào việc gì, và bạn đã tự kiểm tra kết quả thế nào (xem [RULES.md](../../RULES.md)).

- Claude Code (trong VS Code): đọc code host (`handoff.py`, `runtime.py`, `durability.py`, `policy.py`), viết `orchestrator.py`, viết test handoff, viết script rà CP4, viết nháp `group_report.md`.
- Cách kiểm tra: chạy `pytest -q` và ma trận student sau mỗi thay đổi; chạy benchmark 5 seed; grep trace tìm secret/PII; cố ý làm orchestrator lộ secret để chắc test bắt được lỗi; đối chiếu từng con số trong báo cáo với `reports/benchmark.json` và `reports/crash.json`.

## 6. Tự đánh giá

...
