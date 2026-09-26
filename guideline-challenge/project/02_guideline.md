# Annotation guideline — Hierarchical traffic signs

**Version:** v1 — 26/09/2026. **Spec owner:** Nguyễn Tuấn Minh (`@TMTower18`). **Task:** ảnh tĩnh, từng mặt biển báo giao thông. Đây là bản dùng cho calibration; chỉ đổi thành v2 sau khi thảo luận bất đồng và ghi `08_revision_log.md`, rồi v3 sau blind test. Chỉ dùng ví dụ ảnh trong split `example/calibration`; bảng ví dụ cuối file hiện là **tình huống giả định**, phải thay bằng `sample_id` thật trước handoff.

## 1. Objective + scope

Tạo dữ liệu detection và phân loại biển báo **theo bằng chứng nhìn thấy trên ảnh**, gồm cả ca nhỏ/xa/bị che. Vẽ box nếu vừa đủ tin rằng đó là mặt biển giao thông và đủ đặt vùng theo §3. Phân loại cụ thể đến đâu chứng cứ cho phép đến đó; `unknown` tốt hơn một phỏng đoán. Không đánh giá biển có hiệu lực với xe trong ảnh hay có tuân thủ luật quốc gia nào. Cùng một quy tắc áp dụng cho mọi split.

**Bao gồm:** biển báo vật lý cố định bên đường/trên giá long môn/trên cột, mặt trước hoặc mặt sau nhận ra chắc chắn, trong và ngoài làn xe của camera; biển lặp lại và nhiều biển trên một cột; mặt biển tạm thời phục vụ giao thông nếu thực sự là biển báo. **Không bao gồm:** đèn giao thông, vạch mặt đường, quảng cáo/biển hiệu doanh nghiệp, bảng tên đường chỉ là tên phố không có chỉ dẫn giao thông, cột/giá đỡ riêng, phản chiếu trong kính/gương, biển in trên xe, biểu tượng ứng dụng phủ lên ảnh, đối tượng che hoàn toàn. Nếu bộ ảnh có bảng chỉ hướng giao thông chứa chữ địa danh, gán `information/direction` khi đủ bằng chứng là biển chỉ dẫn đường; không loại trừ chỉ vì có tên địa danh.

## 2. Annotation unit

Một **instance = một mặt biển vật lý** trong một ảnh, kể cả hai biển giống nhau. Một biển chứa nhiều biểu tượng/chữ trên cùng một mặt vẫn là **một box**; không vẽ box cho từng ký tự, trị số hay mũi tên. Hai tấm biển ghép trên một cột là hai box nếu có thể thấy ranh giới hai mặt; tấm phụ độc lập là box riêng, `family=other`, `type=supplementary` nếu nhận ra. Nếu không phân biệt được số tấm do chồng lấp, vẽ từng tấm thấy ranh giới; phần còn lại escalation, không nhân bản box theo suy đoán. Chỉ dùng **Shape**, không dùng Track.

## 3. Geometry rule

- Dùng **rectangle axis-aligned** cho từng mặt biển. Box bao sát các pixel **mặt biển nhìn thấy**, kể cả viền gắn liền với mặt, không gồm cột, giá đỡ, bóng đổ hay phần nền. Không kéo box ra để đoán phần khuất (visible-only); không cần polygon/mask.
- Lấy `(x_min,y_min)` là góc trái trên và `(x_max,y_max)` là góc phải dưới của vùng thấy, tại **độ phân giải gốc**. Biển nghiêng vẫn dùng hộp chữ nhật không xoay bao vùng mặt biển; biển tròn thì hộp tiếp xúc gần nhất với mép ngoài. Biển cắt bởi biên ảnh: đặt cạnh box ngay tại biên, không kéo ra ngoài ảnh.
- Bị cây/xe che ở giữa: box ôm biên của các phần **nhìn thấy thuộc cùng một mặt**, có thể chứa vùng vật che giữa chúng; không vẽ box cho từng mảnh. Nếu chỉ còn một góc có thể xác định mặt biển, box chỉ ôm góc đó và escalate khi không rõ ranh giới. Mặt sau biển: box phần mặt sau nếu chắc chắn là mặt biển, phân loại `family=unknown`.
- Chỉ vẽ khi cả chiều rộng và cao phần mặt biển có thể bao **≥ 6 px** ở ảnh gốc. Đúng 6 px thì vẽ; dưới 6 px ở một chiều thì IGNORE. Đây là ngưỡng thao tác, không có nghĩa biển quá nhỏ không quan trọng trong thực tế. Zoom trong CVAT để xem, không dùng upscaling/AI hay biển ở ảnh/frame khác để suy ra chữ số. Mục tiêu QA: từng cạnh sai ≤ `max(2 px, 10% kích thước cạnh tương ứng của box gold)`. Với vật nhỏ, đối chiếu hình và vị trí, vì IoU rất nhạy với vài pixel.

## 4. Taxonomy và attributes

**Schema triển khai đề xuất như sau:** đúng **một CVAT label hình chữ nhật tên `traffic_sign`**, với các select attributes bất biến ở task ảnh tĩnh dưới đây. Phân cấp lưu bằng attributes, không tạo hàng chục label con. Có thêm **một tag ảnh** `image_escalate` cho nghi vấn không thể đặt box. `03_ontology_and_cvat_setup.md` là source of truth; Long phải đồng bộ `03_cvat_labels.json` *trước* tạo task, nếu tên/giá trị khác thì sửa hai bên và guideline đồng thời, không bắt peer tự dịch schema. Không dùng checkbox `needs_review` làm nguồn duy nhất nếu không có lý do.

| Attribute | Giá trị cho phép | Default | Ý nghĩa |
|---|---|---|---|
| `family` | `__undefined__`, `regulatory`, `warning`, `mandatory`, `information`, `other`, `unknown` | `__undefined__` | Nhóm chức năng khi đọc rõ; không suy từ hình dạng/màu đơn lẻ. `unknown` nếu chỉ biết chắc là biển. |
| `type` | `__undefined__`, `stop`, `yield`, `no_entry`, `speed_limit`, `other_regulatory`, `hazard_warning`, `other_warning`, `turn_direction`, `other_mandatory`, `direction`, `place_or_service`, `other_information`, `supplementary`, `other_sign`, `unknown` | `__undefined__` | Loại cụ thể hoặc nhóm con có chứng cứ; dùng `unknown` nếu không đủ đọc loại. |
| `value` | `__undefined__`, `not_applicable`, `unreadable`, `other_readable`, `5`, `10`, `20`, `30`, `40`, `50`, `60`, `70`, `80`, `90`, `100`, `110`, `120` | `__undefined__` | Chỉ ghi trị số tốc độ cho `speed_limit`; không đọc được số dùng `unreadable`; type khác dùng `not_applicable`. |
| `legibility` | `__undefined__`, `clear`, `partly_readable`, `unreadable` | `__undefined__` | Mức độ đọc/nhận biết nội dung trên mặt biển. |


**Ràng buộc taxonomy:**

| `family` | `type` được phép | `value` |
|---|---|---|
| `regulatory` | `stop`, `yield`, `no_entry`, `speed_limit`, `other_regulatory`, `unknown` | `speed_limit`: số đọc đủ / `unreadable` / `other_readable`; các type khác: `not_applicable` |
| `warning` | `hazard_warning`, `other_warning`, `unknown` | `not_applicable` |
| `mandatory` | `turn_direction`, `other_mandatory`, `unknown` | `not_applicable` |
| `information` | `direction`, `place_or_service`, `other_information`, `unknown` | `not_applicable` |
| `other` | `supplementary`, `other_sign`, `unknown` | `not_applicable` |
| `unknown` | `unknown` | `not_applicable` |

`other_*` nghĩa là **biết chắc họ type/family nhưng không thuộc các loại kể tên**. `unknown` nghĩa là **không đủ chứng cứ để biết**. Màu/hình tam giác, tròn, chữ nhật là gợi ý tìm biển, **không tự đủ để xác định family** vì quy ước các nước và mặt sau có thể khác. `hazard_warning` chỉ khi thấy rõ nội dung cảnh báo; không tự điền chỉ vì tam giác. `turn_direction` chỉ khi thấy mũi tên/chỉ dẫn bắt buộc; `direction` là biển chỉ đường thông tin. `place_or_service` cho tên địa điểm/dịch vụ trên biển giao thông. `other_readable` cần QA ghi số thực trong log; không âm thầm chuyển thành một số có trong menu.

## 5. Inclusion / exclusion — cây quyết định

1. Đây có phải **mặt biển giao thông vật lý** theo §1? Nếu rõ là đối tượng ngoài scope: **IGNORE**, không vẽ. Nếu không chắc biển hay quảng cáo: sang bước 2.
2. Có đủ dấu hiệu ảnh để xác định **vị trí và ranh giới mặt biển**, hai chiều ≥ 6 px? Nếu có: vẽ box. Nếu nghi biển nhưng không thể đặt box: gắn tag `image_escalate`, ghi `sample_id` và lý do trong log QA, **không vẽ box tưởng tượng**. Nếu chỉ là đốm không có bằng chứng là biển: IGNORE.
3. Xác định family bằng nội dung thấy rõ; chưa đủ thì `family=unknown`, `type=unknown`. Nếu biết family mà chưa biết type: `type=unknown`. Nếu biết type speed limit nhưng không đọc số: `value=unreadable`. Không dùng ngữ cảnh đường, ảnh khác hay thứ tự biển để lấp chi tiết thiếu.
4. Đặt `visibility`, `legibility`. `decision=label` khi type và value cần thiết đã rõ; `decision=unknown` khi có ít nhất một cấp nội dung dừng ở `unknown` hoặc `speed_limit/unreadable`; `decision=escalate` khi còn bất đồng về **có phải biển**, family/type, box, hoặc giá trị `other_readable`. Chọn `review_reason`. Escalate không có nghĩa trì hoãn lưu annotation.

## 6. Visibility, occlusion, nhỏ/xa

- `full`: mặt không bị vật che hay cắt rìa; dù mờ do khoảng cách vẫn có thể `legibility=unreadable`.
- `occluded`: cây, xe, người, biển khác che một phần mặt. Nếu thấy nhiều mảnh và nhận ra cùng một mặt, dùng một box. Che hoàn toàn: không có box; nếu nghi còn biển tại vị trí đó nhưng không thấy pixel biển, vẫn IGNORE.
- `truncated`: mặt bị mép ảnh cắt; cả che lẫn cắt dùng `occluded_and_truncated`. **Không** đánh dấu occluded chỉ vì camera không lấy nét, mưa, nén ảnh hoặc lóa.
- Biển nhỏ/xa nhưng ≥ 6 px và chắc là biển: box + `unknown` ở cấp không đọc được. Biển ≥ 6 px nhưng bị lóa: box, `legibility=unreadable` nếu không đọc được, `review_reason=glare_or_blur` chỉ khi escalate. Mặt sau: `family=unknown`, `type=unknown`, `decision=unknown`.
- Một chữ số thấy rõ, chữ số còn lại khuất: `speed_limit/unreadable`, tuyệt đối không hoàn thành con số bằng kiến thức phổ biến. Nếu không chắc đó là biển tốc độ, dừng ở family (hoặc unknown).

## 7. Ambiguity / escalation và cách kiểm export

| Quyết định | Thao tác CVAT | Khi nào |
|---|---|---|
| LABEL | một box `traffic_sign`, `decision=label`, attributes đầy đủ | loại và nội dung có đủ bằng chứng |
| UNKNOWN | một box, cấp chưa rõ = `unknown` / số = `unreadable`, `decision=unknown` | chắc là biển và đặt được box; không đủ chi tiết |
| ESCALATE object | box + `decision=escalate`, `review_reason` khác `none` | QA cần chốt class, phân biệt sign/non-sign, box, trị số ngoài menu |
| ESCALATE image | tag `image_escalate` + log QA có `sample_id` | nghi biển nhưng không thể đặt box đáng tin; export có tag, lý do nằm trong log |
| IGNORE | không vẽ box/tag cho đối tượng chắc chắn ngoài scope | ví dụ quảng cáo hoặc quá nhỏ; kiểm bằng gold hình ảnh, không kết luận từ export rỗng một mình |

**Thứ tự xử lý mơ hồ:** xem ảnh gốc → zoom → áp dụng cây quyết định → nếu vẫn khó, chọn mức phân loại cao hơn (`unknown`) → escalate nếu phải xác minh ranh giới/đối tượng hoặc sửa ontology. Annotator không hỏi tác giả guideline để nhận “luật ngầm” trong blind test; câu hỏi được ghi vào clarification log. Trước export, lọc mọi object còn `__undefined__`: đây là **chưa làm xong**, không phải một đáp án. QA kiểm tổ hợp family/type/value theo bảng ràng buộc, vì CVAT select không tự kiểm quan hệ giữa attributes.

## 8. Temporal rule

**Không áp dụng — task ảnh tĩnh.** Mỗi ảnh là độc lập; không nối track, không suy nội dung biển từ ảnh liền kề hoặc ground truth nguồn. Nếu đổi sang clip video, phải thiết kế lại temporal rule, ontology mutable và export; bản guideline này không áp dụng tự động.

## 9. Examples — tình huống minh họa, chưa phải sample của lab

`sample_pack.csv` hiện chỉ có header, nên chưa có `sample_id` thật. **Trước calibration:** gắn các mã `example`/`calibration` có thật, thêm ảnh thật và expected box. **Không dùng ảnh blind làm ví dụ.** Các dòng sau là case mô tả để thống nhất cách ra quyết định, không là evidence đã quan sát.

| Case giả định | Thấy gì | Expected output | Rule |
|---|---|---|---|
| EX-01 | mặt biển STOP rõ, 30×30 px | 1 box; `regulatory/stop/not_applicable`, `full/clear/label/none` | §3–5 |
| EX-02 | biển giới hạn 60 rõ | 1 box; `regulatory/speed_limit/60`, `label` | §4–5 |
| EX-03 | biển tốc độ, chữ số bị tán cây che | 1 box visible-only; `regulatory/speed_limit/unreadable`, `occluded`, `unknown` | §3, §6 |
| EX-04 | mặt biển chắc chắn, 7×8 px, nội dung không đọc | 1 box; `unknown/unknown/not_applicable`, `full/unreadable/unknown` | §3, §5 |
| EX-05 | mặt biển chắc chắn, 5×9 px | IGNORE vì chiều rộng < 6 px | §3 |
| EX-06 | biển thông tin bị cắt rìa, thấy chữ và mũi tên chỉ đường | 1 box clipped; `information/direction/not_applicable`, `truncated`; legibility theo phần đọc được | §3, §6 |
| EX-07 | hai tấm biển trên cùng cột, thấy ranh giới riêng | 2 box, mỗi tấm attributes riêng | §2 |
| EX-08 | bảng quảng cáo tròn đỏ không phải biển giao thông | IGNORE | §1, §5 |
| EX-09 | giống biển nhưng vật che khiến không thể đặt box | tag ảnh `image_escalate` + log QA, không phỏng đoán box | §5, §7 |
| EX-10 | mặt sau một biển chắc chắn, đủ lớn | 1 box; `unknown/unknown/not_applicable`, `decision=unknown` | §4, §6 |

## 10. Common mistakes + checklist

- **Đoán số** tốc độ từ một chữ số hoặc biển trên khung đường: chuyển `value=unreadable`.
- **Đánh đồng hình dáng với ý nghĩa:** chỉ gán family/type khi nội dung đủ bằng chứng; màu/shape chỉ giúp phát hiện ứng viên.
- **Box trùm cả cột/phần bị che:** chỉnh về mặt biển nhìn thấy; không vẽ amodal.
- **Gộp hai biển hoặc cắt một biển thành nhiều box:** đếm từng mặt vật lý theo §2.
- **Quên biển nhỏ:** zoom, áp dụng ngưỡng ở ảnh gốc; ≥ 6 px không phải lý do ignore.
- **Lạm dụng `other_*` thay cho `unknown`:** `other_*` đòi chứng cứ class con khác thực sự, không phải do không đọc nổi.
- **Để default `__undefined__`:** duyệt và sửa tất cả attributes trước Save/Export. `review_reason=none` cho `label/unknown`; `escalate` phải có lý do khác `none`.
- **Trộn ảnh blind vào ví dụ:** chỉ dùng `example/calibration`; giữ gold blind kín cho tới freeze.

**Thao tác giao nhận:** tạo task ảnh với schema đồng bộ; dán nguyên file này vào Guide; mỗi annotator label độc lập rồi Ctrl+S, export `CVAT for images 1.1` (không kèm ảnh) để QA so; sau calibration thay ví dụ bằng `sample_id` thật, cập nhật v2 và revision log. CVAT native XML lưu box, attributes và tag; QA vẫn cần xem ảnh để kiểm các quyết định IGNORE và geometry.

## Nguồn tham khảo và phần nhóm tự quy định

- Ertler et al., *The Mapillary Traffic Sign Dataset for Detection and Classification on a Global Scale*, ECCV 2020 — detection/classification ảnh đường phố đa quốc gia: https://www.ecva.net/papers/eccv_2020/papers_ECCV/papers/123680069.pdf
- Zhu et al., *Traffic-Sign Detection and Classification in the Wild*, CVPR 2016 / TT100K — mục tiêu nhỏ trong ảnh, box/class/mask, biến thiên môi trường: https://cg.cs.tsinghua.edu.cn/traffic-sign/
- Houben et al., *The German Traffic Sign Detection Benchmark*, IJCNN 2013 — ROI và class ID: https://benchmark.ini.rub.de/gtsdb_dataset.html
- CVAT, *CVAT for image* — hỗ trợ boxes, tags và attributes trong native export: https://docs.cvat.ai/docs/dataset_management/formats/format-cvat/

**Ghi chú:** Các tài liệu trên là cơ sở chọn bài toán và format. Taxonomy rút gọn, ngưỡng 6 px, quy tắc dừng ở `unknown`, giới hạn box và thứ tự escalation do nhóm đề xuất để calibration/peer test; không quy chúng cho dataset gốc hoặc tiêu chuẩn pháp luật. Cần kiểm ảnh trong `data/`, thống nhất với CVAT owner và điều chỉnh v2 bằng bằng chứng bất đồng thật.

### Chú thích từng giá trị

#### `family`

| Giá trị | Chú thích |
|---|---|
| `__undefined__` | Chưa xác định hoặc chưa gán nhóm chức năng của biển. |
| `regulatory` | Biển quy định, cấm hoặc hạn chế hành vi giao thông. |
| `warning` | Biển cảnh báo nguy hiểm hoặc điều kiện cần chú ý phía trước. |
| `mandatory` | Biển đưa ra hiệu lệnh bắt buộc phải thực hiện. |
| `information` | Biển cung cấp thông tin, chỉ đường, địa điểm hoặc dịch vụ. |
| `other` | Biết chắc là biển giao thông nhưng chức năng không thuộc các nhóm đã liệt kê. |
| `unknown` | Biết chắc là biển giao thông nhưng không đủ chứng cứ để xác định `family`. |

#### `type`

| Giá trị | Chú thích |
|---|---|
| `__undefined__` | Chưa xác định hoặc chưa gán loại biển. |
| `stop` | Biển yêu cầu phương tiện dừng lại. |
| `yield` | Biển yêu cầu phương tiện nhường đường. |
| `no_entry` | Biển cấm phương tiện đi vào theo hướng được kiểm soát. |
| `speed_limit` | Biển quy định giới hạn tốc độ bằng trị số. |
| `other_regulatory` | Biết chắc thuộc `regulatory` nhưng không phải các loại regulatory đã liệt kê. |
| `hazard_warning` | Biển cảnh báo một nguy hiểm hoặc điều kiện nguy hiểm cụ thể. |
| `other_warning` | Biết chắc thuộc `warning` nhưng không phù hợp loại warning đã liệt kê. |
| `turn_direction` | Biển bắt buộc phương tiện đi theo hướng nhất định như trái, phải hoặc thẳng. |
| `other_mandatory` | Biết chắc thuộc `mandatory` nhưng không phải `turn_direction`. |
| `direction` | Biển cung cấp thông tin chỉ hướng hoặc chỉ đường, không mang tính bắt buộc. |
| `place_or_service` | Biển cung cấp thông tin về địa điểm hoặc dịch vụ. |
| `other_information` | Biết chắc thuộc `information` nhưng không thuộc các type information đã liệt kê. |
| `supplementary` | Biển phụ cung cấp thông tin bổ sung cho biển chính. |
| `other_sign` | Biết chắc thuộc `family=other` nhưng không phải `supplementary`. |
| `unknown` | Biết chắc là biển nhưng không đủ chứng cứ để xác định `type`. |

#### `value`

| Giá trị | Chú thích |
|---|---|
| `__undefined__` | Chưa xác định hoặc chưa gán giá trị. |
| `not_applicable` | Không áp dụng trị số tốc độ cho biển này. |
| `unreadable` | Xác định được là `speed_limit` nhưng không đọc chắc chắn được trị số. |
| `other_readable` | Đọc rõ trị số tốc độ nhưng trị số đó chưa có trong ontology; cần QA kiểm tra. |
| `5` | Đọc chắc chắn giới hạn tốc độ là 5. |
| `10` | Đọc chắc chắn giới hạn tốc độ là 10. |
| `20` | Đọc chắc chắn giới hạn tốc độ là 20. |
| `30` | Đọc chắc chắn giới hạn tốc độ là 30. |
| `40` | Đọc chắc chắn giới hạn tốc độ là 40. |
| `50` | Đọc chắc chắn giới hạn tốc độ là 50. |
| `60` | Đọc chắc chắn giới hạn tốc độ là 60. |
| `70` | Đọc chắc chắn giới hạn tốc độ là 70. |
| `80` | Đọc chắc chắn giới hạn tốc độ là 80. |
| `90` | Đọc chắc chắn giới hạn tốc độ là 90. |
| `100` | Đọc chắc chắn giới hạn tốc độ là 100. |
| `110` | Đọc chắc chắn giới hạn tốc độ là 110. |
| `120` | Đọc chắc chắn giới hạn tốc độ là 120. |

#### `visibility`

| Giá trị | Chú thích |
|---|---|
| `__undefined__` | Chưa đánh giá tình trạng nhìn thấy của biển. |
| `full` | Toàn bộ biển nằm trong ảnh và không bị vật thể khác che khuất. |
| `occluded` | Một phần biển bị vật thể khác che khuất. |
| `truncated` | Một phần biển bị cắt bởi rìa ảnh. |
| `occluded_and_truncated` | Biển vừa bị vật thể khác che khuất vừa bị cắt bởi rìa ảnh. |

#### `legibility`

| Giá trị | Chú thích |
|---|---|
| `__undefined__` | Chưa đánh giá khả năng đọc nội dung biển. |
| `clear` | Nội dung cần thiết trên biển đọc hoặc nhận biết rõ ràng. |
| `partly_readable` | Chỉ đọc hoặc nhận biết được một phần nội dung biển. |
| `unreadable` | Không đọc được nội dung biển với độ tin cậy đủ để phân loại chi tiết. |

#### `decision`

| Giá trị | Chú thích |
|---|---|
| `__undefined__` | Chưa đưa ra quyết định cuối cho annotation. |
| `label` | Annotation đủ rõ và có thể gán nhãn bình thường theo guideline. |
| `unknown` | Biết chắc là biển nhưng không đủ chứng cứ để phân loại sâu hơn; `unknown` được chấp nhận là kết quả cuối. |
| `escalate` | Trường hợp cần QA/reviewer kiểm tra hoặc đưa ra quyết định. |

#### `review_reason`

| Giá trị | Chú thích |
|---|---|
| `__undefined__` | Chưa xác định lý do review. |
| `none` | Không có vấn đề cần QA/reviewer kiểm tra. |
| `small_or_far` | Biển quá nhỏ hoặc quá xa để xác định đáng tin cậy. |
| `occluded` | Biển bị vật thể khác che khuất gây khó khăn cho annotation. |
| `truncated` | Biển bị cắt bởi rìa ảnh gây khó khăn cho annotation. |
| `glare_or_blur` | Biển khó đọc do lóa, phản chiếu, nhòe, mất nét hoặc chất lượng ảnh thấp. |
| `sign_vs_nonsign` | Không chắc đối tượng là biển giao thông hay vật thể không phải biển giao thông. |
| `hierarchy_conflict` | Không thể tạo tổ hợp `family` → `type` → `value` hợp lệ theo taxonomy hiện tại. |
| `geometry` | Không chắc cách đặt hoặc ranh giới bounding box của biển. |
| `other` | Cần review vì nguyên nhân khác ngoài các lý do đã liệt kê. |