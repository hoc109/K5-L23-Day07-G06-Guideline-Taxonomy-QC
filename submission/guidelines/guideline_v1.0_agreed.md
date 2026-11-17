# Guideline gán nhãn — v1.0 (nhóm thống nhất) — G06

- Phiên bản: v1.0 (đã khóa cho Pha 3 gán độc lập)
- Ngày: 2026-11-16
- Nhóm: G06 (`175` — Điều phối, `166` — Reviewer bản cuối)
- Công cụ: CVAT (`http://localhost:8080`), dùng Rectangle 2D, xuất COCO 1.0

---

## 1. Mục tiêu

Bộ nhãn 2D Bounding Box phục vụ huấn luyện và đánh giá mô hình phát hiện người và phương tiện giao thông đường bộ (hệ thống trợ lái nâng cao ADAS Cấp độ 2+ đến 4 và giám sát giao thông thông minh ITS).

- **Nguyên tắc cốt lõi:** Box bám sát ranh giới vật lý nhìn thấy để ước lượng tĩnh không an toàn; ưu tiên không bỏ sót người/xe thô sơ (VRU), bao trọn hàng hóa nguy hiểm nhô ra khỏi xe, và loại bỏ vật thể phi thực để tránh lỗi dương tính giả ("phanh ma").

---

## 2. Lớp và thuộc tính

### 2.1. Danh sách lớp (8 lớp — mỗi vật thuộc đúng một lớp)

| Lớp | Định nghĩa (một câu) | Ví dụ trong bộ ảnh | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Phương tiện cơ giới 4 bánh chở người từ 9 chỗ trở xuống (sedan, hatchback, SUV, crossover, van chở khách ≤ 9 chỗ, bán tải tiêu chuẩn). | Ô tô con lưu thông (`img_015.jpg`, `img_023.jpg`, `img_034.jpg`), xe con trên giá chở (`img_002.jpg`). | Xe tải/van chở hàng (`truck`), xe buýt ≥ 10 chỗ (`bus`), xe mô hình trên nóc (`img_024.jpg`), bia mộ hình xe (`img_027.jpg` → `ignore_region`). |
| `truck` | Phương tiện cơ giới chuyên dùng chở hàng (xe tải thùng kín/mui bạt, đầu kéo rơ-moóc, van chở hàng kín, bán tải hoán cải thùng cao vượt trần cabin, xe cổ chở hàng cồng kềnh quá nóc). | Xe lồng chở ô tô (`img_002.jpg`), xe tải thùng (`img_007.jpg`, `img_036.jpg`), bán tải nắp thùng cao (`img_037.jpg`), bán tải cổ chở tượng ngựa (`img_028.jpg`). | Bán tải gia đình tiêu chuẩn (`car`), xe ba gác máy (`three_wheeler`). |
| `bus` | Phương tiện cơ giới chở khách công cộng hoặc du lịch từ 10 chỗ ngồi trở lên, thân liền khối cỡ trung đến lớn. | Xe khách County (`img_009.jpg`), xe buýt đô thị (`img_031.jpg`, `img_040.jpg`). | Xe van chở khách ≤ 9 chỗ (`car`), xe tải thùng kín (`truck`). |
| `three_wheeler` | Phương tiện cơ giới 3 bánh có động cơ chuyên chở khách hoặc hàng hóa (tuk-tuk, xe lam, xe ba gác máy). | Xe tuk-tuk (`img_004.jpg`, `img_016.jpg`, `img_035.jpg`). | Mô tô 2 bánh gắn sidecar (`motorcycle`, `img_022.jpg`), xích lô đạp chân (`bicycle`, `img_023.jpg`). |
| `motorcycle` | Phương tiện cơ giới 2 bánh (hoặc mô tô gắn thùng sidecar) chạy động cơ đốt trong/điện, điều khiển bằng ghi-đông. | Xe máy số/tay ga (`img_006.jpg`, `img_008.jpg`, `img_021.jpg`, `img_040.jpg`), mô tô sidecar (`img_022.jpg`). | Xe đạp / xích lô đạp chân (`bicycle`), xe tuk-tuk (`three_wheeler`). |
| `bicycle` | Phương tiện thô sơ 2 hoặc 3 bánh vận hành chủ yếu bằng bàn đạp sức người (hoặc trợ lực nhẹ), gồm cả xích lô. | Xe đạp (`img_003.jpg`, `img_006.jpg`, `img_029.jpg`), xích lô (`img_023.jpg`). | Xe máy điện/xe máy (`motorcycle`), xe đẩy hàng không bàn đạp. |
| `person` | Cá thể người thật ở mọi tư thế: đi bộ, đứng, ngồi, gánh hàng hoặc điều khiển phương tiện. | Người đi bộ (`img_034.jpg`), người gánh hàng (`img_003.jpg`), người lái xe máy/xe đạp (`img_006.jpg`, `img_029.jpg`). | Tượng người (`img_017.jpg`), ma-nơ-canh (`img_033.jpg`), ảnh in người trên biển quảng cáo (`img_018.jpg`, `img_026.jpg`). |
| `ignore_region` | Vùng bỏ qua (không phải vật): cụm xe dày đặc ở xa không tách được box, gương phản chiếu xe, vật phi thực cỡ lớn dễ gây nhiễu, hàng hóa nối dài nguy hiểm, thùng xe chủ. | Kẹt xe/bãi xe xa (`img_021.jpg`, `img_030.jpg`), gương phản chiếu (`img_020.jpg`), tủ kính đồ chơi (`img_038.jpg`), bia mộ hình xe (`img_027.jpg`), thanh sắt giữa 2 xe (`img_029.jpg`), thùng xe chủ (`img_012.jpg`). | Người hoặc xe đứng độc lập ở cự ly gần có biên rõ ràng. |

**Quyết định về lớp `van` và vật ngoài taxonomy:**
- Bỏ lớp `van` khỏi taxonomy: xe van chở khách ≤ 9 chỗ xếp vào `car`, xe van chở hàng thùng kín xếp vào `truck` (`R-06`).
- Vật phi thực nhỏ/phẳng (tranh pano, tượng trang trí nhỏ): **loại theo quy tắc** (không gán). Vùng phi thực cỡ lớn/dễ gây nhầm: khoanh `ignore_region` (`R-04`).

### 2.2. Thuộc tính (áp dụng cho mọi lớp trừ `ignore_region`)

| Thuộc tính | Giá trị | Ngưỡng đo được |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | `0`: che `0%` (thấy trọn biên).<br>`1`: che `1% – 30%` diện tích.<br>`2`: che `31% – 70%` diện tích, vẫn nhận dạng chắc chắn loại vật.<br>`3`: che `> 70%` diện tích, chỉ lộ góc nhỏ nhưng đủ cơ sở nhận dạng. |
| `truncated` | `no`, `yes` | `no`: nằm trọn trong khung ảnh.<br>`yes`: chạm mép ảnh và bị mất `>= 5%` diện tích thực tế hoặc mất bộ phận kết cấu chính. |
| `can_review` | `false`, `true` | Tick `true` khi còn lưỡng lự về lớp/biên để đưa ra kiểm tra chéo ở Pha 4–5. |

---

## 3. Quy tắc vẽ box (mã quy tắc `R-01` – `R-10`)

- **`R-01` — Người đi xe hai/ba bánh:** Tách thành 2 box giao cắt: box `person` ôm từ đỉnh mũ bảo hiểm/đầu xuống điểm thấp nhất cơ thể tiếp xúc yên xe hoặc mặt đường (không kéo xuống đáy bánh xe); box phương tiện (`motorcycle`/`bicycle`/`three_wheeler`) ôm trọn lốp, tay lái, gương và ống xả.
- **`R-02` — Độ khít và phần nhìn thấy:** Box Rectangle ôm khít các điểm cực biên nhìn thấy ngoài cùng (dung sai `<= 2 px`), không vẽ dư khoảng trống lớn và không suy đoán phần bị che hoàn toàn.
- **`R-03` — Xe chở xe & phụ kiện trên nóc:** Với xe lồng chở ô tô (`img_002.jpg`), vẽ `1 truck` cho toàn bộ xe chở + rơ-moóc và vẽ thêm từng `car` cho mỗi xe con trên 2 sàn (`occluded_level` 1–2). Với xe gắn mô hình đồ chơi nhỏ trên nóc (`img_024.jpg`), chỉ vẽ `1 car` cho xe thật, bỏ qua xe đồ chơi.
- **`R-04` — Vật phi thực (tượng, ma-nơ-canh, biển quảng cáo, phản chiếu):**
  - Loại theo quy tắc (không gán box): tượng người (`img_017.jpg`), tượng nhân vật (`img_011.jpg`), ma-nơ-canh (`img_033.jpg`), ảnh in người/xe trên biển quảng cáo (`img_018.jpg`, `img_026.jpg`).
  - Khoanh Rectangle `ignore_region`: bia mộ đá hình xe bán tải (`img_027.jpg`), tủ kính trưng bày đồ chơi (`img_038.jpg`), lòng gương chiếu hậu phản chiếu xe (`img_020.jpg`).
- **`R-05` — Vật nhỏ `< 20 px`, đám đông & thùng xe chủ:** Đối tượng đơn lẻ có chiều cao hoặc rộng `< 20 px` ở hậu cảnh sâu thì bỏ qua. Cụm kẹt xe/bãi xe chen chúc ở xa (`img_021.jpg`, `img_030.jpg`): gán đủ xe hàng tiền cảnh nhìn rõ (`> 30%`), phần chen đặc phía sau khoanh bằng Rectangle `ignore_region`. Khu vực thùng xe chủ trước camera (`img_012.jpg`) khoanh `ignore_region`.
- **`R-06` — Ranh giới `car` và `truck` (van, bán tải, chở hàng cao quá nóc):**
  - Xe van chở khách ≤ 9 chỗ và bán tải tiêu chuẩn → `car`.
  - Xe van chở hàng thùng kín, bán tải gắn nắp thùng kỹ thuật cao vượt trần cabin (`img_037.jpg`), và xe bán tải cổ chở tượng ngựa khổng lồ (`img_028.jpg`, vẽ 1 box ôm trọn thân xe + đỉnh tai tượng ngựa) → `truck`.
- **`R-07` — Hàng hóa siêu dài nối giữa hai xe (`img_029.jpg`):** Gán tách 2 cụm phương tiện (`person`+`motorcycle` phía trước, `person`+`bicycle` phía sau); khoảng không gian chứa thanh kim loại nối giữa 2 xe khoanh Rectangle `ignore_region`.
- **`R-08` — Phân biệt `three_wheeler`, `motorcycle` và `bicycle`:** Tuk-tuk, xe lam, xe ba gác có động cơ → `three_wheeler`; xe máy 2 bánh và mô tô gắn thùng sidecar (`img_022.jpg`) → `motorcycle`; xe đạp và xích lô đạp chân (`img_023.jpg`) → `bicycle`.
- **`R-09` — Gán thuộc tính chuẩn hóa:** Gán `occluded_level` theo đúng ngưỡng `0%` / `1–30%` / `31–70%` / `>70%` trong Panel Objects (không dùng phím tắt `Q` mặc định của CVAT); gán `truncated=yes` khi mất `>= 5%` hoặc mất kết cấu chính ở mép ảnh.
- **`R-10` — Ca lưỡng lự khi gán độc lập:** Tick `can_review` trên box nếu đã vẽ được box; nếu chưa vẽ được box thì ghi nhận vào `submission/qc/image_issues.csv` (`image,annotator,region,note`) sau khi khóa nhãn. Ảnh không có đối tượng hợp lệ vẫn giữ nguyên trong Task để xuất COCO 0 annotation.

---

## 4. Ca khó đã chốt (cho Pha 3)

| Ảnh | Tình huống | Rule ID | Quyết định nhóm G06 | Lý do |
|---|---|---|---|---|
| `img_002.jpg` | Xe lồng chở 8 ô tô con 2 tầng | `R-03`, `R-09` | Gán `1 truck` + `8 car` (`occluded_level` 1–2) | Theo dõi đầy đủ tải trọng và từng xe được chở. |
| `img_004.jpg`, `img_016.jpg`, `img_035.jpg` | Xe tuk-tuk 3 bánh | `R-08` | Gán `three_wheeler` | Tách đặc thù động học 3 bánh khỏi xe 2 bánh và ô tô 4 bánh. |
| `img_009.jpg`, `img_031.jpg`, `img_040.jpg` | Xe buýt / xe khách ≥ 10 chỗ | `§2.1` | Gán `bus` | Kích thước và hành vi dừng đỗ khác biệt với `car`/`truck`. |
| `img_017.jpg`, `img_033.jpg`, `img_018.jpg` | Tượng người, ma-nơ-canh, biển quảng cáo | `R-04` | Không gán `person` (loại theo quy tắc) | Tránh nhận nhầm vật tĩnh phi thực là người thật. |
| `img_020.jpg`, `img_027.jpg`, `img_038.jpg` | Phản chiếu gương, bia mộ hình xe, tủ đồ chơi | `R-04` | Không gán `car`; khoanh Rectangle `ignore_region` | Ngăn dương tính giả / phanh ma tại vùng dễ gây nhầm. |
| `img_023.jpg` | Xích lô đạp chân | `R-08` | Gán `bicycle` (+ `person` lái xe) | Phương tiện thô sơ chạy sức người bằng bàn đạp. |
| `img_024.jpg` | Xe cổ gắn ô tô đồ chơi nhỏ trên nóc | `R-03` | Chỉ gán `1 car` cho xe thật bên dưới | Xe đồ chơi là phụ kiện trang trí, không phải phương tiện. |
| `img_028.jpg` | Bán tải cổ chở tượng ngựa cao quá nóc | `R-06` | Gán `1 truck` ôm trọn thân xe + đỉnh tượng ngựa | Hàng hóa quá khổ cao vượt cabin, cần bao trọn tĩnh không. |
| `img_029.jpg` | Xe máy và xe đạp cùng khiêng thanh sắt dài | `R-07`, `R-01` | Gán (`person`+`motorcycle`), (`person`+`bicycle`), thanh sắt giữa 2 xe khoanh `ignore_region` | Ngăn mô hình nhận nhầm khoảng trống giữa 2 xe là an toàn. |
| `img_037.jpg` | Bán tải gắn nắp thùng kỹ thuật cao | `R-06` | Gán `truck` | Hoán cải thùng kín cao vượt cabin mang đặc tính tải nhẹ. |

---

## 5. QC, giải quyết bất đồng và phiên bản

- **Giải quyết các điểm bất đồng giữa hai bản cá nhân (`guideline_175.md` vs `guideline_166.md`):**
  1. **Lớp `van`:** `166` đề xuất giữ `van` (9 lớp); `175` đề xuất gộp để tránh nhập nhằng. Nhóm chốt **bỏ lớp `van`** (gộp van chở khách ≤ 9 chỗ vào `car`, van chở hàng kín vào `truck`, `R-06`).
  2. **Phân biệt `three_wheeler` vs `bicycle` cho xích lô:** `166` xếp mọi xe 3 bánh (kể cả xích lô) vào `three_wheeler`; `175` tách theo động cơ. Nhóm chốt theo nguồn lực vận hành (`R-08`): có động cơ (tuk-tuk, xe ba gác máy) → `three_wheeler`; đạp chân (xe đạp, xích lô `img_023.jpg`) → `bicycle`.
  3. **Ngưỡng `occluded_level` & `truncated`:** `166` đề xuất mốc `0% / 1–25% / 25–50% / >50%` và `truncated=yes` khi chạm mép `>0%`; `175` đề xuất `0% / 1–30% / 31–70% / >70%` và `truncated=yes` khi mất `>= 5%` hoặc mất kết cấu chính. Nhóm chốt mốc của `175` (`R-09`) để tránh đánh cờ quá nhạy khi vật chỉ chạm nhẹ 1–2 px mép ảnh.
  4. **Vật phi thực & đám đông:** `166` chỉ nêu "không gán"; `175` tách rõ phần loại bỏ hẳn (`R-04`: tượng người, ma-nơ-canh, biển quảng cáo) và phần phải khoanh `ignore_region` (`R-04`, `R-05`: gương phản chiếu `img_020`, bia mộ hình xe `img_027`, tủ đồ chơi `img_038`, đám đông xa `img_021`, `img_030`).
  5. **Các ca biên đã chốt:** `img_028.jpg` → `truck` (`R-06`), `img_029.jpg` → thanh sắt khoanh `ignore_region` (`R-07`), `img_037.jpg` → `truck` (`R-06`), `img_009.jpg` (người mờ trên xe buýt trong `guideline_166.md`) → vẫn gán `person` nếu cao `>= 20 px` và nhận dạng được (`R-05`).
- **Ngưỡng QC (`taxonomy.json` -> `rules`):** giữ nguyên toàn bộ ngưỡng của lab (`min_box_height_px=20`, `iou_thr_large=0.85`, `iou_thr_small=0.7`, `large_px=50`, `iaa_iou=0.85`, `iaa_class=0.95`, `error_rate_img_max=0.02`, `match_iou=0.5`).
- **Quy trình calibration & duyệt cuối:** Sau Pha 3, `175` chạy so sánh IAA (`qc_compare.py`); hai thành viên họp Pha 5 chốt các ca bất đồng, và `166` thực hiện duyệt QC toàn lô ở Pha 6.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| `v0 → v1.0` | Bỏ `van` (gộp vào `car`/`truck`); thêm `bus`, `three_wheeler`, `bicycle`; chuẩn hóa ngưỡng `occluded_level`/`truncated`; gắn mã quy tắc `R-01`–`R-10`; chốt `img_028.jpg`=`truck`, `img_029.jpg`=`ignore_region` thanh sắt, `img_037.jpg`=`truck`. | Họp thống nhất Pha 2 giữa `175` và `166` trước khi khóa bộ nhãn CVAT v1.0 cho Pha 3. |
