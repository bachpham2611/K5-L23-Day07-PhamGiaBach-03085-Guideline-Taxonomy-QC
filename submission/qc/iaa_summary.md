# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 4
- Lượt gán đủ **80/80**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 217 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 40 | 217 | 0.835 | 98.6% | 59.9% | 83.9% | 93/117 | 0 | CHƯA ĐẠT |
| 101|102 | 6 | 19 | 0.873 | 100.0% | 100.0% | 100.0% | 12/3 | 0 | ĐẠT |
| 101|103 | 9 | 58 | 0.837 | 96.6% | 51.7% | 74.1% | 30/10 | 0 | CHƯA ĐẠT |
| 101|104 | 5 | 52 | 0.818 | 98.1% | 46.2% | 90.4% | 20/14 | 0 | CHƯA ĐẠT |
| 102|103 | 5 | 21 | 0.818 | 100.0% | 47.6% | 81.0% | 13/20 | 0 | CHƯA ĐẠT |
| 102|104 | 9 | 42 | 0.838 | 100.0% | 76.2% | 90.5% | 11/36 | 0 | CHƯA ĐẠT |
| 103|104 | 6 | 25 | 0.852 | 100.0% | 60.0% | 72.0% | 7/34 | 0 | ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 416 ({'bo_sot_thua': 155, 'iou_thap': 83, 'sai_thuoc_tinh': 96, 'bat_dong_ignore': 79, 'sai_lop': 3}), trên 37 ảnh cần calibration.
