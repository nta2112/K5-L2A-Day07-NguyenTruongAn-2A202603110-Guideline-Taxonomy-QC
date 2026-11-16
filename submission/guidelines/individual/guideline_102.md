# Guideline gán nhãn — v0.1-102 — Thành viên 102

- **Phiên bản:** v0.1-102 — đề xuất cá nhân Pha 1, chưa phải guideline nhóm đã khóa.
- **Ngày cập nhật:** 2026-11-16.
- **Thành viên:** 102.
- **Phạm vi:** đã mở và rà trực quan từng ảnh `img_001.jpg`–`img_040.jpg` để xây dựng quy tắc và bảng ca thực tế.
- **Trạng thái:** bản có AI hỗ trợ rà ảnh và biên soạn; thành viên 102 cần kiểm lại, tự quyết định và ghi cách sử dụng AI trong ghi chú cá nhân theo `RULES.md`. Các quyết định dưới đây là đề xuất, không phải nhãn COCO hoặc kết quả calibration/QC đã hoàn thành.

## 1. Mục tiêu

Tạo bounding box 2D cho người và các nhóm phương tiện quy định trong taxonomy, phục vụ bài thực hành phát hiện đối tượng giao thông. Gán cả vật đang đỗ, đứng yên hoặc trưng bày nếu là vật thật thuộc lớp. Không suy chuyển động từ một ảnh tĩnh.

Bỏ sót người/xe làm thiếu mục tiêu; nhầm tượng, hình in và phản chiếu tạo nhãn thừa; gộp người với xe hoặc nới box theo hàng hóa làm sai phạm vi đối tượng. Nhãn 2D này không tự cung cấp khoảng cách hay tọa độ 3D.

Thứ tự quyết định: vật thật → thuộc lớp nào → phần nhìn thấy → chiều cao box trên ảnh gốc → box và thuộc tính. Không dùng số chỗ ngồi, hãng xe hoặc địa điểm không xác định được từ ảnh để phân lớp.

## 2. Lớp và thuộc tính

### 2.1. Lớp

Giữ sáu tên lớp của `submission/taxonomy.json`. Định nghĩa chi tiết dưới đây là đề xuất cho Pha 2; chưa sửa taxonomy v0 của nhóm.

| Lớp | Định nghĩa | Ví dụ | Phản ví dụ / cách phân biệt |
|---|---|---|---|
| `car` | Ô tô con dạng sedan, hatchback, wagon, SUV/crossover, không có thùng chở hàng riêng. | SUV trái `img_012`, sedan `img_023`, wagon đen `img_024`. | Pickup là truck; xe thân van là van; tượng pickup `img_027` không gán. |
| `truck` | Xe tải hoặc pickup có cabin và kết cấu chở hàng riêng, kể cả thùng kín, bạt, nắp thùng, giàn chở xe. | `img_001`, `img_002`, hai xe `img_007`, `img_036`, `img_037`. | Nắp thùng cao không biến pickup thành van; SUV là car. |
| `van` | Xe dạng van/minivan có thân hộp kín liền khối, khoang lái nối với khoang hàng/khách, không có thùng tải tách rời. | Xe bạc `img_005`, xe trắng `img_016`. | Xe buýt/minibus thân dài chuyên chở khách `img_009` ngoài taxonomy đề xuất; không ép vào van. |
| `motorcycle` | Xe máy/mô tô có hai bánh chính; bao gồm tổ hợp mô tô gắn sidecar nhận ra được xe chính và thùng bên. | Scooter `img_016`, xe `img_026`, sidecar `img_022`. | Xe đạp, xích lô, tuk-tuk cabin ba bánh không thuộc lớp; người lái tách riêng. |
| `person` | Một người thật nhìn thấy được ở bất kỳ tư thế nào, kể cả người lái/hành khách. | Người bán hàng `img_003`, người đi bộ `img_014`, hàng người `img_032`. | Tượng `img_017`, chân dung in `img_018`, ma-nơ-canh `img_033`, đồ chơi `img_038`. |
| `ignore_region` | Vùng kỹ thuật chứa đối tượng mục tiêu quá nhỏ hoặc cụm không thể tách từng vật đáng tin cậy. | Vùng xa `img_021`, vùng xe không tách được `img_030`, sau khi kiểm từng vùng. | Không phải lớp “mọi vật khác”; không dùng để giấu ca chưa biết lớp hay phủ cả đám đông còn tách được. |

**Ngoài taxonomy (R-11):** bỏ thân xe đạp, xích lô, tuk-tuk, xe buýt, động vật, hàng rời và công trình; người thật trên/cạnh chúng vẫn gán riêng. Không tạo ignore cho toàn bộ phương tiện ngoài taxonomy đã rõ, vì có thể che luôn người lái/hành khách hợp lệ. Nếu nhóm muốn mở lớp hoặc đổi cách ignore, phải thống nhất và cập nhật guideline/taxonomy trước khi gán độc lập.

### 2.2. Thuộc tính

Mỗi box có đủ ba thuộc tính với đúng kiểu/tập giá trị trong taxonomy.

| Thuộc tính | Giá trị | Nghĩa và ngưỡng |
|---|---|---|
| `occluded_level` | `0` | Bị vật khác che 0%–10%, kể cả hai đầu mút. |
| `occluded_level` | `1` | Bị che trên 10% đến 35%. |
| `occluded_level` | `2` | Bị che trên 35% đến 75%. |
| `occluded_level` | `3` | Bị che trên 75%, nhưng phần còn thấy vẫn đủ xác định lớp và đạt ngưỡng kích thước. Không có nghĩa là “không biết vật gì”. |
| `truncated` | `yes` / `no` | Yes khi vật tiếp tục ra ngoài mép ảnh và bị cắt; no khi không. Chạm mép nhưng chưa rõ bị cắt thì kiểm lại. |
| `can_review` | `true` / `false` | True khi còn nghi ngờ thật/giả, lớp, biên box hoặc mức thuộc tính; false khi có căn cứ rõ. |

Tỷ lệ che là **ước lượng**: diện tích hình dáng vật bị vật khác che / diện tích hình dáng vật dự kiến nằm trong khung ảnh nếu bỏ vật che. Không dùng diện tích hình chữ nhật thay diện tích thân vật; không tính phần ngoài ảnh, mặt sau vốn không nhìn thấy, nhòe, bóng tối hoặc phản xạ ánh sáng vào tỷ lệ che. Nếu khó ước lượng hoặc sát mốc 10%, 35%, 75%, tick review và ghi lý do.

Mũ, quần áo đang mặc thuộc hình dáng người, không coi là vật che riêng. Người chỉ lộ đầu qua cửa xe có thể bị che nặng nhưng không vẽ thêm thân tưởng tượng. Xét thuộc tính người/xe độc lập; che và cắt mép có thể đồng thời xảy ra.

Với ignore, dùng `occluded_level=0`, `truncated=no` theo quy ước kỹ thuật; `can_review=true` nếu ranh giới vùng cần thống nhất. Không diễn giải mặc định đó như thuộc tính một vật thật.

## 3. Quy tắc vẽ box

### R-01 — Phần nhìn thấy

Box song song trục ảnh, sát bốn cực trị của các phần nhìn thấy thuộc cùng một vật. Nếu các phần bị ngắt bởi vật che nhưng đủ bằng chứng thuộc cùng vật, dùng một box bao chúng. Vật che/khoảng trống có thể nằm bên trong hình chữ nhật; không kéo biên để bù phần hoàn toàn khuất. Không thêm đệm, bóng đổ, phản chiếu hay quầng đèn.

Box người gồm mũ, quần áo, giày đang mặc; không nới theo túi rời, gánh hàng, ô hoặc hàng đang cầm. Box xe gồm các phần kết cấu nhìn thấy như bánh, gương, tay lái và thùng xe.

### R-02 — Tách người và phương tiện

Một xe máy + một người rõ → **1 motorcycle + 1 person**; một xe + hai người → **1 motorcycle + 2 person**. Không gộp hai người, không nới box xe theo đầu/thân người. Các box được phép chồng lấp.

Kiểm trên hai ảnh: nhóm xe chính `img_026` có hai người và một xe, tạo ba box; xe tay ga bên trái `img_006` cũng áp dụng như vậy. Người đi xe đạp/xích lô/tuk-tuk có box person, nhưng phương tiện ngoài lớp không có box mục tiêu. Không suy người lái chỉ từ sự hiện diện của xe.

### R-03 — Xe chở xe, xe trưng bày

`img_002`: một truck cho cabin và giàn chở xe; mỗi xe thật được chở gán riêng theo lớp nếu phần nhìn thấy cao **ít nhất 20 px** và nhận ra được. Không yêu cầu 50 px; 50 px chỉ là mốc QC. Đánh giá che theo từng xe, không mặc định mức 1/2. Box truck có thể chứa xe con bên trong do hình học, nhưng không nới biên theo xe con nhô ngoài kết cấu tải.

Xe thật trưng bày `img_024` vẫn gán. Vật hình xe đặt trên nóc chưa rõ là xe thật/mô hình/vỏ trang trí thì xử lý R-07/R-12, không tự coi giống ca xe chở xe.

### R-04 — Tượng và ma-nơ-canh

Không gán mục tiêu hay ignore cho tượng người `img_017`, tượng pickup trên bệ `img_027`, ma-nơ-canh `img_033`. Dùng dấu hiệu vật liệu, khớp mô hình, bệ và ngữ cảnh. Đứng yên không phải lý do loại trừ vì người thật và xe đỗ vẫn gán.

### R-05 — Hình in và màn hình

Không gán người/xe trong poster, biển quảng cáo, tranh, hình dán hoặc màn hình (`img_018`, `img_026`, `img_030`, `img_033`, `img_036`, `img_040`). Tiếp tục rà vật thật cùng ảnh, ví dụ hàng ô tô dưới billboard `img_018`.

### R-06 — Phản chiếu, xuyên kính và xe chủ

Không gán ảnh phản chiếu trong gương/kính/mặt đường ướt (`img_013`, `img_020`, `img_031`, `img_033`). Vật thật nhìn xuyên kính vẫn gán nếu nhận ra và đủ kích thước; kính trong suốt không tự động là che khuất.

Bỏ phần xe chủ mang camera khi có bằng chứng rõ, như gương/cửa xe `img_020`. Gần camera hoặc cắt mép chưa đủ chứng minh là xe chủ: pickup `img_012` vẫn gán theo đề xuất này.

### R-07 — Đồ chơi, hóa trang và hình dáng lạ

Đồ chơi `img_038` không gán. Nhân vật hóa trang chỉ gán person khi có căn cứ là người thật; hình người chưa đủ chứng minh có người bên trong. Xe hình dáng lạ vẫn có thể là xe thật (`img_019`). Nếu chưa phân biệt chắc mô hình/trang phục hay xe thật/vỏ trang trí, ghi đề xuất và chuyển review, không khẳng định suy đoán.

### R-08 — Hàng rời và kết cấu xe

**Không nới box theo hàng hóa rời**, kể cả hàng buộc trên xe: túi/thùng `img_012`, mô hình ngựa `img_028`, thanh dài `img_029`, kiện hàng `img_035`. Hàng có thể nằm sẵn trong hình chữ nhật xe nhưng không quyết định biên box. Tính che nếu hàng che thân xe/người.

Bao gồm kết cấu xe: thùng/bạt `img_036`, nắp thùng `img_037`, giàn tải `img_002`, sidecar `img_022` và `img_025`. Mô tô chính và sidecar là một motorcycle; không nới theo hàng trên sidecar.

### R-09 — Vật nhỏ và cắt mép

Đo h của box phần nhìn thấy **trên ảnh gốc**, không dùng kích thước sau phóng màn hình. `h < 20 px`: ignore gọn quanh vật/cụm mục tiêu; `h >= 20 px`: gán riêng nếu nhận diện được. Đúng 20 px được gán; 20–49 px không phải lý do bỏ. Không tăng h bằng phần bị che tưởng tượng.

Vật bị cắt vẫn gán phần còn thấy nếu đủ điều kiện, đặt truncated=yes; box kết thúc tại biên ảnh (`img_001`, `img_015`, `img_026`, `img_034`, `img_040`). Xét từng vật riêng.

### R-10 — Đông, nhòe, tối và vùng ignore

Gán từng vật còn phân biệt được, kể cả xe đỗ chen `img_008`/`img_030` hoặc hàng người đông nhưng rõ `img_032`. Chỉ ignore vùng nhỏ hoặc không thể tách đáng tin sau khi xem kỹ; không phủ cả ảnh vì nhiều vật.

Nhòe/tối nhưng còn rõ lớp và biên thì gán. Nhận ra vật nhưng biên chưa chắc thì gán đề xuất + review. Chưa xác định cá thể thì ghi vấn đề cấp ảnh. Không lấy toàn bộ vệt nhòe làm chiều dài xe, không coi nhòe là che (`img_014`, `img_015`, `img_031`).

Không phủ ignore lên mục tiêu rõ. Nếu vùng chữ nhật không tránh được chồng lấp, giữ box mục tiêu và chuyển reviewer thống nhất phạm vi; không xóa mục tiêu để làm sạch số liệu.

### R-11 — Ngoài taxonomy

Áp dụng mục 2.1: không ép buýt thành van/truck, xe đạp thành motorcycle, tuk-tuk thành car. Vật ngoài phạm vi đã rõ không cần review/ignore. Ca chưa biết thuộc hay ngoài taxonomy mới chuyển R-12.

### R-12 — Review và truy vết

Nếu có vật/biên và lớp đề xuất: box + can_review=true, ghi ảnh, vùng, lý do, phương án cần xác nhận. Nếu chưa đủ căn cứ vẽ hoặc chưa rõ thật/giả: ghi vấn đề cấp ảnh theo `image,annotator,region,note`, để tổng hợp vào `submission/qc/image_issues.csv` sau khi khóa nhãn. Không vẽ box giả chỉ để tick review, không dùng ignore thay quyết định còn thiếu.

102 đề xuất; nhóm/reviewer chốt theo quy trình lab. Quyết định làm đổi vật/lớp/box cần Rule ID, phiên bản và phạm vi rà lại. Tài liệu này không ghi nhận calibration đã diễn ra.

## 4. Bảng rà soát 40 ảnh

Liên kết mở ảnh gốc. Bảng ghi tình huống cần chú ý và đề xuất, không phải danh sách từng annotation. Chưa đo/vẽ trong CVAT thì không khẳng định h hoặc tỷ lệ che cụ thể.

| Ảnh | Quan sát thực tế | Rule ID | Quyết định đề xuất và lý do |
|---|---|---|---|
| [img_001](../../../data/images/img_001.jpg) | Pickup trắng trước quán; người đứng tiền cảnh và nhiều người ngồi phía sau; xe/người sát mép dưới. | R-01, R-02, R-09 | Truck và từng person đủ điều kiện, không chỉ người đứng. Xét cắt mép và che riêng; người đứng che xe, bàn/xe che người ngồi, không mặc định che=0. |
| [img_002](../../../data/images/img_002.jpg) | Xe tải giàn chở ô tô hai tầng; người bên phải gần đuôi xe. | R-02, R-03, R-09 | Một truck; xe chở gán riêng car/van theo cấu trúc; person bên phải nếu đạt ngưỡng. Không mặc định mọi xe trên giàn cùng lớp/mức che; mốc gán là 20 px. |
| [img_003](../../../data/images/img_003.jpg) | Người gánh hàng, người cạnh xe đạp, xe máy đỗ/chạy và ô tô xa. | R-01, R-02, R-08, R-11 | Gán person/motorcycle/car nhận ra; bỏ xe đạp; không nới box người theo thúng và đòn gánh. Tách các vật chồng lấp. |
| [img_004](../../../data/images/img_004.jpg) | Hai tuk-tuk xanh trong đêm; có người ngồi ở cả hai xe. | R-02, R-11 | Bỏ thân tuk-tuk; giữ từng person đủ điều kiện. Không ignore cả xe vì sẽ che người hợp lệ. |
| [img_005](../../../data/images/img_005.jpg) | Cận cảnh người đi xe máy chồng nhau; van bạc nền và ô tô trái. | R-01, R-02, R-09 | Tách người/xe; van bạc là van, xe con là car; kiểm cắt mép. Không mô tả nhầm thành dòng xe rất nhỏ ở xa; phần chỉ lộ mũ/đầu cần kiểm đủ nhận diện. |
| [img_006](../../../data/images/img_006.jpg) | Xe máy và xe đạp nhìn từ sau; scooter trái chở hai người; ô tô trắng trái. | R-02, R-09, R-11 | Nhóm scooter trái: 2 person + 1 motorcycle; người đi xe đạp vẫn gán, xe đạp bỏ; car riêng. Đây không phải đoàn xích lô; dùng kiểm lại quy tắc của img_026. |
| [img_007](../../../data/images/img_007.jpg) | Pickup lắp thùng cao giữa ảnh và xe tải thùng lớn bên phải, trong nhà. | R-01, R-08, R-09 | Hai xe chính truck, bao thùng; xe phải cắt mép. Thùng kín không khiến pickup thành van. |
| [img_008](../../../data/images/img_008.jpg) | Phố đêm, dãy xe máy đỗ chéo trái, người ngồi xe trắng tiền cảnh phải, nhiều người nền. | R-02, R-09, R-10 | Tách từng person/motorcycle còn rõ, kiểm cắt mép; chỉ ignore vùng thật sự nhỏ/không tách được. Xe đỗ cũng là mục tiêu. |
| [img_009](../../../data/images/img_009.jpg) | Buýt nhỏ thân dài giữa phố; người qua cửa kính; người/xe máy hai bên. | R-02, R-06, R-09, R-11 | Bỏ thân buýt, không ép vào van; người xuyên kính chỉ gán nếu nhận ra và đạt 20 px; rà xe/người bên ngoài. Không phải xích lô, không suy số người từ cửa sổ. |
| [img_010](../../../data/images/img_010.jpg) | Hai người trên xe máy trái, người bán cạnh xe đạp chở trái cây, một người trên xe máy phải. | R-01, R-02, R-08, R-11 | Từng person riêng, mỗi motorcycle một box; bỏ xe đạp/hàng rời; rà xe đỗ nền. Không gộp hai người bên trái hay nới box người bán theo sạp. |
| [img_011](../../../data/images/img_011.jpg) | Nhân vật Predator lớn; mô tô trang trí kim loại; người thật và ô tô phía sau. | R-04, R-07, R-12 | Gán người/xe nền rõ. Nhân vật và xe trang trí cần xác nhận thật/mô hình. Metadata nguồn mô tả hóa trang và xe độ, hỗ trợ giả thuyết person/motorcycle, nhưng không đủ chứng minh mọi vật trưng bày đều thật; ghi review thay khẳng định. |
| [img_012](../../../data/images/img_012.jpg) | Pickup tiền cảnh có túi/thùng hàng; SUV bạc trái; xe tải trước. | R-01, R-08, R-09 | Pickup và xe tải là truck; SUV car; pickup cắt mép. Không nới theo hàng, không loại xe gần camera nếu chưa có bằng chứng là xe chủ. |
| [img_013](../../../data/images/img_013.jpg) | Phố đêm ướt, hai người rõ gần giữa, xe/người xa, bóng nước. | R-06, R-09, R-10 | Gán người/xe thật đủ rõ; không thêm person cho phản chiếu. Đo vật nhỏ trên ảnh gốc; ánh đèn không phải đối tượng. |
| [img_014](../../../data/images/img_014.jpg) | Hai người đi bộ rõ; người/xe máy nền nhòe chuyển động. | R-02, R-10, R-12 | Hai person rõ; các vật nhòe gán khi tách được cá thể, review nếu biên không chắc. Không bỏ tất cả hoặc lấy cả vệt nhòe làm box. |
| [img_015](../../../data/images/img_015.jpg) | Đầu xe đỏ cắt trái, đuôi xe bạc cắt phải; xe đỗ và vệt xe nhòe nền. | R-01, R-06, R-09, R-10 | Hai car tiền cảnh truncated=yes; rà xe nền riêng, vệt nhòe cần review nếu không tách được. Không bao bóng trên đường. |
| [img_016](../../../data/images/img_016.jpg) | Scooter xanh và người lái; van trắng, pickup có khung thùng trái; tuk-tuk hai bên và người tiền cảnh phải. | R-02, R-08, R-11 | Motorcycle + person; van, truck riêng; bỏ tuk-tuk nhưng giữ người. Cắt mép xét độc lập; cùng ảnh cho thấy van/truck/tuk-tuk khác nhau. |
| [img_017](../../../data/images/img_017.jpg) | Tượng người cưỡi ngựa trên bệ, cọc giao thông trên đầu và chim. | R-04, R-11 | Không gán person/motorcycle hoặc ignore cho tượng; động vật ngoài taxonomy. Không thấy mục tiêu thật rõ thuộc lớp. |
| [img_018](../../../data/images/img_018.jpg) | Chân dung billboard; nhiều ô tô thật sau hàng rào phía dưới. | R-01, R-05, R-09 | Bỏ chân dung; từng xe thật gán nếu nhận ra, đánh che/cắt riêng. Không kết luận cả ảnh không có box chỉ vì billboard nổi bật. |
| [img_019](../../../data/images/img_019.jpg) | Xe vàng kiểu hoạt hình trong diễu hành, người ngồi cạnh nhân vật chuột trắng, nhiều khán giả. | R-02, R-07, R-10, R-12 | Đề xuất car cho xe, person cho người ngồi và khán giả tách được. Nhân vật chuột cần review thật/mô hình; hình dáng lạ không tự loại xe có bánh/cabin/người ngồi. |
| [img_020](../../../data/images/img_020.jpg) | Gương xe chủ phản chiếu xe phía sau, thân xe và cầu vồng; cửa xe ở dưới. | R-06 | Bỏ ảnh ảo trong gương và bộ phận xe chủ. Có thể không có box mục tiêu; không cần ignore. |
| [img_021](../../../data/images/img_021.jpg) | Dòng xe máy rất đông kéo sâu vào giao lộ, ô tô xen kẽ. | R-02, R-09, R-10 | Gán người/xe rõ ở gần; ignore cục bộ vật nhỏ/cụm không tách ở xa. Không gộp cả hàng; đo h trước khi áp mốc 20 px. |
| [img_022](../../../data/images/img_022.jpg) | Ảnh đen trắng mô tô gắn sidecar; chi tiết giống đầu người sau kính sidecar. | R-02, R-08, R-12 | Một motorcycle gồm xe chính và sidecar; phần nghi người cần kiểm lại trước khi gán person. Không suy người từ một đốm nhỏ chưa rõ. |
| [img_023](../../../data/images/img_023.jpg) | Người đạp xích lô nhìn từ sau; sedan đen trái; người/xe máy hai bên. | R-02, R-11 | Person người đạp; bỏ xích lô; sedan car; rà nền. Đây là ca xích lô rõ; không suy hành khách khi ghế khuất. |
| [img_024](../../../data/images/img_024.jpg) | Wagon đen trong khu trưng bày, nhiều ô tô xung quanh; vật hình xe vàng nhỏ trên nóc. | R-03, R-07, R-12 | Xe thật đen và xung quanh car. Vật vàng đề xuất loại theo mô hình/vỏ trưng bày, cần reviewer xác nhận; không nới box car đen theo vật vàng. |
| [img_025](../../../data/images/img_025.jpg) | Lối đi ven biển có người gần/xa; đường phải có ô tô và mô tô gắn thùng bên. | R-02, R-08, R-09 | Gán người/xe đủ rõ; mô tô và khung sidecar một motorcycle, người riêng; bỏ hàng trên thùng. Áp dụng nhất quán img_022; thuyền/ghế ngoài taxonomy. |
| [img_026](../../../data/images/img_026.jpg) | Hai người đội mũ trên xe máy đỏ; tường in cảnh cảng/tàu; đáy xe bị cắt. | R-02, R-05, R-09 | Nhóm chính **2 person + 1 motorcycle**; xe truncated=yes; bỏ cảnh in. Không suy địa điểm là hầm chui; kiểm từng người độc lập. |
| [img_027](../../../data/images/img_027.jpg) | Pickup bằng đá trên bệ trong nghĩa trang. | R-04 | Không car/truck và không ignore. Vật liệu/bệ cho thấy tượng, khác pickup thật img_028. |
| [img_028](../../../data/images/img_028.jpg) | Pickup thật chở mô hình ngựa, ô tô đỏ phía sau. | R-04, R-08 | Truck cho pickup, car cho xe đỏ; không gán mô hình ngựa hay nới box truck theo ngựa. Xe thật và mô hình cần quyết định riêng. |
| [img_029](../../../data/images/img_029.jpg) | Người trên xe máy chở thanh dài; người cạnh xe đạp phải; người/xe đạp xa. | R-02, R-08, R-11 | Motorcycle và từng person; bỏ xe đạp/thanh hàng khỏi biên xe. Không phải chồng xốp che kín người, không mặc định che mức 3. |
| [img_030](../../../data/images/img_030.jpg) | Bãi xe máy chồng lấp, người áo xanh phải, poster chân dung nền. | R-01, R-05, R-10 | Từng motorcycle tách được và người thật; bỏ hình in; ignore chỉ vùng không tách. Tránh tạo nhiều xe từ gương/tay lái của cùng một chiếc. |
| [img_031](../../../data/images/img_031.jpg) | Phố đêm mưa có ô tô, xe máy, người, buýt và phản chiếu đèn. | R-02, R-06, R-10, R-11 | Car/motorcycle/person đủ rõ; bỏ buýt và phản chiếu. Không nới theo vệt đèn, không coi tối là che; kiểm người bên phải tối. |
| [img_032](../../../data/images/img_032.jpg) | Hàng người chờ trạm xe, xe xanh cắt mép trái, xe nền. | R-01, R-06, R-09, R-10 | Tách từng người còn rõ, car tiền cảnh cắt mép; rà xe nền và xuyên kính/phản chiếu. Nhiều người không đồng nghĩa ignore cả hàng. |
| [img_033](../../../data/images/img_033.jpg) | Ma-nơ-canh trần sáng màu, ảnh người mặc váy trái, hình cabin/gương xe trên kính phải. | R-04, R-05, R-06, R-12 | Bỏ mannequin/hình in; đề xuất bỏ hình xe phản chiếu, review nếu chưa phân biệt thật/ảo. Váy thuộc ảnh in, không phải mannequin đang mặc váy. |
| [img_034](../../../data/images/img_034.jpg) | Người sang đường, sedan đen, xe máy/xe đạp; người/xe sát phải. | R-02, R-09, R-11 | Tách person/car/motorcycle; bỏ xe đạp, giữ người đạp; kiểm cắt mép phải. Người đi bộ che sedan, không bỏ người lái vì đã gán xe. |
| [img_035](../../../data/images/img_035.jpg) | Tuk-tuk chở kiện hàng rất cao; người đi mô tô tiền cảnh; tuk-tuk và ô tô nền. | R-02, R-08, R-11, R-12 | Bỏ tuk-tuk/hàng; motorcycle/person tiền cảnh và car nền gán; tài xế cabin chỉ gán khi đủ căn cứ. Không có nhóm học sinh như mô tả cũ; hàng cao không biến xe thành truck. |
| [img_036](../../../data/images/img_036.jpg) | Tải nhẹ xanh có thùng bạt riêng; xe máy, một phần người nền và quảng cáo trên tường. | R-01, R-05, R-08 | Truck gồm thùng/bạt; rà motorcycle/person nền, bỏ người in. Không phải van và không chỉ gán xe lớn tiền cảnh. |
| [img_037](../../../data/images/img_037.jpg) | Pickup trắng lắp nắp thùng cao, buýt phía sau, người đi bộ trái. | R-02, R-08, R-11 | Truck gồm nắp thùng, person riêng; bỏ buýt. Nắp kín không đổi pickup thành van. |
| [img_038](../../../data/images/img_038.jpg) | Tủ kính ô tô đồ chơi, búp bê, thú đồ chơi và phản chiếu. | R-06, R-07 | Không gán đồ chơi/phản chiếu, không ignore cả tủ. Vật lớn trên ảnh không biến mô hình thành xe/người thật. |
| [img_039](../../../data/images/img_039.jpg) | Nhóm người đi bộ giữa phố, người đi xe máy/xe đạp, xe đỗ hai bên. | R-02, R-09, R-11 | Person/motorcycle/car đủ rõ; bỏ xe đạp nhưng giữ người; kiểm h vật xa. Rà cả vỉa hè tối và xe đỗ. |
| [img_040](../../../data/images/img_040.jpg) | Người mũ đen, chỉ lộ một phần xe máy; taxi xanh, SUV, tuk-tuk/buýt nền, ảnh mặt người dán kính. | R-01, R-02, R-05, R-09, R-11, R-12 | Person tiền cảnh cắt mép, taxi/SUV car; motorcycle chỉ bao phần thấy đủ nhận diện, review nếu khó liên kết. Bỏ tuk-tuk/buýt/hình in; không dựng toàn xe từ hai gương. |

## 5. QC và phiên bản

### 5.1. Ngưỡng giữ nguyên theo lab

Theo `submission/taxonomy.json` và `docs/QC_CHEATSHEET.md`; không thay mục rules.

| Hạng mục | Ngưỡng / yêu cầu |
|---|---|
| Chiều cao tối thiểu | 20 px; thấp hơn xử lý R-09. |
| IoU theo cỡ box | Cao từ 50 px: ít nhất 0.85; dưới 50 px: ít nhất 0.70. |
| Ghép cặp | 1–1 không phân biệt lớp, match_iou=0.5. |
| IAA | IoU trung bình cặp ghép ít nhất 0.85; cùng lớp ít nhất 95% trên cặp ghép; không có cặp là N/A. |
| Độ phủ | Đủ 100% lượt assignment trước đo IAA; ít nhất 2 người/ảnh, nhóm 5 người là 3. |
| Lỗi theo ảnh | Ảnh có ít nhất một lỗi / ảnh đã duyệt, tối đa 2%; 1/40=2,5% đã vượt ngưỡng. |
| Lỗi theo đối tượng | Báo thêm annotation lỗi / annotation đã duyệt, kèm mẫu số. |
| Bàn giao | Duyệt toàn lô; Hold khi thiếu bằng chứng/guideline còn vấn đề/chỉ lấy mẫu; Rework khi vượt ngưỡng hoặc lỗi lặp; Release khi reviewer xác nhận đủ điều kiện. |

Không thay ngưỡng IAA trung bình bằng 0.70 của box nhỏ. Box chưa ghép và bất đồng ignore cần xem riêng; IAA cao không chứng minh hai người không cùng sai. Kiểm năm nhóm lỗi: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính.

### 5.2. Checklist trước khi khóa guideline nhóm

- [ ] 102 kiểm lại bản AI hỗ trợ và ghi cách kiểm chứng trong ghi chú cá nhân.
- [ ] Thống nhất car/van/buýt, pickup có nắp, sidecar và xe ngoài taxonomy.
- [ ] Kiểm quy tắc người/xe trên img_026 và img_006.
- [ ] Chốt hàng rời ở img_012/028/029 và xe chở xe img_002.
- [ ] Xác nhận ca cần review: nhân vật/xe trang trí img_011, mascot img_019, phần nghi người img_022, vật vàng img_024, phản chiếu img_033, phần xe máy img_040.
- [ ] Đo vùng nhỏ trên ảnh gốc; không suy kích thước từ ảnh thu nhỏ.
- [ ] Đồng bộ guideline v1.0 và taxonomy, giữ nguyên rules, rồi mới gán độc lập trong CVAT.

Đây là rà guideline Pha 1; chưa xác nhận nhãn COCO, số đo IAA, calibration, QC toàn lô hay quyết định Release.

### 5.3. Lịch sử thay đổi

| Phiên bản | Ngày | Thay đổi | Căn cứ |
|---|---|---|---|
| v0 | Không ghi trong starter | Bản khởi đầu của lab. | guideline/guideline_v0_starter.md. |
| v0.1-102 | 2026-11-16 | Bản cá nhân có sẵn trước lần rà này, gồm quy tắc và một phần ca khó. | Bản đầu vào. |
| v0.2-102 | 2026-11-16 | Bổ sung 40/40 ảnh; sửa mô tả sai 006/009/029/033/035; ghi ca thật/giả chưa chắc; sửa ngưỡng xe được chở từ 50 xuống 20 px; thống nhất người/xe, hàng rời và ngoài taxonomy; thêm ego, nhòe, truncation, IAA/review. Có thay đổi ngữ nghĩa, chưa phải calibration của nhóm. | 40 ảnh gốc, GUIDE, RULES, RUBRIC, template, taxonomy, QC cheatsheet; metadata nguồn chỉ hỗ trợ khi ghi rõ. |

Nếu bản cá nhân đã nộp/khóa trước họp nhóm, giữ bản đã nộp làm bằng chứng và đưa thay đổi ngữ nghĩa vào quy trình phiên bản nhóm; không trình bày bản cập nhật như quyết định độc lập có từ trước.
