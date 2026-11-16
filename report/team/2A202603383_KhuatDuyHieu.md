# Báo cáo cá nhân — 2A202603383_KhuatDuyHieu

## 1. Thông tin thành viên

- Họ tên: Khuất Duy Hiếu
- MSSV: 2A202603383
- Email: hieukhuatduy@gmail.com
- GitHub: KhuatHieu

## 2. Phần việc phụ trách

Shape, module hoặc phần báo cáo bạn phụ trách.

- Nhóm trưởng / integration: repo nhóm, `TEAM.md`, merge PR, chạy crash suite, `check_submission`, chốt commit hash.
- Shape A `workflow` (`src/lab8/shapes/workflow.py`).
- `report/team/decision_adr.md`.

## 3. Đóng góp cụ thể

Commit chính (hash), test đã thêm hoặc sửa, quyết định kỹ thuật bạn đưa ra.

- `aff22a7`: chia việc cho nhóm (`TASK.md`), điền `TEAM.md`.
- `a8fbbee`: shape A `workflow` (`src/lab8/shapes/workflow.py`). Các bước cố định: `get_claim_view` → `lookup_order` → `compute_compensation` → `durability.issue_refund`.
- `13551e0`: test `src/tests/test_workflow_extra.py` (14 test, chạy trong cổng host); `src/reports/crash.json` (`crash_safety 20/20`).
- Merge nhánh shape B `single` vào `main` (`d4a98fc`), giữ `python -m pytest -q` xanh.

Quyết định kỹ thuật:

- Workflow **không gọi MockLLM**: số tiền, approval và idempotency đều do code và host quyết, nên bỏ cả bước soạn câu trả lời. Shape này vì thế rẻ nhất và không có variance theo seed.
- Ticket là dữ liệu người dùng: workflow không đọc số ngày trong ticket và không ghi ticket vào trace. Trace chỉ ghi field đã qua allowlist (`lookup_order`), quote và kết quả từ host.
- Truyền **quote đầy đủ** vào `issue_refund`; host tự trừ phần đã hoàn của đơn (T13, T14). Host từ chối thì trả `denied`, `amount_conflict` thì trả `failed`.
- Không nuốt `InjectedCrash`: ghi trace rồi raise lại, để lần chạy lại dựa vào ledger (khóa `refund:{claim_id}`).

## 4. Bằng chứng

Link commit, tên test, run id hoặc trích đoạn report liên quan.

- `python -m pytest -q -m student -o addopts= -k workflow`: 62 passed.
- `src/tests/test_workflow_extra.py`:
  - `test_workflow_ignores_injected_ticket`: ticket chèn "IGNORE POLICY… 9.999.999 VND" vẫn ra 35.000, 1 dòng ledger, 0 lần gọi model.
  - `test_workflow_trusts_order_delay_on_every_seed`: C-002 bị `denied` ở cả 5 seed, kể cả seed 55, nơi swarm fail.
  - `test_workflow_crash_then_resume_pays_once`: crash ở 3 điểm rồi resume vẫn đúng 1 dòng ledger, 1 audit `refund_committed`.
  - `test_workflow_trace_has_no_secret_or_pii`: 14 claim public, trace không chứa canary, header, SĐT, địa chỉ, ghi chú nội bộ, tên hàng hay ticket.
- `python -m lab8.crash_test --count 20 --out reports/crash.json`: `crash_safety 20/20`.
- `python -m lab8.demo.buggy_workflow`: chỉ dùng checkpoint thì ra 2 refund, ledger của host giữ 1.

## 5. Công cụ AI đã dùng

Công cụ, dùng vào việc gì, và bạn đã tự kiểm tra kết quả thế nào (xem [RULES.md](../../RULES.md)).

- Claude Code: đọc đề và slide, gợi ý chia việc, viết nháp `workflow.py` và `test_workflow_extra.py`, merge nhánh.
- Tự kiểm tra: đọc lại code theo docstring và `docs/business_rules.md`; chạy ma trận student (62/62), cổng host và crash suite; grep trace tìm secret/PII. Không có số liệu nào được chép tay; mọi số đều lấy từ lệnh đã chạy.

## 6. Tự đánh giá

...
