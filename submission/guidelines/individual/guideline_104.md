# Guideline gán nhãn — v0.1 (DEMO) — viết lại từ v0

> **Bản demo tham khảo**, viết sau khi xem cả 40 ảnh trong `data/images/`.
> Bản này cho thấy cách biến v0 thành guideline đủ dùng: định nghĩa có phản ví dụ, ngưỡng đo được,
> quy tắc có mã `R-xx` và ca khó dẫn ID ảnh thật.
> Guideline cá nhân (`submission/guidelines/individual/guideline_<mã>.md`) phải do **từng người tự viết** (xem RULES.md).

- Phiên bản: v1
- Ngày: 2026-11-16
- Người viết / duyệt: (demo)

---

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình **phát hiện người và phương tiện cơ giới 2D** (bounding box) trên ảnh đường phố,
phục vụ camera giao thông / xe tự lái.

- **Sai nguy hiểm nhất:** bỏ sót người thật (đặc biệt người đi xe máy, người qua đường) và xe đang lưu thông.
- **Sai gây nhiễu:** gán box cho vật **không thật** (tượng, ma-nơ-canh, đồ chơi, hình in, phản chiếu). Mô hình học sai sẽ báo động giả.
- **Phạm vi:** chỉ 5 lớp vật + 1 vùng bỏ qua dưới đây. Xe buýt, xe đạp, xích lô, tuk-tuk **không** thuộc phạm vi v0.1 (xem R-04).

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Xe ô tô chở người ≤ 9 chỗ, khoang hành khách và khoang hành lý liền khối (sedan, hatchback, SUV, wagon, taxi). | xe đen `img_023`; taxi xanh `img_040`; xe bạc `img_012`; xe đỗ sau hàng rào `img_018` | pickup có thùng hở → `truck`; xe nhiều chỗ dạng hộp → `van` |
| `truck` | Xe chở hàng có **thùng hàng tách rời cabin** (pickup, xe tải nhỏ, xe tải, đầu kéo + rơ-moóc). | pickup trắng `img_037`; xe tải Isuzu `img_007`; xe tải nhỏ xanh `img_036`; xe tải cũ `img_001`; xe chở xe `img_002` | SUV có cốp liền → `car`; xe van thùng kín liền cabin → `van` |
| `van` | Xe thân hộp liền khối, cabin và khoang sau chung một vỏ, cao hơn car (minivan, xe 12–16 chỗ, xe giao hàng thùng liền). | xe trắng biển 51LD `img_005`; xe Toyota trắng `img_016` | xe buýt cỡ lớn → ngoài phạm vi (R-04); SUV → `car` |
| `motorcycle` | Xe 2 bánh có động cơ (xe máy, tay ga, mô tô), kể cả loại gắn thùng bên (sidecar). | `img_006`, `img_010`, `img_026`; Vespa `img_016`; sidecar `img_022` | xe đạp, xe đạp điện dạng xe đạp → không gán; tuk-tuk 3 bánh → không gán (R-04) |
| `person` | Người thật, nhìn thấy được ít nhất một phần cơ thể, đứng, đi hoặc **ngồi trên xe hở** (xe máy, xe đạp, xích lô, tuk-tuk). | người qua đường `img_014`, `img_034`; người đi xe máy `img_005`, `img_026` | tượng `img_017`; ma-nơ-canh `img_033`; người trên biển quảng cáo `img_018`; người trong xe kín (R-09) |
| `ignore_region` | Vùng có vật thuộc phạm vi nhưng **không thể** vẽ từng box (đám đông dày, vật quá nhỏ thành cụm). Bỏ sót trong vùng này không tính lỗi. | dòng xe máy phía xa `img_021`; đám đông sau rào `img_019` | không dùng để "giấu" vật ngoài phạm vi (xe buýt…) |

**Vật không thuộc lớp nào** (xe buýt, xe đạp, xích lô, tuk-tuk, động vật, tượng…): **không vẽ box, không đặt `ignore_region`**. Người ngồi trên các xe đó vẫn gán `person` (R-04).

### Thuộc tính

| Thuộc tính | Mức | Ngưỡng cụ thể |
|---|---|---|
| `occluded_level` | `0` | bị che **< 10%** diện tích ước lượng của vật |
| | `1` | bị che **10% – < 40%** |
| | `2` | bị che **40% – 80%** |
| | `3` | bị che **> 80%**, **hoặc** không ước lượng được do tối / mờ chuyển động |
| `truncated` | `no` / `yes` | `yes` khi vật **chạm mép ảnh và có phần nằm ngoài khung**. Chỉ chạm mép mà vẫn thấy trọn vật thì vẫn là `no`. Phần bị cắt **không** tính vào `occluded_level`. |
| `can_review` | tick / không | tick khi box rơi vào ca chưa có quy tắc, hoặc khi bạn phân vân giữa hai lớp. Bắt buộc ghi lý do vào nhật ký nhóm. |

Cách ước lượng "% bị che": hình dung toàn bộ vật, rồi ước phần bị vật **khác** che (người che xe, xe che xe, cột, cây).
Người ngồi trên xe máy che một phần xe máy: tính vào `occluded_level` của xe máy.

## 3. Quy tắc vẽ box

| Mã | Quy tắc |
|---|---|
| **R-01** | **Box ôm sát phần nhìn thấy** (modal box). Không đoán phần bị che. Sai lệch cho phép khoảng 2 px mỗi cạnh. |
| **R-02** | **Người đi xe máy = 2 lớp riêng.** Mỗi người (lái và người ngồi sau) một box `person`, xe một box `motorcycle`. Box người bao cả mũ bảo hiểm và chân đặt trên xe. Box xe bao bánh, thân, gương, tay lái, **không** mở rộng để bao người. (`img_005`, `img_006`, `img_010`, `img_026`, `img_040`) |
| **R-03** | **Hàng hóa chở theo:** box vật **không** mở rộng để bao hàng thò ra ngoài (ống dài `img_029`, quang gánh `img_003`, hàng trên tuk-tuk `img_035`). Hàng trong thùng xe tải / pickup thì nằm trong box xe (`img_012`). |
| **R-04** | **Phương tiện ngoài phạm vi** (xe buýt `img_009`, `img_031`, `img_037`, `img_040`; xe đạp `img_006`, `img_029`; xích lô `img_023`, `img_034`; tuk-tuk / xe lam `img_004`, `img_016`, `img_035`, `img_040`): **không vẽ box xe**. Người ngồi / đạp trên xe **hở** vẫn gán `person` theo R-09. |
| **R-05** | **Xe chở xe** (`img_002`): đầu kéo + rơ-moóc là **một** box `truck` (liền khối khi nhìn từ ảnh). Mỗi ô tô **thật** đang được chở là một box `car` riêng, `occluded_level` theo phần bị khung giàn che. Xe trên nóc xe trưng bày (`img_024`) gán theo cùng quy tắc: xe thật gán `car`. |
| **R-06** | **Vật không thật thì không gán:** tượng (`img_017` người cưỡi ngựa, `img_027` bia mộ hình pickup, tượng ngựa trên xe `img_028`), ma-nơ-canh (`img_033`), đồ chơi / mô hình (`img_038`), người hoặc xe trên **biển quảng cáo, tranh, poster** (`img_018`, tranh tường `img_026`, poster `img_030`). Vật thật đứng cạnh vật không thật vẫn gán bình thường (pickup thật ở `img_028` → `truck`). |
| **R-07** | **Phản chiếu không gán:** ảnh trong gương chiếu hậu (`img_020`), bóng trên mặt đường ướt (`img_013`, `img_015`, `img_031`), hình trong kính cửa hàng. Chỉ gán vật thật tạo ra phản chiếu nếu vật đó nằm trong ảnh. |
| **R-08** | **Xe chủ (ego) không gán:** phần thân / gương / tay lái của xe mà người chụp đang ngồi (`img_020` cửa xe và gương; gương xe máy ở đáy `img_040`). |
| **R-09** | **Người trong xe:** người trong xe **kín** (car, van, truck có cabin) **không** gán `person`, kể cả khi thấy qua kính. Người trên xe **hở** (xe máy, xe đạp, xích lô, tuk-tuk, xe mui trần như `img_019`) **gán** `person` nếu thấy ≥ đầu + vai. |
| **R-10** | **Vật nhỏ:** box có chiều cao **< 20 px** thì **không** vẽ box riêng (ngưỡng `min_box_height_px` của taxonomy). Cụm ≥ 3 vật nhỏ sát nhau → một `ignore_region` bao cụm. Vật nhỏ đơn lẻ → bỏ qua. |
| **R-11** | **Đám đông / bãi xe dày:** vẽ box riêng cho mọi vật **cao ≥ 20 px và còn thấy được đường bao riêng**. Phần phía sau mà các vật chồng lên nhau, không tách được từng cá thể, thì gom vào **một** `ignore_region`. `ignore_region` không được phủ lên vật đã có box. (`img_019`, `img_021`, `img_030`, `img_032`) |
| **R-12** | **Xe đỗ cũng gán** như xe đang chạy (`img_008`, `img_030`, `img_036`, `img_018`). |
| **R-13** | **Ảnh mờ chuyển động / ban đêm:** vẫn gán nếu nhận ra lớp. Mờ đến mức không chắc lớp → vẽ box lớp gần đúng nhất, `occluded_level=3` và tick `can_review`. (`img_014`, `img_015`, `img_013`, `img_031`) |
| **R-14** | **Xe trang trí / xe diễu hành chạy được** (có bánh, có người lái, đi trên đường) gán theo hình dáng chức năng gần nhất và tick `can_review` (xe Danger Mouse `img_019` → `car`). Vật **trưng bày không vận hành** (khung xe điêu khắc `img_011`) → không gán; nếu không chắc thì ghi vấn đề cấp ảnh. |
| **R-15** | **Người hóa trang / mascot:** có người thật bên trong (đi lại, ngồi trên xe) → `person` + `can_review` (mascot chuột `img_019`). Hình nộm / tượng hóa trang → không gán (`img_011`). |
| **R-16** | **Ảnh không có vật** thuộc phạm vi (ví dụ `img_017`, `img_027`, `img_033`, `img_038`) vẫn phải có trong file COCO, chỉ là không có annotation. |

## 4. Ca khó (đối chiếu trên 40 ảnh)

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_002` | xe tải chở 7–8 ô tô | R-05 | 1 `truck` (cả đoàn) + mỗi ô tô một `car`, occluded theo giàn | ô tô thật, mô hình cần thấy cả trên giàn; đổi quy tắc này ảnh hưởng `img_024` |
| `img_005` | cận cảnh nhiều người đi xe máy, xe máy bị người che gần hết | R-02, R-01 | mỗi người 1 `person`; xe máy chỉ gán khi thấy ≥ 1 bộ phận nhận dạng được (bánh, đầu xe), `occluded_level=2/3` | tránh vẽ box xe "đoán" khi không thấy xe |
| `img_017` | tượng người cưỡi ngựa đội nón giao thông | R-06 | không gán gì; ảnh rỗng | tượng không phải người; **ca IAA có thể cao nhưng cùng sai** nếu cả hai vẽ `person` |
| `img_020` | xe trong gương chiếu hậu + cửa xe chủ | R-07, R-08 | không gán | phản chiếu và xe chủ |
| `img_026` | 2 người trên xe máy trước tranh tường có tàu | R-02, R-06 | 2 `person` + 1 `motorcycle`; tranh không gán | góc nhìn ngang, người che xe ~40% → `occluded_level=2` cho xe |
| `img_027` | tượng đá hình xe bán tải | R-06 | không gán | không phải phương tiện vận hành |
| `img_028` | pickup thật chở tượng ngựa | R-06 | `truck` cho pickup; ngựa không gán; ô tô đỏ phía sau → `car` | vật thật cạnh vật giả |
| `img_033` | ma-nơ-canh + gương xe tải trong kính | R-06, R-07 | không gán | không phải người; phản chiếu |
| `img_035` | xe lam chở hàng cao + mô tô | R-04, R-03 | xe lam không gán; người lái nhìn thấy → `person`; mô tô + người lái gán; ô tô trắng → `car` | 3 bánh ngoài phạm vi |
| `img_038` | tủ kính xe đồ chơi | R-06 | không gán | mô hình |
| `img_019` | xe diễu hành, mascot, đám đông | R-14, R-15, R-11 | `car` + `can_review`; người lái `person`; mascot `person` + `can_review`; đám đông phía sau → `ignore_region` trừ người tách được | ca chưa chắc → đưa reviewer |
| `img_021` | dòng xe máy rất dày | R-02, R-11 | box cho người và xe ở tiền cảnh; phần xa chồng lấn → `ignore_region` | không tách được cá thể |
| `img_022` | mô tô sidecar, ảnh đen trắng | R-02 | 1 `motorcycle` bao cả sidecar + `person` cho người | sidecar gắn liền xe |
| `img_037` | pickup gắn nắp thùng (truck cap) | lớp `truck` | `truck` | vẫn có thùng tách cabin |
| `img_011` | khung xe điêu khắc, tượng Predator | R-14, R-15 | không gán; người thật nhỏ phía sau gán nếu ≥ 20 px | chưa chắc → ghi vấn đề cấp ảnh |

Nếu gặp ca chưa có trong bảng: tick `can_review` trên box, hoặc ghi vấn đề cấp ảnh (ảnh, vùng, lý do) để nhóm thu vào `image_issues.csv`.

## 5. QC và phiên bản

- **Ngưỡng QC** (giữ nguyên `rules` trong `taxonomy.json`): IoU ghép ≥ 0.5; IAA IoU TB ≥ 0.85, cùng lớp ≥ 95%; tỉ lệ ảnh lỗi ≤ 2% mới Release.
- **Ai quyết ca mơ hồ:** người điều phối chốt ca trong buổi calibration, người review bản cuối kiểm lại. Mỗi ca một dòng trong `calibration_decisions.csv`, dẫn mã `R-xx`.
- Đổi cách chọn vật / lớp / box → `semantic`, phải kiểm lại cả lô. Chỉ sửa câu chữ → `editorial`.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | bản khởi đầu của giảng viên | — |
| v0.1-demo | thêm định nghĩa lớp có phản ví dụ; ngưỡng `occluded_level`; R-01…R-16; bảng ca khó | `img_002`, `img_005`, `img_017`, `img_020`, `img_026`, `img_035`… |
