# Nguồn gốc bản nhãn hiệu chỉnh

**Quyết định QC: Hold.** Bản nhãn này do AI tạo từ nhãn độc lập và ảnh gốc; không phải COCO được thành viên nhóm xuất lại từ CVAT. Thành viên mã 102 xác nhận trong trao đổi ngày 17/11/2026 đã rà và đồng ý cả 416 ca calibration, nên `nguoi_chot=102` ghi người chốt phương án AI. Thành viên 102 chỉ xác nhận các ca calibration, chưa duyệt QC độc lập toàn bộ 40 ảnh.

## Cách tạo

1. `qc_compare.py` so 80/80 lượt gán và tạo `consensus_draft.json` sáu lớp, 427 annotation.
2. Tạo COCO tám lớp theo `taxonomy_v1.1.json` bằng cách giữ các box đồng thuận cũ ở trạng thái `can_review=true`.
3. Đổi lớp trong bản AI: `#39,#40,#183,#385,#386,#425,#424` → `tuk_tuk`; `#131,#315` → `bus`; `#57` → `person`. Box `#424` được thu hẹp theo phần tuk-tuk nhìn thấy.
4. Loại chín annotation cũ: `#150,#151,#152` (điêu khắc trong `img_011`), `#194` (`img_017`), `#202` (box người trùng trong `img_019`), `#221,#222` (phản chiếu trong `img_020`), `#263` (xe mô hình trong `img_024`), `#350` (ma-nơ-canh trong `img_033`).
5. Thêm năm box AI: `#428–#430` bus ở `img_037`, `#431` bus ở `img_040`, `#432` xe diễu hành `car` ở `img_019`. Tọa độ và thuộc tính của các box này cần được reviewer xác nhận.
6. Tạo 416 dòng đề xuất AI bằng cách ghép box của `calibration_candidates.csv` với box gần nhất trong `ai_review_evidence.json`. Sau đó 102 xác nhận đã rà và đồng ý các đề xuất; quyết định vẫn dẫn chiếu bản AI để giữ nguồn gốc. Một dòng `pham_vi=lo` ghi tác động của thay đổi ngữ nghĩa lên toàn bộ 40 ảnh, nhưng chưa thay lượt QC độc lập toàn lô.

COCO chứng cứ là `submission/qc/ai_review_evidence.json` (SHA-256 `9fea1c0debf680ea2043294300b396b84dfebb6852d8534885a1d04b644eecaf`); bản xuất theo hash và nhãn gộp nằm trong cùng thư mục QC. Nhãn gộp có 40 ảnh, 423 annotation. Các box tiếp tục mang `can_review=true` để ngăn việc hiểu nhầm bản AI là dữ liệu đã được người nghiệm thu.

## Giới hạn bắt buộc khi sử dụng

- 102 xác nhận đã rà 416 ca, nhưng chưa có lượt reviewer độc lập duyệt 40 ảnh; phiếu vòng 4 có 0/40 ảnh được xác nhận.
- Số liệu IAA trước calibration không đo chất lượng bản nhãn AI. Xác nhận 416 ca của 102 là xác nhận trong trao đổi, không phải biên bản QC độc lập toàn lô.
- `check_submission.py` kiểm cấu trúc và tính khớp của hồ sơ, không chứng minh độ đúng bằng mắt. Kết quả 0 lỗi tự động không thay quyết định Hold.
