# Guideline gán nhãn — v0.1 (cá nhân) — 104

- Phiên bản: v0.1 (cá nhân, đề xuất từ v0)
- Ngày: 2026-11-16
- Người viết / duyệt: 104

---

## 1. Mục tiêu

- **Mô hình và bài toán:** phát hiện đối tượng 2D bằng bounding box (COCO 1.0) cho **nhận thức giao thông đường phố**, ví dụ camera giao thông hoặc hệ thống hỗ trợ lái. Ảnh có cạnh dài tối đa 1024 px, mọi ngưỡng px trong tài liệu này đo trên ảnh gốc đó.
- **Đối tượng cần nhận diện:** người và phương tiện **thật**, đang hoặc có khả năng tham gia giao thông (kể cả xe đang đỗ).
- **Nguyên tắc chung:** chỉ gán những gì **nhìn thấy trực tiếp** trong ảnh, không suy đoán phần bị che hay phần ngoài mép ảnh.
- **Lỗi nguy hiểm cần tránh (xếp theo mức độ):**
  1. **Dương tính giả với vật giả:** tượng, ma-nơ-canh, ảnh in/biển quảng cáo, tranh tường, đồ chơi, bia mộ, ảnh phản chiếu bị gán thành người/xe thật → mô hình phanh giả hoặc báo động sai.
  2. **Bỏ sót người ngồi trên xe máy** hoặc tách/gộp box không nhất quán → đếm sai, dự đoán quỹ đạo sai.
  3. **Nhãn không nhất quán giữa các annotator** (cùng một tình huống, mỗi người vẽ một kiểu) → mô hình học nhiễu.

---

## 2. Lớp và thuộc tính

### Lớp (mỗi vật thuộc đúng một lớp)

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con 4–9 chỗ (sedan, hatchback, SUV, MPV, taxi con) thật, có khả năng tự di chuyển trên đường. | Taxi xanh-đỏ cận cảnh (`img_040`), SUV xám (`img_012`), Honda đen (`img_023`), xe đỏ và xe bạc cắt mép (`img_015`), Honda Civic cắt mép (`img_032`). | Bán tải, xe tải nhỏ có thùng hàng (→ `truck`); xe van (→ `van`); xe đồ chơi (`img_038`); bia mộ hình xe (`img_027`); xe chỉ xuất hiện trong gương (`img_020`); bóng xe phản chiếu trên mặt đường ướt (`img_015`, `img_031`). |
| `van` | Xe thùng kín cỡ nhỏ/trung, khoang chở liền với cabin, hoặc minibus nhỏ. | Van bạc `51LD-70-65` (`img_005`), van trắng kiểu HiAce (`img_016`), minibus nhỏ (`img_009`, tạm xếp `van` nếu nhóm không thêm lớp `bus`). | Xe bán tải có thùng rời (→ `truck`); xe buýt lớn (`img_037`, `img_040` ở nền, đề xuất lớp `bus`); ô tô con (→ `car`). |
| `truck` | Xe tải hoặc bán tải có khoang hàng/thùng tách khỏi cabin, kể cả đầu kéo kèm rơ-moóc. | Xe chở xe (`img_002`), Isuzu thùng cao và bán tải trắng (`img_007`), xe tải nhỏ thùng lửng (`img_001`), `VERYCA` thùng bạt (`img_036`), bán tải chở hàng (`img_012`), bán tải có thùng mái che (`img_037`), bán tải cổ (`img_028`). | Ô tô con liền khối (→ `car`); van (→ `van`); hàng hóa nhô khỏi khung xe (không phải thân xe). |
| `motorcycle` | Phương tiện cơ giới 2 bánh (xe số, tay ga, mô tô) và xe 3 bánh có động cơ (tuk-tuk, xe lam) **nếu nhóm chưa thêm lớp riêng**. | Xe máy chạy (`img_005`, `img_006`, `img_026`, `img_034`), xe máy đỗ (`img_008`, `img_030`), sidecar (`img_022`), tuk-tuk (`img_004`, `img_016`, `img_035`). | Người lái (box `person` riêng); xe đạp và xích lô đạp chân (`img_003`, `img_023`, `img_029`); xe máy kim loại dạng điêu khắc (`img_011`); chỉ thấy gương/tay lái nhỏ (`img_040`, xem R-03). |
| `person` | Người thật (đi bộ, đứng, ngồi, đạp xe, lái hoặc ngồi trên xe lộ thiên) thấy được ít nhất phần đầu hoặc thân. | Người đi bộ (`img_014`, `img_034`), người lái và người ngồi sau xe máy (`img_005`, `img_026`, `img_008`), tài xế tuk-tuk (`img_004`), người đạp xe (`img_006`, `img_029`), người đứng cạnh xe (`img_001`, `img_002`). | Tượng và điêu khắc (`img_011`, `img_017`, `img_028`), ma-nơ-canh (`img_033`), linh vật (`img_019`), chân dung in trên biển quảng cáo/kính xe (`img_018`, `img_033`, `img_040`), người ngồi trong ô tô thấy qua kính (R-10). |
| `ignore_region` | Vùng không gán từng vật: đám đông/cụm xe dày không tách được, hoặc vật cao dưới 20 px. Bỏ sót trong vùng này không tính là lỗi. | Đám chờ xe buýt (`img_032`), dòng xe máy ở xa (`img_021`), cụm xe đỗ chồng chéo phía sau (`img_008`, `img_030`), người nhỏ ở xa (`img_025`, `img_039`). | Vật đơn lẻ, rõ, cao từ 20 px dù bị che một phần (vẫn phải có box riêng). Không gán vào tượng/ảnh in (R-04). Lớp này không có thuộc tính. |

**Vật không thuộc lớp nào** (xe đạp, xích lô, xe lăn, động vật, tượng, đồ chơi...) **không gán và không bao bằng `ignore_region`**. Nếu ảnh không còn vật hợp lệ thì ảnh vẫn có mặt trong COCO ở dạng rỗng.

> **Đề xuất cho Pha 2** (nhóm chốt có thêm lớp hay không):
> - lớp `bus` cho `img_009`, `img_037`, `img_040`, `img_031`;
> - lớp `tuktuk` (hoặc `three_wheeler`) cho `img_004`, `img_016`, `img_035`.
>
> Nếu không thêm, quyết định tạm: tuk-tuk → `motorcycle`; minibus → `van`; bus lớn → `truck` kèm `can_review`.

### Thuộc tính

| Thuộc tính | Kiểu | Giá trị | Định nghĩa và ngưỡng cụ thể |
|---|---|---|---|
| `occluded_level` | `select` | `0`, `1`, `2`, `3` | Tỷ lệ phần **thân vật bị vật khác che** (ước lượng bằng mắt trên phần vật lẽ ra thấy được trong khung ảnh):<br>- `0`: không che hoặc che dưới 10%.<br>- `1`: che từ 10% đến dưới 40%, vẫn nhận ra hình dáng tổng thể.<br>- `2`: che từ 40% đến dưới 75%, chỉ thấy một phần nhưng đủ nhận ra lớp.<br>- `3`: che từ 75% trở lên, lớp nhận ra chủ yếu nhờ ngữ cảnh. Nếu phần thấy còn dưới 20 px cao thì chuyển sang R-06.<br>**Lưu ý:** (a) phần bị **cắt mép ảnh** không tính vào `occluded_level` mà ghi ở `truncated`; (b) người đang ngồi trên xe **không** được tính là vật che chiếc xe đó; (c) đây là thuộc tính của nhóm, khác với cờ occluded có sẵn của CVAT. |
| `truncated` | `select` | `no`, `yes` | `no`: toàn bộ vật nằm trong khung ảnh. `yes`: vật bị cắt bởi ít nhất một mép ảnh (trên, dưới, trái, phải). |
| `can_review` | `checkbox` | `true` / `false` | Tick `true` khi **đã vẽ box nhưng còn phân vân** về lớp, ranh giới box hoặc mức che. Không tick cho box chắc chắn. Vật chưa có box thì ghi vào `image_issues.csv` (cột `image,annotator,region,note`). |

---

## 3. Quy tắc vẽ box

Các quy tắc được xếp theo thứ tự các chủ đề của khung guideline. Mỗi quy tắc có mã `R-xx` để dẫn lại ở bảng ca khó và nhật ký QC.

1. **R-01 — Ôm sát phần nhìn thấy.** Box ôm sát pixel nhìn thấy của vật: không tự vẽ tràn qua phần bị che, qua mép ảnh, hay ôm cả bóng đổ và phản chiếu. Với phương tiện, box gồm thân, bánh, tay lái, gương cố định; không gồm ăng-ten hoặc hàng nhô ra (R-08).
2. **R-02 — Vật bị che.** Vật bị che một phần vẫn gán nếu nhận ra lớp **và** phần thấy cao từ 20 px; box ôm phần thấy, ghi `occluded_level` theo mục 2. Khi hai vật chồng nhau, mỗi vật giữ box ôm phần **của chính nó**, không cắt box để tránh chồng. Người ngồi trên xe không được tính là vật che chiếc xe đó. Che từ 75% trở lên mà phần thấy còn dưới 20 px thì xử lý theo R-06.
3. **R-03 — Vật bị cắt mép ảnh.** Vật bị cắt nhưng vẫn nhận ra lớp: gán box ôm phần thấy, `truncated=yes`. Nếu chỉ thấy một bộ phận nhỏ (gương xe máy ở góc `img_040`, mép bánh xe, mép cửa) hoặc phần thấy cao dưới 20 px thì không gán, ghi vào `image_issues.csv`.
4. **R-04 — Vật giả, tượng, hình in, đồ chơi, phản chiếu.** Không gán `person`, `car`, `truck`, `motorcycle` cho: tượng và điêu khắc, ma-nơ-canh, đồ chơi/mô hình, bia mộ, chân dung in trên biển quảng cáo/kính xe, tranh tường (kể cả hình tàu thủy ở `img_026`), ảnh phản chiếu qua gương/kính/mặt đường ướt. Không bao bằng `ignore_region` (đây là vật rõ ràng, không phải vùng bất định). Ảnh không còn vật hợp lệ vẫn giữ ở dạng rỗng (`img_017`, `img_020`, `img_027`, `img_033`, `img_038`).
5. **R-05 — Xe chủ (ego).** Không gán bộ phận xe của người chụp: gương chiếu hậu, cửa kính, mui xe (`img_020`, phần góc phải `img_033`).
6. **R-06 — Vật quá nhỏ và đám đông.**
   - Vật đơn lẻ cao **từ 20 px** trở lên: luôn có box riêng, dù bị che nhiều.
   - Vật đơn lẻ cao **dưới 20 px**: không vẽ box riêng, bao bằng một `ignore_region` nhỏ.
   - Cụm hoặc đám đông không tách được từng cá thể: một `ignore_region` bao cả cụm (`img_021`, `img_030`, `img_032`, `img_039`).
   - Nếu có vài vật tiền cảnh rõ (cao từ 20 px), vẽ box cho chúng và dùng `ignore_region` cho phần còn lại.
7. **R-07 — Một vật nhiều phần: người và xe máy.** Mỗi người ngồi trên xe có **một box `person`** riêng; mỗi chiếc xe có **một box `motorcycle`**. Box xe ôm phần thân xe nhìn thấy, không cần bao cả người; hai box được phép chồng lên nhau. Xe chở hai hoặc ba người thì có hai hoặc ba box `person` và một box `motorcycle` (`img_005`, `img_006`, `img_008`, `img_026`).
8. **R-08 — Hàng hóa và vật được chở.** Box phương tiện ôm thân xe cùng phần hàng nằm trong khung xe. **Không** kéo box theo hàng nhô ra khỏi thân xe (ống dài `img_029`, bao hàng chất cao `img_035`); nếu hàng chiếm quá nhiều diện tích box thì tick `can_review`. **Vật được chở** (ô tô trên xe chở xe `img_002`, tượng ngựa trên thùng bán tải `img_028`, hàng trên thùng `img_012`) **không** có box riêng và nằm trong box của xe chở.
9. **R-09 — Nhòe chuyển động.** Vật nhòe nhưng vẫn nhận ra lớp và cao từ 20 px thì gán box ôm phần thấy rõ, tick `can_review` nếu ranh giới không xác định (`img_014`, `img_015`). Nếu không nhận ra lớp thì dùng R-06.
10. **R-10 — Người trong phương tiện.** Người ngồi trong xe **lộ thiên** (xe máy, tuk-tuk, xích lô) gán `person` nếu thấy rõ đầu hoặc thân (`img_004`). Người ngồi trong **xe kín** chỉ thấy qua kính thì **không** gán.
11. **R-11 — Người đạp xe và phương tiện ngoài lớp.** Người đi xe đạp/xích lô vẫn gán `person` (`img_006`, `img_023`, `img_029`). Xe đạp và xích lô không có lớp nên không gán.
12. **R-12 — Xe đỗ, xe trưng bày, xe diễu hành.** Xe thật đang đỗ vẫn gán như xe thường (`img_012`, `img_018`, `img_024`). Xe mô hình, đồ chơi, tượng xe không gán (R-04). Xe thật nhưng thiết kế đặc biệt (xe diễu hành `img_019`, xe đặt trên nóc xe trong bảo tàng `img_024`) thì gán nếu chắc là xe thật, kèm `can_review`.

---

## 4. Ca khó (12 ảnh pilot)

> 12 ca khó, mỗi ca là một ảnh trong lô pilot (`img_002, 004, 005, 006, 008, 012, 017, 020, 026, 027, 033, 040`). Quyết định dưới đây là quyết định ban đầu của cá nhân 104; chỗ ghi "Pha 2" cần nhóm chốt. Nếu chưa chốt, đánh dấu `can_review` trên box hoặc ghi vấn đề cấp ảnh trong `image_issues.csv`. Dùng mã quy tắc `R-xx` trong nhật ký QC.

| Ảnh | Tình huống | Rule ID | Quyết định ban đầu của 104 | Lý do |
|---|---|---|---|---|
| `img_002` | Xe tải chở xe, rơ-moóc hai tầng chở nhiều ô tô con (từ 7 xe), một người đứng ở mép phải. | R-01, R-08 | 1 box `truck` ôm cả đầu kéo và rơ-moóc cùng các xe trên đó; **không** gán `car` riêng cho xe được chở; 1 `person` cho người đứng bên phải. Chốt ở Pha 2: ôm cả tổ hợp hay tách đầu kéo/rơ-moóc. | Xe được chở không phải phương tiện tham gia giao thông độc lập; gán sẽ gây dương tính giả. Cần kiểm lại ảnh nào khác bị ảnh hưởng nếu đổi quy tắc (`img_028`, `img_012`). |
| `img_004` | Hai tuk-tuk 3 bánh đỗ cạnh nhau ban đêm, tài xế ngồi trong cabin bị che một phần. | R-07, R-10 | Tuk-tuk → `motorcycle` (tạm); tài xế → `person`, `occluded_level=2`. Đề xuất lớp `tuktuk` ở Pha 2. | v0 không có lớp xe 3 bánh; tài xế thấy rõ đầu và thân nên vẫn gán. |
| `img_005` | Cận cảnh nhiều người đi xe máy chồng chéo, cắt mép dưới/phải, phía sau có xe van `51LD-70-65`. | R-01, R-07, R-03, R-02 | Mỗi người một `person` (`truncated=yes` nếu bị cắt mép); xe máy chỉ lộ tay lái/đầu xe và cao từ 20 px thì gán `motorcycle`, nếu không thì R-03; xe van → `van`. | Giữ nguyên tắc tách người và xe ngay cả khi mật độ dày. |
| `img_006` | Đường đông, nhiều xe máy nhìn từ phía sau, người đạp xe ở bên phải, ô tô trắng ở trái. | R-07, R-06, R-11 | Mỗi xe máy một `motorcycle` và mỗi người một `person`; người đạp xe → `person`, không gán xe đạp; người/xe ở xa dưới 20 px → `ignore_region`. | Nhất quán với `img_005`; xe đạp không có lớp. |
| `img_008` | Hàng xe máy đỗ dày bên trái, hai người mặc áo cam (xe ôm): một đứng bên trái, một ngồi trên xe máy trắng bên phải (xe cắt mép). | R-07, R-06, R-02 | Xe tiền cảnh rõ → `motorcycle` từng chiếc (`occluded_level` theo mức che); cụm xe đỗ chồng ở nền → `ignore_region`; hai người áo cam → `person`. | Không bỏ sót người; không ép gán từng xe khi không tách được. |
| `img_012` | SUV xám bên trái; bán tải trắng thùng chất đầy hàng ở phía trước, cắt mép; xe khác ở xa. | R-08, R-03, R-12 | SUV → `car`; bán tải → `truck`, `truncated=yes`, box ôm thân xe và hàng trong thùng; xe ở xa gán nếu cao từ 20 px. | Hàng nằm trong khung thùng; không gán riêng thùng hàng. |
| `img_017` | Tượng người cưỡi ngựa bằng đồng, đầu đội cọc giao thông, trước cột đá. | R-04 | **Không gán.** Ảnh rỗng. | Là tượng, không phải người thật. Đây là ca hai người dễ cùng vẽ `person`, IAA cao nhưng vẫn sai nên cần quy tắc rõ. |
| `img_020` | Gương chiếu hậu của xe chủ phản chiếu đường và một ô tô. | R-04, R-05 | **Không gán** (cả xe trong gương lẫn phần xe chủ). | Ảnh phản chiếu có phối cảnh đảo, không phải vật trực tiếp trên đường. |
| `img_026` | Ban đêm, hai người ngồi chung một xe máy đỏ, cắt mép trái, trước tranh tường cảng và tàu thủy. | R-07, R-04, R-03 | 2 `person` + 1 `motorcycle` (`truncated=yes` nếu bị cắt); **không** gán tàu thủy hay người in trên tranh tường. | Tranh tường là ảnh in; nền tối khiến ranh giới khó, tick `can_review` nếu cần. |
| `img_027` | Bia mộ điêu khắc hình xe bán tải bằng đá đen trên bãi cỏ nghĩa trang. | R-04 | **Không gán.** Ảnh rỗng. | Vật giả (điêu khắc), gán `truck` là dương tính giả. |
| `img_033` | Ma-nơ-canh trong tủ kính, ảnh in người mẫu, bóng phản chiếu và cửa/gương xe ở góc phải. | R-04, R-05 | **Không gán** cả ba. Ảnh rỗng. | Ma-nơ-canh, ảnh in và phản chiếu đều là ca đối chứng ngoài phạm vi. |
| `img_040` | Người đội mũ bảo hiểm cận cảnh (cắt mép dưới) trước taxi xanh-hồng; nền có SUV đen, tuk-tuk có biển TAXI, xe buýt; chân dung in trên kính taxi; gương xe máy ở góc trái dưới. | R-01, R-04, R-03 | `person` (người đội mũ, `truncated=yes`); taxi → `car` (`occluded_level` do người che); SUV đen → `car` nếu cao từ 20 px; tuk-tuk → `motorcycle` (tạm); xe buýt → xem đề xuất `bus`; **không** gán chân dung in; gương xe máy ở góc → không gán, ghi `image_issues.csv`. | Nhiều lớp cùng xuất hiện; cần nhất quán giữa người và xe. |

---

## 5. QC và phiên bản

- **Ngưỡng QC của lab** (khớp `taxonomy.json`):
  - Độ phủ trước IAA: 100% lượt trong `assignment.csv`.
  - IAA trước calibration: IoU trung bình ≥ 0.85 và đồng thuận lớp ≥ 95%, tính trên cặp box ghép 1–1 với IoU ghép từ 0.5.
  - Ngưỡng IoU theo box: ≥ 0.85 nếu box cao từ 50 px; ≥ 0.70 nếu nhỏ hơn; vật cao dưới 20 px xử lý theo R-06.
  - Release: tỷ lệ ảnh lỗi ≤ 2% trên **toàn bộ lô** được duyệt.
- **Ca mơ hồ khi gán (Pha 3):** tick `can_review` cho box đã vẽ; vùng không vẽ được ghi vào `image_issues.csv`; không xem nhãn của người khác trước khi nộp bản độc lập.
- **Ai chốt ca mơ hồ:** cả nhóm ở Pha 4–5, ghi lại mã ca (`C01`, `C02`...), rule ID, phiên bản và người xác nhận trong `calibration_decisions.csv`.
- **Quy tắc đặt tên:** rule ID theo dạng `R-xx`; dùng cùng ID trong phiếu QC.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| `v0` | Bản khởi đầu của giảng viên. | Thiếu ngưỡng che khuất, quy tắc người/xe máy và ca đối chứng. |
| `v0.1` | Thêm ngưỡng `occluded_level` và cách ước lượng; R-01 đến R-12; định nghĩa kèm phản ví dụ; bảng 12 ca khó (12 ảnh pilot); đề xuất lớp `bus`/`tuktuk` cho Pha 2. | Chuẩn bị họp Pha 2: `img_002, 004, 005, 006, 008, 012, 017, 020, 026, 027, 033, 040` (12 ảnh pilot). |
