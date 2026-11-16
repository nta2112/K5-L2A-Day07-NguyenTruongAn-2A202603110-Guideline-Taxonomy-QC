# Guideline gán nhãn — v1.0 — Bản thống nhất của nhóm (ĐÃ KHÓA)

- **Phiên bản:** v1.0 (Agreed — Khóa cho lượt gán độc lập Pha 3)
- **Ngày thống nhất:** 2026-11-16
- **Nhóm thực hiện:** Nhóm 5 thành viên (Mã: `101`, `102`, `103`, `104`, `105`)
- **Phạm vi áp dụng:** Toàn bộ 40 ảnh đường phố (`data/images/img_001.jpg` đến `img_040.jpg`)
- **Công cụ & Chuẩn xuất:** CVAT, Bounding Box 2D (Rectangle), định dạng COCO 1.0

---

## 1. Mục tiêu và phạm vi sử dụng

Nhãn bounding box 2D được chuẩn hóa dùng để huấn luyện và đánh giá mô hình thị giác máy tính phát hiện đối tượng (Object Detection) phục vụ **hệ thống xe tự lái (Autonomous Driving) và camera giám sát giao thông đô thị**.

### Rủi ro mô hình cần kiểm soát:
1. **Lỗi bỏ sót (False Negative - FN):** Bỏ sót người đi bộ, người điều khiển phương tiện hoặc xe cơ giới đang lưu thông là lỗi nguy hiểm nhất, dẫn tới nguy cơ va chạm trực tiếp trong thực tế.
2. **Lỗi gán thừa / Nhận diện nhầm vật giả (False Positive - FP):** Gán box cho tượng đá, ma-nơ-canh, hình in 2D trên biển quảng cáo (billboard) hoặc bóng phản chiếu qua gương sẽ khiến hệ thống tự hành hiểu nhầm là vật cản sống, gây hiện tượng **phanh gấp đột ngột (phantom braking)** cực kỳ nguy hiểm cho các xe phía sau.
3. **Lỗi ranh giới (Localization Error):** Box quá lỏng hoặc cố ý nới rộng bao trùm cả khoảng trống, hàng rời quá khổ sẽ làm sai lệch kích thước vật lý của đối tượng, gây sai số khi thuật toán ước tính khoảng cách và không gian an toàn.

---

## 2. Lớp và thuộc tính

### 2.1. Danh mục 6 lớp đối tượng (Khớp với `submission/taxonomy.json`)

Mỗi đối tượng hợp lệ bắt buộc thuộc về **đúng một lớp**:

| Lớp | Định nghĩa chuẩn | Ví dụ bao gồm | Phản ví dụ / Dễ nhầm lẫn |
|---|---|---|---|
| `car` | Ô tô con chở người từ 4 đến 9 chỗ, kết cấu khoang cabin và khoang hành lý liền một khối (sedan, hatchback, wagon, SUV, crossover, coupe, taxi). | Sedan đen `img_023`, taxi xanh `img_040`, SUV `img_012`, wagon `img_024`. | Xe bán tải pickup có thùng hở (`truck`), xe van thùng kín liền khối (`van`), xe tuk-tuk, xe đồ chơi `img_038`. |
| `truck` | Xe cơ giới chở hàng có cabin và kết cấu chở hàng riêng biệt (thùng lửng, thùng kín, thùng bạt, rơ-moóc, giàn chở xe) và xe bán tải (pickup truck). | Pickup `img_001`, `img_012`, `img_037`; xe tải lồng chở ô tô `img_002`; xe tải Isuzu `img_007`; xe tải nhẹ CMC `img_036`. | Xe SUV khoang liền (`car`), xe van giao hàng (`van`). Tượng xe bán tải bằng đá `img_027` **không gán**. |
| `van` | Xe ô tô thân hộp kín một khối liền mạch giữa khoang lái và khoang chở hàng/chở khách (minivan, panel van giao hàng). | Xe minivan bạc `img_005`, xe Toyota trắng `img_016`. | Xe buýt lớn chở khách công cộng (ngoài taxonomy); xe tải thùng rời cabin (`truck`). |
| `motorcycle` | Phương tiện cơ giới 2 bánh hoặc 3 bánh có động cơ (xe máy, xe tay ga, xe phân khối lớn, xe mô tô gắn thùng bên sidecar). | Scooter `img_016`, xe số `img_026`, xe phân khối lớn `img_011`, mô tô có sidecar `img_022`. | Xe đạp, xích lô (ngoài taxonomy). **Không bao gồm người lái trong box xe**. |
| `person` | Con người thật, còn sống, ở mọi tư thế: đi bộ, đứng, ngồi, nằm, hoặc người đang điều khiển / ngồi trên phương tiện. | Người đi bộ `img_014`, `img_034`; người bán hàng `img_003`; người ngồi lái xe máy `img_005`, `img_026`; người cosplay `img_011`. | Tượng danh nhân `img_017`, ma-nơ-canh `img_033`, ảnh người trên billboard `img_018`, búp bê đồ chơi `img_038`. |
| `ignore_region` | Vùng nhãn kỹ thuật hình chữ nhật bao quanh khu vực có đối tượng mục tiêu nhưng quá nhỏ ($< 20\text{ px}$) hoặc đám đông ken đặc không thể tách rời từng vật một cách tin cậy. | Dòng xe máy ở xa hun hút `img_021`, đám đông mờ xa `img_030`. | Không dùng để che giấu ca chưa phân loại được, không dùng đè lên đối tượng đơn lẻ rõ nét ở tiền cảnh. |

---

### 2.2. Biên bản giải quyết bất đồng giữa các bản Guideline cá nhân

Trong buổi họp Pha 2, nhóm đã đối chiếu các bản cá nhân (`101`, `102`, `103`, `104`, `105`) và thống nhất các điểm tranh luận:

1. **Phân loại xe bán tải (Pickup Truck - `img_001`, `img_012`, `img_037`):**
   - *Bất đồng:* Bản 103 xếp bán tải nhỏ vào `car`; bản 102, 104, 105 xếp vào `truck`.
   - *Quyết định nhóm:* Thống nhất xếp tất cả xe bán tải (kể cả có nắp chụp thùng Master 75 như `img_037`) vào **`truck`**.
   - *Lý do:* Xe bán tải có khung gầm chịu tải (chassis rời) và khoang thùng chở hàng tách biệt với cabin; hệ thống điều khiển tự động cần nhận biết tải trọng lớn hơn xe con chở khách thông thường.

2. **Phương tiện ngoài Taxonomy (Xe buýt, Tuk-tuk, Xích lô, Xe đạp):**
   - *Bất đồng:* Bản 103 đề xuất bổ sung thêm lớp `bus` và `tuk_tuk`. Bản 102, 104 giữ nguyên bộ 6 lớp và không gán thân xe ngoài danh mục.
   - *Quyết định nhóm:* **Giữ nguyên đúng 6 lớp của starter taxonomy**, không tự ý mở thêm lớp để đảm bảo tính đồng bộ của hệ thống đánh giá tự động.
   - *Quy tắc thống nhất:* 
     - Thân xe buýt, tuk-tuk (`img_004`, `img_035`), xích lô (`img_006`, `img_023`), xe đạp: **Bỏ qua không gán box phương tiện**.
     - Người lái và hành khách trên xe hở (tuk-tuk, xích lô, xe đạp): **Bắt buộc gán box `person` riêng biệt** nếu nhìn thấy rõ và đạt kích thước $\ge 20\text{ px}$.

3. **Hàng hóa trên xe máy / xe tải (`img_012`, `img_028`, `img_029`, `img_035`):**
   - *Bất đồng:* Có ý kiến muốn nới box xe trùm cả hàng hóa cồng kềnh (kiện hàng lớn `img_035`, ống sắt dài `img_029`, mô hình ngựa `img_028`).
   - *Quyết định nhóm:* **Không nới box theo hàng rời / hàng buộc thêm**. Box xe chỉ ôm khít cấu trúc cơ giới vật lý của phương tiện. Hàng hóa nằm trong lòng thùng xe (`img_012`) thì nằm trong box xe; hàng nhô ra ngoài bị loại khỏi box xe. Nếu hàng rời che khuất thân xe hoặc người lái, ghi nhận vào thuộc tính `occluded_level`.

4. **Tách người và xe máy (`img_005`, `img_026`):**
   - *Đồng thuận 100%:*: Luôn tách thành các box độc lập: 1 xe chở 1 người $\rightarrow$ `1 motorcycle + 1 person`; 1 xe chở 2 người $\rightarrow$ `1 motorcycle + 2 person`.

5. **Loại trừ vật thể phi sinh học / ảo:**
   - *Đồng thuận 100%:*: Tuyệt đối không gán tượng `img_017`, `img_027`; ma-nơ-canh `img_033`; ảnh billboard `img_018`; phản chiếu gương `img_020`; đồ chơi `img_038`.

---

### 2.3. Bảng thuộc tính định lượng đo được (Attributes)

Mỗi box mục tiêu bắt buộc có đủ 3 thuộc tính:

| Thuộc tính | Giá trị | Ngưỡng định lượng cụ thể |
|---|---|---|
| `occluded_level` | `0` | Không bị che hoặc diện tích bị che khuất $\le 10\%$. |
| | `1` | Bị vật khác che khuất trên $10\%$ đến $\le 35\%$ diện tích ước lượng. |
| | `2` | Bị vật khác che khuất trên $35\%$ đến $\le 75\%$ diện tích ước lượng. |
| | `3` | Bị che khuất trên $75\%$, nhưng phần còn lại vẫn đủ bằng chứng nhận diện chắc chắn. |
| `truncated` | `no` | Vật thể nằm trọn vẹn $100\%$ bên trong khung ảnh. |
| | `yes` | Bất kỳ cạnh nào của vật thể bị cắt đứt bởi 4 mép viền của ảnh. |
| `can_review` | `false` | Tự tin với quyết định gán nhãn. |
| | `true` | Đánh dấu khi đối tượng có độ nghi ngờ cao (ca biên, nhòe nặng, tranh chấp lớp) để đưa vào Calibration. |

*Lưu ý kỹ thuật:*
- Tỷ lệ che khuất được ước lượng dựa trên **diện tích hình dáng thực** của vật thể, không lấy diện tích hình chữ nhật của box.
- Không coi bóng đổ, quầng sáng hoặc nhòe chuyển động là vật che khuất.
- Đối với `ignore_region`: Mặc định đặt `occluded_level=0`, `truncated=no`, `can_review=false`.

---

## 3. Bộ quy tắc vẽ Bounding Box (Rule IDs)

- **R-01 (Visible Bound - Ôm sát phần nhìn thấy):** Bounding box phải là hình chữ nhật song song trục ảnh, tiếp xúc khít 4 điểm cực trị của phần vật thể thực sự nhìn thấy. Không phỏng đoán kéo dài box qua phần bị che khuất hoàn toàn. Sai số mép tối đa cho phép $\le 2$ pixel.
- **R-02 (Tách người và phương tiện):**
  - Người ngồi trên xe máy, xe đạp, xích lô, tuk-tuk: **Tách riêng box `person` và box phương tiện**.
  - Box `person` bao toàn bộ cơ thể nhìn thấy (đầu, mũ bảo hiểm, tay, chân).
  - Box `motorcycle` bao toàn bộ khung xe, bánh xe, tay lái, gương chiếu hậu (không bao gồm cơ thể người).
  - Hai box được phép chồng lấn tự nhiên.
- **R-03 (Xe chuyên dụng chở xe - Car Carrier `img_002`):**
  - Xe tải lồng gán 1 box `truck` bao cabin và giàn giàn chở.
  - Từng chiếc ô tô con chở trên giàn: Gán riêng từng box `car` (kèm `occluded_level` tương ứng) nếu chiều cao nhìn thấy $h \ge 20\text{ px}$.
- **R-04 (Loại trừ Tượng và Ma-nơ-canh `img_017`, `img_027`, `img_033`):**
  - Tuyệt đối không gán `person` cho tượng điêu khắc hoặc ma-nơ-canh trưng bày tủ kính.
  - Tuyệt đối không gán `car`/`truck` cho tượng xe tạc bằng đá.
- **R-05 (Loại trừ Hình in 2D, Màn hình, Biển quảng cáo `img_018`):**
  - Hình ảnh người/xe in trên poster, billboard, pano quảng cáo: Không gán. Các xe thật lưu thông phía dưới biển quảng cáo vẫn gán bình thường.
- **R-06 (Loại trừ Phản chiếu, Xuyên kính & Xe chủ `img_013`, `img_020`):**
  - Ảnh ảo phản chiếu qua gương chiếu hậu, kính tòa nhà, mặt nước: Không gán.
  - Bộ phận của xe chủ mang camera (nắp ca-pô, viền gương `img_020`): Không gán.
  - Người/xe thật nhìn xuyên qua kính trong suốt (`img_009`): Vẫn gán nếu nhận diện rõ.
- **R-07 (Loại trừ Đồ chơi & Mô hình thu nhỏ `img_038`):**
  - Xe đồ chơi, búp bê trong tủ kính cửa hàng: Không gán.
- **R-08 (Kết cấu xe vs Hàng hóa rời `img_012`, `img_028`, `img_029`):**
  - Kết cấu gắn liền xe (thùng hàng, bạt xe `img_036`, nắp thùng `img_037`, thùng phụ sidecar `img_022`): Nằm trong box xe.
  - Hàng rời quá khổ, vật chở cồng kềnh buộc thêm: Không nới box xe theo hàng.
- **R-09 (Ngưỡng kích thước tối thiểu - Min Resolution Threshold):**
  - Kích thước đo chiều cao trên ảnh gốc $h < 20\text{ px}$: Bỏ qua không gán box đơn lẻ.
  - Kích thước $h \ge 20\text{ px}$: Bắt buộc gán box riêng nếu nhận dạng được.
- **R-10 (Xử lý Đám đông & Vùng bỏ qua `ignore_region`):**
  - Ở tiền cảnh: Tách từng người và xe nếu còn phân biệt được ranh giới.
  - Ở hậu cảnh xa xôi nơi các đối tượng tạo thành cụm mờ đặc không thể tách: Khoanh 1 box `ignore_region` bao quanh cụm đó.
- **R-11 (Xử lý đối tượng ngoài Taxonomy):**
  - Xe buýt, xe đạp, xích lô, xe tuk-tuk: Bỏ qua không gán thân xe. Người ngồi trên đó vẫn gán `person`.
- **R-12 (Quy tắc Review & Ghi nhật ký):**
  - Khi phân vân hoặc đối tượng ở ranh giới chưa rõ ràng: Vẽ box tốt nhất theo cảm nhận + tick `can_review=true`.
  - Nếu đối tượng quá mờ/không thể vẽ box: Ghi nhận vào danh sách issue theo cấu trúc `image,annotator,region,note` để đối chiếu sau này.

---

## 4. Bảng ca khó tiêu biểu trong 40 ảnh đã thống nhất

| Mã ảnh | Tình huống đặc trưng | Rule ID | Quyết định nhóm v1.0 | Lý do kỹ thuật |
|---|---|---|---|---|
| `img_001.jpg` | Pickup trước quán ăn; người phụ nữ đứng cạnh; khách ngồi sau. | R-01, R-02 | Gán `truck` cho pickup. Tách riêng từng box `person` cho người đứng và khách ngồi. | Pickup có thùng chở hàng riêng $\rightarrow$ `truck`. |
| `img_002.jpg` | Xe tải lồng hai tầng chở loạt ô tô con. | R-02, R-03 | 1 box `truck` cho xe tải lồng. Từng xe trên giàn gán box `car` riêng nếu $h \ge 20\text{ px}$. | Giúp mô hình học cấu trúc xe vận chuyển và phát hiện xe con trên giá. |
| `img_004.jpg` | Hai xe tuk-tuk màu xanh đỗ ban đêm. | R-02, R-11 | Bỏ qua thân xe tuk-tuk (ngoài taxonomy). Gán box `person` cho người ngồi trong cabin. | Giữ taxonomy 6 lớp, không ép tuk-tuk vào `car`. |
| `img_005.jpg` | Dòng xe máy chen chúc ở tiền cảnh; xe van bạc phía sau. | R-01, R-02 | Tách riêng từng `person` và `motorcycle`. Xe van bạc gán `van`. Xe con bên trái gán `car`. | Đảm bảo tính toán đúng số lượng đối tượng dễ bị tổn thương. |
| `img_006.jpg` | Xe máy chở 2 người bên trái; người đi xe đạp bên phải. | R-02, R-11 | Scooter bên trái: 1 `motorcycle` + 2 `person`. Người đi xe đạp: gán `person`, bỏ xe đạp. | Phù hợp quy tắc R-02 và R-11. |
| `img_007.jpg` | Hai xe tải lớn trong kho hàng ban đêm. | R-01, R-08 | Gán 2 box `truck` bao toàn bộ thân và thùng xe. Xe bên phải đặt `truncated=yes`. | Thùng xe là kết cấu chịu lực của xe tải. |
| `img_008.jpg` | Dãy xe máy đỗ chéo ở phố chợ đêm; nhiều người qua lại. | R-02, R-10 | Gán từng xe máy đỗ và người nhìn rõ. Vùng xa mờ tối không phân biệt được khoanh `ignore_region`. | Xe đứng yên vẫn là mục tiêu lưu thông cần nhận diện. |
| `img_009.jpg` | Xe buýt nhỏ thân dài giữa phố; người nhìn qua cửa kính. | R-06, R-11 | Bỏ qua thân xe buýt. Người ngồi nhìn xuyên qua kính chỉ gán `person` nếu nhìn rõ mặt/thân và $h \ge 20\text{ px}$. | Xe buýt ngoài taxonomy; không ép vào `van`. |
| `img_011.jpg` | Nhân vật cosplay Predator đứng cạnh dàn mô tô độ. | R-01, R-07 | Gán `person` cho nhân vật Predator (có người bên trong). Dàn mô tô gán từng `motorcycle`. | Thực thể sống đang cử động trên đường phải được bảo vệ. |
| `img_012.jpg` | Pickup chở sọt hàng ở thùng sau; SUV bạc bên trái. | R-01, R-08 | Pickup gán `truck` (box bao sọt hàng trong lòng thùng). SUV bạc gán `car`. | Hàng nằm gọn trong lòng thùng xe tính vào box phương tiện. |
| `img_017.jpg` | Tượng Duke of Wellington cưỡi ngựa trên bệ cao. | R-04 | **Không gán bất kỳ box nào**. | Tượng đá/đồng tĩnh vật, tránh gây phantom braking. |
| `img_018.jpg` | Biển quảng cáo lớn có ảnh chân dung; dãy ô tô phía dưới. | R-05, R-09 | Bỏ qua ảnh in trên biển quảng cáo. Dãy ô tô thật đỗ phía dưới gán từng box `car`. | Phân biệt rạch ròi giữa vật ảo 2D và vật thể thật 3D. |
| `img_020.jpg` | Cầu vồng và xe phản chiếu trong gương chiếu hậu ô tô. | R-06 | **Không gán** ảnh ảo phản chiếu trong gương. Bỏ qua viền gương xe chủ. | Loại trừ xe chủ mang camera và ảnh ảo. |
| `img_022.jpg` | Xe mô tô 3 bánh có thùng phụ bên hông (sidecar). | R-02, R-08 | Gán 1 box `motorcycle` bao toàn bộ cả mô tô và thùng sidecar. Người lái gán `person`. | Sidecar là bộ phận gắn cố định vào xe máy. |
| `img_026.jpg` | Đôi nam nữ đi xe máy đỏ trong hầm chui; chân xe bị cắt mép. | R-01, R-02 | Gán 1 `motorcycle` (`truncated=yes`) và 2 box `person` riêng biệt cho 2 người. | Quy tắc chuẩn khi 1 phương tiện chở nhiều người. |
| `img_027.jpg` | Tượng xe bán tải điêu khắc bằng đá trong nghĩa trang. | R-04 | **Không gán**. | Tượng điêu khắc bằng đá không phải phương tiện giao thông. |
| `img_028.jpg` | Xe bán tải chở mô hình ngựa lớn phía sau. | R-04, R-08 | Gán `truck` cho xe bán tải. Mô hình ngựa không thuộc taxonomy, không nới box xe trùm lên ngựa. | Ngựa mô hình buộc rời không phải kết cấu xe. |
| `img_029.jpg` | Xe máy chở bó thanh sắt dài nhô ra phía sau. | R-02, R-08 | Gán `motorcycle` ôm thân xe; không kéo box theo thanh sắt nhô ra. Người lái gán `person`. | Tránh làm méo mó đặc trưng hình học của xe máy. |
| `img_033.jpg` | Ma-nơ-canh trưng bày trong tủ kính tiệm thời trang. | R-04 | **Không gán**. | Ma-nơ-canh sau tủ kính không phải người tham gia giao thông. |
| `img_035.jpg` | Xe Auto Rickshaw chở bao tải hàng khổng lồ chèn ép. | R-08, R-11 | Bỏ qua thân xe rickshaw và kiện hàng. Người lái/khách nhìn thấy rõ gán `person`. | Nhất quán với quy tắc tuk-tuk ngoài taxonomy. |
| `img_036.jpg` | Xe tải nhẹ CMC có thùng bạt che chắn. | R-01, R-08 | Gán 1 box `truck` bao toàn bộ cabin và thùng bạt. | Thùng bạt là kết cấu chở hàng chuyên dụng của xe tải. |
| `img_037.jpg` | Xe bán tải lắp nắp thùng chụp cứng Master 75. | R-01, R-08 | Gán `truck` (box ôm cả phần nắp thùng chụp phía sau). | Nắp chụp mui cứng gắn liền khung thùng xe bán tải. |
| `img_038.jpg` | Tủ kính bày xe ô tô đồ chơi và búp bê thu nhỏ. | R-07 | **Không gán**. | Đồ chơi trong tủ kính không thuộc đối tượng giao thông thực tế. |

---

## 5. Tiêu chuẩn QC và Lịch sử phiên bản

### 5.1. Tiêu chuẩn QC áp dụng (Theo quy định chặt chẽ của Lab)
- **Độ trùng khớp hình học (IoU Threshold):**
  - Đối tượng lớn ($h \ge 50\text{ px}$): $IoU \ge 0.85$.
  - Đối tượng nhỏ ($20\text{ px} \le h < 50\text{ px}$): $IoU \ge 0.70$.
  - Ngưỡng ghép cặp 1-1 giữa hai annotator: $IoU_{\text{match}} \ge 0.50$.
- **Độ đồng thuận phân loại (IAA Class Agreement):** $\ge 95.0\%$ trên tổng số cặp box ghép được.
- **Tỉ lệ lỗi tối đa cho phép bàn giao (Max Error Rate):** $\le 2.0\%$ trên số ảnh đã duyệt của cả lô ($0/40$ ảnh lỗi mới đạt Release).

### 5.2. Lịch sử thay đổi

| Phiên bản | Ngày | Người duyệt | Nội dung thay đổi |
|---|---|---|---|
| `v0` | 2026-11-16 | Giảng viên | Bản khởi đầu sơ sài, thiếu quy tắc ca biên và định nghĩa thuộc tính. |
| `v1.0` | 2026-11-16 | Cả nhóm (101, 102, 103, 104, 105) | Bản thống nhất chính thức: Giải quyết bất đồng bán tải (`truck`), giữ nguyên 6 lớp (xử lý buýt/tuk-tuk theo R-11), chốt quy tắc hàng rời (R-08), tách người và xe máy (R-02), chuẩn hóa 4 mức che khuất đo được. **Khóa bản để gán độc lập Pha 3**. |
