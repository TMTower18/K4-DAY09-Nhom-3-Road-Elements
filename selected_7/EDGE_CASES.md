# Edge cases — selected_7

Áp dụng schema `guideline-challenge/project/03_cvat_labels.json` và **`02_guideline_v2.md`** (v2):

- 1 label `traffic_sign` (rectangle), attribute `family` / `type` / `value` / `legibility` (default `__undefined__`).
- 1 tag ảnh `image_escalate`.
- Ghi tắt trong file này: `family/type/value/legibility`, ví dụ `regulatory/speed_limit/40/clear`.
- Toạ độ `(xtl, ytl, xbr, ybr)` tính bằng pixel trên ảnh gốc 960×540. Dấu `~` = toạ độ ước lượng từ ảnh, **không có
  trong nhãn gốc**, cần vẽ lại cho sát.

| Ảnh | Loại | Box nhãn gốc | Box theo guideline | Điểm khó chính |
|---|---|---|---|---|
| 0338 | normal | 2 | 2 | — |
| 0664 | normal | 2 | 2 | — |
| 0824 | normal | 2 | 2 | 2 biển chồng trên một cột |
| 0959 | normal* | 1 | 2 | có 1 biển bị cắt ở mép trên |
| 1979 | normal* | 1 | 3 | có 1 biển bị cắt ở mép trên + 1 tấm phụ |
| **0756** | **EDGE** | 12 | **23** | cảnh dày đặc, biển phụ, biển 1 làn, bảng tên đường |
| **0961** | **EDGE** | 3 | **3** | biển cực nhỏ, dưới ngưỡng 10 px, tấm phụ |

\* Vẫn là ảnh normal, nhưng nhãn gốc thiếu box: label theo guideline sẽ **nhiều box hơn** nhãn gốc.

---

## 1. Nhãn gốc ≠ guideline — đọc trước khi review

Nhãn gốc của dataset (mã QCVN như `P.102`) và guideline của nhóm khác nhau ở 4 điểm. Khi bài label khác nhãn gốc, **chấm
theo guideline**, không chấm theo nhãn gốc.

| Điểm | Nhãn gốc dataset | Guideline nhóm |
|---|---|---|
| Biển bị cắt > 80% ở mép ảnh | — | IGNORE (§6, EX-11) |
| Biển phụ (tấm chữ, loại xe, khung giờ) | không vẽ | **box riêng**, `other/supplementary` (§2) |
| Biển nhỏ | vẽ cả biển 8×5 px | chỉ vẽ khi **cạnh dài nhất ≥ 10 px** (§3) |
| Biển xa không đọc được nội dung | vẫn ghi đúng mã biển | `unknown`, **không** suy từ hình dạng/màu (§4) |
| Biển 1 làn (R.412), biển chỉ đường | không vẽ | vẽ nếu là mặt biển giao thông (§1) |

### Bảng quy đổi mã biển → attribute (dành cho reviewer)

Annotator **không** được dùng bảng này để đoán: chỉ gán khi đọc được nội dung trên ảnh.

| Mã gốc | Tên | Expected khi đọc rõ |
|---|---|---|
| P.102 | Cấm đi ngược chiều | `regulatory/no_entry/not_applicable` |
| P.127*40, P.127*60 | Giới hạn tốc độ | `regulatory/speed_limit/40` · `/60` |
| P.103a, P.106a, P.111, P.123a, P.129, P.130, P.131a | các biển cấm khác | `regulatory/other_regulatory/not_applicable` |
| R.302a | Phải đi vòng sang phải | `mandatory/turn_direction/not_applicable` |
| R.412 | Làn dành riêng từng loại xe | `mandatory/other_mandatory/not_applicable` — xem case 2.2 |
| W.224, W.221b, W.239b*, W.246a | cảnh báo nguy hiểm cụ thể | `warning/hazard_warning/not_applicable` |
| W.245a | Đi chậm | `warning/other_warning/not_applicable` |
| I.434a | Bến xe buýt | `information/place_or_service/not_applicable` |
| Camera | Đường có camera giám sát | `information/other_information/not_applicable` |

---

## 2. EDGE 1 — `0756.jpg`: giàn biển dày đặc ở cửa ngõ

Giàn biển trên đường + cụm biển hai bên lề, nhìn từ xe máy đang kẹt xe. Nhãn gốc có 12 box; theo guideline
**23 box** — xem [examples/0756_expected.png](examples/0756_expected.png). Rủi ro chính: **bỏ sót** biển phụ và biển nhỏ, **gộp** biển chính với biển phụ, **gán số** cho biển không
phải tốc độ.

### 2.1 Biển tốc độ và tấm phụ loại xe bên dưới

| Đối tượng | Box | Expected |
|---|---|---|
| Biển 40 | (305, 114, 343, 150) | `regulatory/speed_limit/40/clear` |
| Tấm phụ dưới biển 40 (hình ô tô, xe tải) | ~(307, 155, 337, 183) | `other/supplementary/not_applicable` |
| Biển 60 | (351, 116, 389, 153) | `regulatory/speed_limit/60/clear` |
| Tấm phụ dưới biển 60 (hình ô tô, xe khách) | ~(353, 155, 382, 184) | `other/supplementary/not_applicable` |

- **Bẫy:** box biển 40/60 kéo dài xuống ôm cả tấm phụ → sai geometry. Biển chính và tấm phụ là **2 mặt vật lý = 2 box**.
- **Reviewer kiểm:** đủ 4 box; `value` đúng `40`, `60`.

### 2.2 Biển 1 làn trên giàn (R.412) — nhãn gốc không có

| # | Hình trên biển | Box | Expected |
|---|---|---|---|
| 1 | mũi tên thẳng + ô tô | ~(253, 121, 298, 182) | `mandatory/other_mandatory/not_applicable/clear` |
| 2 | mũi tên thẳng + ô tô + xe buýt | ~(397, 124, 443, 186) | như trên |
| 3 | mũi tên thẳng + xe máy + ô tô | ~(570, 130, 617, 193) | như trên |

- **Ambiguity thật:** biển có mũi tên, annotator dễ chọn `turn_direction`. Theo §4, `turn_direction` = bắt buộc đi theo
  hướng; biển này chủ yếu quy định **làn cho loại xe** → `other_mandatory`. Guideline chưa nói rõ trường hợp này →
  **cần thêm một dòng vào §4 hoặc một ví dụ ở §9**.
- **Chấp nhận khi calibration:** `mandatory/turn_direction` là lỗi `type`, không phải lỗi `family`.

### 2.3 Bảng chữ tuyên truyền trên giàn — IGNORE

Bảng xanh "PHÍA TRƯỚC TAY LÁI LÀ SỰ SỐNG…" ~(640, 120, 728, 208).

- **Expected:** không vẽ. Không phải mặt biển giao thông (§1).
- **Bẫy:** cùng màu xanh, cùng giàn với biển thật.

### 2.4 Cụm biển bên phải: biển cấm + tấm giờ + tấm chữ

| Đối tượng | Box | Expected |
|---|---|---|
| Biển cấm xe tải (hình xe tải "3.5t") | (830, 212, 878, 259) | `regulatory/other_regulatory/not_applicable` |
| Tấm giờ "06:00-22:00" | ~(827, 260, 878, 284) | `other/supplementary/not_applicable` |
| Tấm xanh "XE PHỤC VỤ KHU CÔNG NGHỆ CAO ĐƯỢC PHÉP LƯU THÔNG" | ~(822, 284, 877, 320) | `other/supplementary/not_applicable` |
| Biển tam giác "2.7m" | (881, 217, 940, 267) | `warning/hazard_warning/not_applicable` |
| Bảng "XA LỘ HÀ NỘI / HANOI HIGHWAY" | ~(914, 269, 956, 296) | **IGNORE** — xem dưới |

- **Bẫy `value`:** biển có số "3.5t" và "2.7m" nhưng **không phải tốc độ** → `value=not_applicable`, không chọn số.
- **Bảng "XA LỘ HÀ NỘI":** chỉ là tên đường, không có mũi tên hay chỉ dẫn → theo §1 là "bảng tên đường chỉ là tên phố" →
  IGNORE. Nếu annotator thấy có chỉ dẫn hướng → `information/direction`. Đây là **ambiguity cần ghi vào calibration
  report** nếu 2 người làm khác nhau.

### 2.5 Biển nhỏ lẫn giữa cảnh

| Đối tượng | Box | Cỡ | Expected |
|---|---|---|---|
| W.221b (gồ giảm tốc), trái | (152, 250, 181, 277) | 29×27 | `warning/hazard_warning` |
| Tấm chữ đỏ dưới biển trên | ~(149, 279, 179, 312) | 30×33 | `other/supplementary` |
| W.221b, phải | (762, 289, 792, 315) | 30×26 | `warning/hazard_warning` |
| Tấm chữ đỏ dưới biển trên | ~(758, 317, 794, 353) | 36×36 | `other/supplementary` |
| W.221b, nhỏ | (684, 338, 696, 352) | **12×14** | cạnh dài 14 ≥ 10 px → vẽ; nhiều khả năng `unknown/unknown`, `legibility=unreadable` |
| Tấm đỏ nhỏ dưới W.221b nhỏ | ~(683, 353, 698, 369) | 15×16 | `other/supplementary` hoặc `unknown` — nhãn gốc bỏ sót |
| P.130 | (548, 266, 570, 287) | 22×21 | `regulatory/other_regulatory` |
| P.111 | (548, 289, 569, 310) | 21×21 | `regulatory/other_regulatory` |
| Tấm đỏ nhỏ dưới P.111 | ~(547, 312, 567, 325) | 20×13 | `other/supplementary` hoặc `unknown` — nhãn gốc bỏ sót |
| P.131a | (696, 338, 715, 357) | 19×19 | `regulatory/other_regulatory` hoặc `unknown` nếu không đọc được |
| P.129 | (696, 358, 714, 374) | 18×16 | như trên |
| Camera | (737, 307, 760, 341) | 23×34 | `information/other_information` hoặc `unknown` |

- **Biển 12–22 px:** chấp nhận cả expected lẫn `unknown` + `legibility=unreadable` — §4 ưu tiên `unknown` hơn phỏng đoán.
  **Không** chấp nhận `family` đoán từ màu/hình mà `legibility=unreadable`.
- **Reviewer kiểm:** zoom vùng x 540–800, y 260–380; mỗi mặt vật lý một box, box không trùm sang biển kế bên.

---

## 3. EDGE 2 — `0961.jpg`: biển cực nhỏ ở xa

Minh hoạ: [examples/0961_expected.png](examples/0961_expected.png).

| Đối tượng | Box | Cỡ | Expected |
|---|---|---|---|
| Tam giác trên (gốc: W.245a) | (241, 196, 255, 208) | 14×12 | vẽ; `unknown/unknown/not_applicable/unreadable` |
| Tam giác dưới (gốc: W.246a) | (242, 208, 255, 220) | 13×12 | vẽ; `unknown/unknown/not_applicable/unreadable` |
| Tấm đỏ "CHÚ Ý QUAN SÁT" dưới 2 tam giác | ~(236, 219, 260, 231) | 24×12 | vẽ; `other/supplementary` nếu đọc được chữ, không thì `unknown/unknown` |
| Biển tròn bên trái (gốc: P.130) | (168, 221, 176, 226) | **8×5** | **IGNORE** — cạnh dài nhất 8 px < 10 px |
| Khung đỏ/vàng mép trên bên phải | ~(800, 0, 960, 20) | — | **IGNORE** — kết cấu cổng, không phải mặt biển |

### 3.1 Nhãn gốc cụ thể hơn bằng chứng trên ảnh

Nhãn gốc ghi W.245a / W.246a, nhưng ở 12–14 px không đọc được hình bên trong tam giác. Theo §4: "màu/hình tam giác không tự
đủ để xác định family" → expected là **`unknown`**.

- **Chấp nhận:** `warning/unknown` nếu annotator zoom và thấy rõ viền tam giác cảnh báo — ghi lý do trong log.
- **Không chấp nhận:** `warning/hazard_warning` hoặc `other_warning` với `legibility=unreadable` (tự mâu thuẫn).

### 3.2 Biển 8×5 px — ngưỡng 10 px

Nhãn gốc **có** box này, guideline v2 bảo **IGNORE** (§3: cạnh dài nhất < 10 px). Peer vẽ box này → sai theo guideline dù khớp nhãn gốc.

- Nếu annotator nghi là biển nhưng không chắc → **không** vẽ box; có thể gắn tag `image_escalate` + ghi log (§5 bước 2).
- **Mâu thuẫn trong v2:** EX-10 ("phần mặt nhìn thấy 8×6 px → để unknown") trái với ngưỡng 10 px ở §3/§6. Theo
  §3 thì biển 8×6 phải IGNORE. Cần sửa EX-10 trước freeze, nếu không peer sẽ hỏi hoặc vẽ box này.

### 3.3 Hai tam giác + tấm phụ trên cùng cột

3 mặt vật lý → **3 box**, xếp dọc, không chồng lên nhau. Bẫy: gộp thành 1 box hoặc bỏ tấm phụ.

---

## 4. Ảnh normal có biển nhãn gốc bỏ sót

| Ảnh | Đối tượng | Box | Expected |
|---|---|---|---|
| 0959 | Tam giác vàng bị mép trên cắt, phía trên biển W.224 | ~(604, 0, 710, 22) | mất khoảng 40–60% mặt (< 80%), còn thấy một phần hình vẽ → vẽ box sát mép ảnh; `unknown/unknown/not_applicable`, `legibility=partly_readable` hoặc `unreadable` |
| 1979 | Biển tròn xanh viền đỏ bị mép trên cắt, phía trên P.111 | ~(227, 0, 316, 27) | mất khoảng 70–75% mặt (< 80%), còn thấy nền xanh + viền đỏ + gạch chéo → vẽ box sát mép ảnh; `regulatory/other_regulatory/not_applicable/partly_readable` |
| 1979 | Tấm đỏ "CẤM XE 2 VÀ 3 BÁNH" dưới P.111 | ~(236, 137, 309, 169) | `other/supplementary/not_applicable/clear` |

Các biển có trong nhãn gốc của ảnh normal:

| Ảnh | Box | Expected |
|---|---|---|
| 0338 | (520, 234, 581, 292) P.123a | `regulatory/other_regulatory/not_applicable/clear` |
| 0338 | (736, 31, 825, 159) I.434a | `information/place_or_service/not_applicable/clear` — chữ "BUS STOP" nằm trên cùng mặt → 1 box |
| 0664 | (135, 123, 197, 185) P.130 | `regulatory/other_regulatory/not_applicable/clear` |
| 0664 | (631, 103, 690, 164) P.103a | `regulatory/other_regulatory/not_applicable/clear` |
| 0824 | (539, 156, 589, 201) P.102 | `regulatory/no_entry/not_applicable/clear` |
| 0824 | (534, 201, 592, 245) R.302a | `mandatory/turn_direction/not_applicable/clear` |
| 0959 | (606, 36, 710, 132) W.224 | `warning/hazard_warning/not_applicable/clear` |
| 1979 | (230, 39, 324, 134) P.111 | `regulatory/other_regulatory/not_applicable/clear` |

IGNORE ở ảnh normal: biển hiệu cửa hàng ("MUA SẮT PHẾ LIỆU" ở 0338), đèn giao thông (0824) — ngoài scope §1.

---

## 5. Schema và các nghi vấn ở cấp object

Guideline v2 §4 chốt: **không** có attribute `visibility`, `decision`, `review_reason`. Hệ quả với edge case:

| Case | Cách xử lý theo v2 |
|---|---|
| Biển bị cắt mép (0959, 1979) | Không ghi `truncated`; chỉ thể hiện qua box dừng tại biên ảnh và `legibility` |
| R.412: `turn_direction` hay `other_mandatory` | Chọn theo file này (`other_mandatory`); nghi vấn ghi vào **calibration/review log** kèm `sample_id` — không có cách đánh dấu trên box |
| Biển 12–14 px không đọc được | `family=unknown`, `type=unknown`, `legibility=unreadable` |
| Nghi là biển nhưng không đặt được box | Tag ảnh `image_escalate` + log |

**Còn tồn đọng cho spec owner:**

- `02_guideline.md` vẫn ghi **v1**, v2 nằm ở file riêng `02_guideline_v2.md`. `make freeze` và `make handoff` đọc
  `02_guideline.md` và đòi Version ≥ v2 → cần chép v2 vào `02_guideline.md` (hoặc đổi tên file).
- `08_revision_log.md` chưa có dòng v2.
- EX-10 mâu thuẫn với ngưỡng 10 px (xem 3.2).
- Chưa có rule cho biển làn theo loại xe có mũi tên (R.412) và bảng chỉ có tên đường ("XA LỘ HÀ NỘI").

---

## 6. Checklist reviewer

- [ ] Không còn attribute nào = `__undefined__`.
- [ ] `value` ≠ `not_applicable` **chỉ** khi `type=speed_limit`; biển "3.5t", "2.7m" phải là `not_applicable`.
- [ ] Tổ hợp `family → type` hợp lệ theo bảng ràng buộc §4 (ví dụ không có `warning/no_entry`).
- [ ] Biển phụ là box riêng, box biển chính không ôm tấm phụ.
- [ ] Không có box nào có cạnh dài nhất < 10 px (0961: biển 8×5 phải vắng).
- [ ] Biển bị cắt mép: box dừng tại biên ảnh; biển mất > 80% chỉ còn viền thì không có box.
- [ ] Không box cho bảng tuyên truyền, bảng tên đường, biển hiệu cửa hàng, kết cấu cổng.
- [ ] `legibility=unreadable` đi cùng `family`/`type` = `unknown`, không đi cùng một type cụ thể.
- [ ] Đếm box: 0756 = 23, 0961 = 3, 0959 = 2, 1979 = 3, các ảnh còn lại = 2.
