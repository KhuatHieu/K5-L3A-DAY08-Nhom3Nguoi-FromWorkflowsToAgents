# Team

## Thông tin nhóm

- Tên nhóm: Nhom3Nguoi
- Lớp: L3A
- Repo nhóm: `K5-L3A-DAY08-Nhom3Nguoi-FromWorkflowsToAgents` (xem [SUBMISSION.md](SUBMISSION.md))
- URL repo, nhánh nộp, commit chốt: https://github.com/KhuatHieu/K5-L3A-DAY08-Nhom3Nguoi-FromWorkflowsToAgents · `main` · (chốt ở CP5)

## Thành viên

| STT | Họ tên | MSSV | Email | GitHub | Vai trò |
| --: | ------ | ---- | ----- | ------ | ------- |
|   1 | Khuất Duy Hiếu | 2A202603383 | hieukhuatduy@gmail.com | KhuatHieu | Nhóm trưởng, integration; shape A `workflow`; crash suite, ADR, nộp bài |
|   2 | Lê Văn Hoàng | 2A202603357 | hoanglv2k4@gmail.com | hoanglv2k9 | Shape B `single`; benchmark, tự kiểm hidden, research card |
|   3 | Đoàn Gia Long      | 2A202603443     |  doanlong932004@gmail.com     | doanlong9304 | Shape C `orchestrator` + handoff; rà must-pass (approval, secret/PII), group report |

Gợi ý chia vai:

- Nhóm 6 người: integration; workflow; single agent; orchestrator; ledger/policy/crash; eval/report.
- Nhóm 4 người: workflow + integration; single; orchestrator + handoff; ledger + evaluation.

## Bảng đóng góp

`python -m lab8.check_submission` đọc bảng này bằng máy: giữ nguyên tiêu đề `## Bảng đóng góp` và các cột. Mỗi người một dòng: commit có nội dung và ít nhất một test đã thêm hoặc sửa.

| Người | Vai | Commit | Test đã thêm hoặc sửa |
| --- | --- | --- | --- |
| Khuất Duy Hiếu (KhuatHieu) | Nhóm trưởng; shape A `workflow` | `a8fbbee` workflow shape | `src/tests/test_workflow_extra.py` (injection, delay từ order, crash resume, trace sạch) |
| Lê Văn Hoàng (hoanglv2k9) | Shape B `single`; benchmark, research card | `d04bba4` single shape, `f458875` benchmark (nhánh `2A202603357_LeVanHoang`) | `src/tests/test_single_extra.py` (ticket injection, C-002 trên 5 seed, `step_cap`, tool chỉ đọc, crash resume, trace sạch) |
| Đoàn Gia Long (doanlong9304) | Shape C `orchestrator`; bảo mật/policy; group report | `82936fe` orchestrator shape, `6a2888d` test handoff, `5c3eb55` benchmark 4 shape | `src/tests/test_orchestrator_handoff.py` (handoff chỉ field allowlist, trace không có CANARY/PII trên 14 claim, make_handoff từ chối tool ghi, investigator không ghi được kể cả khi handoff bị sửa) |

## Cá nhân

Mỗi thành viên đồng thời tạo một báo cáo riêng trong `report/` theo dạng:

```text
report/team/MSSV_HoTen.md
```

Ví dụ: `report/team/20241234_NguyenVanAn.md`.

Sao chép mục dưới đây cho từng thành viên.

## MSSV_HoTen

### Phần việc phụ trách

-

### Đóng góp cụ thể

-

### Bằng chứng

-
