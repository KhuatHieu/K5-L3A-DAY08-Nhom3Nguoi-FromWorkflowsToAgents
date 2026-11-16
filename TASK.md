# TASK — Phân công Lab 8 (nhóm 3 người)

Nguồn: `README.md`, `CHECKPOINTS.md`, `RUBRIC.md`, `SUBMISSION.md` và [slide giảng viên](https://minhhai203.github.io/k5-day08-instructor-slides/).
Mọi lệnh chạy trong `src/`. Mốc giờ theo lịch chiều 13:30–17:30; ca khác giữ nguyên thứ tự và thời lượng.

## Nguyên tắc chia việc

- **CP2: mỗi người viết một shape** (theo slide). Ba file không đụng nhau, nên ít conflict khi merge.
- Mỗi người còn giữ thêm **một mảng dùng chung** (integration, đo đạc, bảo mật) để không còn việc nào không ai nhận.
- **C8:** mỗi người phải có ít nhất một commit kỹ thuật và **ít nhất một test tự viết** trong `src/tests/`, rồi ghi vào bảng đóng góp trong `TEAM.md`.
- Chỉ sửa `src/lab8/shapes/{workflow,single,orchestrator}.py`, `src/tests/`, `src/bonus/`. **Không sửa** `metrics.py`, `durability.py`, `policy.py`, `config/`, `data/`, và không sửa tay JSON trong `src/reports/`.
- Mỗi người làm trên nhánh riêng `feat/<shape>`, mở PR vào `main`. Trước khi merge thì `python -m pytest -q` phải xanh.

## Tổng quan

| Người | Shape (CP2–CP3) | Mảng dùng chung | Test tự viết (C8) | Báo cáo |
| --- | --- | --- | --- | --- |
| **khuathieu** (Khuất Duy Hiếu) | A `workflow.py` | Nhóm trưởng / integration: repo, `TEAM.md`, merge, crash suite, `check_submission`, chốt hash | `tests/test_workflow_extra.py` | `decision_adr.md`, phần nộp bài |
| **hoanglv2k9** | B `single.py` (ReAct) | Đo đạc: benchmark 4 shape × 5 seed, tự kiểm hidden, đọc metric | `tests/test_single_extra.py` | `research_card.md`, số liệu cho ADR |
| **doanlong9304** | C `orchestrator.py` + handoff | Bảo mật và policy: rà trace/secret/PII, nhánh approval, đơn đã hoàn (CP4) | `tests/test_orchestrator_handoff.py` | `group_report.md` (chủ bút) |

Mỗi người tự viết `report/team/<MSSV>_<HoTen>.md` của mình (đã có: `report/team/2A202603383_KhuatDuyHieu.md`).

---

## khuathieu (Khuất Duy Hiếu) — Shape A `workflow` + integration

**CP1 (13:30–13:45)**
- [x] Fork hoặc tạo repo nhóm tên `K5-L3A-DAY08-<TenNhom>-FromWorkflowsToAgents`, để public hoặc mời giảng viên; mời 2 bạn vào.
- [x] Điền phần thông tin nhóm và bảng thành viên trong `TEAM.md`.
- [ ] Kiểm cả 3 máy: smoke pass; `pytest -q` không có `failed`; ma trận student ra `186 failed, 62 passed`. _(máy khuathieu: đạt, 54 passed ở cổng host; chờ 2 máy còn lại)_

**CP2 (13:45–14:30): `src/lab8/shapes/workflow.py`**
- [x] Đọc claim qua `get_claim_view`, thiếu `order_id` thì trả `needs_info` (không ghi ledger).
- [x] Gọi `lookup_order`; principal không sở hữu đơn thì trả `escalated`, không chép SĐT, địa chỉ, tên hàng hay ghi chú nội bộ.
- [x] Quote bằng `compute_compensation` từ **order row**, bỏ qua số ngày ghi trong ticket. Chỉ dùng policy `is_current` đã có hiệu lực.
- [x] Quote = 0 thì trả `denied`; quote > 0 thì gọi `durability.issue_refund` với **quote đầy đủ** (T13, T14).
- [x] Host từ chối (lỗi approval, đơn đã hoàn) thì trả `denied`; amount conflict thì trả `failed`. Trả kết quả bằng `finish()`, có trace ở **mọi** nhánh return.
- [x] `pytest -q -m student -o addopts= -k workflow` ra 62 passed.

**CP3 (14:30–15:15)**
- [x] Truyền `crash_at` vào `issue_refund`, **không** bắt `InjectedCrash`; T11 và T12 xanh.
- [x] Chạy `python -m lab8.demo.buggy_workflow` để cả nhóm thấy vì sao cần ledger.

**CP4–CP5 (integration)**
- [ ] Merge PR của 3 shape; giữ `pytest -q` xanh trên `main`.
- [x] `python -m lab8.crash_test --count 20 --out reports/crash.json`, phải ra `crash_safety = 20/20`.
- [x] Viết test: ví dụ ticket ghi "ignore policy" hoặc ghi sai số ngày thì workflow vẫn quote theo order; hoặc gọi lại cùng claim thì ledger vẫn đúng 1 dòng.
- [ ] Điền `decision_adr.md` bằng số của hoanglv2k9 (shape chọn, run id). Commit, chạy `git rev-parse --short HEAD`, ghi hash vào ADR rồi commit tiếp.
- [ ] `python -m lab8.check_submission` ra `0 FAIL`, rồi push. Nhắc cả nhóm **tự nộp link trên LMS**.

## hoanglv2k9 — Shape B `single` + benchmark

**CP1**
- [x] Cài môi trường, đọc `docs/metrics.md`, `lab8/runtime.py` (`run_react`), `config/tools.manifest.v1.1.yaml`.

**CP2: `src/lab8/shapes/single.py`**
- [x] Dùng `run_react(max_steps=8)`; `decide` chỉ được chọn **tool đọc** (`get_claim_view`, `lookup_order`, `lookup_refund_history`…), không bao giờ gọi `issue_refund`.
- [x] Sau vòng lặp, **code** quote bằng `compute_compensation` từ order row (không tin ticket), rồi gọi `issue_refund` với quote đầy đủ.
- [x] Đếm mọi lần gọi model vào `Budget` để cost so sánh được giữa các shape.
- [x] Có trace ở mọi nhánh, kể cả khi hết 8 bước. `-k single` ra 62 passed.

**CP3**
- [x] Truyền `crash_at`, không bắt `InjectedCrash`; T11 và T12 xanh.

**CP5 (đo đạc)**
- [x] `python -m lab8.benchmark --shapes workflow,single,orchestrator,swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/benchmark.json` _(đã chạy sơ bộ khi `orchestrator` còn stub; chạy lại sau khi merge `orchestrator`)_
- [ ] Sau 16:15: đặt `student_drop/` vào `src/` rồi chạy self-check hidden ra `reports/hidden_selfcheck.json`; kiểm không có `status=error` hay `leak_suspect`.
- [x] Lập bảng pass rate, pass^5, variance, cost/success cho 4 shape, kèm `run_id`. Giải thích vì sao swarm fail T02 ở seed 55. _(đã lập số liệu public cho `workflow`, `single`, `swarm`; chờ `orchestrator`)_
- [x] Điền `research_card.md` (R1–R4, ablation); cột "Nhóm điền" không để trống.
- [x] Viết test: ví dụ single dừng ở bước 8 và vẫn trả status hợp lệ; hoặc ticket ghi trễ 5 ngày, order ghi 2 ngày thì phải `denied`.

## doanlong9304 — Shape C `orchestrator` + bảo mật/policy

**CP1**
- [ ] Cài môi trường, đọc `lab8/handoff.py` (allowlist), `docs/contracts.md` (Trace contract v1), `docs/business_rules.md`.

**CP2: `src/lab8/shapes/orchestrator.py`**
- [ ] Dựng handoff bằng `make_handoff` với **chỉ tool đọc**; facts không chứa `parent_secret`, header hay claim row đầy đủ.
- [ ] Chạy `run_readonly_investigators`; orchestrator quote từ order view của investigator, rồi gọi `issue_refund` với quote đầy đủ.
- [ ] Trace chỉ ghi **handoff đã sanitize**, không ghi parent context. Tóm tắt bằng LLM không được tính là redaction.
- [ ] `-k orchestrator` ra 62 passed.

**CP3**
- [ ] Truyền `crash_at`, không bắt `InjectedCrash`; T11 và T12 xanh.

**CP4 (15:30–16:15): rà must-pass cho cả 3 shape**
- [ ] Rà các nhánh approval: hết hạn, bị thu hồi, cấp cho claim khác, amount ≤ 0. Phải không có refund và phải có audit `policy_denied`.
- [ ] Rà trường hợp đơn đã hoàn đủ (`order_already_refunded`) và trường hợp cộng dồn ship + hỏng hàng (lấy số lớn hơn, không cộng hai số).
- [ ] Grep `traces/` và stdout tìm `CANARY_SECRET`, SĐT, địa chỉ, tên hàng, ghi chú nội bộ; phải không còn gì. Xóa mọi `print` debug in claim row hoặc order row thô.
- [ ] Viết test: handoff không chứa field ngoài allowlist hoặc `CANARY_SECRET_*`; investigator không có tool ghi.

**CP5**
- [ ] Làm chủ bút `group_report.md` (tối đa 5 trang): 4 kiến trúc, ranh giới code/model, durability, bảo mật, kết quả đo, giới hạn của MockLLM.

---

## Việc chung cả nhóm (CP5, 16:15–17:00)

- [ ] Cùng chọn shape bằng số đo. Mặc định nghiêng về **shape ít tự chủ nhất mà vẫn đạt** (thường là `workflow`), trừ khi số liệu nói khác. Ghi rõ điều kiện nào thì quyết định đảo lại.
- [ ] Mỗi con số trong ADR phải dò ngược được tới một file trong `reports/` và một `run_id`.
- [ ] Điền dòng của mình trong bảng đóng góp `TEAM.md` (commit và test) và file `report/team/<MSSV>_<HoTen>.md`.
- [ ] Không commit `traces/`, `*.sqlite`, `student_drop/`, `.venv/`, `.env`.

## Bonus (tùy chọn, chỉ làm khi đã xong bài chính)

Gate: shape chọn qua 3 must-pass, hidden ≥ **0.80**, `pytest -q` xanh. Đề xuất người nào xong shape sớm nhất thì nhận:

- **B7 (kiểm toán grader):** độc lập với shape, không cần `--extra-shape`, chỉ viết test trong `tests/test_bonus_grader_audit.py`. Ít rủi ro nhất.
- **B3 (swarm + verifier):** hợp nếu nhóm muốn phân tích sâu hơn về cost và variance.

Nộp code trong `src/bonus/`, test `tests/test_bonus_*.py`, `reports/bonus_*.json` và `report/team/bonus_report.md`.

## Anti-pattern cần tránh (từ slide)

- Để MockLLM quyết số tiền; copy khối `REFERENCE_WEAKNESS` của swarm.
- Thiếu trace ở nhánh return sớm; `try/except` nuốt `InjectedCrash`; lưu checkpoint rồi mới ghi tiền.
- Sửa test hoặc JSON cho xanh; chọn orchestrator vì "nghe xịn hơn"; chép số vào ADR mà không có `run_id`.
- Cả nhóm code chung trên một máy (ai cũng phải có commit riêng).
