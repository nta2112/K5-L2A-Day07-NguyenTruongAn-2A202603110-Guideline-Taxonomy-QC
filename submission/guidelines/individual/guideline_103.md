# Guideline gán nhãn — v0.1 — 2A202603110 - Nguyễn Trường An

- **Phiên bản:** v0.1 (Bản cá nhân — Pha 1) | **Ngày:** 2026-11-16
- **Công cụ & Định dạng:** CVAT, Bounding Box 2D, xuất COCO 1.0 (không kèm ảnh).

---

## 1. Mục tiêu
- **Ứng dụng:** Huấn luyện mô hình nhận diện người và phương tiện cho camera giao thông và xe tự hành (ADAS).
- **Lỗi nghiêm trọng cần tránh:**
  - *Bỏ sót (FN)* `person` hoặc `motorcycle` $\rightarrow$ nguy cơ va chạm trực tiếp.
  - *Nhầm lớp* (`truck` thành `car`) $\rightarrow$ tính sai khoảng cách an toàn.
  - *Gán nhầm vật giả (FP)* (tượng, biển quảng cáo, hình phản chiếu) $\rightarrow$ gây phanh gấp ảo.

---

## 2. Lớp và thuộc tính

### 2.1. Các lớp đối tượng (Mỗi vật chỉ thuộc 1 lớp)
| Lớp | Định nghĩa ngắn gọn | Ví dụ | Không gán / Dễ nhầm |
|---|---|---|---|
| `person` | Con người thật (đi bộ, đứng, ngồi, lái xe). | Người đi bộ (`img_001`), người lái xe máy (`img_007`). | Tượng (`img_011`, `img_017`), ma-nơ-canh (`img_033`), ảnh in quảng cáo (`img_018`). |
| `motorcycle` | Xe 2–3 bánh gắn động cơ xăng hoặc điện. | Xe số, xe ga, xe máy điện (`img_001`, `img_007`). | Xe đạp (`img_006`), xích lô (`img_029`, `img_039`) $\rightarrow$ **không gán**. Xe tuk-tuk (`img_004`) $\rightarrow$ đề xuất tách lớp riêng ở Pha 2. |
| `car` | Ô tô chở người 4–9 chỗ. | Sedan, SUV, Hatchback, bán tải nhỏ (`img_001`, `img_003`). | Xe tải thùng kín (`van`), xe buýt (`img_009`), ô tô đồ chơi (`img_038`). |
| `van` | Ô tô chở người/hàng thân hộp liền 1 khoang. | Xe tải van mini, xe đưa đón cỡ nhỏ (`img_005`, `img_010`). | Xe buýt lớn (`bus`), xe tải thùng rời (`truck`). |
| `truck` | Xe chở hàng có cabin tách rời thùng hoặc tải trọng lớn. | Xe tải thùng, đầu kéo, xe chở ô tô 2 tầng (`img_002`). | Xe bán tải không thùng rời (`car`), xe van liền khối (`van`). |
| `ignore_region` | Vùng bỏ qua (không tính lỗi nếu bỏ sót). | Đám đông/cụm xe ở xa cao `< 20px` (`img_013`), gương phản chiếu lớn (`img_020`). | Vật thể đơn lẻ rõ nét cao `>= 20px` (phải vẽ box riêng). |

* **Vật ngoài danh sách v0:** Xe buýt (`img_009`, `img_031`, `img_037`, `img_040`) và xe tuk-tuk (`img_004`, `img_035`) tạm tick `can_review` để họp Pha 2 bổ sung lớp `bus` và `tuk_tuk`. Xe đạp/xích lô không gán.

### 2.2. Thuộc tính
| Thuộc tính | Giá trị | Quy tắc chọn |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | Mức độ bị vật khác che khuất: `0` (0%), `1` (1–30%), `2` (31–70%), `3` (>70% nhưng vẫn nhận ra). Che >90% thì bỏ qua. |
| `truncated` | `no`, `yes` | `no`: nằm trọn trong ảnh. `yes`: bị cắt mất một phần bởi 4 mép biên ảnh. |
| `can_review` | `false`, `true` | Tick `true` khi gặp ca mơ hồ, xe lai hoặc vật thể gây tranh cãi để nhóm duyệt lại. |

---

## 3. Quy tắc vẽ Bounding Box
1. **R-01 (Ôm sát phần nhìn thấy):** Box chạm sát mép phần nhìn thấy được (lệch tối đa 2–3 px). Không đoán mò kéo dài box qua phần bị che khuất hoàn toàn.
2. **R-02 (Người đi xe máy):** Tách thành **2 box độc lập**: 1 box `person` (từ đỉnh mũ bảo hiểm đến bàn chân) và 1 box `motorcycle` (trọn xe từ bánh trước đến đuôi xe).
3. **R-03 (Xe chở xe):** Vẽ 1 box `truck` lớn bao toàn bộ xe tải, đồng thời vẽ box `car` riêng cho từng ô tô chở trên sàn nếu cao `>= 20px` (`img_002`).
4. **R-04 (Hàng hóa trên xe):** Box phương tiện phải ôm trùm cả phần hàng hóa/bao tải buộc trên xe (`img_035`).
5. **R-05 (Kích thước tối thiểu):** Vật lẻ cao `< 20px` $\rightarrow$ **bỏ qua**. Cụm nhiều vật `< 20px` hoặc đám đông chen chúc $\rightarrow$ khoanh 1 box `ignore_region`.
6. **R-06 (Vật thể giả):** Tượng đá/đồng, hình in trên biển quảng cáo, ảnh phản chiếu qua gương/kính, ma-nơ-canh, đồ chơi $\rightarrow$ **tuyệt đối không gán**.
7. **R-07 (Xe chủ - Ego vehicle):** Không gán nắp ca-pô của chính xe gắn camera ở đáy ảnh.

---

## 4. Các ca khó thực tế (Từ 40 ảnh mẫu)
| Ảnh | Tình huống | Rule | Cách xử lý ban đầu | Lý do |
|---|---|---|---|---|
| `img_002` | Xe tải chở nhiều ô tô con 2 tầng. | R-03 | Vẽ 1 box `truck` lớn + vẽ riêng từng box `car` bên trên. | Cần nhận diện cả xe mẹ và các xe con bên trên. |
| `img_004` | Xe 3 bánh tuk-tuk chở khách. | Mới | Tạm gán `car` (hoặc `motorcycle`) + tick `can_review` để Pha 2 thêm lớp `tuk_tuk`. | Cấu tạo 3 bánh có cabin, không khớp hẳn lớp nào của v0. |
| `img_009` | Xe buýt công cộng cỡ lớn. | Mới | Tạm gán `truck` + tick `can_review` để Pha 2 thêm lớp `bus`. | v0 thiếu lớp `bus`, kích thước khác hẳn xe con. |
| `img_011`, `017`, `027` | Tượng người và tượng ô tô bằng đá/đồng. | R-06 | **Không gán nhãn.** | Vật tĩnh trang trí, không tham gia giao thông. |
| `img_018` | Người mẫu in trên biển quảng cáo lớn. | R-06 | **Không gán nhãn.** | Hình ảnh 2D tĩnh, tránh gây phanh ảo. |
| `img_020` | Người và xe phản chiếu trong gương chiếu hậu. | R-06 | **Không gán box vật thể** (có thể khoanh `ignore_region` cho gương). | Ảnh ảo quang học, sai vị trí vật lý 3D. |
| `img_035` | Xe tuk-tuk chở bao tải hàng cao trên nóc. | R-04 | Vẽ box ôm trọn cả xe lẫn khối hàng trên nóc + tick `can_review`. | Hàng hóa làm thay đổi kích thước thực tế khi xe di chuyển. |
| `img_037` | Xe bán tải lắp thùng kín cao phía sau. | Mới | Tạm gán `car` + tick `can_review` để nhóm chốt ở Pha 2. | Dễ nhầm lẫn giữa `car`, `van` và `truck`. |
| `img_033`, `038` | Ma-nơ-canh và xe đồ chơi sau tủ kính. | R-06 | **Không gán nhãn.** | Vật trưng bày trong cửa hàng, không phải đối tượng giao thông. |

---

## 5. QC và Lịch sử phiên bản
* **Ngưỡng QC (`taxonomy.json`):** Chiều cao box $\ge 20\text{px}$ | IoU khớp box: $\ge 0.85$ (vật $\ge 50\text{px}$) và $\ge 0.70$ (vật $< 50\text{px}$) | IAA mIoU $\ge 0.85$ | Đồng thuận lớp $\ge 95\%$.
* **Đề xuất cho nhóm (Pha 2):** Bổ sung 2 lớp `bus` và `tuk_tuk` vào `taxonomy.json`; thống nhất bỏ qua xe đạp/xích lô.

| Phiên bản | Ngày | Thay đổi chính | Lý do |
|---|---|---|---|
| `v0` | 2026-11-16 | Bản mẫu 6 lớp cơ bản của giảng viên. | Bản khởi đầu, còn nhiều điểm mơ hồ. |
| `v0.1` | 2026-11-16 | Bổ sung ví dụ/phản ví dụ, lượng hóa % che khuất, thêm quy tắc `R-01` đến `R-07`, tổng hợp 10 ca khó thực tế từ 40 ảnh và đề xuất thêm lớp `bus`, `tuk_tuk`. | Hoàn thiện bản cá nhân Pha 1 để đối chiếu họp nhóm Pha 2. |