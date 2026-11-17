# Báo cáo QC — Nhóm 5 thành viên

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Đặc tả và QC nhãn bounding box 2D cho bài toán phát hiện đối tượng giao thông, phục vụ mô hình lái xe tự hành; giới hạn tập 40 ảnh đô thị ban ngày và ban đêm.
- Nguồn và phép sử dụng dữ liệu: Xem data/ATTRIBUTION.csv, ảnh thu thập phục vụ học tập và nghiên cứu nội bộ.
- Manifest và checksum nhãn cuối: batch_manifest.json, SHA-256 nhãn cuối: bbeaa73fc925aa693013729b84a623ecd5ea2147d0bab93ea84f14bbfe22c238.
- Guideline và taxonomy khi gán / khi bàn giao: Khi gán dùng guideline_v1.0_agreed.md và taxonomy.json (v1.0); khi bàn giao dùng guideline_v1.1_calibrated.md và taxonomy.json (v1.0), giữ nguyên taxonomy và chỉ làm rõ câu chữ editorial.

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 120/120 lượt gán (5 thành viên gán chéo 40 ảnh, mỗi ảnh 3 người gán).
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 736 cặp box ghép, IoU trung bình đạt 0.832, đồng thuận lớp đạt 97.4% trên mẫu số 736 cặp box ghép.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: 328 box chưa ghép bên A / 325 box bên B, 5 box loại do ignore, 239 trường hợp bất đồng vùng ignore.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: img_002.jpg (xe tải chở ô tô), hai người cùng bỏ sót ô tô nhỏ trên giàn xe tải hoặc không gán sát viền theo quy tắc R-03.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: Đã xử lý 1680 ca bất đồng ghi nhận trong calibration_decisions.csv, áp dụng các rule ID từ R-01 đến R-12, người xác nhận: 101 và 103.
- Tác động v1.1 lên nhãn và ảnh: Làm rõ cách xác định mức occluded_level và xử lý các ca truncated sát mép ảnh, thống nhất giữ nguyên 40 ảnh đã gán theo bản draft consensus đã hiệu chỉnh.
- Tham chiếu bằng chứng khi Rework: Không áp dụng Rework, bản sửa được duyệt qua vòng 2 đạt chuẩn.

## 4. Kết quả duyệt

- Vòng QC: 2
- Người review và ngày: 103, 2026-11-17
- Đã duyệt / cả lô: 40/40 ảnh
- Ảnh lỗi / ảnh đã duyệt: 0/40 = 0.0%
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: 0/526 = 0.0%
- Vật bỏ sót, ảnh và vùng: 0 vật bỏ sót
- Ca khó / phần còn lại, lỗi lặp và ví dụ: Ca khó đã duyệt 38/38 (0% lỗi), phần còn lại đã duyệt 2/2 (0% lỗi), chi tiết tại submission/qc/reviews/round-02/qc_review.csv.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Máy chặn được lỗi sai taxonomy, tọa độ box ra ngoài ảnh hoặc box rỗng; người đọc ảnh cần phát hiện vật ảo như tượng đá img_027 hay ảnh billboard img_018 mà mô hình/người gán nhầm.

## 5. Quyết định bàn giao

- Quyết định: Release
- Lý do, người nhận và phạm vi sử dụng: Dữ liệu đạt tỷ lệ lỗi 0.0% (< 2.0%), đồng thuận lớp cao, bàn giao cho bộ phận huấn luyện mô hình Object Detection xe tự lái.
- Nếu còn việc: Không còn việc tồn đọng, toàn bộ 40 ảnh đã được chuẩn hóa và kiểm tra hoàn tất.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: Ca C001, ảnh img_001.jpg, quy tắc R-01, nhãn tại labels_calibrated.json khớp hoàn toàn với đặc tả.
