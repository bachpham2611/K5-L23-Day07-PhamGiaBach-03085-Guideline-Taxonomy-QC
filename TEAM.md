# Thông tin nhóm

- Tên nhóm: Đại đi
- Mã thành viên: 101, 102, 103, 104
- Người điều phối: 103
- Người review bản cuối: 102 (QC vòng 6)
- Commit chốt bài (điền ở kênh nộp sau khi commit): Chưa có SHA; cập nhật sau khi hoàn tất commit và push lên GitHub.

## Kết quả tóm tắt

- Pilot hay cả bộ 40 ảnh: Cả bộ 40 ảnh.
- Lượt gán hoàn tất / được giao: 80/80 lượt (4 thành viên, mỗi ảnh được phân công cho 2 người).
- IAA trước calibration (kèm mẫu số): 217 cặp Bounding Box ghép được; IoU trung bình 0.835, thấp hơn ngưỡng 0.85; đồng thuận lớp 98.6%, vượt ngưỡng 95%.
- Calibration: Ghi nhận 416 trường hợp bất thường, sử dụng guideline v1.1 và taxonomy v1.1. Bộ nhãn hiệu chỉnh gồm 40 ảnh và 423 annotation.
- Vòng QC và số ảnh đã duyệt / cả lô: Vòng 6, đã duyệt 40/40 ảnh; 37/37 ảnh ca khó và 3/3 ảnh còn lại.
- Kết quả QC: Không ghi nhận ảnh lỗi (0/40) hoặc annotation lỗi (0/423) trong phiếu QC vòng 6. Không còn annotation đánh dấu `can_review=true`.
- Quyết định và lý do: Release. Công cụ QC vòng 6 trả về RELEASE trên bộ nhãn hiệu chỉnh. Nhóm bàn giao bộ dữ liệu cùng guideline, taxonomy, kết quả IAA, nhật ký Calibration, manifest và bằng chứng QC theo phạm vi thực hành Lab 7.

## Hồ sơ bàn giao

- Báo cáo QC: `submission/qc/QC_REPORT.md`
- Nhãn cuối: `submission/qc/labels_calibrated.json`
- Manifest: `submission/qc/batch_manifest.json`
- Bằng chứng vòng QC cuối: `submission/qc/reviews/round-06/`
- Ghi chú cá nhân: `submission/individual_notes/`
- Guideline: `submission/guidelines/`
