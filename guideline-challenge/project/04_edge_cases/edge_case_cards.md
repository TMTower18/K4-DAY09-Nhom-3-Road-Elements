# Edge-case library

Tối thiểu **8 card**, khuyến nghị 10–12. Một edge case tốt là case mà hai annotator hợp lý có thể làm khác nhau nếu
guideline chưa rõ. Tám ảnh dễ có label rõ ràng không được tính là edge-case library.

Cần có đủ độ đa dạng: occlusion / truncation / small-far · ambiguous semantics · conflicting road elements · **một case
critical-risk** · **một case guideline cho phép escalation**.

File này là kho nội bộ của nhóm, **không gửi cho peer**. Card dùng ảnh example/calibration thì chép rule + ví dụ sang
`02_guideline.md` (mục 7 và 9) để peer đọc được. Card về ảnh blind chỉ nằm ở đây, và decision của nó phải có trong
`gold_decisions.csv` trước `make freeze`.

`make status` đếm số dòng `CASE ID:` đã điền (đã thay placeholder). Copy khối dưới cho mỗi case.

---

CASE ID: 1
Sample:  (sample_id)
Scene: 0756
Observation: Biển tròn 
Decision: UNKNOWN — LABEL / IGNORE / UNKNOWN / ESCALATE
Expected: Biển cấm — class, attribute, geometry cụ thể
Rationale: Gán `unknown` cho các mức phân loại không đủ chứng cứ pixel (biển nhỏ/xa) thay vì đoán mò. Điều này tuân thủ nguyên tắc tránh gán nhầm nội dung cụ thể có thể gây failure nghiêm trọng cho hệ thống downstream, tuân thủ khoản 2 và 3 của Downstream contract ở `01_problem_statement.md`.
Common mistake: Biển tròn 
Diversity: small far— occlusion / small_far / ambiguity / conflict / critical / escalation / …

---
Scene: 0959
Observation: Biển tam giác bị cắt 
Decision: IGNORE — LABEL / IGNORE / UNKNOWN / ESCALATE
Expected: Biển cảnh báo — class, attribute, geometry cụ thể
Rationale: Gán `unknown` cho các mức phân loại không đủ chứng cứ pixel (biển nhỏ/xa) thay vì đoán mò. Điều này tuân thủ nguyên tắc tránh gán nhầm nội dung cụ thể có thể gây failure nghiêm trọng cho hệ thống downstream, tuân thủ khoản 2 và 3 của Downstream contract ở `01_problem_statement.md`.
Common mistake: Biển cảnh báo 
Diversity: ambiguity— occlusion / small_far / ambiguity / conflict / critical / escalation / …

Scene: 1979
Observation: Biển hình chữ nhật 
Decision: IGNORE — LABEL / IGNORE / UNKNOWN / ESCALATE
Expected: Biển thông tin — class, attribute, geometry cụ thể
Rationale: Gán `unknown` cho các mức phân loại không đủ chứng cứ pixel (biển nhỏ/xa) thay vì đoán mò. Điều này tuân thủ nguyên tắc tránh gán nhầm nội dung cụ thể có thể gây failure nghiêm trọng cho hệ thống downstream, tuân thủ khoản 2 và 3 của Downstream contract ở `01_problem_statement.md`.
Common mistake: Biển hình chữ nhật 
Diversity: ambiguity— occlusion / small_far / ambiguity / conflict / critical / escalation / …

Scene: 0961
Observation: Biển tam giác bị cắt 
Decision: IGNORE — LABEL / IGNORE / UNKNOWN / ESCALATE
Expected: Biển cảnh báo — class, attribute, geometry cụ thể
Rationale: Gán `unknown` cho các mức phân loại không đủ chứng cứ pixel (biển nhỏ/xa) thay vì đoán mò. Điều này tuân thủ nguyên tắc tránh gán nhầm nội dung cụ thể có thể gây failure nghiêm trọng cho hệ thống downstream, tuân thủ khoản 2 và 3 của Downstream contract ở `01_problem_statement.md`.
Common mistake: Biển tam giác 
Diversity: ambiguity— occlusion / small_far / ambiguity / conflict / critical / escalation / …

 