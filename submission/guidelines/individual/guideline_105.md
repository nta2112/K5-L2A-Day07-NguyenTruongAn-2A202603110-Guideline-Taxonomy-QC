# Guideline gán nhãn — v0.1 (cá nhân) — 105

> Bản guideline cá nhân Pha 1, viết độc lập sau khi rà soát toàn bộ 40 ảnh (`img_001.jpg`–`img_040.jpg`) dựa trên khung `guideline/TEMPLATE_guideline.md`, `guideline/guideline_v0_starter.md` (v0) và `submission/taxonomy.json`. Những điểm v0 còn mơ hồ được làm rõ theo một cách hiểu duy nhất kèm ngưỡng đo được và đánh dấu các tình huống cần thống nhất với nhóm ở Pha 2.

- Phiên bản: v0.1 (cá nhân Pha 1)
- Ngày: 2026-11-16
- Người viết / duyệt: 105

---

## 1. Mục tiêu

- **Bài toán và mô hình sử dụng:** Bộ nhãn dùng để huấn luyện và đánh giá mô hình phát hiện vật thể 2D (`bounding box`, xuất COCO 1.0 trên CVAT) nhận diện **người và phương tiện giao thông** trên ảnh đường phố (phục vụ camera giám sát giao thông đô thị và hệ thống hỗ trợ lái xe / xe tự hành).
- **Lỗi nào nguy hiểm:**
  - **Bỏ sót (`False Negative`)** người đi bộ, người đi xe máy hoặc phương tiện đang lưu thông làm mô hình mất khả năng cảnh báo va chạm.
  - **Gán thừa (`False Positive`)** tượng người/xe, ma-nơ-canh, đồ chơi mô hình, ảnh chân dung trên biển quảng cáo hoặc ảnh phản chiếu qua gương/kính khiến xe phanh giả hoặc học sai đặc trưng vật thể thật ngoài đường.
  - **Box vẽ lỏng hoặc sai lớp** làm giảm IoU xuống dưới ngưỡng QC (`0.85` / `0.70`) và gây nhiễu giữa các nhóm phương tiện có kích thước, tốc độ khác nhau (ví dụ `car` vs `truck` vs `bus` vs `tuk_tuk`).

---

## 2. Lớp và thuộc tính

### 2.1. Lớp (mỗi vật thuộc đúng một lớp)

| Lớp | Định nghĩa (một câu) | Ví dụ (kèm ảnh) | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con chở khách cỡ nhỏ đến vừa (sedan, hatchback, SUV, crossover, taxi 4–7 chỗ, xe cổ kiểu coupe/wagon). | `img_002.jpg` (8 ô tô trên sàn xe), `img_012.jpg` (SUV bạc), `img_015.jpg` (SUV đỏ, hatchback bạc), `img_018.jpg` (dãy xe con sau hàng rào), `img_023.jpg` (sedan đen), `img_024.jpg` (xe Pacer đen trong bảo tàng), `img_040.jpg` (taxi xanh-đỏ). | Không gồm xe mô hình đồ chơi (`img_038.jpg`), vỏ xe mini trưng bày trên nóc (`img_024.jpg`), xe phản chiếu trong gương (`img_020.jpg`) hoặc xe bán tải/tải nhỏ (`img_001.jpg`, `img_012.jpg`, `img_036.jpg`). |
| `truck` | Xe tải chở hàng có khoang/sàn hàng tách biệt hoặc đầu kéo chuyên dụng (bao gồm xe tải thùng hở/bạt/kín, xe chở ô tô, và đề xuất tạm xếp xe bán tải thùng hở vào nhóm này cho tới khi họp Pha 2). | `img_002.jpg` (xe tải Renault chở 8 ô tô), `img_007.jpg` (xe tải thùng Isuzu bên phải), `img_012.jpg` (xe tải thùng giữa đường và bán tải chở hàng phía trước), `img_036.jpg` (xe tải nhẹ thùng phủ bạt Veryca 1.3L), `img_001.jpg` (bán tải Suzuki chở khung sắt). | Không gồm bia mộ/tượng đá hình xe bán tải (`img_027.jpg`). Dễ nhầm với `van` ở các xe bán tải gắn nắp thùng kín cao (`img_007.jpg` Hilux trắng, `img_016.jpg` bán tải nắp cao, `img_037.jpg` bán tải có nắp `Master 75`) → cần chốt rõ ranh giới ở Pha 2. |
| `van` | Xe ô tô thân liền khối cỡ vừa chuyên chở nhiều người (xe 9–16 chỗ) hoặc chở hàng dạng thùng kín liền cabin. | `img_005.jpg` (xe MB 100 bạc phía sau), `img_011.jpg` (xe cứu thương/van trắng phía xa), `img_016.jpg` (xe van xám bạc bên phải). | Không gán `van` cho xe buýt cỡ lớn/trung (`img_009.jpg`, `img_031.jpg`, `img_037.jpg`, `img_040.jpg`) và không gán cho xe tải có thùng rời cabin (`img_007.jpg`, `img_036.jpg`). |
| `motorcycle` | Phương tiện hai bánh gắn động cơ (xe số, xe tay ga, mô tô phân khối lớn), bao gồm cả xe máy gắn thùng bên (sidecar) hoặc xe máy kéo thùng hàng gắn cố định. | `img_005.jpg`, `img_006.jpg`, `img_008.jpg`, `img_010.jpg`, `img_021.jpg`, `img_026.jpg`, `img_029.jpg` (xe tay ga đen), `img_022.jpg` (mô tô có sidecar), `img_025.jpg` (xe máy gắn quầy hàng bên cạnh). | Không gồm xe đạp, xích lô đạp không động cơ (`img_003.jpg`, `img_006.jpg`, `img_010.jpg`, `img_023.jpg`, `img_029.jpg`, `img_034.jpg`, `img_039.jpg`), xe ba bánh gắn máy tuk-tuk (`img_004.jpg`, `img_016.jpg`, `img_035.jpg`), và không gộp người lái vào box `motorcycle`. |
| `person` | Người thật đang đi bộ, đứng, ngồi hoặc điều khiển/ngồi trên phương tiện giao thông trong cảnh thực. | `img_001.jpg` (cô gái đứng và khách ngồi ăn), `img_004.jpg` (tài xế trong tuk-tuk), `img_005.jpg` & `img_026.jpg` (người đi xe máy), `img_014.jpg` & `img_034.jpg` (người qua đường), `img_019.jpg` (tài xế trong xe diễu hành), `img_029.jpg` (người đi xe máy và người đi xe đạp). | Không gồm tượng người (`img_011.jpg` Predator, `img_017.jpg` tượng cưỡi ngựa), ma-nơ-canh (`img_033.jpg`), búp bê/đồ chơi (`img_038.jpg`), nhân vật linh vật nhựa (`img_019.jpg`), ảnh chân dung/tranh vẽ 2D trên biển quảng cáo, áp phích, decal kính (`img_008.jpg`, `img_015.jpg`, `img_018.jpg`, `img_030.jpg`, `img_031.jpg`, `img_036.jpg`, `img_040.jpg`). |
| `ignore_region` | Vùng hình chữ nhật đánh dấu đám đông người hoặc cụm xe quá dày đặc/chồng chéo/nhỏ khó tách rời từng cá thể để không tính lỗi bỏ sót bên trong vùng đó. | `img_003.jpg` (dãy xe máy đỗ sát nhau góc trên phải), `img_011.jpg` (đám đông khán giả lễ hội phía xa), `img_019.jpg` (hàng rào người xem phía sau), `img_021.jpg` (biển xe máy xa phía trước), `img_030.jpg` (khu vực xe máy xếp dày phía sau), `img_032.jpg` (đoàn người đứng chờ xe buýt sau dải phân cách). | Không dùng `ignore_region` để che một vài vật đơn lẻ rõ ràng mà người gán lười vẽ hoặc không chắc lớp (ca đơn lẻ chưa chắc phải vẽ box riêng và tick `can_review`). |

#### Xử lý vật ngoài 6 lớp v0 (cần chốt ở Pha 2)

Phép thử thiết kế: *"Mô hình phát hiện 2D khi suy luận có cần phân biệt/nhận ra lớp này không?"*
1. **Đề xuất bổ sung lớp mới vào `taxonomy.json` ở Pha 2 (không ép gán vào lớp sai nghĩa):**
   - `bus`: Xe buýt / xe khách cỡ vừa và lớn (`img_009.jpg` xe khách 29 chỗ, `img_031.jpg` xe buýt số 62, `img_037.jpg` xe buýt GO Transit xanh, `img_040.jpg` xe buýt 511 và xe buýt phía xa). Không nên gộp vào `van` hay `truck`.
   - `tuk_tuk`: Xe ba bánh có động cơ chuyên chở khách/hàng (`img_004.jpg`, `img_016.jpg`, `img_035.jpg`, `img_040.jpg`). Khác hẳn cấu trúc `motorcycle` 2 bánh và `car` 4 bánh.
2. **Đề xuất giữ ngoài phạm vi (loại theo quy tắc) — hoặc hỏi nhóm xác nhận có cần thêm lớp `bicycle` không:**
   - **Xe đạp và xích lô đạp không gắn động cơ** (`img_003.jpg`, `img_006.jpg`, `img_010.jpg`, `img_023.jpg`, `img_029.jpg`, `img_034.jpg`, `img_039.jpg`): theo bộ lớp v0 hiện tại **không gán box cho thân xe đạp/xích lô**, chỉ gán `person` cho người đang đạp/dắt xe. Ở Pha 2 cần chốt thống nhất với cả nhóm giữa phương án loại hoàn toàn hay mở thêm lớp `bicycle`.
   - **Động vật** (chó ở `img_009.jpg`, chim mòng biển trên tượng ở `img_017.jpg`): loại theo quy tắc, không gán.

---

### 2.2. Thuộc tính (giữ tên và giá trị cố định khớp `taxonomy.json`)

| Thuộc tính | Kiểu & giá trị | Định nghĩa và ngưỡng đo cụ thể |
|---|---|---|
| `occluded_level` | `select`: `0`, `1`, `2`, `3` (mặc định `0`) | **Mức độ bị che khuất bởi vật thể khác** (người, xe khác, cột, rào chắn, cây...; phần bị cắt ở mép ảnh tính vào `truncated`, không cộng vào đây):<br>- **`0` (Không che, 0%):** Thấy trọn vẹn bề mặt hướng về camera của vật.<br>- **`1` (Che nhẹ, 1%–30% diện tích ước lượng):** Che một phần nhỏ (ví dụ đầu xe bị chắn nhẹ bởi xe trước ở `img_007.jpg`, người đi bộ bị người khác che một phần chân/tay ở `img_025.jpg`, dãy ô tô sau song sắt thưa ở `img_018.jpg`).<br>- **`2` (Che vừa–nhiều, 31%–70%):** Mất khoảng một phần ba đến quá nửa thân vật (ví dụ ô tô ở tầng dưới xe chở xe `img_002.jpg`, tài xế ngồi trong tuk-tuk `img_004.jpg`, người đi xe máy chen hàng ở `img_005.jpg`, `img_021.jpg`).<br>- **`3` (Che rất nặng, >70% nhưng vẫn nhận diện chắc chắn lớp):** Chỉ còn lộ một phần nhỏ (<30% vật thể), ví dụ hành khách ngồi sau cửa kính xe buýt `img_009.jpg`, người lái xe máy ở rìa xa trong kẹt xe `img_021.jpg`, xe máy chỉ lộ đầu/yếm trong bãi xe `img_030.jpg`. Nếu che gần hết không nhận ra được là lớp nào → không gán đơn lẻ (gom vào `ignore_region` nếu thuộc cụm đông). |
| `truncated` | `select`: `no`, `yes` (mặc định `no`) | **Vật bị cắt bởi ít nhất một trong 4 cạnh khung ảnh:**<br>- **`no`:** Toàn bộ phần cấu trúc vật nằm lọt hoàn toàn bên trong khung ảnh.<br>- **`yes`:** Một bộ phận rõ ràng của vật bị cắt đứt tại mép trái/phải/trên/dưới khung ảnh (ví dụ bán tải Suzuki bên phải `img_001.jpg`, xe tải Isuzu bên phải `img_007.jpg`, hai ô tô hai rìa `img_015.jpg`, người đi xe máy ở góc dưới/phải `img_005.jpg`, `img_040.jpg`). |
| `can_review` | `checkbox`: `false` / `true` (mặc định `false`) | Tick (`true`) khi đã vẽ box nhưng **chưa chắc chắn** về lớp (ví dụ bán tải gắn thùng `Master 75` ở `img_037.jpg`), ranh giới box (hàng hóa cồng kềnh ở `img_035.jpg`), hoặc mức `occluded_level`. Không dùng trên `ignore_region`. |

*(Lưu ý thao tác CVAT: chỉ chọn `occluded_level` trong panel **Objects**, không dùng cờ `occluded` phím `Q` mặc định của CVAT).*

---

## 3. Quy tắc vẽ box

- **`R-01` Ôm sát phần nhìn thấy (visible-only tight box):** Vẽ box ôm khít rìa pixel ngoài cùng nhìn thấy được của vật (tính cả gương chiếu hậu, bánh xe, cản xe). Không suy đoán kéo rộng box qua vùng bị vật khác che hoặc vượt ra ngoài biên ảnh.
- **`R-02` Người đi xe máy và người ngồi trên xe (tách box độc lập):**
  - Với mọi cặp người–xe máy (`img_005.jpg`, `img_006.jpg`, `img_010.jpg`, `img_021.jpg`, `img_026.jpg`), luôn vẽ **box riêng cho `person`** (từ đỉnh mũ bảo hiểm/đầu đến bàn chân nhìn thấy) và **box riêng cho `motorcycle`** (từ bánh trước/tay lái/gương đến đuôi xe/bánh sau, không kéo theo phần đầu/lưng người ngồi trên yên).
  - Xe có `2` người ngồi (`img_006.jpg`, `img_010.jpg`, `img_026.jpg`) → vẽ `2` box `person` và `1` box `motorcycle`.
  - Kiểm thử nhất quán: áp dụng quy tắc này trên `img_006.jpg` cho kết quả cùng số box dự đoán với `img_005.jpg` (mỗi người thấy rõ một box `person`, mỗi xe máy thấy rõ một box `motorcycle`, tự động tính `occluded_level` cho người bị che phía sau).
  - Xe máy có gắn thùng bên (`img_022.jpg` sidecar, `img_025.jpg` xe đẩy hàng gắn hông): gán **1 box `motorcycle`** bao trùm cả xe máy và phần thùng gắn liền khối thành một phương tiện.
- **`R-03` Vật bị che khuất và nhòe chuyển động:** Vật bị che (`occluded_level` `1`–`3`) vẫn gán bình thường phần nhìn thấy nếu xác định chắc chắn lớp và phần lộ có chiều cao `>= 20 px`. Nếu bị nhòe chuyển động quá nặng chỉ còn vệt màu mờ không xác định được ranh giới rõ (`img_014.jpg` các vệt xe lướt phía sau, `img_015.jpg` vệt xe đen chuyển động giữa đường), **không tự đoán vẽ box lỏng**; ghi nhận ca cấp ảnh để chốt thống nhất với reviewer.
- **`R-04` Vật bị cắt mép ảnh (`truncated`):** Box chạm tối đa đúng biên `[0, width]` hoặc `[0, height]`, tuyệt đối không vượt biên, và đặt `truncated = yes`. Nếu chỉ lộ một mẩu vụn quá nhỏ (`< 20 px`) sát mép ảnh không đủ nhận dạng (ví dụ đầu tay lái lộ rất nhỏ ở rìa trái `img_006.jpg`), không gán.
- **`R-05` Xe chở xe và hàng hóa / vật mang vác trên xe:**
  - **Xe chở ô tô (`img_002.jpg`):** Vẽ `1` box `truck` ôm trọn xe tải đầu kéo + rơ-moóc 2 tầng, **và vẽ riêng từng box `car`** cho cả 8 ô tô thật nằm trên 2 sàn xe (đặt `occluded_level = 1` hoặc `2` tùy mức bị khung sàn che).
  - **Hàng hóa thông thường vs hàng hóa quá khổ:** Hàng nằm gọn trong lòng thùng xe (`img_012.jpg` thùng hàng trong bán tải) nằm bên trong box xe. Ngược lại, **không kéo phình box xe/người theo hàng hóa quá khổ vươn ra ngoài khung** như chồng bao tải cao vọt trên nóc tuk-tuk (`img_035.jpg`) hay thanh kim loại dài vác ngang vai người đi xe máy và xe đạp (`img_029.jpg`): box chỉ ôm sát khung xe hoặc cơ thể người.
  - **Vật trưng bày gắn trên xe (`img_024.jpg`, `img_028.jpg`):** Chỉ gán xe thật bên dưới (xe Pacer đen ở `img_024.jpg`, xe bán tải cổ nâu ở `img_028.jpg`); không gán vỏ thân xe mini trên nóc (`img_024.jpg`) hay tượng ngựa trắng đứng trên thùng bán tải (`img_028.jpg`).
- **`R-06` Vật không thật và phản chiếu (loại bỏ):**
  - **Không gán** tượng/bia mộ (`img_011.jpg` tượng Predator, `img_013.jpg` tượng gà vàng & sư tử đá, `img_017.jpg` tượng người cưỡi ngựa, `img_027.jpg` bia mộ hình xe bán tải, `img_028.jpg` tượng ngựa trắng), ma-nơ-canh (`img_033.jpg`), đồ chơi mô hình (`img_038.jpg`), linh vật giả trong xe (`img_019.jpg` nhân vật chuột trắng).
  - **Không gán** hình ảnh 2D in trên biển quảng cáo, áp phích, tường vẽ, decal dán kính (`img_008.jpg`, `img_015.jpg`, `img_018.jpg`, `img_026.jpg`, `img_030.jpg`, `img_031.jpg`, `img_036.jpg`, `img_040.jpg`).
  - **Không gán** hình phản chiếu qua gương chiếu hậu (`img_020.jpg`), kính cửa tiệm (`img_012.jpg`, `img_033.jpg`) hoặc vũng nước mặt đường (`img_007.jpg`, `img_015.jpg`, `img_031.jpg`).
- **`R-07` Xe chủ (ego vehicle):** Không gán gương chiếu hậu hoặc phần khung cửa của chính xe gắn camera người chụp (`img_020.jpg`).
- **`R-08` Vật quá nhỏ (`< 20 px`) và đám đông / bãi xe dày (`ignore_region`):**
  - Mọi người/xe đơn lẻ có chiều cao box `< 20 px` (`min_box_height_px = 20`) thì **không gán box đơn lẻ** (ví dụ xe/người ở rất xa cuối đường trong `img_013.jpg`, `img_034.jpg`).
  - Cụm nhiều người hoặc nhiều xe xếp chồng, san sát nhau khó phân tách từng cá thể và dễ gây sai khác IoU (`img_003.jpg` dãy xe máy đỗ bên phải, `img_008.jpg` dãy xe cuối chợ, `img_011.jpg` đám đông phía xa, `img_019.jpg` hàng rào khán giả phía sau, `img_021.jpg` đoàn xe máy dày đặc ở xa, `img_025.jpg` cụm người trên ghế bãi biển bên trái, `img_030.jpg` bãi xe máy dày phía sau, `img_032.jpg` đoàn người đứng chờ xe buýt sau dải phân cách): gom thành **1 box `ignore_region`** thay vì vẽ rời rạc thiếu ổn định; những cá thể đứng tách biệt rõ ràng bên ngoài cụm (ví dụ 2 người đứng rìa phải `img_032.jpg`, 2 người đứng gần hàng rào trái `img_011.jpg`, các xe máy hàng đầu `img_021.jpg`) vẫn gán `person` / `motorcycle` bình thường.

---

## 4. Ca khó (khảo sát trên 40 ảnh)

Phân tách rõ giữa **ca loại theo quy tắc** (tự tin loại bằng định nghĩa) và **ca cần hỏi reviewer / chốt ở Pha 2**:

| Ảnh | Tình huống | Loại xử lý | Quyết định đề xuất của `105` | Lý do |
|---|---|---|---|---|
| `img_001.jpg` | Xe bán tải Suzuki thùng hở bị cắt ở mép phải, cô gái đứng chắn trước thân xe | Cần hỏi reviewer (Pha 2) | Tạm gán `truck` (`truncated=yes`, `occluded_level=1`), cô gái `person`; nhóm thống nhất bán tải thùng hở xếp `truck` hay `car` | v0 chưa định nghĩa xe bán tải (pickup) thuộc `car` hay `truck` |
| `img_002.jpg` | Xe tải 2 tầng chở 8 ô tô con trên sàn + công nhân đứng bên phải | Cần hỏi reviewer (Pha 2) | Gán `1` `truck` bao cả xe chở + `8` `car` trên 2 sàn (`occluded_level=1`/`2`) + `1` `person` | Các ô tô trên sàn là vật thể `car` thật nhìn thấy rõ, nhưng dễ phát sinh 2 cách hiểu nếu không chốt trước |
| `img_003.jpg`, `img_023.jpg` | Người dắt/đạp xe đạp (`img_003.jpg`) và đạp xích lô (`img_023.jpg`); dãy xe máy đỗ xa bên phải | Cần hỏi reviewer (lớp xe đạp/xích lô) + Quy tắc (`R-08`) | Gán `person` cho người; **không gán** xe đạp/xích lô vào `motorcycle`; dãy xe đỗ xa bên phải `img_003.jpg` đặt `ignore_region` | Xe đạp/xích lô không có động cơ, nằm ngoài 6 lớp v0; cần nhóm chốt giữ loại bỏ hay thêm lớp `bicycle` |
| `img_004.jpg`, `img_016.jpg`, `img_035.jpg` | Xe 3 bánh `tuk-tuk` chở khách/hàng (`img_035.jpg` chất bao tải cao vọt trên nóc) | Cần hỏi reviewer (Pha 2) | Đề xuất thêm lớp `tuk_tuk` (chưa ép sang `motorcycle`/`car`); tài xế bên trong gán `person` (`occluded_level=2`); ở `img_035.jpg` box ôm khung xe, **không bao đống bao tải trên nóc** (`R-05`) | v0 thiếu lớp `tuk-tuk` và không nói ranh giới box khi hàng chất vượt nóc |
| `img_005.jpg`, `img_006.jpg`, `img_010.jpg`, `img_026.jpg` | Nhiều người ngồi trên 1 xe máy; người trước/sau che khuất nhau và bị cắt mép ảnh | Quy tắc (`R-02`, `R-04`) | Tách `1` `motorcycle` + từng `person` riêng biệt; bật `truncated=yes` cho người/xe chạm mép (`img_005.jpg`), đặt `occluded_level` theo % che | Bảo đảm mô hình học riêng con người và xe máy đúng số lượng cá thể |
| `img_007.jpg`, `img_016.jpg`, `img_036.jpg`, `img_037.jpg` | Xe bán tải / tải nhẹ gắn thùng kín hoặc phủ bạt cao (`img_007.jpg` Hilux thùng kín, `img_036.jpg` Veryca bạt, `img_037.jpg` bán tải gắn nắp `Master 75`) | Cần hỏi reviewer (Pha 2) | Tạm gán `truck` (vì có khoang chở hàng tách rời cabin/thùng hàng công dụng) + tick `can_review` | Ranh giới giữa `truck` và `van` trên các xe tải nhẹ/bán tải độ thùng là điểm dễ lệch lớp nhất |
| `img_009.jpg`, `img_031.jpg`, `img_037.jpg`, `img_040.jpg` | Xe buýt / xe khách cỡ lớn (`img_009.jpg` County, `img_031.jpg` bus 62, `img_037.jpg` GO Transit, `img_040.jpg` bus 511) | Cần hỏi reviewer (Pha 2) | Đề xuất thêm lớp **`bus`** vào `taxonomy.json` (không gộp vào `van` hay `truck`). Hành khách thấy rõ qua kính `img_009.jpg` gán `person` (`occluded_level=3`) hoặc hỏi nhóm có bỏ qua người ngồi sâu trong bus | v0 không có lớp `bus`; nếu mỗi người tự chọn `van` hoặc `truck` sẽ tụt đồng thuận lớp |
| `img_011.jpg`, `img_013.jpg`, `img_017.jpg`, `img_027.jpg`, `img_028.jpg` | Tượng Predator (`img_011`), tượng gà/sư tử (`img_013`), tượng cưỡi ngựa (`img_017`), bia mộ xe bán tải (`img_027`), tượng ngựa trên thùng xe (`img_028`) | Loại theo quy tắc (`R-06`) | **Không gán** bất kỳ tượng/bia mộ nào; chỉ gán người thật, xe thật xung quanh (`img_011` đám đông xa → `ignore_region`, `img_028` bán tải thật → `truck`, SUV đỏ → `car`) | Vật tĩnh/điêu khắc không phải đối tượng tham gia giao thông thật; `img_017.jpg` và `img_027.jpg` là ảnh 0 đối tượng |
| `img_014.jpg`, `img_015.jpg` | Vệt người đi xe máy nhòe chuyển động mạnh (`img_014.jpg`) và vệt đen giữa đường + phản chiếu mặt đường ướt (`img_015.jpg`) | Cần hỏi reviewer (`img_014.jpg`) + Loại theo quy tắc phản chiếu (`R-06`) | Không gán phản chiếu dưới đường ướt (`img_015.jpg`); với các vệt mờ quá nặng không rõ biên xe ở `img_014.jpg`/`img_015.jpg` tạm không vẽ box rời, ghi vấn đề cấp ảnh để chốt ngưỡng motion blur | Cần nhóm thống nhất mức nhòe tối đa vẫn được gán để tránh IoU thấp |
| `img_018.jpg`, `img_030.jpg`, `img_031.jpg`, `img_036.jpg`, `img_040.jpg` | Ảnh người trên biển quảng cáo (`img_018`, `img_036`), poster ca sĩ (`img_030`), nhân vật hoạt hình lớn trên tường (`img_031`), ảnh dán kính taxi (`img_040`) | Loại theo quy tắc (`R-06`) | **Không gán** người/nhân vật in 2D; ở `img_018.jpg` chỉ gán các xe `car` thật sau hàng rào (`occluded_level=1`) | Hình in/tranh vẽ 2D không phải người thật |
| `img_019.jpg` | Xe diễu hành DangerMouse màu vàng (có biển số thật `YR04 NUM` và tài xế thật bên cạnh linh vật chuột) + đám đông sau rào | Cần hỏi reviewer (xe diễu hành) + Quy tắc (`R-06`, `R-08`) | Gán `car` cho xe vàng + `person` cho tài xế (`occluded_level=2`); **không gán** linh vật chuột; khán giả sau rào gom `ignore_region` | Xe chạy trên đường thật có biển số nhưng ngoại hình lạ giống đồ chơi |
| `img_020.jpg` | Gương chiếu hậu xe chủ soi một ô tô phía sau trong gương | Loại theo quy tắc (`R-06`, `R-07`) | **Không gán** xe phản chiếu trong gương và không gán phần thân/gương xe chủ (ảnh 0 đối tượng) | Phản chiếu gương không phải vị trí thực trong không gian trước mắt và dễ gây nhầm khoảng cách |
| `img_021.jpg`, `img_030.jpg`, `img_032.jpg` | Giao thông xe máy dày đặc (`img_021`), bãi giữ xe máy (`img_030`), đám đông chờ xe buýt (`img_032`) | Quy tắc (`R-08`) + Cần hỏi ranh giới cụm | Các xe/người cận cảnh tách rõ được gán từng box; cụm chồng chéo dày đặc ở xa/sau gom vào `ignore_region` | Giảm bất đồng `bo_sot_thua` và `iou_thap` ở vùng chồng chéo dày |
| `img_022.jpg`, `img_025.jpg` | Mô tô gắn sidecar (`img_022`) và xe máy gắn quầy hàng bên hông (`img_025`) | Cần hỏi reviewer (Pha 2) | Gán `1` `motorcycle` ôm liền cả xe và thùng bên; người ngồi trong/trên gán `person` riêng | Cần chốt rõ xem sidecar/xe bán hàng gắn hông là 1 box `motorcycle` hay tách phần |
| `img_024.jpg` | Ô tô Pacer đen trong bảo tàng chở vỏ thân xe mini màu vàng trên giá nóc | Cần hỏi reviewer (Pha 2) | Gán `car` cho các ô tô thật trong phòng (Pacer đen, xe cam, xe kem...); **không gán** vỏ xe vàng mini trên nóc | Vỏ xe mini không bánh/không phải phương tiện hoàn chỉnh |
| `img_029.jpg` | Người đi xe máy và người đi xe đạp cùng vác thanh kim loại rất dài | Cần hỏi reviewer (hàng hóa dài) + Quy tắc | Gán `1` `motorcycle`, `2` `person` (người đi xe máy + người đi xe đạp), **không kéo box ôm theo thanh kim loại**; không gán xe đạp | Bao thanh kim loại sẽ làm box phình rộng chứa toàn nền đường, phá vỡ IoU |
| `img_033.jpg`, `img_038.jpg` | Ma-nơ-canh và phản chiếu xe van qua kính (`img_033.jpg`); tủ kính đầy xe mô hình và đồ chơi (`img_038.jpg`) | Loại theo quy tắc (`R-06`) | **Không gán** bất kỳ vật nào (cả hai là ảnh 0 đối tượng, vẫn giữ trong Task khi xuất COCO) | Ma-nơ-canh, phản chiếu qua kính và xe mô hình đồ chơi đều không phải người/xe thật |
| `img_040.jpg` | Người đội mũ bảo hiểm cận cảnh ở góc dưới, chỉ lộ một phần đầu xe máy/gương (`motorcycle` bị cắt rất sâu) | Cần hỏi reviewer (Pha 2) | Gán `person` (`truncated=yes`); nếu phần xe máy lộ rõ tay lái/yếm/2 gương đạt `>= 20 px` vẫn gán `motorcycle` (`truncated=yes`, `occluded_level=2`/`3`) + tick `can_review` | Vùng lộ xe máy rất nhỏ ở cạnh dưới dễ gây lệch giữa người gán |

---

## 5. QC và phiên bản

- **Ngưỡng QC áp dụng** (giữ nguyên mục `rules` trong `submission/taxonomy.json`):
  - Chiều cao box tối thiểu: `min_box_height_px = 20` px (nhỏ hơn không gán đơn lẻ, dùng `ignore_region` nếu là cụm nhiều vật).
  - Ngưỡng ghép cặp khi đo IAA: `match_iou = 0.5`.
  - Ngưỡng IoU chuẩn theo kích thước (`large_px = 50` px): box cao `>= 50 px` yêu cầu `IoU >= 0.85`; box cao `< 50 px` yêu cầu `IoU >= 0.70`.
  - Chỉ tiêu IAA nhóm (Pha 4): `iaa_iou >= 0.85` và `iaa_class >= 0.95` trên các cặp ghép được (đồng thời kiểm mắt người cho các ca bỏ sót/thừa và lỗi cả nhóm cùng sai).
  - Ngưỡng bàn giao lô (Pha 6): duyệt toàn lô `40/40` ảnh, `error_rate_img_max <= 0.02` (`<= 2%` số ảnh có lỗi, tức 0/40 ảnh lỗi mới đạt Release).
- **Quy trình quyết định ca mơ hồ:**
  - Khi gán độc lập ở Pha 3 gặp tình huống chưa rõ trong bản thống nhất v1.0: vẽ box và tick `can_review`, hoặc chưa vẽ được box thì ghi nhận `(image, annotator, region, note)` để tổng hợp vào `submission/qc/image_issues.csv` sau khi khóa nhãn.
  - Mọi quyết định sửa quy tắc ở Pha 2 và Pha 5 đều tăng phiên bản guideline (`v1.0`, `v1.1`...) và gắn mã quy tắc `R-01`, `R-02`... để truy vết.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| `v0` | Bản khởi đầu của giảng viên | Sơ sài, thiếu định nghĩa ranh giới lớp, ngưỡng thuộc tính và ca biên |
| `v0.1` (`105`) | Chuẩn hóa 5 phần: thêm ví dụ/phản ví dụ cho 6 lớp, định lượng % `occluded_level`, tách rõ quy tắc `R-01`–`R-08`, đề xuất lớp `bus`/`tuk_tuk`, tách ca loại theo quy tắc vs ca hỏi reviewer | Phát hiện qua 40 ảnh: `img_002`, `img_004`, `img_007`, `img_009`, `img_011`, `img_014`, `img_017`–`img_022`, `img_024`, `img_027`–`img_033`, `img_035`–`img_040` |
