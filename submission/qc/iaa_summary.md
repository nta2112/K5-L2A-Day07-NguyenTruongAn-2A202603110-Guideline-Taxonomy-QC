# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 5
- Lượt gán đủ **120/120**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 736 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 120 | 736 | 0.832 | 97.4% | 46.3% | 88.7% | 328/325 | 5 | CHƯA ĐẠT |
| 101|102 | 13 | 66 | 0.841 | 100.0% | 21.2% | 90.9% | 10/59 | 1 | CHƯA ĐẠT |
| 101|103 | 12 | 78 | 0.803 | 98.7% | 43.6% | 75.6% | 14/56 | 2 | CHƯA ĐẠT |
| 101|104 | 12 | 89 | 0.791 | 97.8% | 43.8% | 91.0% | 14/87 | 0 | CHƯA ĐẠT |
| 101|105 | 11 | 54 | 0.757 | 98.1% | 61.1% | 92.6% | 31/15 | 0 | CHƯA ĐẠT |
| 102|103 | 13 | 102 | 0.877 | 100.0% | 36.3% | 94.1% | 37/9 | 2 | ĐẠT |
| 102|104 | 11 | 83 | 0.908 | 97.6% | 49.4% | 95.2% | 15/27 | 0 | ĐẠT |
| 102|105 | 11 | 43 | 0.793 | 97.7% | 37.2% | 95.3% | 43/7 | 0 | CHƯA ĐẠT |
| 103|104 | 11 | 94 | 0.863 | 93.6% | 100.0% | 86.2% | 20/45 | 0 | CHƯA ĐẠT |
| 103|105 | 12 | 50 | 0.800 | 94.0% | 20.0% | 74.0% | 54/12 | 0 | CHƯA ĐẠT |
| 104|105 | 14 | 77 | 0.813 | 96.1% | 29.9% | 89.6% | 90/8 | 0 | CHƯA ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 1680 ({'bo_sot_thua': 528, 'iou_thap': 308, 'sai_thuoc_tinh': 437, 'co_can_review': 149, 'bat_dong_ignore': 239, 'sai_lop': 19}), trên 38 ảnh cần calibration.
