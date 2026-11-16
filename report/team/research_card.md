# Research card

File này nằm cố định ở `report/team/research_card.md`; `python -m lab8.check_submission` đọc cột "Nhóm điền" bằng máy.

Phần này nối tiếp phần mini research ở buổi lý thuyết (slide 62–63). Chọn **một** câu R1–R4, điền 6 dòng, rồi biến nó thành **một ablation** chạy được trong lab: giữ nguyên model (MockLLM), tool, seed, corpus và giá, chỉ thay đổi một biến.

| Câu | Nguồn (thư viện tham khảo, slide 68–69) | Câu hỏi gợi ý cho lab |
| --- | --- | --- |
| R1 · Scaling | Google Research 2026, *Towards a Science of Scaling Agent Systems* | Task claims chủ yếu tuần tự hay song song được? Dự đoán thêm investigator sẽ làm pass rate và cost/success thay đổi thế nào. |
| R2 · Failure taxonomy | MAST, NeurIPS 2025, *Why Do Multi-Agent LLM Systems Fail?* | Các trial fail của swarm (seed 55) thuộc nhóm nào: lỗi giao việc, lệch mục tiêu, hay lỗi kiểm chứng? |
| R3 · Architecture choices | AgentArch, AACL 2026 | Nếu đổi topology mà giữ nguyên mọi thứ khác, biến nào phải cố định để kết luận còn đúng? |
| R4 · Evaluation confounders | EMNLP 2026, *Rethinking Evaluation of MAS Efficiency* | Làm sao chứng minh chênh lệch giữa A/B/C/D là do topology, không phải do prompt, tool hay seed? |

## Phiếu của nhóm

Điền vào cột cuối. Không xóa hai cột đầu.

| Field | Hướng dẫn | Nhóm điền |
| --- | --- | --- |
| CÂU | R1 / R2 / R3 / R4 | R2 |
| SOURCE | Paper/blog, năm, URL, loại bằng chứng (benchmark, production report, blog kinh nghiệm) | MAST (NeurIPS 2025), *Why Do Multi-Agent LLM Systems Fail?*, https://arxiv.org/abs/2503.13657, nghiên cứu thực nghiệm phân tích hơn 1.600 trace trên 7 framework MAS |
| CLAIM | Một kết luận gốc của nguồn. Không viết rộng hơn phạm vi paper | Lỗi trong MAS phần lớn bắt nguồn từ thiết kế hệ thống, lệch phối hợp giữa các agent và thiếu bước kiểm chứng (14 failure mode chia 3 nhóm), nên chỉ thêm agent hay sửa nhẹ prompt không loại bỏ được lỗi hệ thống |
| BOUNDARY | Model / dataset / setup giới hạn kết luận đó | Phân tích trên 7 framework MAS mã nguồn mở và 1.600+ trace trong các tác vụ coding, toán và general agent; sang bài toán LogiX-VN dùng dữ liệu tổng hợp, `MockLLM` có seed, tool đọc cố định và host SQLite độc lập |
| HYPOTHESIS | Nếu đổi shape A→B (hoặc bật/tắt một thành phần), metric X sẽ đổi theo hướng nào, khoảng bao nhiêu | Khi giữ `compute_compensation` gắn chặt với order row trong DB và không tin chữ trong ticket (`workflow` A, `single` B, `orchestrator` C), pass rate giữ `1.000`, `pass^5` = `1.000`, variance = `0`; ngược lại nếu để agent lấy `delay_days` từ ticket khi `seed % 5 == 0` mà thiếu kiểm chứng chéo với DB (`swarm` D), lỗi nhóm *Task Verification / Incomplete Verification* xuất hiện ở T02 seed 55 làm `pass_rate` giảm xuống dưới `1.0`, `pass^5` giảm còn ~`0.93` và `cost/success` tăng |
| METHOD | Biến giữ cố định: model, tools, seed, corpus, giá, evaluator. Biến duy nhất được thay đổi: … | Biến giữ cố định: `MockLLM`, danh sách tool đọc, `seeds = [11, 22, 33, 44, 55]`, corpus `data/public_claims.jsonl` (14 claim, 70 trial/shape), `as_of = 2026-11-15`, `config/pricing.yaml`, evaluator `lab8.metrics`, mỗi trial một SQLite mới. Biến duy nhất thay đổi: kiến trúc shape (`workflow` vs `single` vs `orchestrator` vs `swarm`) và cơ chế kiểm chứng dữ kiện `delay_days` trước khi gọi `compute_compensation` |
| FAILURE / METRIC | Một lỗi dự kiến và cách đo nó: pass rate, pass^5, variance, cost/success, leak, bypass | Lỗi dự kiến ở T02 (`C-002` seed 55, `R-swarm-C-002-s55`): ticket ghi trễ 5 ngày nhưng order row chỉ trễ 2 ngày, khiến agent trả `resolved` (hoàn 20.000 VND) thay vì `denied`. Đo bằng `pass_rate`, `pass^5`, `seed_variance`, `seed_min/max`, `cost_per_success_vnd`, `mean_steps` trong `src/reports/benchmark.json` và `trial_pass` (kiểm leak/approval bypass) |

## Kết quả ablation

- Lệnh đã chạy (copy nguyên văn): `python -m lab8.benchmark --shapes workflow,single,orchestrator,swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/benchmark.json` (chạy trong `src/`, đủ cả 4 shape)
- Run id / file report: `src/reports/benchmark.json` (`R-workflow-C-002-s55`, `R-single-C-002-s55`, `R-orchestrator-C-002-s55`, `R-swarm-C-002-s55`)
- Kết quả so với hypothesis (đúng / sai / không kết luận được), kèm số liệu: **Đúng.** Giữ nguyên mọi biến kiểm soát trên đủ 4 shape: `workflow` đạt `pass_rate = 1.000` (70/70), `pass^5 = 1.000`, `seed_variance = 0.000000`, `cost/success = 34.29 VND`, `mean_steps = 4.43`; `single` đạt `pass_rate = 1.000` (70/70), `pass^5 = 1.000`, `seed_variance = 0.000000`, `cost/success = 45.86 VND` (+33.7%), `mean_steps = 8.07`; `orchestrator` đạt `pass_rate = 1.000` (70/70), `pass^5 = 1.000`, `seed_variance = 0.000000`, `cost/success = 57.19 VND` (+66.8%), `mean_steps = 4.36`; `swarm` đạt `pass_rate = 0.986` (69/70), `pass^5 = 0.929` (13/14), `seed_variance = 0.000816` (`seed_min = 0.929`, `seed_max = 1.000`), `cost/success = 49.57 VND`, `mean_steps = 2.80`. Đúng 1 trial fail ở `R-swarm-C-002-s55` (`status resolved != denied`, cost `62.22 VND`) thuộc lỗi kiểm chứng không đối chiếu lại `delay_days` từ DB, trong khi `R-workflow-C-002-s55` (`30.00 VND`), `R-single-C-002-s55` (`43.14 VND`) và `R-orchestrator-C-002-s55` (`61.44 VND`) đều `denied` đúng.
- Giới hạn: MockLLM không phải model sản xuất, latency là `unavailable`; bộ public gồm 14 claim × 5 seed (70 trial/shape). Trên `student_drop/` (40 hidden claim × 5 seed không nhãn, `src/reports/hidden_selfcheck.json`), cả `workflow`, `single` và `orchestrator` đều đạt `0 error_trials`, `0 leak_suspect_trials` và phân bố `status_counts` trùng khớp (`denied`: 95, `resolved`: 90, `needs_info`: 5, `escalated`: 10).

Muốn đi xa hơn một ablation để lấy điểm cộng thì xem [docs/bonus_topics.md](../../docs/bonus_topics.md).
