# Guideline gán nhãn — v0.1-candidate — 102

- Phiên bản: `v0.1-candidate` (bản cá nhân, trước khi họp nhóm)
- Ngày: 2026-11-16
- Người viết / duyệt: `102`
- Loại nhãn: bounding box trên ảnh 2D, thao tác bằng CVAT, xuất COCO 1.0
- Phạm vi khảo sát: toàn bộ 40 ảnh `img_001.jpg` đến `img_040.jpg`

## 1. Mục tiêu

Nhãn dùng để huấn luyện và đánh giá mô hình phát hiện người cùng các phương tiện thuộc taxonomy trên ảnh đường phố 2D. Đầu ra có thể hỗ trợ camera giao thông hoặc hệ thống quan sát cho xe; vì vậy bỏ sót người hay phương tiện thật có thể làm hệ thống đánh giá thiếu nguy cơ va chạm, còn gán nhầm tượng, hình in, phản chiếu, ma-nơ-canh hoặc đồ chơi sẽ tạo báo động giả.

Guideline ưu tiên tính lặp lại: cùng một vật phải dẫn đến cùng quyết định về việc có gán hay không, lớp, phạm vi box và thuộc tính. Đây là bài toán nhận biết vật xuất hiện trực tiếp trong ảnh, không suy đoán vật nằm hoàn toàn sau vật che và không thêm lớp ngoài taxonomy.

## 2. Lớp và thuộc tính

### 2.1. Lớp

Mỗi vật thật đủ điều kiện chỉ nhận đúng một lớp. Khi phân loại phương tiện, dùng cấu tạo và công năng nhìn thấy được, không dựa riêng vào kích thước phối cảnh.

| Lớp | Định nghĩa một câu | Ví dụ trong bộ ảnh | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô chở người cỡ nhỏ như sedan, hatchback, wagon, taxi hoặc SUV, có thân hành khách là phần chính. | Taxi/SUV ở `img_040`; ô tô trên xe chở xe ở `img_002`. | Pickup có thùng hàng là `truck`; minibus/bus không có trong taxonomy; xe mô hình ở `img_038` không gán. |
| `truck` | Xe cơ giới có cabin và thùng/sàn chở hàng riêng, gồm pickup, xe tải hộp và đầu kéo gắn rơ-moóc đang tạo thành một tổ hợp. | Pickup ở `img_001`, xe chở ô tô ở `img_002`, xe tải nhỏ ở `img_036`. | SUV là `car`; xe thân kín một khối cỡ nhỏ là `van`; tượng pickup ở `img_027` không gán. |
| `van` | Xe cơ giới cỡ nhỏ có khoang kín kéo dài liền thân để chở người hoặc hàng. | Xe van phía sau người đi xe máy ở `img_005`; van trắng ở `img_016`. | Pickup dù có nắp thùng vẫn là `truck`; bus/minibus lớn không thuộc taxonomy; sedan/wagon là `car`. |
| `motorcycle` | Phương tiện cơ giới hai hoặc ba bánh điều khiển kiểu xe máy, gồm scooter, mô tô, tuk-tuk/auto-rickshaw và xe máy gắn sidecar. | Tuk-tuk ở `img_004`, scooter ở `img_010`, xe sidecar ở `img_022`. | Xe đạp và cyclo chạy bằng sức người không gán; người lái được gán riêng là `person`. |
| `person` | Một người thật đang hiện diện trực tiếp, gồm người đi bộ, ngồi/đứng trên phương tiện và người thật mặc trang phục nhân vật. | Người và người lái ở `img_005`, hai người trên xe ở `img_026`, người mặc đồ Predator ở `img_011`. | Tượng ở `img_017`, người trên billboard ở `img_018`, ma-nơ-canh ở `img_033`, búp bê/mascot trang trí không gán. |
| `ignore_region` | Một vùng 2D mà các vật mục tiêu quá nhỏ hoặc chồng lấn dày đặc đến mức không thể gán từng vật một cách ổn định; đây không phải một vật thể. | Cụm xe/người xa và không tách được ở `img_021` hoặc cụm xe dày ở `img_030`. | Không dùng để bao một bus, xe đạp, tượng, phản chiếu hay vật ngoài taxonomy; các vật đó chỉ đơn giản không gán. |

Quy tắc phân giải lớp:

1. Pickup luôn là `truck`, kể cả thùng trống (`img_001`, `img_012`), chở vật (`img_028`) hoặc có nắp/cap kín (`img_037`).
2. Tuk-tuk/auto-rickshaw dùng kết cấu ba bánh kiểu xe máy nên gán `motorcycle` (`img_004`, `img_016`, `img_035`, `img_040`).
3. Bus/minibus, xe đạp, cyclo, ngựa, đồ vật và các loại phương tiện không có trong sáu lớp trên không được ép vào lớp gần giống.
4. Xe thật vẫn được gán khi đang đỗ, ở trong kho/bảo tàng hoặc được chở trên xe khác; đồ chơi, mô hình, tượng và hình mô phỏng thì không.

### 2.2. Vật ngoài phạm vi

Không gán và không tạo lớp mới cho:

- bus/minibus, xe đạp, cyclo, động vật và vật chở hàng;
- hình in, ảnh quảng cáo, người/xe trên màn hình hoặc tranh;
- tượng, ma-nơ-canh, búp bê, mô hình, đồ chơi và phương tiện làm tượng đài;
- vật chỉ xuất hiện qua gương hay bề mặt phản chiếu;
- phần xe chủ (ego vehicle) gắn với camera như gương, cửa, nắp ca-pô của xe đang chụp;
- một vật riêng lẻ có chiều cao box phần nhìn thấy dưới 20 px.

`ignore_region` chỉ dùng theo R-10 cho cụm vật mục tiêu không thể gán ổn định, không dùng như lớp “khác”.

### 2.3. Thuộc tính

#### `occluded_level`

Ước lượng tỷ lệ diện tích của vật bị một vật thể tiền cảnh che mất so với hình dạng đầy đủ của chính vật đó. Không tính phần ra ngoài khung ảnh, bóng tối, nhòe chuyển động hoặc độ trong suốt của kính là che khuất.

| Giá trị | Ngưỡng phần bị che | Cách hiểu |
|---|---:|---|
| `0` | 0% | Không có vật khác che; có thể tối hoặc nhòe nhưng đường bao vẫn hiện diện. |
| `1` | Trên 0% đến 30% | Che ít, đa số vật vẫn thấy. |
| `2` | Trên 30% đến 70% | Che nhiều nhưng vẫn đủ đặc điểm để nhận dạng và tách cá thể. |
| `3` | Trên 70% đến dưới 100% | Chỉ còn một phần nhỏ nhưng vẫn nhận ra chắc chắn vật và lớp. |

Nếu không còn đủ bằng chứng để xác định cá thể hoặc lớp thì không dựng box bằng suy đoán; xử lý theo R-10 hoặc R-14.

#### `truncated`

- `yes`: hình dạng tự nhiên của vật bị đường biên ảnh cắt; box chạm đúng biên tương ứng.
- `no`: toàn bộ phần ngoài cùng của vật vẫn nằm trong ảnh, kể cả khi vật bị vật khác che.

`truncated` và `occluded_level` độc lập. Một vật vừa bị cắt mép vừa bị che phải ghi cả hai thuộc tính đúng thực tế.

#### `can_review`

- Không tick khi quyết định đã theo được rule rõ ràng.
- Tick khi đã có box nhưng còn bất định hợp lý về lớp, ranh giới box, việc hai phần có thuộc cùng một vật hay mức che nằm sát ngưỡng.
- Nếu chưa có box để tick, ghi vấn đề cấp ảnh với tên ảnh, vùng và lý do; không vẽ box giả chỉ để mang cờ.

## 3. Quy tắc vẽ box

### R-01 — Điều kiện gán

Chỉ gán một vật thật, xuất hiện trực tiếp, thuộc một lớp mục tiêu, nhận dạng được và có chiều cao box phần nhìn thấy ít nhất 20 px. Ngưỡng 20 px được đo sau khi đặt box sát phần nhìn thấy, không ước lượng kích thước đầy đủ sau che khuất.

### R-02 — Box theo phần nhìn thấy

Box chữ nhật đi qua các pixel ngoài cùng đang nhìn thấy của vật, sát nhất có thể và không suy đoán phần nằm ngoài ảnh hoặc sau vật che. Nếu một vật bị che làm phần nhìn thấy tách thành nhiều mảnh, vẫn vẽ một box duy nhất bao các mảnh nhìn thấy ngoài cùng; không tách một vật thành nhiều box.

Đối với `person`, box gồm cơ thể và quần áo/mũ đang mặc; không nới box theo túi xách, hàng đang cầm, xe hoặc thanh dài đang vác. Đối với phương tiện, box gồm thân và bộ phận kết cấu gắn liền như bánh, mui, gương nhìn thấy; không nới theo hàng hóa rời.

### R-03 — Che khuất

Vẽ box sát phần nhìn thấy như R-02 rồi đặt `occluded_level` theo bảng ở mục 2.3. Khung xe, người khác, cây, hàng hóa hoặc phương tiện phía trước có thể là vật che. Kính trong không tự động được tính là che; cột cửa, khung kính hoặc hàng rào đục có che pixel thì được tính.

### R-04 — Cắt mép ảnh

Vật đủ nhận dạng vẫn được gán khi bị cắt bởi mép ảnh. Box dừng đúng ở mép ảnh và đặt `truncated=yes`. Không suy rộng box ra ngoài khung. Một mảnh quá nhỏ, không đủ xác định lớp hoặc cao dưới 20 px thì bỏ theo R-01.

### R-05 — Người và phương tiện là các cá thể riêng

Mỗi người thật và phương tiện thật có box riêng dù các box chồng nhau. Người lái, người ngồi sau hoặc người trong xe được gán `person` nếu phần nhìn thấy đủ 20 px và nhận dạng được; không vẽ một box ghép “người + xe”. Xe máy chở hai người ở `img_026` vì vậy có ba box mục tiêu: hai `person` và một `motorcycle`.

### R-06 — Xe chở xe, tổ hợp và hàng hóa

- Xe tải/đầu kéo cùng rơ-moóc đang nối và vận hành như một tổ hợp nhận một box `truck` bao phần kết cấu nhìn thấy của cả tổ hợp.
- Mỗi ô tô thật trên xe chở xe ở `img_002` nhận một box `car` riêng nếu đạt ngưỡng, dù đang là hàng hóa.
- Vật chở rời như hộp, rau, thanh kim loại, bao tải, tượng ngựa hoặc khối hàng không nằm trong box của xe nếu nó vươn ra ngoài thân xe.
- Xe máy gắn sidecar cố định (`img_022`) là một `motorcycle` với một box bao cả cụm; tuk-tuk cũng là một phương tiện duy nhất.

### R-07 — Ranh giới giữa `car`, `truck`, `van` và `motorcycle`

Ưu tiên dấu hiệu cấu tạo: thùng/sàn hàng tách cabin → `truck`; thân kín kéo dài một khối cỡ nhỏ → `van`; thân chủ yếu chở người dạng sedan/hatchback/wagon/SUV/taxi → `car`; xe cơ giới hai/ba bánh điều khiển kiểu mô tô → `motorcycle`. Khi dấu hiệu bị che đến mức không phân giải được hai lớp, đặt lớp hợp lý nhất và tick `can_review`, không đổi sang lớp ngoài taxonomy.

### R-08 — Vật giả, hình ảnh và trang trí

Không gán tượng (`img_017`), tượng đài hình pickup (`img_027`), ma-nơ-canh (`img_033`), xe mô hình/đồ chơi (`img_038`), người trên billboard (`img_018`) hoặc mô tô điêu khắc ở `img_011`. Người thật mặc hóa trang vẫn là `person` khi có bằng chứng là một người biểu diễn; mascot/đạo cụ chỉ có hình nhân vật nhưng không thấy bằng chứng người thật thì không gán.

### R-09 — Phản chiếu, vật qua kính và xe chủ

Vật chỉ xuất hiện trong gương hoặc phản chiếu trên kính/kim loại không gán; `img_020` vì vậy không có box mục tiêu. Vật thật nhìn trực tiếp xuyên qua kính vẫn được gán nếu đủ rõ. Không gán các phần của chính xe mang camera, nhưng một xe khác bị cắt mép gần camera vẫn được gán theo R-04.

### R-10 — Đám đông và `ignore_region`

1. Nếu từng người/xe có thể tách cá thể và cao ít nhất 20 px, gán từng vật dù số lượng nhiều.
2. Nếu có một cụm từ ba vật mục tiêu trở lên chồng lấn hoặc quá nhỏ, không thể đặt box cá thể ổn định, vẽ một `ignore_region` sát vùng cụm đó.
3. Không đồng thời gán cá thể nằm hoàn toàn trong `ignore_region`. Nếu vài cá thể ở rìa tách rõ, thu nhỏ vùng ignore để gán chúng riêng.
4. Không dùng một vùng ignore phủ cả ảnh chỉ vì ảnh đông; không dùng ignore cho vật ngoài taxonomy.

### R-11 — Vật nhỏ, xa, tối hoặc nhòe

Chiều cao box dưới 20 px thì không gán riêng. Vật tối hoặc nhòe chuyển động vẫn được gán nếu nhận dạng được lớp, tách được cá thể và đạt 20 px; nhòe không phải che khuất. Nếu đường bao nhòe, box ôm vùng pixel thuộc vật mà mắt người còn xác định được, không ôm cả vệt mờ dài không chắc chắn.

### R-12 — Trạng thái hoạt động và bối cảnh

Phương tiện thật được gán dù đang chạy, đỗ, ở kho, trong bảo tàng hoặc trên xe chở hàng. Ví dụ xe tải trong kho ở `img_007` và ô tô thật trưng bày ở `img_024` vẫn thuộc phạm vi. Trạng thái “không chạy” không biến vật thật thành đồ chơi; dùng kích thước, vật liệu, kết cấu và bối cảnh để phân biệt.

### R-13 — Lớp vắng trong taxonomy

Không gán bus/minibus, xe đạp hoặc cyclo và không thay bằng lớp gần nhất. Người trên các phương tiện ấy vẫn nhận box `person` riêng nếu đạt R-01. Ví dụ cyclo ở `img_023` không có box phương tiện nhưng người đạp và hành khách nhìn rõ vẫn có thể là `person`.

### R-14 — Cách xử lý bất định

Nếu vật chắc chắn thuộc phạm vi nhưng lớp/box/thuộc tính còn lưỡng lự, vẽ quyết định tốt nhất và tick `can_review`. Nếu chưa chắc vùng đó có phải vật mục tiêu thật hay không và không có box hợp lệ, không vẽ; ghi vấn đề cấp ảnh. Nhật ký phải nêu ảnh, vùng, hai khả năng và rule gây vướng để reviewer có thể chốt.

## 4. Ca khó

### 4.1. Các quyết định chính

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_002` | Một xe chở nhiều ô tô. | R-02, R-03, R-06 | Một box `truck` cho tổ hợp xe chở; mỗi ô tô thật nhìn thấy và cao ≥20 px có một box `car`; người bên phải có box `person`. | Xe chở là một tổ hợp thật, còn các ô tô là những cá thể thật riêng dù đang là hàng hóa; thanh sàn có thể gây che khuất. |
| `img_004` | Hai tuk-tuk ba bánh và người lái. | R-05, R-07 | Mỗi tuk-tuk là một `motorcycle`; mỗi người nhìn rõ là một `person` riêng. | Kết cấu và điều khiển thuộc họ xe máy ba bánh, không phải `car` hoặc `van`. |
| `img_006` | Nhiều xe máy, xe đạp và người chồng lấn. | R-01, R-10, R-13 | Gán người/xe máy tách được và đủ 20 px; xe đạp không gán; cụm xa không tách được dùng `ignore_region`. | Không ép xe đạp vào `motorcycle`, đồng thời tránh box suy đoán trong cụm dày. |
| `img_011` | Người hóa trang Predator bên các mô tô điêu khắc. | R-08, R-12 | Predator là `person` nếu xác định là người biểu diễn; các mô tô làm từ chi tiết kim loại mang tính điêu khắc không gán; người/xe thật nền sau xử lý bình thường nếu đủ ngưỡng. | Taxonomy nhắm vật thật; trang phục không làm mất tư cách người nhưng tác phẩm mô phỏng không phải phương tiện vận hành. |
| `img_014` | Người rõ nét, xe máy chạy ngang bị nhòe. | R-05, R-11 | Hai người đi bộ có box `person`; xe máy/người lái nhòe chỉ gán khi còn tách được cá thể, nhận ra lớp và cao ≥20 px. | Nhòe không đồng nghĩa với che nhưng không được dựng box khi không còn biên ổn định. |
| `img_015` | Nhiều ô tô bị cắt mép và xe đang chạy bị nhòe. | R-04, R-11 | Các ô tô nhận dạng được đều là `car`; xe bị mép ảnh cắt đặt `truncated=yes`; xe nhòe vẫn gán nếu đủ ngưỡng. | Vật thật không bị loại chỉ vì cắt mép hoặc chuyển động. |
| `img_017` | Tượng người cưỡi ngựa. | R-08 | Không có box `person`; không dùng `ignore_region`. | Đây là tượng, không phải người thật, và ngựa cũng ngoài taxonomy. |
| `img_018` | Chân dung trên billboard, ô tô thật sau hàng rào. | R-03, R-08 | Không gán người trên bảng; gán các xe thật đủ 20 px nhìn trực tiếp sau hàng rào, với mức che tương ứng. | Hình in bị loại nhưng vật thật trong cùng ảnh vẫn thuộc phạm vi. |
| `img_019` | Xe diễu hành hình hoạt họa, tài xế và nhân vật mascot. | R-08, R-12, R-14 | Xe có khả năng vận hành là `car`, tài xế thật là `person`; nhân vật mascot không gán khi không có bằng chứng trực quan đủ chắc đó là người thật. | Hình dáng tùy biến không loại xe thật; không suy đoán một đạo cụ là người. |
| `img_020` | Xe phía sau chỉ xuất hiện trong gương chiếu hậu, kèm phần xe chủ. | R-09 | Không gán box nào. | Phản chiếu và ego vehicle đều ngoài phạm vi. |
| `img_021` | Đám đông xe máy/người rất dày. | R-10 | Gán riêng cá thể tách được; dùng `ignore_region` cho phần trung tâm/xa nơi không thể phân ranh ổn định. | Một vùng ignore có kiểm soát giúp tránh vừa bỏ sót vừa tạo box suy đoán. |
| `img_022` | Xe máy gắn sidecar. | R-06, R-07 | Một box `motorcycle` bao phần nhìn thấy của xe máy và sidecar gắn cố định. | Đây là một tổ hợp phương tiện ba bánh, không phải hai xe riêng. |
| `img_024` | Ô tô thật trong bảo tàng và mô hình xe màu vàng đặt trên nóc. | R-08, R-12 | Gán từng ô tô thật cỡ đầy đủ; không gán xe mô hình màu vàng. | Bối cảnh trưng bày không loại xe thật, nhưng mô hình/đồ chơi bị loại theo bản chất. |
| `img_026` | Hai người ngồi trên một xe máy. | R-05 | Hai box `person` và một box `motorcycle`, các box được phép chồng nhau. | Người và xe là ba cá thể mục tiêu, không ghép chung. |
| `img_027` | Tượng đài có hình pickup kích thước lớn. | R-08 | Không gán `truck`. | Bệ trưng bày và bối cảnh nghĩa trang cho thấy đây là tượng/monument, không phải xe thật. |
| `img_029` | Xe máy chở thanh dài, cạnh người và xe đạp. | R-02, R-05, R-13 | Gán xe máy và từng người đủ ngưỡng; không gán xe đạp; thanh hàng không làm nới box người hay xe. | Hàng hóa rời và xe đạp ngoài taxonomy. |
| `img_030` | Bãi xe máy dày, nhiều xe che nhau. | R-03, R-10 | Gán riêng xe tách được; vùng các xe chồng lấn không thể phân cá thể dùng `ignore_region`; người thật đủ ngưỡng gán riêng. | Không nên đoán số xe hoặc box amodal trong cụm. |
| `img_033` | Ma-nơ-canh sau kính và ảnh xe/phố phản chiếu. | R-08, R-09 | Không có box `person`; các hình chỉ do phản chiếu không gán. Nếu reviewer xác nhận một xe được nhìn trực tiếp chứ không phải phản chiếu, ghi issue cấp ảnh trước khi thêm box. | Ảnh cố ý trộn vật giả và phản chiếu nên cần bằng chứng về đường truyền trực tiếp. |
| `img_035` | Tuk-tuk chở khối hàng quá cao. | R-02, R-05, R-06, R-07 | Tuk-tuk là `motorcycle`; box xe không nới theo bao hàng; người lái và các phương tiện/người khác gán riêng nếu đủ ngưỡng. | Hàng rời không phải bộ phận kết cấu của phương tiện. |
| `img_038` | Tủ kính đầy ô tô mô hình và búp bê. | R-08, R-09 | Không gán `car` hay `person`; phản chiếu trên kính cũng không gán. | Tất cả đối tượng chính là đồ chơi/mô hình, không phải vật mục tiêu thật. |
| `img_040` | Tuk-tuk, taxi, SUV, bus và người bị che/cắt. | R-03, R-04, R-07, R-13 | Taxi/SUV là `car`, tuk-tuk là `motorcycle`, người thật là `person`; bus không gán; từng box đặt mức che/cắt riêng. | Một ảnh chứa cả lớp hợp lệ và loại phương tiện không có trong taxonomy. |

### 4.2. Nhật ký rà soát đủ 40 ảnh

Bảng này ghi điểm cần kiểm khi gán, không thay cho việc đếm box trực tiếp trong CVAT.

| Ảnh | Quan sát và áp dụng rule |
|---|---|
| `img_001` | Người thật gán `person`; pickup gán `truck`; người bị bàn/xe che đặt `occluded_level` theo R-03. |
| `img_002` | Xe chở xe và các xe con tách riêng theo R-06; người cạnh rơ-moóc gán riêng. |
| `img_003` | Gán người và xe máy đủ ngưỡng; xe đạp không gán theo R-13; xử lý che khuất từng cá thể. |
| `img_004` | Hai tuk-tuk là `motorcycle`, người lái là `person` theo R-05/R-07. |
| `img_005` | Mỗi người và xe máy có box riêng; van nền là `van`; vật cắt mép theo R-04. |
| `img_006` | Người/xe máy tách được gán riêng, xe đạp loại; cụm không tách dùng R-10. |
| `img_007` | Xe tải hộp và pickup trong kho đều là `truck`; xe cắt mép trái xử lý R-04. |
| `img_008` | Xe máy đỗ vẫn gán; người và phương tiện nền chỉ gán khi đạt 20 px; vùng dày áp dụng R-10. |
| `img_009` | Minibus/bus không ép thành `van`; chỉ gán người/xe máy thật đủ ngưỡng theo R-01/R-13. |
| `img_010` | Gán người và xe máy; xe đạp chở hàng không gán; box không bao hàng rời. |
| `img_011` | Người hóa trang có thể là `person`; mô tô điêu khắc không gán theo R-08. |
| `img_012` | Pickup là `truck`; thùng hàng không phải box riêng và không làm nới box xe; xe nền xử lý theo lớp thật. |
| `img_013` | Người, xe máy và ô tô xa chỉ gán nếu đạt 20 px; tượng/trang trí ở vỉa hè không gán. |
| `img_014` | Hai người chính gán rõ; xe/người nhòe chỉ gán khi còn đủ điều kiện R-11. |
| `img_015` | Ô tô thật đều là `car`; hai xe tiền cảnh bị cắt đặt `truncated=yes`; xe nhòe áp dụng R-11. |
| `img_016` | Tuk-tuk/scooter là `motorcycle`, van là `van`, pickup là `truck`, người gán riêng. |
| `img_017` | Tượng người và ngựa: không có box mục tiêu theo R-08. |
| `img_018` | Người in trên billboard loại; ô tô thật sau hàng rào gán nếu đủ ngưỡng, tính che theo R-03. |
| `img_019` | Xe diễu hành vận hành được là `car`, tài xế là `person`; đạo cụ mascot xử lý R-08/R-14; đám đông theo R-10. |
| `img_020` | Phản chiếu trong gương và phần ego vehicle: không có box theo R-09. |
| `img_021` | Đám đông người/xe máy: tách được thì gán, phần không tách dùng `ignore_region`. |
| `img_022` | Xe máy-sidecar là một `motorcycle`; người nhìn thấy xử lý riêng nếu đạt R-01. |
| `img_023` | Cyclo/xe đạp không gán; car, motorcycle và person đủ ngưỡng vẫn gán riêng. |
| `img_024` | Ô tô thật cỡ đầy đủ trong bảo tàng gán `car`; mô hình vàng trên nóc loại. |
| `img_025` | Người, ô tô và xe máy thật gán khi cao ≥20 px; vật quá xa áp dụng R-01/R-10. |
| `img_026` | Hai `person` và một `motorcycle`; không ghép box, không coi người ngồi sau là phần xe. |
| `img_027` | Tượng đài hình pickup: không gán `truck`. |
| `img_028` | Pickup thật là `truck`, ô tô nền là `car`; tượng ngựa trên thùng không gán. |
| `img_029` | Người và xe máy gán riêng; xe đạp và thanh hàng loại khỏi box mục tiêu. |
| `img_030` | Bãi motorcycle dày: cá thể rõ gán riêng, cụm không phân ranh dùng `ignore_region`; người gán riêng. |
| `img_031` | Ô tô là `car`, xe máy/người xử lý bình thường; bus không gán theo R-13. |
| `img_032` | Người tách được ở trạm xe buýt gán riêng; cụm chồng lấn dùng R-10; car tiền cảnh cắt mép dùng R-04. |
| `img_033` | Ma-nơ-canh và phản chiếu không gán; vùng xe qua kính còn mơ hồ ghi issue theo R-14. |
| `img_034` | Người, car và motorcycle gán nếu đủ ngưỡng; xe đạp không gán; vật biên phải kiểm `truncated`. |
| `img_035` | Tuk-tuk là `motorcycle`; khối hàng không nới box xe; người, xe máy, car khác gán riêng. |
| `img_036` | Xe tải nhỏ chính là `truck`; motorcycle/person nền chỉ gán nếu cao ≥20 px. |
| `img_037` | Pickup có cap vẫn là `truck`; người gán `person`; bus nền không gán. |
| `img_038` | Xe mô hình, búp bê và phản chiếu tủ kính: không có box mục tiêu. |
| `img_039` | Người và motorcycle đủ ngưỡng gán riêng; xe đạp loại; phương tiện xa áp dụng ngưỡng 20 px. |
| `img_040` | Taxi/SUV là `car`, tuk-tuk là `motorcycle`, người gán riêng; bus loại; vật che/cắt đặt thuộc tính tương ứng. |

## 5. QC và phiên bản

### 5.1. Tự kiểm trước khi xuất

1. Xem lần lượt mọi ảnh, kể cả ảnh dự kiến không có box; ảnh âm phải vẫn xuất hiện trong COCO.
2. Kiểm mỗi vật chỉ có một box và đúng một lớp; người và xe không bị ghép.
3. Kiểm mọi box có chiều cao phần nhìn thấy ít nhất 20 px và nằm trong khung ảnh.
4. Kiểm box sát pixel nhìn thấy, không ôm hàng rời, bóng đổ hoặc phần bị suy đoán.
5. Kiểm `truncated=yes` cho mọi vật bị biên ảnh cắt và không dùng thuộc tính này thay cho che khuất.
6. Kiểm `occluded_level` theo đúng các ngưỡng 0%, 30% và 70%.
7. Tìm lại các ca âm `img_017`, `img_020`, `img_027`, `img_033`, `img_038` để bảo đảm không gán nhầm tượng, phản chiếu, ma-nơ-canh hoặc đồ chơi.
8. Kiểm `ignore_region` không được dùng để che lỗi phân lớp hoặc bao vật ngoài taxonomy.
9. Lọc toàn bộ box có `can_review=true`; mỗi box phải có lý do có thể trình bày cho reviewer. Với ca chưa có box, kiểm nhật ký vấn đề cấp ảnh.
10. Chạy `python scripts/validate_labels.py submission/labels/102.json` sau khi xuất COCO.

### 5.2. Ngưỡng QC dùng trong lab

- Ghép hai box khi IoU ≥ 0.50.
- Cặp có chiều cao box trung bình từ 50 px trở lên: mục tiêu IoU ≥ 0.85; nhỏ hơn 50 px: mục tiêu IoU ≥ 0.70.
- Mức đồng thuận chung: IoU ≥ 0.85 và đồng thuận lớp ≥ 0.95 trên các cặp ghép được.
- Tỷ lệ lỗi ảnh tối đa để Release: 0.02; ca bỏ sót và ảnh không có cặp box vẫn phải kiểm bằng mắt, không chỉ dựa vào IAA.
- Ca mơ hồ do người `102` nêu phải được nhóm/reviewer chốt bằng ảnh và Rule ID; giữ `can_review` hoặc issue cho đến khi có quyết định.

### 5.3. Lịch sử phiên bản

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| `v0` | Bản starter: mới có tên lớp và ba quy tắc tổng quát. | Chưa giải quyết xe chở xe, tuk-tuk, phản chiếu, tượng, ma-nơ-canh, vật nhỏ và đám đông. |
| `v0.1-candidate` | Bổ sung định nghĩa lớp, ngưỡng thuộc tính, R-01 đến R-14, ca khó và nhật ký rà soát đủ 40 ảnh. | Khảo sát `img_001`–`img_040`, đặc biệt các ca `img_002`, `img_004`, `img_017`, `img_020`, `img_027`, `img_033`, `img_038`, `img_040`. |

Mọi thay đổi sau họp làm đổi việc chọn vật, lớp, phạm vi box hoặc thuộc tính phải tăng phiên bản semantic của guideline nhóm. Sửa chính tả/diễn đạt không đổi nhãn được ghi là thay đổi editorial.
 
    