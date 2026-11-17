# Guideline v1.1 

**Trạng thái:** Được dùng để tạo nhãn hiệu chỉnh có hỗ trợ AI. Thành viên 102 xác nhận đã rà và đồng ý 416 ca calibration ngày 17/11/2026; chưa duyệt QC độc lập 40 ảnh và các box vẫn mang `can_review`. Bản bàn giao giữ quyết định Hold. Giữ guideline v1.0 và nhãn độc lập làm lịch sử.

## 1. Mục tiêu mô hình 2D

Gán box cho người thật và phương tiện thật nhìn trực tiếp trong 40 ảnh. Mỗi cá thể đủ điều kiện có một lớp trong `submission/taxonomy_v1.1.json`. Box ôm phần nhìn thấy, không suy đoán phần bị che hoặc ngoài ảnh. Vùng `ignore_region` chỉ đánh dấu cụm không thể tách cá thể ổn định.

## 2. Lớp và thuộc tính

| Lớp | Gán | Phản ví dụ |
|---|---|---|
| `car` | Sedan, hatchback, taxi, SUV/MPV cỡ ô tô con, kể cả xe thật đang được chở. | Pickup → `truck`; đồ chơi/tượng xe → loại. |
| `truck` | Xe tải, pickup, đầu kéo và rơ-moóc nối liền; box chỉ bao kết cấu xe. | Hàng rời và các ô tô được chở không thuộc box `truck`. |
| `van` | Van nhỏ thân kín liền khối. | Bus/minibus thiết kế chuyên dụng → `bus`; pickup → `truck`. |
| `bus` | Bus lớn hoặc minibus có kết cấu xe buýt chuyên dụng. | Van nhỏ thân kín → `van`; xe tải → `truck`. |
| `tuk_tuk` | Xe ba bánh có động cơ kiểu tuk-tuk. | Mô tô hai bánh hoặc sidecar → `motorcycle`; xích lô đạp chân → loại. |
| `motorcycle` | Mô tô, scooter, xe số, xe máy gắn sidecar cố định. | Tuk-tuk → `tuk_tuk`; xe đạp → loại. |
| `person` | Người thật trực tiếp, gồm người lái/ngồi xe và người nhìn rõ qua kính trong suốt. | Tượng, ma-nơ-canh, hình in, phản chiếu → loại. |
| `ignore_region` | Cụm ít nhất 3 mục tiêu quá nhỏ hoặc quá chồng lấn, không thể tách ổn định. | Vật nhỏ đơn lẻ, vật ngoài taxonomy hoặc vật giả → không tạo ignore. |

`occluded_level`: `0` = không bị che; `1` = >0–30%; `2` = >30–70%; `3` = >70% nhưng vẫn nhận dạng chắc. `truncated=yes` khi bị mép ảnh cắt, độc lập với mức che. `can_review=true` khi đã có box nhưng chưa chắc lớp, biên hoặc thuộc tính; ca chưa có box ghi vào `image_issues.csv`.

## 3. Quy tắc vẽ box

- **R-01 — Điều kiện gán:** Vật thật, nhìn trực tiếp, thuộc một trong bảy lớp đối tượng, còn nhận dạng chắc và phần nhìn thấy cao ít nhất 20 px.
- **R-02 — Biên box:** Ôm sát pixel ngoài cùng đang nhìn thấy của một cá thể; không bao bóng đổ, hàng rời, phản chiếu hoặc phần bị suy đoán.
- **R-03 — Che khuất:** Vẫn gán khi đủ R-01; chọn `occluded_level` theo ngưỡng ở mục 2.
- **R-04 — Cắt mép:** Dừng box ở ranh ảnh và đặt `truncated=yes`; không nới box ngoài ảnh.
- **R-05 — Người và xe:** Người lái/ngồi và xe có box riêng; hai người trên một xe máy có 2 `person` + 1 `motorcycle` nếu cả ba đủ điều kiện.
- **R-06 — Xe chở xe:** Một `truck` cho tổ hợp đầu kéo/rơ-moóc; mỗi ô tô thật được chở có `car` riêng nếu đủ R-01. Không kéo box xe chở theo hàng hoặc xe trên sàn.
- **R-07 — Phân lớp phương tiện:** Pickup kể cả có nắp thùng → `truck`; van thân kín nhỏ → `van`; bus/minibus chuyên dụng → `bus`; tuk-tuk → `tuk_tuk`; mô tô gắn sidecar → `motorcycle`. Dựa trên cấu tạo nhìn thấy, không đoán theo bối cảnh.
- **R-08 — Vật giả:** Không gán tượng, ma-nơ-canh, đồ chơi, mô hình, hình in/quảng cáo hoặc đạo cụ không chứng minh được là người thật.
- **R-09 — Phản chiếu và xe chủ:** Không gán vật chỉ xuất hiện qua gương/kính phản chiếu hoặc bộ phận xe đang mang camera. Vật khác nhìn trực tiếp xuyên kính vẫn có thể gán.
- **R-10 — Cụm khó:** Gán cá thể tách được trước; tạo `ignore_region` sát cụm ít nhất 3 mục tiêu không thể tách ổn định, không phủ vật rõ ở rìa.
- **R-11 — Nhỏ, tối, nhòe:** Bỏ box vật cao <20 px đơn lẻ. Tối/nhòe không tự động là che khuất; gán nếu vẫn đủ R-01 và đánh cờ review khi biên thiếu chắc.
- **R-12 — Trạng thái:** Xe thật vẫn gán khi đỗ, trưng bày hoặc đang được vận chuyển. Loại mô hình và tượng.
- **R-13 — Ngoài taxonomy:** Không ép xe đạp, xích lô đạp chân, động vật hoặc vật khác vào lớp gần nhất; người thật trên các xe đó vẫn có thể được gán.
- **R-14 — Ca không chắc:** Box hợp lệ nhưng phân vân → `can_review`; không đủ bằng chứng để vẽ → ghi vấn đề cấp ảnh, không dựng box giả.

## 4. Ca khó phải kiểm lại trên cả lô

| Ảnh | Quyết định v1.1 cần áp dụng khi sửa trong CVAT | Quy tắc |
|---|---|---|
| `img_002` | Xe chở là `truck`; ô tô thật trên sàn là các `car` riêng. | R-02, R-06 |
| `img_004`, `img_035` | Đổi tuk-tuk đủ điều kiện từ `motorcycle`/`car` sang `tuk_tuk`; không ôm hàng nhô. | R-02, R-07 |
| `img_009`, `img_031`, `img_037`, `img_040` | Rà từng bus/minibus; thêm `bus` khi đủ R-01, giữ `van` cho van đúng cấu tạo. | R-01, R-07 |
| `img_017`, `img_020`, `img_027`, `img_033`, `img_038` | Kiểm ca âm: tượng, phản chiếu, ma-nơ-canh, đồ chơi không được thành box đối tượng. | R-08, R-09 |
| Cả 40 ảnh | Rà mọi box/ca bỏ sót, thuộc tính, vùng ignore và vật ngoài phạm vi sau đổi taxonomy. | R-01–R-14 |

## 5. Calibration, QC và lịch sử phiên bản

| Phiên bản | Nội dung | Trạng thái |
|---|---|---|
| v1.0 | Sáu lớp, dùng cho nhãn độc lập đã nộp. | Bản lịch sử, trong file nhóm còn ghi chờ khóa. |
| v1.1 | Tám lớp theo HTML; đổi nghĩa `motorcycle`, thêm `bus` và `tuk_tuk`. | 102 xác nhận 416 ca từ đề xuất AI; chưa QC độc lập toàn lô. |

Đổi nghĩa taxonomy là `semantic`, nên ít nhất một dòng trong `calibration_decisions.csv` ghi `pham_vi=lo` và bản COCO sửa bao gồm 40/40 ảnh. 
