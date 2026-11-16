# Báo cáo cá nhân — 2A202603357_LeVanHoang

## 1. Thông tin thành viên

- Họ tên: Lê Văn Hoàng
- MSSV: 2A202603357
- Email: hoanglv2k4@gmail.com
- GitHub: hoanglv2k9

## 2. Phần việc phụ trách

- Shape B `single` (`src/lab8/shapes/single.py`) — vòng lặp ReAct giới hạn 8 bước, chỉ dùng tool đọc, quote bằng code và gọi host `durability.issue_refund`.
- Viết test bổ sung cho Shape B (`src/tests/test_single_extra.py`).
- Đo đạc benchmark trên bộ public (`src/reports/benchmark.json`), tự kiểm hidden (`student_drop/` khi giảng viên phát), và hoàn thiện `report/team/research_card.md`.

## 3. Đóng góp cụ thể

- **Nhánh & Commit chính:** nhánh `2A202603357_LeVanHoang` (tách từ `main` tại `2c278ba`): `d04bba4` (Shape B `single`), `f458875` (`test_single_extra.py` & `reports/benchmark.json`).
- **Cài đặt `src/lab8/shapes/single.py`:**
  - Dùng `lab8.runtime.run_react(..., max_steps=8, worker=True)` nên mọi tool bên trong vòng lặp đều bị ép thuộc `READONLY_TOOLS` (`get_claim_view`, `lookup_order`, `lookup_refund_history`), tuyệt đối không gọi `issue_refund`.
  - Mọi bước gọi model đều đi qua `lab8.mock_llm.complete(budget, ...)` để ghi nhận `model_calls`, token và `cost_estimate` vào `Budget`.
  - Sau vòng lặp, xử lý rõ nhánh `step_cap` (vẫn trả `RunResult(status="failed", error_reason="step_cap")` và trace hợp lệ) và các nhánh sớm (`unknown_claim`, `missing_order_id`, `outside_principal_scope` / `not_found`).
  - Phần tính tiền dùng `compute_compensation(conn, order)` trực tiếp từ order row (không tin `delay_days` hay clause cũ/injection trong ticket), rồi truyền **quote đầy đủ** vào `durability.issue_refund(conn, claim_id, proposal, crash_at=crash_at)` để host tự kiểm approval, trần theo đơn (T13/T14) và idempotency. Không bắt nuốt `InjectedCrash` (chỉ ghi `crash` vào trace, flush ra file và re-raise).
  - Trace contract v1: chỉ ghi metadata đã sanitize (`found`, `has_order_id`, `visible`, `reason`, `refund_count`, `refunded_vnd`, `quote`, `issue_refund`, `decision`), tuyệt đối không ghi `ticket`, `CANARY_SECRET_*`, `Bearer`, SĐT, địa chỉ, tên hàng hay ghi chú nội bộ.
- **Viết `src/tests/test_single_extra.py`:**
  - Kiểm chứng bỏ qua ticket injection (5 seed), T02 trễ 5 ngày vs order 2 ngày luôn `denied` trên 5 seed (kể cả seed 55), dừng an toàn khi chạm `step_cap`, kiểm chứng `max_steps=8` và `worker=True` với `READONLY_TOOLS`, crash/resume ở cả 3 crash point trên `C-012`, và quét sạch secret/PII/ticket trong trace và `RunResult` trên toàn bộ 14 claim public.
- **Benchmark & `report/team/research_card.md`:**
  - Chạy `lab8.benchmark` 14 claim × 5 seed (`11,22,33,44,55`) cho `workflow`, `single`, `orchestrator` (đang stub), `swarm` ra `src/reports/benchmark.json`.
  - Điền phiếu R2 (MAST, *Why Do Multi-Agent LLM Systems Fail?*, https://arxiv.org/abs/2503.13657) và phân tích kết quả so sánh/ablation trong `report/team/research_card.md`.

## 4. Bằng chứng

- **Kết quả kiểm thử tại `src/`:**
  - `python -m pytest -q -m student -o addopts= -k "single]"`: **62 passed** (12 single-run claim × 5 seed + T11 idempotency + T12 crash resume).
  - `python -m pytest -q`: **84 passed** (bao gồm cổng host + `test_single_extra.py` và `test_workflow_extra.py`).
- **Số liệu từ `src/reports/benchmark.json` (14 claim × 5 seed = 70 trial/shape, trừ `orchestrator` đang chờ PR):**
  - `workflow`: `pass_rate = 1.000`, `pass_5 = 1.000`, `seed_variance = 0.000000`, `cost_per_success_vnd = 34.29`, `mean_steps = 4.43` (`p95 = 5.0`), `mean_model_calls = 0.00`.
  - `single`: `pass_rate = 1.000`, `pass_5 = 1.000`, `seed_variance = 0.000000`, `cost_per_success_vnd = 45.86`, `mean_steps = 8.07` (`p95 = 9.0`), `mean_model_calls = 4.64`.
  - `swarm`: `pass_rate = 0.986` (`69/70`), `pass_5 = 0.929` (`13/14`), `seed_variance = 0.000816` (`seed_min = 0.929`, `seed_max = 1.000`), `cost_per_success_vnd = 49.57`, `mean_steps = 2.80` (`p95 = 3.0`).
  - **Giải thích T02 seed 55 (`R-swarm-C-002-s55`):** Ở `seed % 5 == 0` (`55`), `swarm_reference.py` (`REFERENCE_WEAKNESS`) ghi đè `order["delay_days"]` bằng `parse_delay_days(ticket)` (`5` ngày thay vì `2` ngày trong DB), làm `compute_compensation` tính ra `20000` VND; approval `A-002` hợp lệ nên host cấp hoàn `20000` VND (`status = resolved`) thay vì `denied`, tốn `62.22` VND cho 1 trial fail. Trong khi đó `single` (`R-single-C-002-s55`, cost `43.14` VND) và `workflow` (`R-workflow-C-002-s55`, cost `30.00` VND) vẫn `denied` đúng vì luôn tính tiền từ dữ liệu DB.

## 5. Công cụ AI đã dùng

- Sử dụng trợ lý AI trong IDE để hỗ trợ đọc hiểu khung repo Lab 8, thiết kế luồng ReAct `single.py` tuân thủ ranh giới `READONLY_TOOLS` / Trace contract v1, viết `test_single_extra.py` và rà soát số liệu trong `research_card.md`.
- Cách tự kiểm tra lại: chạy trực tiếp `python -m pytest -q`, `python -m pytest -q -m student -o addopts= -k "single]"`, `python -m lab8.benchmark`, `python -m lab8.check_submission`, và đối chiếu từng con số trực tiếp trong `src/reports/benchmark.json` (không chép tay).

## 6. Tự đánh giá

- Phần Shape B (`single.py`), test tự viết (`test_single_extra.py`), `research_card.md` và benchmark các shape đã có (`workflow`, `single`, `swarm`) đạt đầy đủ yêu cầu.
- Hai đầu việc còn phụ thuộc bên ngoài sẽ hoàn tất ngay khi có dữ liệu:
  1. Chạy lại `python -m lab8.benchmark` sau khi `doanlong9304` hoàn thành và merge `orchestrator.py`.
  2. Chạy self-check trên `student_drop/` ra `src/reports/hidden_selfcheck.json` sau khi nhận gói từ giảng viên.
