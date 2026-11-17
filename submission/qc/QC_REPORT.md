# Báo cáo QC — Đại đi

**Trạng thái bàn giao: RELEASE.**

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Thực hành xây dựng guideline, gán nhãn và kiểm soát chất lượng Bounding Box 2D trên bộ dữ liệu 40 ảnh ca biên. Bộ nhãn được bàn giao phục vụ thực hành và đánh giá theo phạm vi Lab 7, không mặc nhiên bảo đảm tính đại diện cho dữ liệu vận hành thực tế.
- Nguồn và phép sử dụng dữ liệu: Bộ dữ liệu gồm 40 ảnh. Thông tin nguồn, tác giả và giấy phép được lưu trong `data/ATTRIBUTION.csv`. Nhóm giữ nguyên yêu cầu ghi công và tuân thủ điều kiện sử dụng của từng ảnh.
- Manifest và checksum nhãn cuối: Bản nhãn cuối `submission/qc/labels_calibrated.json` và manifest `submission/qc/batch_manifest.json` được tạo lại sau khi hoàn thành Calibration và xử lý trạng thái `can_review`. Checksum SHA-256 chính thức được lưu trong manifest của phiên bản bàn giao vòng QC 6. Không sử dụng checksum của các phiên bản trước khi hiệu chỉnh.
- Guideline và taxonomy khi gán / khi bàn giao: Lượt gán độc lập sử dụng guideline v1.0 và taxonomy v1.0 gồm sáu lớp. Bộ nhãn bàn giao sử dụng guideline v1.1 và taxonomy v1.1 gồm tám lớp. Thay đổi thuộc loại `semantic`, được áp dụng trong phạm vi toàn bộ 40 ảnh.

## 2. Phân công và đồng thuận trước Calibration

- Lượt hoàn thành / được giao: 80/80 lượt gán, với bốn thành viên mang mã 101, 102, 103 và 104. Mỗi ảnh được phân công cho hai thành viên.
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: Tổng cộng 217 cặp Bounding Box được ghép. IoU trung bình đạt 0.835, thấp hơn ngưỡng 0.85. Đồng thuận lớp đạt 98.6%, cao hơn ngưỡng 95%. Các chỉ số được tính trên những cặp box ghép thành công trước Calibration.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: Ghi nhận 93/117 box chưa ghép theo hai phía, không có cặp bị loại do ignore và 79 trường hợp bất đồng vùng ignore.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: `img_019.jpg` có phương tiện diễu hành nhưng bản đồng thuận ban đầu chỉ ghi nhận `person`. Phương án Calibration bổ sung nhãn `car` để khắc phục khả năng bỏ sót phương tiện; quyết định cần được truy vết về ảnh, guideline và nhãn hiệu chỉnh.

Kết quả IAA được sử dụng để phát hiện bất đồng giữa các thành viên và xác định phạm vi Calibration. Việc IoU ban đầu chưa đạt không đồng nghĩa với việc bản nhãn cuối không thể được bàn giao sau khi đã hiệu chỉnh và kiểm tra chất lượng.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: Có 416 trường hợp bất thường được ghi nhận trong `submission/qc/calibration_decisions.csv`, kèm mã quy tắc, phương án xử lý và tham chiếu tới `ai_review_evidence.json`. Thành viên 102 tham gia rà soát phương án Calibration. Các quyết định và bộ nhãn hiệu chỉnh được đưa vào phạm vi QC cuối để kiểm tra.
- Tác động v1.1 lên nhãn và ảnh: Guideline v1.1 và taxonomy v1.1 bổ sung các lớp `bus`, `tuk_tuk`, đồng thời điều chỉnh phạm vi lớp `motorcycle`. Thay đổi có tác động đến toàn bộ 40 ảnh. Bộ nhãn cuối gồm 423 annotation. Sau quá trình xác nhận thuộc tính, số annotation còn `can_review=true` là 0.
- Tham chiếu bằng chứng khi Rework: Lịch sử các vòng QC trước được lưu tại `submission/qc/reviews/`. Vòng QC 6 được tạo cho phiên bản nhãn đã cập nhật sau khi xử lý trạng thái chờ review. Bản chụp nhãn, manifest, phiếu QC và kết quả kiểm tra tương ứng được lưu trong `submission/qc/reviews/round-06/`.

Nhóm đã tạo lại bộ nhãn hiệu chỉnh và manifest để duy trì tính nhất quán giữa dữ liệu, phiên bản Calibration và bằng chứng QC.

## 4. Kết quả duyệt

- Vòng QC: 6
- Người review và ngày: Thành viên mã 102; kết quả review được ghi trong `submission/qc/reviews/round-06/qc_review.csv`. Ngày review: 17/11/2026.
- Đã duyệt / cả lô: 40/40 ảnh, đạt 100% độ phủ kiểm tra theo phiếu QC vòng 6.
- Ảnh lỗi / ảnh đã duyệt: 0/40 = 0.0%, theo kết quả QC tự động vòng 6.
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: 0/423 = 0.0%, theo kết quả QC tự động vòng 6.
- Vật bỏ sót, ảnh và vùng: Không ghi nhận trường hợp bỏ sót trong phiếu QC vòng 6.
- Ca khó / phần còn lại, lỗi lặp và ví dụ: Đã duyệt 37/37 ảnh ca khó và 3/3 ảnh còn lại. Không ghi nhận ảnh lỗi trong hai nhóm ảnh. Các trường hợp bất đồng trước Calibration được lưu trong báo cáo IAA và hồ sơ quyết định.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Công cụ tự động kiểm tra cấu trúc COCO, kích thước Bounding Box, phạm vi ảnh và tính đầy đủ của dữ liệu. Các tình huống như `bus`/`van`, `tuk_tuk`/`motorcycle`, tượng/người thật, hình phản chiếu và độ ôm sát của Bounding Box cần được đánh giá bằng quan sát ảnh.

**Kết quả kiểm tra tự động vòng 6: RELEASE.**

Báo cáo QC tự động ghi nhận:

- 40/40 ảnh đã duyệt.
- 0/40 ảnh có lỗi được ghi nhận.
- 0/423 annotation có lỗi được ghi nhận.
- 0 vật thể bỏ sót được ghi nhận.
- 37/37 ảnh ca khó và 3/3 ảnh còn lại đã được duyệt.
- Không còn annotation đánh dấu `can_review=true` theo kết quả kiểm tra dữ liệu sau hiệu chỉnh.

Các tỷ lệ lỗi phản ánh nội dung được ghi trong phiếu QC, không phải cam kết độ chính xác tuyệt đối cho mọi đối tượng trong ảnh.

## 5. Quyết định bàn giao

- Quyết định: Release
- Lý do, người nhận và phạm vi sử dụng: Nhóm đã hoàn thành các lượt gán được phân công, phân tích IAA, ghi nhận quyết định Calibration và tạo bộ nhãn hiệu chỉnh. Phiếu QC vòng 6 ghi nhận đã duyệt 40/40 ảnh, không phát hiện lỗi theo các kết quả được khai báo và đã xử lý trạng thái chờ review. Công cụ `qc_report.py` trả về RELEASE. Bộ hồ sơ được bàn giao cho coach phục vụ đánh giá Lab 7 và sử dụng trong phạm vi thực hành gán nhãn 2D đã mô tả.
- Nếu còn việc: Hoàn tất kiểm tra tính đầy đủ của gói nộp, lưu giữ các phiên bản và bằng chứng QC, bảo đảm coach có thể truy vết từ báo cáo tới ảnh, rule và bản nhãn. Không còn hạng mục kỹ thuật đang được báo cáo QC vòng 6 đánh dấu là điều kiện chặn Release.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: Trường hợp `C046` trong `submission/qc/calibration_candidates.csv` liên kết đến `img_004.jpg`, quy tắc R-07 trong guideline v1.1 và annotation `#39 tuk_tuk` trong bộ nhãn hiệu chỉnh. Có thể đối chiếu ảnh gốc, quyết định Calibration, bản nhãn cuối và bằng chứng review để truy vết phương án xử lý.

## 6. Tổng kết

Nhóm đã hoàn thành các công việc chính của Lab 7:

- Hoàn thành 80/80 lượt gán độc lập trên 40 ảnh.
- Phân tích IAA trên 217 cặp Bounding Box ghép được.
- Ghi nhận IoU trung bình trước Calibration là 0.835 và đồng thuận lớp 98.6%.
- Xử lý quy trình Calibration đối với 416 trường hợp bất thường.
- Xây dựng guideline v1.1 và taxonomy v1.1.
- Tạo bộ nhãn hiệu chỉnh gồm 40 ảnh và 423 annotation.
- Hoàn thành ghi nhận QC vòng 6 cho 40/40 ảnh.
- Không ghi nhận lỗi trong phiếu QC vòng 6.
- Xử lý trạng thái `can_review` của các annotation đã được xác nhận.
- Nhận kết quả `RELEASE` từ công cụ báo cáo QC tự động.

**QUYẾT ĐỊNH CUỐI CÙNG: RELEASE.**

Bộ nhãn được bàn giao cùng với guideline, taxonomy, kết quả IAA, hồ sơ Calibration, manifest và bằng chứng QC để phục vụ kiểm tra và đánh giá theo yêu cầu Lab 7.