# Guideline gán nhãn cá nhân — Pha 1

> Bản đề xuất độc lập dựa trên guideline v0 và việc rà soát 40 ảnh trong `data/images/`.
> Đây chưa phải guideline v1.0 đã thống nhất; giữ nguyên bản này sau buổi họp nhóm.

- Phiên bản: Bản cá nhân Pha 1 (tham chiếu v0)
- Ngày: 16/11/2025
- Họ và tên: Nguyễn Hoàng Việt Anh *(chỉ điền nếu coach yêu cầu; hướng dẫn nộp bài yêu cầu không đưa họ tên vào gói nộp)*
- Mã thành viên (3 chữ số): 101
- Tên file mẫu hiện tại: `guideline_101.md`
- Đổi `xxx` thành mã thành viên ba chữ số trước khi nộp, ví dụ `guideline_101.md`.
  Không dùng họ tên trong tên file.

## 1. Mục tiêu

Gán bounding box trên ảnh đường phố cho các lớp có trong taxonomy v0: `car`, `truck`,
`van`, `motorcycle`, `person` và `ignore_region`. Mục tiêu là phát hiện phương tiện và
người thật nhìn thấy trong ảnh; không suy đoán vật thể bị che hoàn toàn hoặc không đủ
bằng chứng để xác định.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa dùng trong bản cá nhân | Ví dụ | Phản ví dụ / dễ nhầm |
|---|---|---|---|
| `car` | Ô tô chở khách dạng sedan, hatchback, wagon, SUV hoặc ô tô thật có kiểu dáng đặc biệt. | Xe con trong `img_002`, `img_015`, `img_024`, `img_040`; xe diễu hành cỡ thật trong `img_019`. | Xe tải có thùng hàng riêng; xe mô hình trong `img_038`. |
| `truck` | Xe có khoang/thùng chở hàng riêng, gồm xe tải, xe chở ô tô và pickup có thùng hàng. | Xe chở xe trong `img_002`; pickup trong `img_001`, `img_012`, `img_028`, `img_037`. | Xe con đặt trên xe chở xe vẫn là một vật riêng; không gộp vào box xe tải. |
| `van` | Xe thân kín cỡ nhỏ, kiểu van/minibus nhỏ; dùng khi hình dáng xe phù hợp định nghĩa này. | Xe minibus nhỏ trong `img_009`; xe thân kín cỡ nhỏ trong `img_016`. | Xe buýt cỡ lớn không có lớp riêng trong taxonomy v0; không tự ép vào `van`. |
| `motorcycle` | Phương tiện gốc xe máy có động cơ xăng hoặc điện: xe máy, mô tô, scooter, moped và tuk-tuk/auto-rickshaw chở khách kiểu xe máy cải tiến. Tổ hợp xe máy gắn sidecar vẫn là một `motorcycle`. | Xe máy trong `img_005`, `img_008`, `img_030`; xe máy gắn sidecar trong `img_022`; tuk-tuk trong `img_004`, `img_016`, `img_035`. | Xe đạp (kể cả xe đạp trợ lực nếu vẫn là xe đạp), xe buýt, xe mô hình hoặc hình phản chiếu. |
| `person` | Người thật nhìn thấy trong cảnh, gồm người đi bộ, người lái và hành khách trên xe. Mỗi người là một box riêng nếu nhận diện được và cao ít nhất 20 px. | Người đi bộ ở `img_014`, người ngồi xe máy ở `img_005`, người mặc hóa trang ở `img_011`. | Tượng, mannequin, hình in trên billboard, búp bê hoặc nhân vật đồ chơi. |
| `ignore_region` | Vùng có vật ngoài taxonomy hoặc cụm vật không thể gán riêng đáng tin cậy; đây là vùng bỏ qua, không phải lớp của vật thể. | Vùng quanh xe buýt không có lớp phù hợp hoặc cụm xe quá nhỏ/không tách được. | Không dùng thay cho box của một vật thuộc lớp đã biết và nhìn thấy rõ. |

**Ca cần nhóm chốt:** taxonomy v0 chưa nêu riêng `bus`, `bicycle` hay `cyclo`. Quyết định
cá nhân là xếp tuk-tuk/auto-rickshaw chở khách kiểu xe máy cải tiến vào `motorcycle`; mang
quyết định này ra thống nhất ở Pha 2. Không xếp xe buýt hoặc xe đạp/cyclo vào `motorcycle`.
Xe ba bánh có thùng hàng riêng, nhận diện rõ là xe chở hàng, có thể thuộc `truck`. Dùng
`ignore_region` cho phương tiện ngoài taxonomy nếu biểu diễn được trong CVAT; nếu không,
ghi ảnh/vùng/lý do để nhóm xử lý. Không tự thêm lớp mới vào CVAT hoặc `taxonomy.json`.

### Phân biệt phương tiện có động cơ

| Dấu hiệu phương tiện | Lớp cá nhân đề xuất | Cách phân biệt |
|---|---|---|
| Xe hai bánh tự hành bằng động cơ xăng hoặc điện: motorcycle, scooter, moped | `motorcycle` | Hai bánh là cấu hình chính; người lái ngồi/đứng điều khiển xe. Xe máy gắn sidecar vẫn là một phương tiện và một box `motorcycle`. |
| Xe đạp thường hoặc xe đạp trợ lực có cấu tạo/chức năng chính là xe đạp | Không có lớp trong v0 | Không gán `motorcycle`; xử lý phần xe ngoài taxonomy theo quy tắc `ignore_region`/review ở mục 3.3. Nếu không rõ là xe đạp trợ lực hay moped, hỏi reviewer. |
| Tuk-tuk/auto-rickshaw chở khách kiểu xe máy cải tiến | `motorcycle` | Theo quyết định cá nhân, xem đây là xe máy ba bánh chở khách; vẽ một box `motorcycle` cho toàn bộ phương tiện. Đưa quy tắc ra nhóm chốt ở Pha 2. |
| Xe ba bánh có thùng hàng riêng, nhận diện rõ là xe chở hàng | `truck` (nếu đáp ứng định nghĩa) | Dựa vào cấu tạo thùng hàng, không chỉ dựa vào số bánh hoặc việc có động cơ. Nếu không rõ mục đích/cấu tạo, cần reviewer. |
| Ô tô chở khách nhiều bánh | `car` | Không xếp thành `motorcycle` dù có động cơ. |
| Xe thân kín cỡ nhỏ | `van` | Dựa vào kiểu thân xe theo định nghĩa `van`; xe buýt lớn chưa có lớp riêng trong v0. |
| Xe tải/pickup có khoang hoặc thùng chở hàng riêng | `truck` | Gán phương tiện chở hàng; hàng hóa trên xe không phải box riêng. |

Không dùng riêng tiêu chí “có động cơ” để quyết định `motorcycle`: trước hết nhận diện
cấu hình và kiểu phương tiện, sau đó chọn lớp taxonomy phù hợp. Nếu không chắc xe là
moped/motorcycle hay xe đạp trợ lực, tick `can_review` nếu đã tạo được box hợp lệ hoặc ghi
ảnh/vùng/lý do để reviewer quyết định.

### Thuộc tính

| Thuộc tính | Giá trị | Quy tắc áp dụng |
|---|---|---|
| `occluded_level` | `0` | Không bị che hoặc chỉ bị che không đáng kể (ước lượng ≤15% diện tích vật). |
| `occluded_level` | `1` | Bị che trên 15% đến 30%; phần lớn hình dạng vẫn nhìn rõ. |
| `occluded_level` | `2` | Bị che trên 35% đến 65%; vẫn xác định được vật và phần nhìn thấy. |
| `occluded_level` | `3` | Bị che trên 65% hoặc rất khó xác định hình dạng, nhưng vẫn đủ bằng chứng để gán box. Nếu không đủ bằng chứng để nhận diện vật, không đoán box; ghi ca để review. |
| `truncated` | `no` | Vật không bị cắt bởi biên ảnh. Bị che bởi vật khác không phải là truncated. |
| `truncated` | `yes` | Vật bị biên ảnh cắt mất một phần. Vẽ đến sát biên ảnh, không ước lượng phần nằm ngoài ảnh. |
| `can_review` | tick / không | Tick khi chưa chắc lớp, ranh giới box hoặc mức che; ghi vấn đề cấp ảnh nếu không thể tạo box hợp lệ. |

Ước lượng phần trăm che khuất theo toàn bộ vật nếu có thể nhận ra hình dáng; không cần
đo chính xác từng pixel. Nếu không thể phân biệt che khuất với vật bị cắt do ảnh, ưu tiên
đặt `can_review`.

## 3. Quy tắc vẽ box

### 3.1 Chọn vật và phân biệt các vật

1. Tạo một box cho mỗi người/phương tiện thuộc taxonomy, đủ nhìn thấy và có chiều cao box
   **từ 20 px trở lên**. Vật thấp hơn 20 px không gán box đối tượng; nếu nằm trong cụm
   khiến không thể gán riêng, đánh dấu vùng phù hợp bằng `ignore_region` và ghi ca cần xem xét.
2. Box ôm sát phần nhìn thấy của đúng vật; không mở rộng qua người/vật liền kề và không
   vẽ phần bị che theo suy đoán.
3. Người và xe là hai đối tượng riêng. Với người lái/hành khách, vẽ box người trên phần cơ
   thể nhìn thấy và box phương tiện riêng; không gộp người vào box xe. Nón bảo hiểm, mũ,
   quần áo hoặc hàng hóa đang mang không tạo box riêng.
4. Với xe chở xe, vẽ box xe tải chở và từng ô tô trên sàn chở nếu mỗi xe nhìn thấy đủ rõ
   và đạt ngưỡng 20 px. Xe chở xe không thay thế các xe được chở (`img_002`, `img_024`).
5. Xe máy có sidecar là một phương tiện, một box `motorcycle` bao cả cụm xe gắn liền
   (`img_022`). Hàng hóa trên xe không phải phương tiện; không vẽ box cho hàng hóa.
6. Xe thật đang chạy/đỗ vẫn được gán dù ở bãi xe, trong kho, trên sân khấu hoặc có kiểu dáng
   lạ. Xe đồ chơi/mô hình, xe trên billboard, tượng xe và hình xe trong phản chiếu không gán
   (`img_017`, `img_018`, `img_020`, `img_027`, `img_038`).
7. Chỉ gán `person` cho người thật. Không gán tượng người, mannequin, búp bê, hình in hoặc
   hình phản chiếu như người thật (`img_017`, `img_018`, `img_033`, `img_038`).

### 3.2 Quy tắc cụ thể cho người đi xe máy

- Một xe máy và mỗi người ngồi trên xe là các đối tượng riêng. Với một xe có một người,
  dự kiến có **hai box** nếu cả xe và người đều nhìn thấy đủ rõ: một box `motorcycle` ôm
  phần xe nhìn thấy (không bao gồm người), và một box `person` ôm phần cơ thể nhìn thấy
  (không bao gồm xe).
- Nếu có người lái và hành khách, mỗi người có một box `person`; xe vẫn chỉ có một box
  `motorcycle`. Không gộp nhiều người vào một box và không vẽ box riêng cho mũ bảo hiểm.
- Khi người hoặc xe bị che, vẽ từng box quanh phần nhìn thấy của đối tượng tương ứng; không
  dự đoán phần khuất. Dùng `occluded_level` theo mức che của chính đối tượng đó.
- **Áp dụng thử:** ở `img_005`, tách xe máy tiền cảnh khỏi người lái; ở `img_026`, áp dụng
  lại cùng quy tắc cho xe máy chở hai người: một box xe máy và hai box người nếu từng người
  đủ nhận diện/đạt ngưỡng chiều cao. Cách này cho cùng một cách đếm dù số người trên xe đổi.
- Chỉ áp dụng quy tắc này cho motorcycle/scooter/moped đúng định nghĩa. Không dùng cho xe
  đạp; tuk-tuk/auto-rickshaw chở khách được tính là một `motorcycle` theo quyết định cá nhân,
  nhưng không dùng quy tắc đếm người/xe máy hai bánh để gộp người vào box phương tiện.

### 3.3 Quy tắc cho xe đạp và cyclo

- Xe đạp đạp bằng bàn đạp không thuộc lớp `motorcycle`; taxonomy v0 không có lớp `bicycle`.
  Không gán xe đạp vào lớp xe khác và không tự tạo lớp mới trong CVAT/taxonomy.
- Người đang đạp xe vẫn là một đối tượng `person`: vẽ một box ôm phần cơ thể nhìn thấy,
  riêng với xe. Không gộp người và xe thành một box. Nếu người thấp hơn 20 px, áp dụng
  ngưỡng kích thước của guideline.
- Phần xe đạp ngoài taxonomy: nếu CVAT cho phép, đánh dấu `ignore_region` ôm vùng xe đạp
  nhưng tránh che phần người cần gán; nếu không thể đánh dấu vùng chính xác, ghi ảnh, vị trí
  và lý do vào danh sách vấn đề để reviewer/nhóm quyết định. Không tự suy diễn rằng bỏ qua
  xe là đã được nhóm chấp thuận.
- Cyclo/xe xích lô/xe ba bánh đạp bằng sức người cũng không phải `motorcycle`. Đánh dấu
  vùng xe là `ignore_region` nếu biểu diễn được và ghi vấn đề để nhóm chốt; người đạp hoặc
  hành khách là `person` riêng nếu nhìn thấy đủ rõ.
- Ví dụ: `img_006` có người đi xe đạp giữa dòng xe; `img_003` có người và xe đạp chở hàng;
  `img_023`, `img_034`, `img_039` có cyclo/xe đạp và người đi cùng. Áp dụng cùng quy tắc
  cho người và ghi nhận phần xe ngoài taxonomy ở từng ảnh phù hợp.

### 3.4 Che khuất, ảnh mờ và biên ảnh

- Khi nhiều xe máy/người chen lấp nhau (`img_005`, `img_008`, `img_030`, `img_031`), tách
  từng vật còn nhận diện được; mỗi box chỉ bao phần nhìn thấy của vật đó. Không vẽ một box
  bao cả đám đông thay cho các vật riêng.
- Nếu một vật bị che gần hết hoặc nhòe chuyển động nhưng vẫn nhận diện được và thấy đủ
  đường biên để đặt box, gán box phần nhìn thấy, điền `occluded_level` và tick `can_review`
  nếu ranh giới còn không chắc (`img_014`).
- Nếu chỉ có các mảnh không đủ để xác định vật hoặc không thể đặt box đáng tin cậy, không
  tự nối các mảnh thành box; ghi ảnh/vùng/lý do để nhóm review.
- Vật bị cắt ở mép ảnh: box chạm biên ảnh, `truncated=yes`. Không đánh dấu truncated chỉ
  vì vật bị một phương tiện khác che (`img_015`, `img_040`).
- Vật phản chiếu trong gương/kính hoặc hình phản chiếu trên bề mặt không phải một vật thể
  trực tiếp trong cảnh để gán box. Không gán xe/người phản chiếu trong `img_020` hoặc hình
  phản chiếu trên kính cửa hàng ở `img_033`.

### 3.5 Đám đông và vùng bỏ qua

- Cố gắng gán riêng các đối tượng nhìn rõ; không bỏ qua cả ảnh chỉ vì ảnh đông.
- Chỉ dùng `ignore_region` để khoanh vùng không thể gán từng vật nhất quán (ví dụ vật ngoài
  taxonomy hoặc các vật quá nhỏ/dính chồng không thể tách). Không dùng vùng này để che đi
  một vật thuộc lớp hợp lệ, nhìn rõ, cao ≥20 px.
- Không cần tạo `ignore_region` cho toàn bộ nền trống hoặc mọi khu vực không có vật mục tiêu.
- Ghi lại ảnh, khu vực và lý do mỗi khi dùng `ignore_region` do bất định để nhóm kiểm tra.

## 4. Ca khó quan sát trong bộ 40 ảnh

| Ảnh | Tình huống | Rule ID | Quyết định ban đầu | Phân loại ca |
|---|---|---|---|---|
| `img_005`; kiểm tra lại trên `img_026` | Một người hoặc nhiều người ngồi trên xe máy | R-01 | Tách riêng xe và từng người: `img_005` dự kiến một box xe + một box người cho xe/người tiền cảnh; `img_026` dự kiến một box xe + hai box người nếu mỗi người đạt ngưỡng. Box chỉ ôm phần nhìn thấy. | Quy tắc áp dụng; tick `can_review` nếu ranh giới/nhận diện không chắc. |
| `img_002` | Xe chở nhiều ô tô | R-02 | Một box `truck` cho xe chở, thêm box riêng cho mỗi ô tô nhìn thấy đủ rõ và cao ≥20 px; không gộp ô tô vào xe tải. | Gán theo lớp; cần reviewer nếu xe nào không thể tách hoặc xác định. |
| `img_005`, `img_008`, `img_030` | Người/xe máy bị che bởi phương tiện hoặc người khác | R-03 | Gán từng đối tượng nhận diện được theo phần nhìn thấy, đánh mức che riêng cho từng box; không đoán phần bị che. | Gán nếu đủ bằng chứng; reviewer nếu không thể xác định vật hoặc ranh giới box. |
| `img_025`, `img_032` | Người/phương tiện ở xa, có thể thấp hơn ngưỡng 20 px | R-04 | Đo chiều cao box trên ảnh gốc. Nếu <20 px thì không tạo box đối tượng; không coi đây là lỗi bỏ sót. Nếu ≥20 px và nhận diện được thì gán bình thường. | Loại theo quy tắc khi <20 px; reviewer nếu sát ngưỡng/không rõ vật hoặc không chắc cách biểu diễn vùng bỏ qua. |
| `img_017`, `img_027` | Tượng người, ngựa và pickup | R-05 | Không gán tượng là `person` hoặc phương tiện thật; không vẽ box đối tượng cho các tượng. | Loại theo quy tắc (vật không phải người/phương tiện thật). |
| `img_020`, `img_033` | Phương tiện/người phản chiếu trên gương hoặc kính | R-06 | Chỉ gán vật thật trong cảnh; không gán hình phản chiếu thành đối tượng riêng. | Loại theo quy tắc (ảnh phản chiếu, không phải vật thể trực tiếp). |
| `img_032` | Xe buýt cỡ lớn ở trạm | R-07 | Taxonomy v0 không có lớp bus; không tự gán thành `van` hoặc `truck`. Ghi lại để reviewer/nhóm chốt cách xử lý phù hợp schema. | Cần hỏi reviewer/nhóm (thiếu lớp phù hợp), không tự loại im lặng. |
| `img_003`, `img_006`, `img_023`, `img_034`, `img_039` | Xe đạp/cyclo và người đi cùng | R-08 | Xe đạp/cyclo không có lớp trong v0, không gán thành `motorcycle`; đánh dấu phần xe bằng `ignore_region` nếu có thể, còn người đạp/ngồi xe là `person` riêng nếu đủ rõ. Ghi ảnh/vùng/lý do để nhóm chốt. | Xe ngoài taxonomy cần reviewer/nhóm; người thật được gán theo quy tắc `person`. |
| `img_004`, `img_016`, `img_035` | Tuk-tuk/auto-rickshaw chở khách ba bánh có động cơ | R-09 | Xếp `motorcycle` theo quyết định cá nhân: xem đây là xe máy cải tiến ba bánh. Vẽ một box cho toàn bộ tuk-tuk; gán người lái/hành khách thành box `person` riêng nếu đủ rõ. Thống nhất quy tắc này với nhóm ở Pha 2. | Đề xuất cá nhân; cần thống nhất nhóm trước khi khóa v1.0. |
| `img_014` | Người/phương tiện bị nhòe chuyển động | R-10 | Chỉ gán khi nhận diện được vật và có đủ bằng chứng đặt box; nếu không, ghi ảnh/vùng/lý do, không suy đoán. | Cần reviewer khi không đủ bằng chứng hoặc box không đáng tin cậy. |
| `img_018`, `img_033`, `img_038` | Người trong billboard, mannequin, xe đồ chơi/mô hình | R-11 | Không gán hình in, mannequin hoặc đồ chơi thành người/phương tiện thật. | Loại theo quy tắc (không phải đối tượng thật thuộc lớp). |
| `img_001`, `img_012`, `img_028`, `img_036`, `img_037` | Pickup/xe có thùng chở hàng | R-12 | Gán `truck` nếu nhận ra thùng hàng riêng; hàng hóa trên xe không có box riêng. | Gán theo quy tắc; reviewer nếu không xác định được loại xe. |
| `img_022` | Xe máy có sidecar | R-13 | Một box `motorcycle` bao xe máy và sidecar gắn liền; người thật nếu nhìn thấy thì box riêng. | Gán theo quy tắc; reviewer nếu không rõ phần nào thuộc phương tiện. |

Ảnh còn lại trong bộ dữ liệu được rà soát để kiểm tra các trường hợp đường phố thông thường,
người đi bộ, phương tiện đỗ/chạy và vật thể nhỏ. Chỉ gán những gì đáp ứng quy tắc chung ở
trên; không suy ra nhãn từ chú thích/metadata ảnh. Các quyết định ở trên là quan điểm cá
nhân để đem ra họp, không phải kết quả đồng thuận của nhóm.

## 5. QC và phiên bản

- Giữ nguyên taxonomy và các ngưỡng của lab: box nhỏ hơn 20 px cần xử lý theo quy tắc vật
  nhỏ; IoU kiểm box lớn `0.85`, box nhỏ `0.70`, ngưỡng ghép IAA `0.50`; IAA yêu cầu IoU
  trung bình `0.85` và cùng lớp `95%`; tỉ lệ ảnh lỗi tối đa `2%`.
- Các ngưỡng được nhắc lại để áp dụng, không tự sửa `rules` trong `taxonomy.json`.
- Khi không chắc, tick `can_review` trên box nếu đã có box hợp lệ; nếu chưa thể tạo box thì
  ghi ảnh/vùng/lý do vào danh sách vấn đề cấp ảnh theo quy trình của nhóm.
- Không ghi số đo IAA, kết quả QC, quyết định calibration hay nhãn như thể đã thực hiện.
  Các kết quả này chỉ được điền sau khi chạy đúng quy trình và có bằng chứng.
- Ca taxonomy chưa rõ (đặc biệt bus, cyclo/xe đạp, xe đạp trợ lực và quy tắc tuk-tuk) cần được thống nhất trước khi
  khóa guideline nhóm và gán nhãn độc lập.

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| Bản cá nhân Pha 1 | Làm rõ box, người/xe chồng lấp, phương tiện chở xe, vật không phải thật, vật nhỏ và các lớp còn thiếu. | Rà soát `img_001`–`img_040`; các ca chưa có lớp phải được nhóm chốt. |
