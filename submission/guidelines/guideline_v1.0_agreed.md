# Guideline gán nhãn — v1.0 — bản hợp nhất đề xuất cho nhóm

- **Phiên bản mục tiêu:** `v1.0`
- **Ngày tổng hợp:** 2026-11-16
- **Căn cứ:** `guideline_101.md` (v0.1), `guideline_102.md` (v0.1-candidate), `guideline_103.md` (v0.1-candidate), `guideline_104.md` (v0.1); `submission/taxonomy.json` (v0); `GUIDE.md` và `RULES.md` của bài Day 07.
- **Người đề xuất phương án:** Bản tổng hợp hỗ trợ từ ba guideline cá nhân; **không thay thế cuộc họp và quyết định của nhóm**.
- **Trạng thái:** **DỰ THẢO CHỜ NHÓM XÁC NHẬN**. Chỉ đổi thành `ĐÃ THỐNG NHẤT — KHÓA v1.0` sau khi các thành viên xác nhận những điểm ở mục 6 và taxonomy được cập nhật, chạy kiểm tra thành công.
- **Phạm vi:** 40 ảnh `img_001` đến `img_040`; Object Detection 2D; thao tác Bounding Box thủ công trong CVAT; xuất COCO 1.0.

## 1. Mục tiêu

Tạo dữ liệu gán nhãn nhất quán cho mô hình nhận diện **người thật và phương tiện thật xuất hiện trực tiếp** trong ảnh giao thông. Mỗi đối tượng hợp lệ có một box thuộc đúng một lớp trong taxonomy. Ưu tiên tránh bỏ sót người/phương tiện có thể nhận ra, tránh nhầm vật giả hoặc phản chiếu với vật thật, tránh suy đoán vùng không nhìn thấy và đảm bảo người gán nhãn khác nhau có thể áp dụng cùng quy tắc.

Chỉ sử dụng nhãn do con người quan sát và vẽ. Không dùng mô hình tự động pre-label trong Day 07. Không bổ sung lớp mới ngoài taxonomy đã được cung cấp.

## 2. Lớp và thuộc tính

### 2.1. Sáu lớp được giữ nguyên

| Lớp | Định nghĩa vận hành | Ví dụ | Không gán / không nhầm với |
|---|---|---|---|
| `car` | Ô tô con thân chính phục vụ chở người: sedan, hatchback, wagon, taxi con, SUV/MPV cỡ ô tô con. | Taxi/SUV `img_040`; xe con thật trên xe vận chuyển `img_002`. | Pickup có thùng hàng là `truck`; van thân kín là `van`; xe mô hình `img_038`. |
| `truck` | Xe có cabin và thùng/sàn chở hàng riêng: bán tải, xe tải thùng, xe chở ô tô, đầu kéo + rơ-moóc đang nối thành tổ hợp. | Pickup `img_001`, `img_012`, `img_037`; xe chở xe `img_002`; xe tải `img_036`. | SUV là `car`, van liền khối là `van`; hàng rời không tính vào thân xe. |
| `van` | Xe van thân kín cỡ nhỏ, khoang liền thân dùng chở người hoặc hàng; không phải xe buýt chuyên dụng. | Van `img_005`, van trắng `img_016`. | Xe có thùng hàng tách cabin là `truck`; xe buýt/minibus thiết kế như bus không ép vào `van`. |
| `motorcycle` | Xe máy/mô tô/scooter hai bánh có động cơ, và xe ba bánh có động cơ điều khiển theo kiểu xe máy, gồm tuk-tuk và sidecar cố định. | Tuk-tuk `img_004`, `img_035`; sidecar `img_022`; scooter `img_010`. | Xe đạp, xích lô đạp chân không gán; người ngồi trên xe phải có box `person` riêng. |
| `person` | Người thật xuất hiện trực tiếp, đi bộ, ngồi, đứng, lái xe hoặc đạp xe; người mặc đồ hóa trang nếu nhận ra là người thật. | Người lái `img_004`, hai người trên xe `img_026`; người đi bộ `img_014`. | Tượng `img_017`, ma-nơ-canh `img_033`, chân dung quảng cáo, người chỉ xuất hiện trong phản chiếu. |
| `ignore_region` | Vùng khó gán cá thể riêng vì nhiều mục tiêu quá nhỏ hoặc chồng lấn dày đặc; **không phải một vật thể**. | Các cụm người/xe không phân ranh ổn định trong `img_021`, `img_030`, `img_032`. | Không dùng làm lớp `other`, không bao vật giả hoặc vật ngoài taxonomy, không che cá thể rõ. |

**Quy tắc phạm vi:** Bus/minibus loại xe buýt, xe đạp, xích lô đạp chân, ngựa/động vật, tượng, đồ chơi, mô hình, tranh/hình in, ma-nơ-canh và vật chỉ xuất hiện qua phản chiếu **không được gán**. Không thay bus thành `truck` hay `van` chỉ để có box. Nếu một phương tiện nhỏ trông giống van lẫn minibus, dựa vào kết cấu/chức năng nhìn thấy; không xác định chắc thì ghi vấn đề để reviewer chốt. Xe thật đang đỗ, trong kho/bảo tàng hoặc đang được vận chuyển vẫn có thể gán nếu đủ điều kiện.

### 2.2. Thuộc tính

| Thuộc tính | Giá trị | Quy tắc chung được đề xuất |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | Ước lượng phần cơ thể/thân xe bị **vật tiền cảnh** che: `0`: 0%; `1`: trên 0% đến 30%; `2`: trên 30% đến 70%; `3`: trên 70% đến dưới 100% nhưng vẫn nhận ra rõ đối tượng. |
| `truncated` | `no`, `yes` | `yes` khi vật bị một hay nhiều mép ảnh cắt; box dừng tại biên ảnh. Cắt mép không tính vào che khuất. |
| `can_review` | `false`, `true` | Tick `true` cho box đã vẽ nhưng còn phân vân về lớp, ranh giới, mức che hoặc việc tách cá thể. Không dựng box giả để đánh dấu; ca chưa vẽ được ghi vấn đề cấp ảnh. |

Không coi trời tối, chuyển động nhòe hoặc kính trong suốt là che khuất. Một vật vừa bị vật khác che vừa bị cắt mép có thể có `occluded_level > 0` và `truncated=yes`.

**Lưu ý quyết định nhóm:** Bản 104 đề xuất các mốc che khuất 10% / 40% / 75%, khác bản 102. Bản hợp nhất **đề xuất** mốc 0% / 30% / 70% để các annotator áp dụng đồng nhất; nhóm phải xác nhận lựa chọn trước khi khóa.

## 3. Quy tắc gán nhãn (Rule ID thống nhất)

- **R-01 — Điều kiện gán:** Chỉ gán vật thật thuộc `car`, `truck`, `van`, `motorcycle`, `person`, nhìn thấy trực tiếp, nhận ra được lớp và có **chiều cao box phần nhìn thấy ≥ 20 px** trên ảnh dùng để gán. Vật đơn lẻ dưới 20 px không tạo box cá thể.
- **R-02 — Box theo phần nhìn thấy:** Một box chữ nhật ôm sát các pixel ngoài cùng **đang nhìn thấy** của một cá thể; không dựng hình các phần ở sau vật che hoặc ngoài ảnh. Nếu các phần nhìn thấy của cùng vật bị chia thành nhiều mảnh, vẫn dùng một box bao các mảnh nhìn thấy. Không ôm bóng đổ, vật phản chiếu hoặc hàng rời nhô ra.
- **R-03 — Che khuất:** Vật bị che vẫn gán khi còn đủ dấu hiệu để xác định lớp và phần nhìn thấy cao ≥ 20 px; đánh `occluded_level` theo bảng mục 2.2. Nếu không đủ bằng chứng phân loại/cá thể, không đoán box.
- **R-04 — Cắt mép ảnh:** Vật thật bị cắt mép vẫn gán nếu đạt R-01; dừng box đúng ranh ảnh và đặt `truncated=yes`. Không nới box ngoài ảnh. Chỉ nhìn thấy một bộ phận không đủ nhận dạng thì không gán.
- **R-05 — Người và phương tiện:** Người lái/ngồi và xe là các cá thể riêng; box được phép chồng nhau. Hai người trên một xe máy → **2 `person` + 1 `motorcycle`** (`img_026`). Người đạp xe/xích lô vẫn là `person` khi đạt ngưỡng, dù xe không thuộc taxonomy. **Đề xuất** gán cả người thật nhìn trực tiếp qua cửa kính xe kín khi có đủ bằng chứng và cao ≥ 20 px; nếu không rõ thì ghi issue, không suy đoán.
- **R-06 — Xe chở xe, tổ hợp và hàng:** Đầu kéo + rơ-moóc đang nối tạo một box `truck`; ô tô thật trên xe chở xe (`img_002`) **có box `car` riêng** khi đạt R-01. Không nới box của xe chở theo ô tô/hàng được chở; box truck chỉ bao phần kết cấu riêng của xe, dù các box `car` bên trên có thể chồng box `truck`. Xe máy gắn sidecar là một box `motorcycle`. Hàng/ống/bao tải và tượng ngựa không có box riêng, không làm box xe phình theo hàng rời.
- **R-07 — Phân biệt phương tiện:** Pickup dù có cap/nắp vẫn là `truck`; sedan/SUV/taxi con là `car`; xe van thân kín liền khối là `van`; tuk-tuk/sidecar có động cơ là `motorcycle`. Không ép xe buýt vào `van` hay `truck` khi taxonomy không có `bus`. Nếu không rõ giữa hai lớp hợp lệ: chọn lớp có bằng chứng mạnh nhất, tick `can_review` và ghi lý do.
- **R-08 — Loại vật giả:** Không gán tượng, điêu khắc, ma-nơ-canh, đồ chơi, mô hình, hình in/quảng cáo/tranh hoặc mascot chỉ là đạo cụ. Người thật đang mặc trang phục nhân vật được gán `person` khi có bằng chứng đủ chắc là người thật. Không dùng `ignore_region` cho các vật bị loại theo quy tắc.
- **R-09 — Phản chiếu và xe chủ:** Không gán xe/người chỉ nhìn thấy trong gương hoặc phản chiếu trên kính/kim loại/mặt đường; không gán gương, nắp ca-pô, cửa của chính xe mang camera (`ego`). Vật khác nhìn trực tiếp xuyên qua kính vẫn có thể gán nếu đạt R-01. `img_020` là ca âm.
- **R-10 — Đám đông / `ignore_region`:** Gán cá thể tách được và cao ≥ 20 px trước. Chỉ tạo vùng `ignore_region` nhỏ/sát cụm từ **ba mục tiêu trở lên** bị chồng lấn hoặc quá nhỏ khiến không thể phân cá thể ổn định. Không bao cả ảnh và không gán trùng cá thể nằm hoàn toàn trong ignore; thu nhỏ vùng để giữ các vật rõ ở rìa. Vật nhỏ **đơn lẻ** dưới 20 px bỏ qua, không tự tạo ignore riêng cho từng vật.
- **R-11 — Nhỏ, xa, tối, nhòe:** Một vật cao < 20 px thì bỏ box riêng. Vật tối hoặc nhòe nhưng xác định được lớp, cá thể và chiều cao ≥ 20 px vẫn gán theo biên còn quan sát được; tick `can_review` nếu biên thiếu chắc chắn. Nhòe không đồng nghĩa bị che.
- **R-12 — Trạng thái xe và bối cảnh:** Xe thật vẫn gán nếu đang chạy, đứng yên, đỗ, đặt trong kho/bảo tàng hoặc là xe thật được vận chuyển. Loại xe đồ chơi, mô hình và tượng, kể cả khi hình dạng giống xe thật. Xe diễu hành vẫn gán nếu là xe thật vận hành được.
- **R-13 — Vật ngoài taxonomy:** Không có lớp `bus`, `bicycle`, `cyclo`, `horse` hoặc `other`; không thêm lớp và không ép vào lớp gần nhất. Người thật trên các phương tiện ngoài taxonomy vẫn có thể nhận box `person` theo R-01.
- **R-14 — Ca chưa chắc chắn:** Đã có box hợp lệ nhưng còn mơ hồ → tick `can_review` và ghi lý do. Không có bằng chứng để vẽ box hợp lệ → không vẽ; ghi `image,annotator,region,note` vào nhật ký cấp ảnh **sau khi khóa nhãn độc lập**, để reviewer phân xử. Không xem nhãn người khác trong pha gán độc lập.

## 4. Ca khó và quyết định đề xuất

Các quyết định dưới đây là **đề xuất của bản hợp nhất từ 3 tài liệu**, chưa thay cho xác nhận nhóm.

| Ảnh | Ca khó | Rule ID | Quyết định áp dụng đề xuất | Lý do |
|---|---|---|---|---|
| `img_002` | Xe tải kéo rơ-moóc chở nhiều xe con | R-02, R-06 | Một `truck` cho tổ hợp đầu kéo/rơ-moóc và **mỗi xe con thật** có một `car` riêng nếu ≥20 px; gán `person` đủ điều kiện. | 102 gán xe con riêng; 104 không gán. Ưu tiên tính nhất quán: xe con thật vẫn là vật thật; không gộp hàng được chở vào box kết cấu xe tải. **Cần cả nhóm phê duyệt.** |
| `img_004` | Tuk-tuk và người lái | R-05, R-07 | Mỗi tuk-tuk → `motorcycle`; mỗi người thật đủ điều kiện → `person`. | Giữ taxonomy sáu lớp, không tự thêm `tuktuk`. |
| `img_005` | Xe máy/người chồng lấn, van nền | R-03, R-05 | Người và xe máy box riêng, `van` cho xe van, thuộc tính che/cắt theo từng vật. | Không ghép người+xe. |
| `img_006` | Xe máy, người đạp xe, xe đạp, cụm đông | R-05, R-10, R-13 | Gán `person` và `motorcycle` tách được, không gán xe đạp; cụm không thể tách mới dùng ignore. | Không ép xe đạp thành xe máy. |
| `img_008` | Bãi xe máy đỗ rất sát nhau | R-03, R-10, R-12 | Xe nào tách được thì gán, cụm chồng lấn không rõ mới ignore; người có box riêng. | Xe đỗ vẫn là xe thật. |
| `img_009` | Minibus/van nhỏ | R-07, R-13 | Chỉ gán `van` nếu hình thái là van liền khối; nếu rõ là xe buýt/minibus chuyên dụng thì không gán phương tiện, ghi issue khi chưa phân giải được. | 102 loại minibus/bus, 104 muốn `van`; không được thay taxonomy không qua nhóm. **Cần xác nhận.** |
| `img_011` | Người hóa trang, mô tô điêu khắc | R-08 | Người biểu diễn thật đủ điều kiện → `person`; mô tô điêu khắc không gán. | Phân biệt người thật và hình mô phỏng. |
| `img_012` | Pickup chở hàng và SUV | R-02, R-06, R-07 | Pickup → `truck`, SUV → `car`; không nới box pickup theo hàng nhô ra. | Hàng không phải bộ phận kết cấu xe. |
| `img_014` | Xe và người nhòe do chuyển động | R-01, R-11 | Vẽ riêng nếu vẫn nhận diện được cá thể và ≥20 px; không đoán phần nhòe không xác định. | Nhòe khác che. |
| `img_017` | Tượng người cưỡi ngựa | R-08 | Không gán, giữ ảnh COCO rỗng nếu không có vật thật khác. | Không phải người thật. |
| `img_018` | Người quảng cáo và xe thật sau hàng rào | R-03, R-08 | Không gán hình in; ô tô thật còn nhìn thấy và đủ ngưỡng thì gán với che khuất thích hợp. | Vật thật và ảnh in phải phân biệt. |
| `img_019` | Xe diễu hành, tài xế và mascot | R-08, R-12, R-14 | Xe thật → `car` nếu đủ dấu hiệu; tài xế thật → `person`; mascot/đạo cụ không gán khi không đủ bằng chứng là người thật. | Có thể ghi issue cho mascot. |
| `img_020` | Xe trong gương chiếu hậu, phần xe chủ | R-09 | Không gán. | Phản chiếu và ego đều ngoài phạm vi. |
| `img_021` | Cụm xe máy/người đông | R-10 | Tách cá thể rõ để gán; dùng ignore cho phần còn lại không thể phân ranh. | Không bỏ toàn bộ ảnh vào ignore. |
| `img_022` | Xe máy gắn sidecar | R-06, R-07 | Một box `motorcycle` cho tổ hợp gắn cố định, người có box riêng nếu đủ điều kiện. | Một phương tiện. |
| `img_023` | Xích lô và người | R-05, R-13 | Không gán phương tiện xích lô; người thật đủ điều kiện vẫn gán. | Xích lô không thuộc taxonomy. |
| `img_024` | Xe thật trong bảo tàng, mô hình trên nóc | R-08, R-12 | Gán ô tô thật; không gán xe mô hình màu vàng. | Trưng bày không đồng nghĩa đồ chơi. |
| `img_026` | Hai người trên một xe máy | R-05 | Hai `person`, một `motorcycle` (nếu đủ điều kiện); gán `truncated` đúng vật bị cắt mép. | Ba đối tượng độc lập. |
| `img_027` | Bia mộ/tượng hình pickup | R-08 | Không gán. | Vật điêu khắc. |
| `img_028` | Pickup thật chở tượng ngựa | R-06, R-08 | Pickup `truck`; tượng ngựa không gán, không kéo box theo tượng. | Hàng rời ngoài thân. |
| `img_029` | Xe máy chở ống dài và người đạp xe | R-02, R-05, R-13 | Gán xe máy và từng người đủ điều kiện; không gán xe đạp, không kéo box theo ống. | Hàng nhô và xe đạp ngoài taxonomy. |
| `img_030` | Nhiều xe máy đỗ dày | R-03, R-10 | Xe riêng tách được gán, cụm không tách được ignore, người rõ gán riêng. | Tránh box suy đoán. |
| `img_032` | Người chờ bus đông, ô tô cắt mép | R-04, R-10 | Người có thể phân cá thể và ≥20 px gán riêng; phần quá dày ignore; ô tô thật cắt mép → `car`, `truncated=yes`. | Không tự động ignore toàn bộ đám đông. |
| `img_033` | Ma-nơ-canh, ảnh in và phản chiếu | R-08, R-09, R-14 | Không gán vật giả/phản chiếu; nếu có vật thật nhìn trực tiếp chưa rõ thì ghi issue cấp ảnh. | Ca đối chứng dễ nhầm. |
| `img_035` | Tuk-tuk chở bao hàng cao | R-02, R-06, R-07 | `motorcycle` cho tuk-tuk, không kéo box theo hàng; người thật đủ điều kiện gán riêng. | Không thêm lớp xe ba bánh. |
| `img_036` | Xe tải nhỏ thùng kín | R-07 | `truck` nếu có cabin và thùng hàng riêng; nếu thực tế một thân van liền khối thì `van` và ghi issue nếu mơ hồ. | Căn cứ cấu tạo, không dựa tên gọi chung. |
| `img_037` | Pickup có cap và bus nền | R-07, R-13 | Pickup → `truck`; bus nền không gán; người đủ điều kiện gán riêng. | Không ép bus thành truck. |
| `img_038` | Tủ kính ô tô đồ chơi | R-08, R-09 | Không gán; giữ ảnh COCO rỗng nếu không có vật thật khác. | Đồ chơi và phản chiếu. |
| `img_040` | Taxi/SUV, tuk-tuk, bus, người bị che và hình in | R-03, R-04, R-07, R-08, R-13 | Taxi/SUV → `car`, tuk-tuk → `motorcycle`, người thật → `person`; không gán bus/chân dung in; thuộc tính theo từng box. | Áp dụng nhất quán nhiều lớp. |

**Ảnh còn lại (`img_001`, `003`, `007`, `010`, `013`, `015`, `016`, `025`, `031`, `034`, `039`)**: áp dụng trực tiếp R-01 đến R-14 và danh sách quan sát trong bản cá nhân 102/104. Bảng trên là hướng dẫn các ca dễ tranh luận, **không phải kết quả gán nhãn hay số box đã QC**.

## 5. QC và quản lý phiên bản

### 5.1. Tự kiểm khi gán và trước xuất COCO

1. Đảm bảo mỗi ảnh được phân công được duyệt, cả ảnh không có box cũng xuất hiện trong COCO.
2. Mỗi vật thật đủ điều kiện một box, một lớp; người và xe không ghép chung.
3. Các box nằm trong biên ảnh; chiều cao box cá thể ≥20 px; sát phần thấy, không bao phần bị suy đoán, hàng rời và bóng.
4. Đặt đúng `occluded_level`, `truncated`, `can_review`; không dùng cờ `truncated` thay `occluded_level`.
5. Kiểm các ca âm `img_017`, `img_020`, `img_027`, `img_033`, `img_038` nếu nằm trong phân công.
6. Chỉ dùng `ignore_region` theo R-10; lọc lại tất cả box `can_review=true` và danh sách vấn đề cấp ảnh.
7. Đối chiếu đủ lượt gán độc lập theo `assignment.csv` trước khi tính IAA. Không xem file người khác trước khi tự khóa và nộp lượt gán riêng.
8. Sau khi xuất chạy `python scripts/validate_labels.py submission/labels/<ma>.json` (thay `<ma>` bằng mã thật, ví dụ `102`).

### 5.2. Ngưỡng QC theo `taxonomy.json` gốc (bắt buộc giữ nguyên)

| Kiểm tra | Ngưỡng |
|---|---|
| Chiều cao box tối thiểu | `min_box_height_px = 20` |
| Phân biệt box lớn | `large_px = 50` |
| IoU tối thiểu box lớn | `iou_thr_large = 0.85` |
| IoU tối thiểu box nhỏ | `iou_thr_small = 0.70` |
| IoU tối thiểu để ghép cặp | `match_iou = 0.50` |
| IAA IoU mục tiêu | `iaa_iou = 0.85` |
| IAA đồng thuận class | `iaa_class = 0.95` |
| Tỷ lệ ảnh lỗi tối đa khi release | `error_rate_img_max = 0.02` |

IAA chỉ phản ánh đồng thuận trên các cặp box ghép được; phải kiểm riêng vật bị bỏ sót, object ngoài phạm vi và ảnh rỗng. Không tự tạo số liệu QC hoặc đánh dấu Release khi chưa chạy kiểm và duyệt.

### 5.3. Phiên bản

| Phiên bản | Nội dung | Trạng thái |
|---|---|---|
| `v0` | Taxonomy, starter và ngưỡng QC do lab cung cấp. | Nguồn |
| `v0.1` / `v0.1-candidate` | Guideline cá nhân của 101, 102 và 104. | Đã đọc, còn bất đồng |
| `v1.0` | Quy tắc hợp nhất R-01…R-14, giữ nguyên 6 lớp, đưa ca khó về cùng cách xử lý. | **Chờ nhóm phê duyệt** |
| `v1.1` | Chỉ lập khi Pha 5 calibration có chỉnh sửa sau v1.0; ghi semantic/editorial đúng yêu cầu GUIDE. | Chưa thực hiện |

## 6. Biên bản xác nhận để thực sự khóa v1.0

**Những điểm có bất đồng cần nhóm chốt trực tiếp (không ghi đã họp khi chưa họp):**

| Mã | Điểm khác nhau giữa các bản cá nhân | Phương án tổng hợp đang ghi | Xác nhận nhóm |
|---|---|---|---|
| D01 | `img_002`: 102 gán các xe thật trên xe chở riêng; 104 không gán và gộp vào xe chở. | Mỗi xe con thật có `car` riêng; một `truck` cho tổ hợp kéo-rơ-moóc, không nới theo hàng. | Đã xác nhận |
| D02 | `img_009`: 102 loại minibus/bus; 104 đề xuất minibus → van, bus lớn → truck hoặc thêm lớp. | Giữ taxonomy 6 lớp: chỉ van đúng cấu tạo mới là `van`; bus/minibus chuyên dụng không gán. | Đã xác nhận |
| D03 | 102 chia mức che 0 / 30 / 70%; 104 chia 10 / 40 / 75%. | Mốc từ 102: 0%, 30%, 70%. | Đã xác nhận |
| D04 | 102 gán người thật nhìn trực tiếp xuyên kính khi đủ rõ; 104 loại người trong xe kín qua kính. | Gán khi nhìn trực tiếp, đủ rõ, ≥20 px; nếu thiếu bằng chứng ghi issue. | Đã xác nhận |
| D05 | 104 đề xuất `ignore_region` cho từng vật nhỏ; 102 chỉ dùng cho cụm không tách được. | Vật nhỏ đơn lẻ bỏ box; ignore cho cụm ≥3 vật mục tiêu không thể tách ổn định. | Đã xác nhận |
| D06 | 104 ghi hàng/xe được chở nằm trong box xe chở; 102 tách xe thật và loại hàng rời khỏi box. | Tách xe thật và không nới box xe chở theo hàng; box có thể chồng nhau. | Đã xác nhận |

**Phiếu xác nhận nhóm:**

- [ ] 101 đã đọc các mục D01–D06 và đồng ý bản chốt (hoặc ghi sửa đổi).
- [ ] 102 đã đọc các mục D01–D06 và đồng ý bản chốt (hoặc ghi sửa đổi).
- [ ] 104 đã đọc các mục D01–D06 và đồng ý bản chốt (hoặc ghi sửa đổi).
- [ ] 103 đã đọc các mục D01–D06 và đồng ý bản chốt (hoặc ghi sửa đổi).
- [ ] Đã thống nhất và sửa mọi hàng D01–D06 còn thay đổi.
- [ ] `submission/taxonomy.json` đã đặt `version = v1.0`, đồng bộ định nghĩa lớp/thuộc tính **và giữ nguyên hoàn toàn object `rules`**.
- [ ] Đã chạy thành công `python scripts/make_cvat_labels.py` và kiểm `cvat/labels_generated.json`.
- [ ] Nhóm công bố khóa guideline + taxonomy v1.0 trước khi phân ảnh và gán nhãn độc lập.

**Chỉ khi các bước trên thực sự xong, cập nhật phần đầu tài liệu thành `Trạng thái: ĐÃ THỐNG NHẤT — KHÓA v1.0`, 16/11/2026.**
