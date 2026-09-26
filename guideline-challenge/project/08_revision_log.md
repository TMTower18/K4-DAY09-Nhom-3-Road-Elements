# Revision log

Guideline v1 = bản nháp đầu; v2 = sau calibration nội bộ; v3 = sau blind handoff.

| Version | Đổi gì | Vì sao | Bằng chứng |
|---|---|---|---|
| v1 | Tạo guideline ban đầu cho bài toán Hierarchical Traffic Signs. | Tạo baseline để nhóm thực hiện calibration nội bộ. | Guideline v1. |
| v2 | Điều chỉnh ngưỡng biển nhỏ/xa: cạnh dài nhất của phần mặt biển nhìn thấy ≥ 10 px thì LABEL, < 10 px thì IGNORE. | Làm rõ tiêu chí xử lý biển nhỏ/xa sau calibration nội bộ. | Calibration nội bộ; bổ sung sample_id / dòng calibration report tương ứng. |
| v2 | Bổ sung quy tắc biển bị cắt khỏi khung: nếu hơn 80% mặt biển bị cắt và phần còn lại chỉ là viền thì IGNORE. | Làm rõ cách xử lý trường hợp truncation để giảm khác biệt giữa annotator. | Calibration nội bộ; bổ sung sample_id / dòng calibration report tương ứng. |
| v2 | Bổ sung chú thích tiếng Việt trực tiếp cho các giá trị `family`, `type`, `value`, `legibility`. | Giúp annotator hiểu và chọn attribute thống nhất hơn. | Calibration/clarification nội bộ. |
| v3 | Chưa cập nhật — chưa thực hiện blind handoff/kiểm chéo với nhóm peer. | Chưa có kết quả blind test và peer feedback để xác định thay đổi từ v2 → v3. | Chưa có peer feedback / blind score. |