# Team

Điền trước phút 15. Thay mọi placeholder; còn sót thì `make status` báo ở gate G1.

- **Team:** TODO (ví dụ `team07`)
- **Nhóm peer test bài của mình:** TODO (cặp A ↔ B; số nhóm lẻ thì ring 3 nhóm A → B → C → A — Lab Coach công bố)
- **Nhóm mình test bài của:** TODO
- **Problem family:** TODO (xem README mục "1 · Chọn bài toán")
- **Nguồn ảnh:** TODO (`bdd100k`, `gtsdb`, `lisa` — chỉ dùng ảnh trong `data/`)

| Thành viên | GitHub | Vai trò chính | File phụ trách |
|---|---|---|---|
| Nguyễn Tuấn Minh | [@TMTower18](https://github.com/TMTower18) | Team Lead / Spec Owner | `00_team.md`, `01_problem_statement.md`, `02_guideline.md` |
| Nguyễn Đình Bảo Phúc | [@NightFury002](https://github.com/NightFury002) | CVAT / Ontology Owner | `03_ontology_and_cvat_setup.md`, `03_cvat_labels.json` |
| Trần Đăng Ka Song | [@arksong1](https://github.com/arksong1) | Sample / Gold Owner | `sample_pack.csv`, `04_edge_cases/`, `gold_decisions.csv` |
| Bùi Thành Long | [@ThanhLong012](https://github.com/ThanhLong012) | QA / Calibration Owner | `05_qa_plan.md`, `06_calibration_report.csv` |
| Nguyễn Bình Dương | [@Duong43203](https://github.com/Duong43203) | Blind Test / Revision Owner | `07_blind_handoff/`, `08_revision_log.md`, `09_cvat_export_or_task_reference.txt` |

Gợi ý chia vai (nhóm 2–3 người thì gộp): **spec owner** (`01`, `02`), **CVAT owner** (`03_*`, `sample_pack.csv`,
`09`), **gold owner** (`04_edge_cases/`), **QA owner** (`05`, `06`, `07_blind_handoff/`). Mỗi file một người sửa
chính để tránh xung đột git. Calibration thì mọi người cùng label.
