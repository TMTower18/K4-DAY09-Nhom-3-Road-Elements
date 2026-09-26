# Problem statement + downstream contract

**Topic lock — Day 9:** Label traffic signs — hierarchical sign taxonomy cho biển nhỏ, xa hoặc bị che. **Phiên bản:** v1, 26/09/2026. Người phụ trách spec: Nguyễn Tuấn Minh (`@TMTower18`).

## Bài toán

Trên ảnh đường phố, phát hiện **từng mặt biển báo giao thông cố định** và phân loại theo mức chi tiết **cao nhất mà pixel còn nhìn thấy thực sự chứng minh được**. Biển nhỏ, ở xa, bị cây/xe che, lóa hoặc cắt mép ảnh có thể nhận ra là biển báo nhưng không đọc được loại hay trị số. Quy trình phải giữ được đối tượng để huấn luyện detector, đồng thời không tạo nhãn chi tiết sai do suy đoán.

## Downstream contract

1. **Người dùng:** nhóm huấn luyện và đánh giá hệ thống nhận diện biển báo trên ảnh camera giao thông; đầu ra phục vụ nghiên cứu perception, **không** trực tiếp quyết định điều khiển xe hay diễn giải hiệu lực pháp lý của biển.

2. **Đầu ra:** một bounding box hình chữ nhật cho **mỗi mặt biển**, kèm `family`, `type`, `value`, `legibility`. Taxonomy theo bậc `traffic_sign → family → type → value`; mức chưa xác định được ghi `unknown`, không đoán. `IGNORE` thể hiện theo quy tắc không vẽ với ca loại trừ rõ ràng trong guideline. Trường hợp **nghi có biển nhưng không thể đặt box đáng tin** dùng tag ảnh `image_escalate` (nếu schema có tag), kèm ảnh/sample_id trong log QA.

3. **Failure nghiêm trọng nhất (critical):** bỏ sót một biển nhìn đủ rõ để vẽ, đặc biệt biển có vẻ thuộc nhóm điều tiết; hoặc gán nhầm nội dung cụ thể như `stop`, `no_entry`, `speed_limit=60` khi pixel không đủ chứng cứ. Nhầm trị số/hướng/mệnh lệnh nguy hiểm hơn một nhãn `unknown` đúng quy tắc.

4. **Escalation:** annotator lưu lại ảnh/sample_id và box của trường hợp chưa thể xác định chắc chắn để QA owner Nguyễn Đình Bảo Phúc xem lại độc lập. Nếu QA vẫn không quyết định được, spec owner Nguyễn Tuấn Minh chốt theo bằng chứng hình ảnh, ghi quyết định và sửa `02_guideline.md`/`08_revision_log.md` trước vòng tiếp theo. Không sửa gold đã freeze theo kết quả peer.

## Scope và dữ liệu

- **Trong scope:** biển báo giao thông cố định có mặt biển nhìn thấy đủ để xác định vùng và kích thước tối thiểu **≥ 6 px theo mỗi chiều** ở độ phân giải ảnh gốc; kể cả biển cho chiều đường khác, biển bị che/cắt, hoặc nhiều biển trên một cột. Từng mặt biển là một instance. Chỉ dùng **ảnh trong `data/` của lab**, chọn `example/calibration/blind` trong `sample_pack.csv` sau khi kiểm tra ảnh thật; nguồn ảnh cụ thể sẽ được điền vào `00_team.md`.

- **Ngoài scope:** đèn tín hiệu, vạch đường, biển quảng cáo, biển tên cửa hàng, biển số xe, biển chỉ nhìn qua gương/phản chiếu, trụ/cột không có mặt biển, biển bị che hoàn toàn, mảng màu không đủ bằng chứng là biển, đối tượng < 6 px ở bất kỳ chiều nào. Không dùng luật giao thông theo quốc gia để suy ra phần chữ/số bị mất.

- **Geometry:** box axis-aligned ôm **phần mặt biển nhìn thấy** (visible-only), không gồm cột/khung đỡ và không ước lượng phần sau vật che. Clip box tại biên ảnh. Với đối tượng kiểm được, mục tiêu sai lệch mỗi cạnh ≤ max(2 px, 10% chiều tương ứng của box gold) ở ảnh gốc; QA chấm bằng đối chiếu hình, không coi đây là ngưỡng khoa học của dataset gốc.

## Output chấm được trong blind test

Mỗi gold item xác định `sample_id`, sự hiện diện hoặc không có box, `family`, `type`, `value`, `legibility`, và vị trí box khi có geometry. `unknown` là **giá trị phân loại** cho instance nhận diện được. Ảnh không có biển thật là negative đã kiểm, không suy từ export rỗng. Gold cần ít nhất 10 quyết định, 2 critical và 1 geometry trước `make freeze` theo README.

## Cơ sở nghiên cứu và giới hạn

Mapillary Traffic Sign Dataset báo cáo ảnh đường phố nhiều vùng địa lý với nhãn loại biển và bounding boxes; TT100K ghi nhận biển chỉ chiếm phần nhỏ ảnh, có biến đổi ánh sáng/thời tiết và cung cấp box/class/mask. GTSDB dùng ROI và class ID cho detection. Từ đó nhóm chọn box từng instance và phân loại phân cấp; **ngưỡng 6 px, taxonomy rút gọn, quy tắc unknown và tolerance là quyết định vận hành của nhóm**, cần hiệu chỉnh qua calibration, không phải tiêu chuẩn của các bộ dữ liệu. Không dùng ontology của một nước làm chân lý toàn cầu.

- Ertler et al., *The Mapillary Traffic Sign Dataset*, ECCV 2020: https://www.ecva.net/papers/eccv_2020/papers_ECCV/papers/123680069.pdf
- Zhu et al., *Traffic-Sign Detection and Classification in the Wild*, CVPR 2016 / TT100K: https://cg.cs.tsinghua.edu.cn/traffic-sign/
- Houben et al., *Detection of Traffic Signs in Real-World Images*, IJCNN 2013 / GTSDB: https://benchmark.ini.rub.de/gtsdb_dataset.html
- CVAT native image format (box, tag, attributes): https://docs.cvat.ai/docs/dataset_management/formats/format-cvat/