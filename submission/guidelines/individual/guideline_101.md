# Guideline gán nhãn — v0.1 — 101

> Copy file này (hoặc `guideline_v0_starter.md`) rồi viết. Một guideline đủ gồm 5 phần. Câu nào của v0
> làm hai người đọc ra hai cách khác nhau thì phải viết lại cho một cách hiểu duy nhất.

- Phiên bản: v0.1 (bản cá nhân)
- Ngày: 16/11/2026
- Người viết / duyệt: 101 (bản cá nhân, chưa duyệt)

## 1. Mục tiêu

Xây dựng bộ dữ liệu gán nhãn để huấn luyện mô hình AI phát hiện vật thể (Object Detection) trong ảnh giao thông bằng Bounding Box.
Mô hình nhận diện và xác định vị trí các đối tượng như người, ô tô, xe tải, xe van và xe máy, phục vụ hệ thống giám sát giao thông thông minh.
Các lỗi như bỏ sót vật thể, phân loại sai hoặc vẽ box không chính xác có thể làm giảm hiệu quả nhận diện và ảnh hưởng đến an toàn giao thông.
Vì vậy, việc gán nhãn phải chính xác, đầy đủ, nhất quán và tuân thủ cùng một bộ quy tắc.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa | Ví dụ | Dễ nhầm với |
|---|---|---|---|
| car | Ô tô con, SUV | Sedan, SUV | van |
| truck | Xe tải, có thùng hàng riêng | Xe tải chở hàng | van |
| van | Xe thùng kín cỡ nhỏ | Xe van giao hàng | truck, car |
| motorcycle | Xe máy, mô tô | Xe tay ga, mô tô | bicycle |
| person | Người đi bộ hoặc ngồi trên xe | Người đi bộ, người lái xe | Hình người trên quảng cáo |
| ignore_region | Vùng bỏ qua, không gán riêng từng vật | Vùng nhiều vật thể quá nhỏ | Vật thể cần gán riêng |

Vật không thuộc các lớp trên: cần xác định rõ trường hợp không gán và trường hợp sử dụng `ignore_region` theo quy tắc đã thống nhất.

### Thuộc tính

- occluded_level: `0` không bị che; `1` che ít; `2` che nhiều; `3` không rõ mức che. Cần bổ sung ngưỡng phần trăm cụ thể cho từng mức.
- truncated: `yes` nếu vật bị cắt bởi mép ảnh; `no` nếu không bị cắt.
- can_review: Tick khi chưa chắc chắn về lớp, vị trí box hoặc thuộc tính để được kiểm tra lại.

## 3. Quy tắc vẽ box

- R01: Gán Bounding Box cho người và phương tiện thuộc các lớp đã quy định.
- R02: Box phải ôm sát phần vật thể nhìn thấy, hạn chế lấy thừa nền.
- R03: Chọn đúng lớp và các thuộc tính `occluded_level`, `truncated`, `can_review`.
- R04 – Vật bị che: Xác định phần nhìn thấy và mức độ che khuất.
- R05 – Vật cắt mép ảnh: Xác định giới hạn box và thuộc tính `truncated`.
- R06 – Phản chiếu, hình in, tượng, đồ chơi: Cần thống nhất trường hợp nào được gán và trường hợp nào bỏ qua.
- R07 – Xe chủ (ego): Quy định việc có gán phần xe chứa camera xuất hiện trong ảnh hay không.
- R08 – Vật quá nhỏ, đám đông: Xác định khi nào dùng `ignore_region`.
- R09 – Người và xe: Quy định cách xử lý khi người ngồi hoặc điều khiển phương tiện.

Các trường hợp chưa chắc chắn cần đánh dấu `can_review` và ghi lại để thảo luận.


## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| img_004 | Xe tuk-tuk ba bánh | R03 | Cần tự chốt | Chưa có lớp riêng |
| img_017 | Tượng người cưỡi ngựa | R06 | Cần tự chốt | Phân biệt người thật và tượng |
| img_020 | Ô tô trong gương chiếu hậu | R06 | Cần tự chốt | Trường hợp phản chiếu |
| img_021 | Xe máy và người chen chúc | R04, R08 | Cần tự chốt | Nhiều vật bị che |
| img_023 | Xe xích lô | R03 | Cần tự chốt | Chưa có lớp riêng |
| img_027 | Vật trưng bày hình xe bán tải | R06 | Cần tự chốt | Phân biệt vật thật và vật mô phỏng |
| img_033 | Ma-nơ-canh và hình quảng cáo | R06 | Cần tự chốt | Phân biệt người thật và vật thể giả |
| img_035 | Xe ba bánh chở hàng | R03 | Cần tự chốt | Khó phân loại phương tiện |
| img_036 | Xe tải nhỏ thùng kín | R03 | Cần tự chốt | Dễ nhầm truck và van |
| img_038 | Ô tô đồ chơi trong tủ kính | R06 | Cần tự chốt | Phân biệt đồ chơi và xe thật |

## 5. QC và phiên bản

### Quy trình QC

- Kiểm tra vật thể bị bỏ sót hoặc gán thừa.
- Kiểm tra đúng lớp và các thuộc tính.
- Kiểm tra Bounding Box có ôm sát vật thể.
- Đánh dấu `can_review` đối với trường hợp chưa chắc chắn.
- Đối chiếu kết quả giữa các thành viên bằng IAA và IoU theo ngưỡng của đề bài.
- Các trường hợp bất đồng được nhóm thảo luận và ghi nhận trong quá trình Calibration.

### Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v0.1 | Xây dựng Guideline cá nhân 101 | Hoàn thiện quy tắc từ bản v0 |
| v1.0 | Chưa thực hiện | Chờ nhóm thống nhất |
| v1.1 | Chưa thực hiện | Chờ kết quả Calibration |