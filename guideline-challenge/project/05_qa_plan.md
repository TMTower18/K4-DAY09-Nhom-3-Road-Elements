# QA plan + quality gates

**QA owner:** Bùi Thành Long (`@ThanhLong012`). **Áp dụng cho:** bản `02_guideline.md` hiện hành, schema `03_cvat_labels.json`
(`traffic_sign` + `family` / `type` / `value` / `legibility`, tag `image_escalate`).

QA bám theo downstream contract ở `01_problem_statement.md`: lỗi nặng nhất là **bỏ sót biển đủ rõ để vẽ** và **gán nội
dung cụ thể không có bằng chứng** (sai trị số tốc độ, `stop`, `no_entry`…). Một nhãn `unknown` đúng quy tắc luôn tốt hơn
một nhãn cụ thể sai.

## Flow

Guideline → Calibration → Production → Self-QC → Review → Rework → Quality Gate.

- **Self-QC (annotator, trước khi export):**
  1. Lọc object còn `__undefined__` ở bất kỳ attribute nào → phải bằng 0.
  2. Tự kiểm tổ hợp `family → type → value` theo bảng ràng buộc family → type của guideline (xem ghi chú ở Metrics).
  3. Zoom quét lại toàn ảnh theo lưới trái → phải, trên → dưới, tìm biển nhỏ và tấm phụ.
  4. Ctrl+S rồi export `CVAT for images 1.1`, không kèm ảnh.
- **Ai review, review bao nhiêu:**
  - **Reviewer chính:** QA owner (Long) review **100% ảnh và 100% box** của calibration và của export peer trong blind test
    — bộ ảnh nhỏ (5–8 ảnh calibration, 4–5 ảnh blind) nên review toàn bộ rẻ hơn rủi ro bỏ lọt.
  - **Review độc lập lần 2:** mọi item gắn `image_escalate`, mọi issue `Critical` và mọi item `Question` được một thành
    viên thứ hai (không phải người label ảnh đó) xem lại trước khi đóng.
  - **Khi scale lên production** (không áp dụng trong buổi lab): 100% ảnh của 20 ảnh đầu tiên của mỗi annotator mới;
    sau đó 20% ngẫu nhiên + 100% ảnh có tag rủi ro (xem dòng dưới).
- **Chọn sample theo rule nào:** ưu tiên theo rủi ro trước, ngẫu nhiên sau.
  - Luôn review: ảnh có tag `edge`, `critical`, `small_far`, `occlusion`, `ambiguity` trong `sample_pack.csv`; ảnh có
    `image_escalate`; ảnh có box `speed_limit` hoặc `family=regulatory`; ảnh có từ 5 biển trở lên.
  - Checklist rủi ro khi review từng ảnh:
    - biển phụ (tấm giờ, tấm loại xe, tấm chữ) phải là box riêng, `other/supplementary`;
    - biển có số không phải tốc độ ("3.5t", "2.7m") phải có `value=not_applicable`;
    - bảng tuyên truyền, bảng chỉ có tên đường, biển hiệu cửa hàng → không có box;
    - biển dưới ngưỡng kích thước §3, hoặc bị cắt gần hết chỉ còn viền → không có box;
    - biển nhỏ không đọc được → `unknown` + `legibility=unreadable`, không suy loại từ màu hoặc hình dạng.
- **Issue được ghi ở đâu, đóng thế nào:**
  - Ghi vào `project/05_qa_issue_log.csv`, mỗi lỗi một dòng: `issue_id, sample_id, object (toạ độ box hoặc mô tả),
    severity, defect, annotator, found_by, status, resolution`.
  - `status` đi theo: `open` → `fixed` (annotator sửa và export lại) → `verified` (reviewer mở export mới xác nhận) →
    đóng. `wont_fix` chỉ dùng cho `Question` đã được spec owner quyết định là không đổi, phải ghi lý do.
  - Issue chỉ được đóng khi reviewer đã xem **export mới**, không đóng theo lời báo của annotator.
- **Khi phát hiện guideline gap thì update và version ra sao:**
  - Issue được đánh `Question` + ghi `guideline_gap`; **không tính là lỗi của annotator**.
  - Spec owner (Minh) quyết định trong vòng đó: thêm rule, thêm ví dụ, hoặc thêm đường escalation vào `02_guideline.md`.
  - Gap phát hiện trong calibration → vào **v2**; gap phát hiện qua blind test → vào **v3**. Mỗi thay đổi thêm một dòng
    vào `08_revision_log.md` kèm bằng chứng (`sample_id`, dòng `06_calibration_report.csv` hoặc câu hỏi trong
    clarification log).
  - Sau `make freeze`: **không sửa gold và sample pack** theo kết quả QA; gold sai ghi `gold sai:` trong
    `transfer_score.csv` rồi đưa vào v3.

## Defect severity

Mapping theo downstream contract: sai nội dung cụ thể và bỏ sót biển điều tiết nặng hơn lỗi hình học.

| Severity | Định nghĩa cho project này | Ví dụ | Action mặc định |
|---|---|---|---|
| Critical | Bỏ sót một biển đạt ngưỡng vẽ và đọc được nội dung; **hoặc** gán `type`/`value` cụ thể mà pixel không chứng minh được; **hoặc** sai trị số tốc độ | Bỏ sót biển tốc độ 60 rõ nét; ghi `speed_limit=60` cho biển thực tế là 40; ghi `no_entry` cho biển tròn đỏ ở xa không đọc được | Rework toàn ảnh; review lại 100% ảnh khác của cùng annotator để tìm cùng loại lỗi; nếu từ 2 người trở lên mắc cùng lỗi → mở `Question` cho guideline |
| Major | Sai phân loại hoặc sai đơn vị đếm nhưng không tạo nội dung cụ thể sai; hoặc export không dùng được | Bỏ sót tấm phụ hoặc biển nhỏ không đọc được; vẽ box cho bảng tên đường hoặc biển dưới ngưỡng kích thước §3; gộp 2 biển thành 1 box, hoặc box biển chính ôm cả tấm phụ; sai `type` trong đúng `family`; tổ hợp `family/type/value` không hợp lệ; còn `__undefined__` | Sửa ảnh đó rồi reviewer verify lại |
| Minor | Lỗi không đổi ý nghĩa nhãn | Cạnh box lệch vượt `max(2 px, 10% cạnh gold)` nhưng vẫn ôm đúng mặt biển; `legibility` lệch 1 mức (`clear` ↔ `partly_readable`); để `unknown` trong khi nội dung đọc được (quá thận trọng) | Sửa trong lượt rework chung, không cần review lại riêng |
| Question | Guideline không đủ để quyết định, hoặc hai cách hiểu đều hợp lý theo văn bản hiện tại | Biển làn dành riêng có mũi tên: `turn_direction` hay `other_mandatory`; bảng xanh có tên đường: `information/direction` hay IGNORE; tag `image_escalate` | Không tính lỗi annotator; spec owner quyết định, sửa guideline, ghi `08_revision_log.md` |

## Metrics

Đối sánh export với gold theo **từng mặt biển**, do reviewer xác nhận bằng mắt trên CVAT: box được coi là khớp gold khi
cùng một mặt biển vật lý. Với biển có cạnh dài ≥ 20 px dùng thêm IoU ≥ 0.5; với biển nhỏ hơn, dùng điều kiện tâm box
nằm trong box gold, vì IoU của vật nhỏ quá nhạy với vài pixel.

| Metric | Cách tính | Vì sao phù hợp với bài toán |
|---|---|---|
| Sign recall | box khớp gold / tổng box gold | Bỏ sót biển là failure critical theo contract; detector cần đủ mẫu |
| False positive rate | box không khớp gold nào / tổng box annotator | Bắt box cho quảng cáo, bảng tên đường, biển dưới ngưỡng §3 |
| Family accuracy | box khớp có `family` đúng / box khớp | Cấp phân loại đầu tiên của taxonomy |
| Type accuracy | box khớp có `type` đúng / box khớp | Cấp phân loại chi tiết |
| Speed value accuracy | box `speed_limit` có `value` đúng / box `speed_limit` trong gold | Sai trị số là lỗi nguy hiểm nhất của contract |
| Over-specification rate | box khớp có `family`/`type`/`value` **cụ thể hơn** gold (gold là `unknown`/`unreadable`) / box khớp | Đo đúng rủi ro "đoán thay vì dùng `unknown`" |
| Completeness | object còn `__undefined__` / tổng object | Default chưa chọn = chưa làm xong |
| Hierarchy validity | box có tổ hợp `family → type → value` sai bảng ràng buộc / tổng box | CVAT không tự kiểm quan hệ giữa các attribute |
| Geometry pass rate | box khớp có mọi cạnh trong `max(2 px, 10% cạnh gold)` / box khớp | Tolerance đã chốt trong problem statement |

**Ghi chú:** Hierarchy validity cần bảng ràng buộc `family → type → value` trong guideline. Bảng này hiện **không còn**
trong `02_guideline.md`; tới khi spec owner khôi phục, QA chỉ kiểm theo định nghĩa từng `type` ở §4 và ghi tổ hợp đáng ngờ
thành issue `Question`, không tính lỗi annotator.

**Metric high-risk tách riêng:**

- **Critical defect escape rate** = số lỗi `Critical` phát hiện **sau khi** batch đã PASS QA (qua review lần 2 hoặc
  qua chấm blind) / tổng số lỗi `Critical` tìm được. Mục tiêu 0. Mỗi lần escape: ghi vào issue log và thêm mục đó vào
  checklist rủi ro ở trên.
- **Critical count** theo số tuyệt đối, không chỉ tỉ lệ: một bộ blind chỉ có khoảng 20–40 box, một lỗi đã là 2.5–5%.

## Quality gate

Đơn vị gate là **một batch** = một export của một annotator cho một task. Ngưỡng dưới đây là đề xuất của nhóm cho buổi
lab, không phải chuẩn ngành.

```text
PASS if:
  Critical = 0
  AND Completeness: __undefined__ = 0
  AND Hierarchy validity: 0 tổ hợp sai
  AND Sign recall >= 95%
  AND Speed value accuracy = 100%
  AND Over-specification rate <= 2%
  AND Family accuracy >= 95% AND Type accuracy >= 90%
  AND False positive rate <= 5%
  AND Geometry pass rate >= 90%
  AND Major <= 1 lỗi trên mỗi 20 box

REWORK if: Critical từ 1 đến 2, HOẶC còn __undefined__ / tổ hợp sai, HOẶC bất kỳ metric nào dưới ngưỡng PASS
  nhưng chưa chạm ngưỡng REJECT. Sửa các ảnh có lỗi rồi review lại 100% các ảnh đó; tối đa 2 vòng rework.

REJECT / ESCALATE if: Critical >= 3, HOẶC Sign recall < 85%, HOẶC Over-specification rate > 5%, HOẶC cùng một loại
  Critical lặp lại sau 1 vòng rework → label lại batch từ đầu. Nếu từ 2 annotator trở lên cùng mắc một lỗi → coi là
  guideline gap: escalate cho spec owner, sửa guideline và tăng version, không quy lỗi cho người label.
```

**Trade-off:**

- **Nghiêm với nội dung, nới với hình học.** Contract coi sai trị số hay mệnh lệnh nguy hiểm hơn box lệch vài pixel, nên
  Critical = 0 và Speed value accuracy = 100% là điều kiện cứng, còn geometry chỉ cần 90%. Biển nhỏ 10–20 px lệch 2 px
  đã vượt 10%; đặt geometry 100% sẽ bắt rework liên tục mà không cải thiện dữ liệu cho detector.
- **Recall trước precision.** Recall ≥ 95% chặt hơn false positive ≤ 5% không nhiều, nhưng ngưỡng REJECT của recall
  (85%) được đặt riêng vì bỏ sót biển làm hỏng cả detector lẫn đánh giá; box thừa dễ lọc hơn ở bước QA.
- **Over-specification ≤ 2% thay vì 0.** Ranh giới "đọc được / không đọc được" ở biển 10–20 px có tranh luận thật; đòi 0%
  sẽ đẩy annotator chọn `unknown` cho mọi thứ, làm giảm type accuracy. Vượt 5% thì coi là annotator đang đoán có hệ thống.
- **Chi phí review.** Review 100% hợp lý vì bộ lab nhỏ (≈ 10–13 ảnh). Ở quy mô production, chuyển sang review theo rủi ro
  + 20% ngẫu nhiên; khi Critical defect escape rate > 0 ở hai batch liên tiếp thì quay lại 100% cho annotator đó.
