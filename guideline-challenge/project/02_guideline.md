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

**Schema triển khai đề xuất cho Long:** đúng **một CVAT label hình chữ nhật tên `traffic_sign`**, với các select attributes bất biến ở task ảnh tĩnh dưới đây. Phân cấp lưu bằng attributes, không tạo hàng chục label con. Không tạo attributes `visibility`, `decision` hoặc `review_reason`. Có thêm **một tag ảnh** `image_escalate` cho nghi vấn không thể đặt box; các nghi vấn ở object có box được ghi trong calibration/review log bằng `sample_id`. `03_ontology_and_cvat_setup.md` là source of truth; Long phải đồng bộ `03_cvat_labels.json` *trước* tạo task, nếu tên/giá trị khác thì sửa hai bên và guideline đồng thời, không bắt peer tự dịch schema. Không dùng checkbox `needs_review` làm nguồn duy nhất nếu không có lý do.

| Attribute | Giá trị cho phép | Default | Ý nghĩa |
|---|---|---|---|
| `family` | `__undefined__`, `regulatory`, `warning`, `mandatory`, `information`, `other`, `unknown` | `__undefined__` | Nhóm chức năng **khi đọc rõ**, không suy từ hình dạng/màu đơn lẻ. `unknown` nếu chỉ biết chắc là biển. |
| `type` | `__undefined__`, `stop`, `yield`, `no_entry`, `speed_limit`, `other_regulatory`, `hazard_warning`, `other_warning`, `turn_direction`, `other_mandatory`, `direction`, `place_or_service`, `other_information`, `supplementary`, `other_sign`, `unknown` | `__undefined__` | Loại cụ thể hoặc nhóm con có chứng cứ; dùng `unknown` nếu không đủ đọc loại. |
| `value` | `__undefined__`, `not_applicable`, `unreadable`, `other_readable`, `5`, `10`, `20`, `30`, `40`, `50`, `60`, `70`, `80`, `90`, `100`, `110`, `120` | `__undefined__` | Chỉ ghi **trị số tốc độ** cho `speed_limit` nếu đọc được toàn bộ. Giá trị đọc được ngoài danh sách: `other_readable` + escalate để QA mở rộng ontology trước freeze. `unreadable` cho speed limit thấy rõ loại nhưng không đọc được số; còn lại `not_applicable`. |
| `legibility` | `__undefined__`, `clear`, `partly_readable`, `unreadable` | `__undefined__` | Mức đọc nội dung trên mặt; không đồng nghĩa kích thước. |

`other_*` nghĩa là **biết chắc họ type/family nhưng không thuộc các loại kể tên**. `unknown` nghĩa là **không đủ chứng cứ để biết**. Màu/hình tam giác, tròn, chữ nhật là gợi ý tìm biển, **không tự đủ để xác định family** vì quy ước các nước và mặt sau có thể khác. `hazard_warning` chỉ khi thấy rõ nội dung cảnh báo; không tự điền chỉ vì tam giác. `turn_direction` chỉ khi thấy mũi tên/chỉ dẫn bắt buộc; `direction` là biển chỉ đường thông tin. `place_or_service` cho tên địa điểm/dịch vụ trên biển giao thông. `other_readable` cần QA ghi số thực trong log; không âm thầm chuyển thành một số có trong menu.

## 5. Inclusion / exclusion — cây quyết định

1. Đây có phải **mặt biển giao thông vật lý** theo §1? Nếu rõ là đối tượng ngoài scope: **IGNORE**, không vẽ. Nếu không chắc biển hay quảng cáo, xem kỹ ảnh gốc và tiếp tục bước 2.
2. Có đủ dấu hiệu ảnh để xác định **vị trí và ranh giới mặt biển**, hai chiều ≥ 6 px? Nếu có: vẽ box. Nếu nghi biển nhưng không thể đặt box đáng tin: gắn tag `image_escalate`, ghi `sample_id` và lý do trong log QA, **không vẽ box tưởng tượng**. Nếu chỉ là đốm không có bằng chứng là biển: IGNORE.
3. Xác định family bằng nội dung thấy rõ; chưa đủ thì `family=unknown`, `type=unknown`. Nếu biết family mà chưa biết type: `type=unknown`. Nếu biết type `speed_limit` nhưng không đọc số: `value=unreadable`. Không dùng ngữ cảnh đường, ảnh khác hay thứ tự biển để lấp chi tiết thiếu.
4. Gán `legibility`: `clear` nếu nội dung cần thiết đọc/nhận biết rõ; `partly_readable` nếu nhận ra một phần nhưng không đủ phân loại sâu; `unreadable` nếu không đọc được nội dung. Các mức `unknown` trong family/type/value là đầu ra hợp lệ, không cần thuộc tính quyết định riêng. Khi cần QA phân xử object có box, ghi `sample_id`, class dự kiến và câu hỏi vào calibration/review log. Không chắc có thể đặt box: dùng tag ảnh `image_escalate`.

## 6. Occlusion, cắt khung và biển nhỏ/xa

- Vẽ visible-only theo §3. Không tạo thuộc tính visibility: tình trạng che/cắt chỉ ảnh hưởng box và việc nội dung có thể đọc hay không.
- Cây, xe, người hoặc biển khác che một phần mặt: nếu thấy nhiều mảnh thuộc cùng một mặt, dùng một box bao vùng mặt nhìn thấy; vùng bị che giữa các mảnh có thể nằm bên trong box. Không vẽ box cho từng mảnh. Che hoàn toàn: không có box; nếu nghi có biển nhưng không thấy pixel nào của mặt biển thì ghi tag `image_escalate` nếu vị trí còn đáng tin, nếu không thì IGNORE.
- Biển cắt bởi mép ảnh: box kết thúc tại biên ảnh, không kéo ra ngoài. Nếu bị che và cắt cùng lúc, vẫn chỉ vẽ visible-only box.
- Mất nét, mưa, nén ảnh, phản chiếu hoặc lóa không phải occlusion. Nếu biển vẫn chắc chắn nhưng nội dung không đọc được, gán type/value ở mức đọc được cao nhất và `legibility=unreadable` hoặc `partly_readable`.
- Biển nhỏ/xa nhưng ≥ 6 px và chắc là biển: vẽ box; dùng `unknown` ở cấp không đọc được. Biển < 6 px ở một chiều: IGNORE theo ngưỡng vận hành §3.
- Một chữ số thấy rõ, chữ số còn lại khuất: `speed_limit/unreadable`; tuyệt đối không hoàn thành con số bằng kiến thức phổ biến. Nếu không chắc đó là biển tốc độ, dừng ở family (hoặc `unknown`).

## 7. Ambiguity / escalation và cách kiểm export

| Trường hợp | Thao tác CVAT | Ghi nhận / xử lý |
|---|---|---|
| LABEL rõ | Một box `traffic_sign`, gán `family`, `type`, `value`, `legibility` phù hợp | Đủ bằng chứng thì không cần thêm tag |
| Không đủ chi tiết nhưng chắc là biển | Một box; dừng ở `unknown` cho family/type hoặc `unreadable` cho value; `legibility` phản ánh khả năng đọc | Đây là kết quả hợp lệ; không ép chọn class chi tiết |
| Cần review object đã có box | Giữ box và nhãn trung thực nhất; không có thuộc tính escalate riêng | Ghi `sample_id`, box/class, lý do cần chốt vào calibration/review log để reviewer xử lý |
| Nghi là biển nhưng không thể đặt box đáng tin | Tag ảnh `image_escalate`; không tạo box suy đoán | Ghi `sample_id` và lý do trong log QA |
| IGNORE | Không vẽ box cho đối tượng chắc chắn ngoài scope hoặc dưới ngưỡng | Reviewer kiểm ảnh/gold; export rỗng không tự chứng minh đây là IGNORE |

**Quy trình khi mơ hồ:** xem ảnh gốc → zoom → áp dụng cây quyết định → nếu biết chắc là biển thì chọn mức phân loại cao nhất có bằng chứng, phần còn lại `unknown`/`unreadable` → ghi vào log nếu cần reviewer chốt. Trước export, lọc mọi object còn `__undefined__`: đây là **chưa hoàn thành**, không phải đáp án. QA kiểm các tổ hợp family/type/value theo bảng ràng buộc vì CVAT select không tự kiểm quan hệ giữa attributes. Tình trạng cần review được theo dõi qua log, không thêm `decision` hay `review_reason` vào CVAT.

## 8. Temporal rule

**Không áp dụng — task ảnh tĩnh.** Mỗi ảnh là độc lập; không nối track, không suy nội dung biển từ ảnh liền kề hoặc ground truth nguồn. Nếu đổi sang clip video, phải thiết kế lại temporal rule, ontology mutable và export; bản guideline này không áp dụng tự động.

## 9. Examples — tình huống minh họa, chưa phải sample của lab

| Case giả định | Thấy gì | Expected output | Rule |
|---|---|---|---|
| EX-01 | mặt biển STOP rõ, 30×30 px | 1 box; `regulatory/stop/not_applicable`, `legibility=clear` | §3–5 |
| EX-02 | biển giới hạn 60 rõ | 1 box; `regulatory/speed_limit/60`, `legibility=clear` | §4–5 |
| EX-03 | biển tốc độ, chữ số bị tán cây che | 1 box visible-only; `regulatory/speed_limit/unreadable`, `legibility=partly_readable`; ghi log nếu cần review | §3, §6 |
| EX-04 | mặt biển chắc chắn, 7×8 px, nội dung không đọc | 1 box; `unknown/unknown/not_applicable`, `legibility=unreadable` | §3, §5 |
| EX-05 | mặt biển chắc chắn, 5×9 px | IGNORE vì chiều rộng < 6 px | §3 |
| EX-06 | biển thông tin bị cắt rìa, thấy chữ và mũi tên chỉ đường | 1 box clipped tại biên; `information/direction/not_applicable`; legibility theo phần đọc được | §3, §6 |
| EX-07 | hai tấm biển trên cùng cột, thấy ranh giới riêng | 2 box, mỗi tấm attributes riêng | §2 |
| EX-08 | bảng quảng cáo tròn đỏ không phải biển giao thông | IGNORE | §1, §5 |
| EX-09 | giống biển nhưng vật che khiến không thể đặt box | tag ảnh `image_escalate` + log QA, không phỏng đoán box | §5, §7 |
| EX-10 | mặt sau một biển chắc chắn, đủ lớn | 1 box; `unknown/unknown/not_applicable`, `legibility=unreadable` | §4, §6 |

## 10. Common mistakes + checklist

- **Đoán số** tốc độ từ một chữ số hoặc biển trên khung đường: chuyển `value=unreadable`.
- **Đánh đồng hình dáng với ý nghĩa:** chỉ gán family/type khi nội dung đủ bằng chứng; màu/shape chỉ giúp phát hiện ứng viên.
- **Box trùm cả cột/phần bị che:** chỉnh về mặt biển nhìn thấy; không vẽ amodal.
- **Gộp hai biển hoặc cắt một biển thành nhiều box:** đếm từng mặt vật lý theo §2.
- **Quên biển nhỏ:** zoom, áp dụng ngưỡng ở ảnh gốc; ≥ 6 px không phải lý do ignore.
- **Lạm dụng `other_*` thay cho `unknown`:** `other_*` đòi chứng cứ class con khác thực sự, không phải do không đọc nổi.
- **Để default `__undefined__`:** duyệt và sửa tất cả attributes trước Save/Export. Không để thuộc tính nào còn `__undefined__`; nghi vấn cần QA phải được ghi vào calibration/review log kèm `sample_id`.
- **Trộn ảnh blind vào ví dụ:** chỉ dùng `example/calibration`; giữ gold blind kín cho tới freeze.

**Thao tác giao nhận:** tạo task ảnh với schema đồng bộ; dán nguyên file này vào Guide; mỗi annotator label độc lập rồi Ctrl+S, export `CVAT for images 1.1` (không kèm ảnh) để QA so; sau calibration thay ví dụ bằng `sample_id` thật, cập nhật v2 và revision log. CVAT native XML lưu box, attributes và tag; QA vẫn cần xem ảnh để kiểm các quyết định IGNORE và geometry.

## Nguồn tham khảo và phần nhóm tự quy định

- Ertler et al., *The Mapillary Traffic Sign Dataset for Detection and Classification on a Global Scale*, ECCV 2020 — detection/classification ảnh đường phố đa quốc gia: https://www.ecva.net/papers/eccv_2020/papers_ECCV/papers/123680069.pdf
- Zhu et al., *Traffic-Sign Detection and Classification in the Wild*, CVPR 2016 / TT100K — mục tiêu nhỏ trong ảnh, box/class/mask, biến thiên môi trường: https://cg.cs.tsinghua.edu.cn/traffic-sign/
- Houben et al., *The German Traffic Sign Detection Benchmark*, IJCNN 2013 — ROI và class ID: https://benchmark.ini.rub.de/gtsdb_dataset.html
- CVAT, *CVAT for image* — hỗ trợ boxes, tags và attributes trong native export: https://docs.cvat.ai/docs/dataset_management/formats/format-cvat/

**Ghi chú:** Các tài liệu trên là cơ sở chọn bài toán và format. Taxonomy rút gọn, ngưỡng 6 px, quy tắc dừng ở `unknown`, giới hạn box và thứ tự escalation do nhóm đề xuất để calibration/peer test; không quy chúng cho dataset gốc hoặc tiêu chuẩn pháp luật. Cần kiểm ảnh trong `data/`, thống nhất với CVAT owner và điều chỉnh v2 bằng bằng chứng bất đồng thật.

## Phụ lục — Chú thích từng giá trị

Các giá trị `__undefined__` là trạng thái mặc định của CVAT để báo rằng annotator chưa chọn. Phải thay bằng giá trị có căn cứ trước khi lưu/export; không được xem `__undefined__` là nhãn `unknown`.

### `family`

| Giá trị | Chú thích |
|---|---|
| `__undefined__` | Chưa gán nhóm; cần hoàn thành trước export. |
| `regulatory` | Biển quy định, cấm hoặc hạn chế hành vi giao thông. |
| `warning` | Biển cảnh báo nguy hiểm hoặc điều kiện cần chú ý phía trước. |
| `mandatory` | Biển yêu cầu người tham gia giao thông phải thực hiện một hướng/hành động. |
| `information` | Biển cung cấp thông tin, chỉ đường, địa điểm hoặc dịch vụ. |
| `other` | Biết chắc là biển giao thông nhưng chức năng không thuộc các nhóm đã liệt kê. |
| `unknown` | Biết chắc là biển giao thông nhưng ảnh không đủ bằng chứng để xác định nhóm. |

### `type`

| Giá trị | Chú thích |
|---|---|
| `__undefined__` | Chưa gán loại; cần hoàn thành trước export. |
| `stop` | Biển yêu cầu phương tiện dừng lại. |
| `yield` | Biển yêu cầu phương tiện nhường đường. |
| `no_entry` | Biển cấm phương tiện đi vào theo hướng được kiểm soát. |
| `speed_limit` | Biển giới hạn tốc độ bằng trị số. |
| `other_regulatory` | Biết chắc thuộc `regulatory` nhưng không phải các loại regulatory đã liệt kê. |
| `hazard_warning` | Biển cảnh báo nguy hiểm hoặc điều kiện nguy hiểm cụ thể. |
| `other_warning` | Biết chắc thuộc `warning` nhưng không thuộc loại warning đã liệt kê. |
| `turn_direction` | Biển bắt buộc đi theo hướng được chỉ định, như trái, phải hoặc thẳng. |
| `other_mandatory` | Biết chắc thuộc `mandatory` nhưng không phải `turn_direction`. |
| `direction` | Biển cung cấp thông tin chỉ hướng hoặc chỉ đường, không mang tính bắt buộc. |
| `place_or_service` | Biển giao thông cung cấp thông tin về địa điểm hoặc dịch vụ. |
| `other_information` | Biết chắc thuộc `information` nhưng không thuộc các loại information đã liệt kê. |
| `supplementary` | Biển phụ độc lập bổ sung thông tin cho biển chính. |
| `other_sign` | Biết chắc thuộc `family=other` nhưng không phải `supplementary`. |
| `unknown` | Biết chắc là biển nhưng không đủ bằng chứng để xác định loại. |

### `value`

| Giá trị | Chú thích |
|---|---|
| `__undefined__` | Chưa gán giá trị; cần hoàn thành trước export. |
| `not_applicable` | Biển này không có trị số tốc độ cần gán. |
| `unreadable` | Đã xác định là `speed_limit` nhưng không đọc chắc trị số. |
| `other_readable` | Đọc rõ trị số tốc độ nhưng giá trị chưa có trong danh sách; ghi trị số thực trong review log để QA xem xét cập nhật ontology. |
| `5` | Đọc chắc giới hạn tốc độ là 5. |
| `10` | Đọc chắc giới hạn tốc độ là 10. |
| `20` | Đọc chắc giới hạn tốc độ là 20. |
| `30` | Đọc chắc giới hạn tốc độ là 30. |
| `40` | Đọc chắc giới hạn tốc độ là 40. |
| `50` | Đọc chắc giới hạn tốc độ là 50. |
| `60` | Đọc chắc giới hạn tốc độ là 60. |
| `70` | Đọc chắc giới hạn tốc độ là 70. |
| `80` | Đọc chắc giới hạn tốc độ là 80. |
| `90` | Đọc chắc giới hạn tốc độ là 90. |
| `100` | Đọc chắc giới hạn tốc độ là 100. |
| `110` | Đọc chắc giới hạn tốc độ là 110. |
| `120` | Đọc chắc giới hạn tốc độ là 120. |

### `legibility`

| Giá trị | Chú thích |
|---|---|
| `__undefined__` | Chưa đánh giá khả năng đọc; cần hoàn thành trước export. |
| `clear` | Nội dung cần thiết đọc hoặc nhận biết rõ. |
| `partly_readable` | Chỉ đọc hoặc nhận biết được một phần nội dung; chưa đủ để gán đầy đủ loại hoặc trị số. |
| `unreadable` | Không đọc được nội dung đủ tin cậy để phân loại chi tiết. |
