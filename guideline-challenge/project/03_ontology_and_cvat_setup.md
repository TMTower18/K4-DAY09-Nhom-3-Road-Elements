# Ontology + CVAT setup

Bảng ontology là **source of truth** cho schema CVAT: `03_cvat_labels.json` phải khớp từng dòng ở đây. Thay mọi
placeholder mới là xong (gate G2).

## Ontology table

| Name | Geometry | Type (class / attribute) | Allowed values | Default | Mutable? | Rationale |
|---|---|---|---|---|---|---|
| `traffic_sign` | rectangle | class | n/a | n/a | n/a | Một instance tương ứng với một mặt biển giao thông vật lý; dùng rectangle axis-aligned và box visible-only theo guideline §2–3. |
| `family` | n/a | attribute (select) | `__undefined__`, `regulatory`, `warning`, `mandatory`, `information`, `other`, `unknown` | `__undefined__` | No | Lưu nhóm chức năng khi có bằng chứng; `unknown` tránh suy diễn từ màu hoặc hình dạng. |
| `type` | n/a | attribute (select) | `__undefined__`, `stop`, `yield`, `no_entry`, `speed_limit`, `other_regulatory`, `hazard_warning`, `other_warning`, `turn_direction`, `other_mandatory`, `direction`, `place_or_service`, `other_information`, `supplementary`, `other_sign`, `unknown` | `__undefined__` | No | Lưu loại cụ thể đến mức bằng chứng cho phép; `unknown` khi chưa đủ chi tiết. |
| `value` | n/a | attribute (select) | `__undefined__`, `not_applicable`, `unreadable`, `other_readable`, `5`, `10`, `20`, `30`, `40`, `50`, `60`, `70`, `80`, `90`, `100`, `110`, `120` | `__undefined__` | No | Chỉ dùng trị số cho `speed_limit`; số không đọc được là `unreadable`, không đoán số. |
| `visibility` | n/a | attribute (select) | `__undefined__`, `full`, `occluded`, `truncated`, `occluded_and_truncated` | `__undefined__` | No | Phân biệt che bởi vật thể và cắt bởi rìa ảnh; độ mờ không tự động là occlusion. |
| `legibility` | n/a | attribute (select) | `__undefined__`, `clear`, `partly_readable`, `unreadable` | `__undefined__` | No | Ghi mức đọc được nội dung trên mặt biển, độc lập với kích thước biển. |
| `decision` | n/a | attribute (select) | `__undefined__`, `label`, `unknown`, `escalate` | `__undefined__` | No | Phân biệt annotation đã chốt, chốt ở mức unknown và trường hợp cần QA. |
| `review_reason` | n/a | attribute (select) | `__undefined__`, `none`, `small_or_far`, `occluded`, `truncated`, `glare_or_blur`, `sign_vs_nonsign`, `hierarchy_conflict`, `geometry`, `other` | `__undefined__` | No | Ghi nguyên nhân chính khi `decision=escalate`; dùng `none` cho `label` hoặc `unknown`. |
| `image_escalate` | tag | tag | n/a | n/a | n/a | Tag ảnh cho nghi vấn không thể đặt box đáng tin cậy; lý do và `sample_id` được ghi trong QA log. |

## Class hay attribute

Chỉ có `traffic_sign` là class hình chữ nhật vì đây là đối tượng cần phát hiện và đo geometry. `image_escalate` là
tag ảnh vì dùng cho trường hợp nghi có biển nhưng không thể đặt box đáng tin cậy. `family`, `type`, `value`,
`visibility`, `legibility`, `decision` và `review_reason` là attributes của cùng một mặt biển; tách thành nhiều class
sẽ làm phình taxonomy, khó duy trì ràng buộc phân cấp và không phản ánh annotation unit.

Tất cả select attributes dùng default `__undefined__` để phát hiện annotation chưa hoàn tất trước khi lưu/export.
Default này không phải đáp án hợp lệ. Annotator phải thay bằng giá trị phù hợp; `unknown` và `unreadable` chỉ dùng khi
đã áp dụng cây quyết định trong guideline. `review_reason=none` chỉ hợp lệ cho `decision=label` hoặc `unknown`, còn
`decision=escalate` phải có lý do khác `none`.

## CVAT

- **Phiên bản CVAT** (`make cvat-status`): v2.74.1; đã xác nhận đang chạy tại `http://localhost:8080`.
- **Tên task calibration** (có version guideline, ví dụ `team07-calib-v1`): Chưa tạo trong repo/CVAT; sau khi tạo
	task, dùng tên theo mẫu `<team>-calib-v1` và ghi thêm task ID tại đây.
- **Guide của task đã dán `02_guideline.md`?** Chưa thể kiểm tra vì task calibration chưa được tạo. Trước khi label,
	dán nguyên nội dung `02_guideline.md` v1 vào Guide của task.
- **Nhóm dùng Track hay Shape, vì sao:** Shape. Đây là task ảnh tĩnh, mỗi ảnh độc lập và guideline §8 quy định không
	nối track hoặc suy nội dung từ ảnh liền kề.

## Setup test

Một thành viên **chưa tham gia setup** mở task và trả lời: label gì, dùng tool nào, gán attribute nào, khi nào
escalate. Ghi lại ai test và chỗ họ vấp:

Chưa thể thực hiện vì task calibration chưa được tạo. Sau khi task sẵn sàng, một thành viên không tham gia setup phải
mở task độc lập và trả lời đúng các điểm sau; người phụ trách ghi người test, ngày test và điểm vấp vào phần này:

- Label: `traffic_sign`; công cụ: Rectangle/Shape; không dùng Track.
- Attributes: `family`, `type`, `value`, `visibility`, `legibility`, `decision`, `review_reason`.
- Tag ảnh: `image_escalate` khi nghi có biển nhưng không thể đặt box đáng tin cậy.
- Escalate object: dùng box với `decision=escalate` và `review_reason` khác `none` khi còn bất đồng về đối tượng,
  phân loại, geometry hoặc giá trị ngoài ontology.
- Không vẽ box khi đối tượng chắc chắn ngoài scope, bị che hoàn toàn hoặc một chiều nhỏ hơn 6 px.
