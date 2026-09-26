# Annotation guideline — Hierarchical traffic signs

**Version:** v2 — 26/09/2026. **Spec owner:** Nguyễn Tuấn Minh (`@TMTower18`). **Task:** ảnh tĩnh, từng mặt biển báo giao thông. Bản v2 cập nhật ngưỡng biển nhỏ/xa, trường hợp biển bị cắt quá 80%, và đưa chú thích tiếng Việt vào ngay bảng attributes. Ghi các thay đổi vào `08_revision_log.md`; sau blind test mới cập nhật v3. Chỉ dùng ví dụ ảnh trong split `example/calibration`; bảng ví dụ cuối file hiện là **tình huống giả định**, phải thay bằng `sample_id` thật trước handoff.

## 1. Objective + scope

Tạo dữ liệu detection và phân loại biển báo **theo bằng chứng nhìn thấy trên ảnh**, gồm cả ca nhỏ/xa/bị che. Vẽ box nếu vừa đủ tin rằng đó là mặt biển giao thông và đủ đặt vùng theo §3. Phân loại cụ thể đến đâu chứng cứ cho phép đến đó; `unknown` tốt hơn một phỏng đoán. Không đánh giá biển có hiệu lực với xe trong ảnh hay có tuân thủ luật quốc gia nào. Cùng một quy tắc áp dụng cho mọi split.

**Bao gồm:** biển báo vật lý cố định bên đường/trên giá long môn/trên cột, mặt trước hoặc mặt sau nhận ra chắc chắn, trong và ngoài làn xe của camera; biển lặp lại và nhiều biển trên một cột; mặt biển tạm thời phục vụ giao thông nếu thực sự là biển báo. **Không bao gồm:** đèn giao thông, vạch mặt đường, quảng cáo/biển hiệu doanh nghiệp, bảng tên đường chỉ là tên phố không có chỉ dẫn giao thông, cột/giá đỡ riêng, phản chiếu trong kính/gương, biển in trên xe, biểu tượng ứng dụng phủ lên ảnh, đối tượng che hoàn toàn. Nếu bộ ảnh có bảng chỉ hướng giao thông chứa chữ địa danh, gán `information/direction` khi đủ bằng chứng là biển chỉ dẫn đường; không loại trừ chỉ vì có tên địa danh.

## 2. Annotation unit

Một **instance = một mặt biển vật lý** trong một ảnh, kể cả hai biển giống nhau. Một biển chứa nhiều biểu tượng/chữ trên cùng một mặt vẫn là **một box**; không vẽ box cho từng ký tự, trị số hay mũi tên. Hai tấm biển ghép trên một cột là hai box nếu có thể thấy ranh giới hai mặt; tấm phụ độc lập là box riêng, `family=other`, `type=supplementary` nếu nhận ra. Nếu không phân biệt được số tấm do chồng lấp, vẽ từng tấm thấy ranh giới; phần còn lại escalation, không nhân bản box theo suy đoán. Chỉ dùng **Shape**, không dùng Track.

## 3. Geometry rule

- Dùng **rectangle axis-aligned** cho từng mặt biển. Box bao sát các pixel **mặt biển nhìn thấy**, kể cả viền gắn liền với mặt, không gồm cột, giá đỡ, bóng đổ hay phần nền. Không kéo box ra để đoán phần khuất (visible-only); không cần polygon/mask.
- Lấy `(x_min,y_min)` là góc trái trên và `(x_max,y_max)` là góc phải dưới của vùng thấy, tại **độ phân giải gốc**. Biển nghiêng vẫn dùng hộp chữ nhật không xoay bao vùng mặt biển; biển tròn thì hộp tiếp xúc gần nhất với mép ngoài. Biển cắt bởi biên ảnh: đặt cạnh box ngay tại biên, không kéo ra ngoài ảnh.
- Bị cây/xe che ở giữa: box ôm biên của các phần **nhìn thấy thuộc cùng một mặt**, có thể chứa vùng vật che giữa chúng; không vẽ box cho từng mảnh. Nếu chỉ còn một góc có thể xác định mặt biển, box chỉ ôm góc đó và escalate khi không rõ ranh giới. Mặt sau biển: box phần mặt sau nếu chắc chắn là mặt biển, phân loại `family=unknown`.
- Chỉ vẽ biển có cạnh dài nhất của phần mặt biển nhìn thấy **≥ 10 px** ở ảnh gốc. Nếu cạnh dài nhất **< 10 px**, IGNORE. Đúng 10 px thì vẽ. Đo trên ảnh gốc, không phóng to để vượt ngưỡng; đây là ngưỡng thao tác của nhóm. Zoom trong CVAT để xem, không dùng upscaling/AI hay biển ở ảnh/frame khác để suy ra chữ số. Mục tiêu QA: từng cạnh sai ≤ `max(2 px, 10% kích thước cạnh tương ứng của box gold)`. Với vật nhỏ, đối chiếu hình và vị trí, vì IoU rất nhạy với vài pixel.

## 4. Taxonomy và attributes

Dùng một CVAT label hình chữ nhật tên `traffic_sign`; phân cấp nhãn lưu bằng attributes. Không tạo attributes `visibility`, `decision` hoặc `review_reason`. Có thêm tag ảnh `image_escalate` cho nghi vấn không thể đặt box; nghi vấn ở object có box ghi trong calibration/review log bằng `sample_id`. Đồng bộ bảng này với `03_ontology_and_cvat_setup.md` và `03_cvat_labels.json` trước khi tạo task.

| Attribute | Giá trị cho phép và chú thích tiếng Việt | Default |
|---|---|---|
| `family` | `__undefined__` (chưa chọn, phải sửa trước export); `regulatory` (biển quy định, cấm hoặc hạn chế); `warning` (biển cảnh báo nguy hiểm); `mandatory` (biển yêu cầu phải thực hiện); `information` (biển cung cấp thông tin/chỉ đường/dịch vụ); `other` (biết là biển giao thông nhưng không thuộc các nhóm trên); `unknown` (chắc chắn là biển nhưng không đủ bằng chứng xác định nhóm). | `__undefined__` |
| `type` | `__undefined__` (chưa chọn, phải sửa trước export); `stop` (dừng lại); `yield` (nhường đường); `no_entry` (cấm đi vào); `speed_limit` (giới hạn tốc độ); `other_regulatory` (loại quy định khác); `hazard_warning` (cảnh báo nguy hiểm cụ thể); `other_warning` (loại cảnh báo khác); `turn_direction` (bắt buộc đi theo hướng chỉ định); `other_mandatory` (hiệu lệnh bắt buộc khác); `direction` (chỉ hướng/chỉ đường); `place_or_service` (địa điểm hoặc dịch vụ); `other_information` (loại thông tin khác); `supplementary` (biển phụ bổ sung thông tin); `other_sign` (loại biển khác); `unknown` (chắc là biển nhưng không đủ chứng cứ xác định loại). | `__undefined__` |
| `value` | `__undefined__` (chưa chọn, phải sửa trước export); `not_applicable` (không áp dụng trị số tốc độ); `unreadable` (biết là biển tốc độ nhưng không đọc chắc số); `other_readable` (đọc được số nhưng chưa có trong danh sách, ghi số thật vào review log); `5`, `10`, `20`, `30`, `40`, `50`, `60`, `70`, `80`, `90`, `100`, `110`, `120` (trị số tốc độ đọc rõ tương ứng). | `__undefined__` |
| `legibility` | `__undefined__` (chưa đánh giá, phải sửa trước export); `clear` (nội dung cần thiết đọc/nhận biết rõ); `partly_readable` (chỉ đọc/nhận biết được một phần); `unreadable` (không đọc được nội dung đủ tin cậy để phân loại chi tiết). | `__undefined__` |

`other_*` nghĩa là biết chắc thuộc nhóm cha nhưng là một loại khác đã liệt kê; `unknown` nghĩa là không đủ chứng cứ để xác định loại. Màu và hình dạng chỉ là gợi ý tìm biển, không tự đủ để kết luận family. `hazard_warning` chỉ dùng khi nội dung cảnh báo đủ rõ; `turn_direction` là hướng bắt buộc, còn `direction` là thông tin chỉ đường. Chọn `value` tốc độ chỉ khi đọc chắc trị số.

## 5. Inclusion / exclusion — cây quyết định

1. Đây có phải **mặt biển giao thông vật lý** theo §1? Nếu rõ là đối tượng ngoài scope: **IGNORE**, không vẽ. Nếu không chắc biển hay quảng cáo, xem kỹ ảnh gốc và tiếp tục bước 2.
2. Có đủ dấu hiệu ảnh để xác định **vị trí và ranh giới mặt biển**, cạnh dài nhất ≥ 10 px? Nếu có: vẽ box. Nếu nghi biển nhưng không thể đặt box đáng tin: gắn tag `image_escalate`, ghi `sample_id` và lý do trong log QA, **không vẽ box tưởng tượng**. Nếu chỉ là đốm không có bằng chứng là biển: IGNORE.
3. Xác định family bằng nội dung thấy rõ; chưa đủ thì `family=unknown`, `type=unknown`. Nếu biết family mà chưa biết type: `type=unknown`. Nếu biết type `speed_limit` nhưng không đọc số: `value=unreadable`. Không dùng ngữ cảnh đường, ảnh khác hay thứ tự biển để lấp chi tiết thiếu.
4. Gán `legibility`: `clear` nếu nội dung cần thiết đọc/nhận biết rõ; `partly_readable` nếu nhận ra một phần nhưng không đủ phân loại sâu; `unreadable` nếu không đọc được nội dung. Các mức `unknown` trong family/type/value là đầu ra hợp lệ, không cần thuộc tính quyết định riêng. Khi cần QA phân xử object có box, ghi `sample_id`, class dự kiến và câu hỏi vào calibration/review log. Không chắc có thể đặt box: dùng tag ảnh `image_escalate`.

## 6. Occlusion, cắt khung và biển nhỏ/xa

- Vẽ visible-only theo §3. Không tạo thuộc tính visibility: tình trạng che/cắt chỉ ảnh hưởng box và việc nội dung có thể đọc hay không.
- Cây, xe, người hoặc biển khác che một phần mặt: nếu thấy nhiều mảnh thuộc cùng một mặt, dùng một box bao vùng mặt nhìn thấy; vùng bị che giữa các mảnh có thể nằm bên trong box. Không vẽ box cho từng mảnh. Che hoàn toàn: không có box; nếu nghi có biển nhưng không thấy pixel nào của mặt biển thì ghi tag `image_escalate` nếu vị trí còn đáng tin, nếu không thì IGNORE.
- Biển cắt bởi mép ảnh: nếu còn đủ mặt biển thì box kết thúc tại biên ảnh, không kéo ra ngoài. Nếu hơn 80% mặt biển bị cắt khỏi ảnh và phần còn lại chỉ là viền, IGNORE, không vẽ box. Nếu bị che và cắt cùng lúc, vẫn chỉ vẽ visible-only box khi đạt ngưỡng kích thước và còn đủ phần mặt để nhận diện.
- Mất nét, mưa, nén ảnh, phản chiếu hoặc lóa không phải occlusion. Nếu biển vẫn chắc chắn nhưng nội dung không đọc được, gán type/value ở mức đọc được cao nhất và `legibility=unreadable` hoặc `partly_readable`.
- Biển nhỏ/xa nhưng cạnh dài nhất của phần mặt nhìn thấy ≥ 10 px và chắc là biển: vẽ box; dùng `unknown` ở cấp không đọc được. Nếu cạnh dài nhất < 10 px: IGNORE theo ngưỡng §3.
- Một chữ số thấy rõ, chữ số còn lại khuất: `speed_limit/unreadable`; tuyệt đối không hoàn thành con số bằng kiến thức phổ biến. Nếu không chắc đó là biển tốc độ, dừng ở family (hoặc `unknown`).

## 7. Ambiguity / escalation và cách kiểm export

| Trường hợp | Thao tác CVAT | Ghi nhận / xử lý |
|---|---|---|
| LABEL rõ | Một box `traffic_sign`, gán `family`, `type`, `value`, `legibility` phù hợp | Đủ bằng chứng thì không cần thêm tag |
| Không đủ chi tiết nhưng chắc là biển | Một box; dừng ở `unknown` cho family/type hoặc `unreadable` cho value; `legibility` phản ánh khả năng đọc | Đây là kết quả hợp lệ; không ép chọn class chi tiết |
| Cần review object đã có box | Giữ box và nhãn trung thực nhất; không có thuộc tính escalate riêng | Ghi `sample_id`, box/class, lý do cần chốt vào calibration/review log để reviewer xử lý |
| Nghi là biển nhưng không thể đặt box đáng tin | Tag ảnh `image_escalate`; không tạo box suy đoán | Ghi `sample_id` và lý do trong log QA |
| IGNORE | Không vẽ box cho đối tượng chắc chắn ngoài scope hoặc dưới ngưỡng | Reviewer kiểm ảnh/gold; export rỗng không tự chứng minh đây là IGNORE |

**Quy trình khi mơ hồ:** xem ảnh gốc → zoom → áp dụng cây quyết định → nếu biết chắc là biển thì chọn mức phân loại cao nhất có bằng chứng, phần còn lại `unknown`/`unreadable` → ghi vào log nếu cần reviewer chốt. Trước export, lọc mọi object còn `__undefined__`: đây là **chưa hoàn thành**, không phải đáp án. Annotator chọn từng giá trị theo bằng chứng hình ảnh; nếu chưa đủ căn cứ cho một cấp thì dùng `unknown` hoặc `unreadable`. Tình trạng cần review được theo dõi qua log, không thêm `decision` hay `review_reason` vào CVAT.

## 8. Temporal rule

**Không áp dụng — task ảnh tĩnh.** Mỗi ảnh là độc lập; không nối track, không suy nội dung biển từ ảnh liền kề hoặc ground truth nguồn. Nếu đổi sang clip video, phải thiết kế lại temporal rule, ontology mutable và export; bản guideline này không áp dụng tự động.

## 9. Examples — tình huống minh họa, chưa phải sample của lab

| Case giả định | Ảnh minh họa | Thấy gì | Expected output | Rule |
|---|---|---|---|---|
| EX-01 | ![EX-01](../../images/bien-nguoc-chieu.png) | mặt biển ngược chiều | 1 box; `regulatory/no_entry/not_applicable`, `legibility=clear` | §3–5 |
| EX-02 | ![EX-02](../../images/bien_toc_do.png) | biển giới hạn 60 rõ | 1 box; `regulatory/speed_limit/60`, `legibility=clear` | §4–5 |
| EX-03 | ![EX-03](../../images/bien_bi_cay_che.png) | biển bị tán cây che | 1 box visible-only; `legibility=partly_readable`; ghi log nếu cần review | §3, §6 |
| EX-04 | ![EX-04](../../images/2_bien.png) | hai tấm biển trên cùng cột, thấy ranh giới riêng | 2 box, mỗi tấm attributes riêng | §2 |
| EX-05 | ![EX-05](../../images/bien_tuyen_truyen.png) | bảng tuyên truyền về an toàn giao thông | 1 box; `information/other_information/not_applicable`, `legibility=clear` | §1, §5 |
| EX-06 | ![EX-06](../../images/bien_xa.png) | biển ở xa thấy biển những không thấy rõ nội dung | 1 box; để tất cả thuộc tính là unknow hoặc tương tự | §1, §5 |
| EX-07 | ![EX-07](../../images/bien_vien.png) | biển bị cắt ở viền, không thấy hơn 80% biển | ignore, không tạo box | §1, §5 |


## 10. Common mistakes + checklist

- **Đoán số** tốc độ từ một chữ số hoặc biển trên khung đường: chuyển `value=unreadable`.
- **Đánh đồng hình dáng với ý nghĩa:** chỉ gán family/type khi nội dung đủ bằng chứng; màu/shape chỉ giúp phát hiện ứng viên.
- **Box trùm cả cột/phần bị che:** chỉnh về mặt biển nhìn thấy; không vẽ amodal.
- **Gộp hai biển hoặc cắt một biển thành nhiều box:** đếm từng mặt vật lý theo §2.
- **Quên biển nhỏ:** zoom, áp dụng ngưỡng ở ảnh gốc; nếu cạnh dài nhất dưới 10 px thì IGNORE.
- **Lạm dụng `other_*` thay cho `unknown`:** `other_*` đòi chứng cứ class con khác thực sự, không phải do không đọc nổi.
- **Để default `__undefined__`:** duyệt và sửa tất cả attributes trước Save/Export. Không để thuộc tính nào còn `__undefined__`; nghi vấn cần QA phải được ghi vào calibration/review log kèm `sample_id`.
- **Trộn ảnh blind vào ví dụ:** chỉ dùng `example/calibration`; giữ gold blind kín cho tới freeze.

**Thao tác giao nhận:** tạo task ảnh với schema đồng bộ; dán nguyên file này vào Guide; mỗi annotator label độc lập rồi Ctrl+S, export `CVAT for images 1.1` (không kèm ảnh) để QA so; sau calibration thay ví dụ bằng `sample_id` thật, cập nhật v2 và revision log. CVAT native XML lưu box, attributes và tag; QA vẫn cần xem ảnh để kiểm các quyết định IGNORE và geometry.

## Nguồn tham khảo và phần nhóm tự quy định

- Ertler et al., *The Mapillary Traffic Sign Dataset for Detection and Classification on a Global Scale*, ECCV 2020 — detection/classification ảnh đường phố đa quốc gia: https://www.ecva.net/papers/eccv_2020/papers_ECCV/papers/123680069.pdf
- Zhu et al., *Traffic-Sign Detection and Classification in the Wild*, CVPR 2016 / TT100K — mục tiêu nhỏ trong ảnh, box/class/mask, biến thiên môi trường: https://cg.cs.tsinghua.edu.cn/traffic-sign/
- Houben et al., *The German Traffic Sign Detection Benchmark*, IJCNN 2013 — ROI và class ID: https://benchmark.ini.rub.de/gtsdb_dataset.html
- CVAT, *CVAT for image* — hỗ trợ boxes, tags và attributes trong native export: https://docs.cvat.ai/docs/dataset_management/formats/format-cvat/

**Ghi chú:** Các tài liệu trên là cơ sở chọn bài toán và format. Taxonomy rút gọn, ngưỡng 10 px, quy tắc dừng ở `unknown`, giới hạn box và thứ tự escalation do nhóm đề xuất để calibration/peer test; không quy chúng cho dataset gốc hoặc tiêu chuẩn pháp luật. Cần kiểm ảnh trong `data/`, thống nhất với CVAT owner và điều chỉnh v2 bằng bằng chứng bất đồng thật.
