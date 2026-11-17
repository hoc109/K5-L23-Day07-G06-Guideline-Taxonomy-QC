# Guideline gán nhãn — v0.1 (cá nhân) — 175

- Phiên bản: v0.1 (cá nhân — Pha 1)
- Ngày: 2026-11-16
- Người viết / duyệt: 175
- Công cụ: CVAT (`http://localhost:8080`), xuất COCO 1.0 (bounding box chữ nhật / Rectangle)

---

## 1. Mục tiêu

Tài liệu thiết lập quy chuẩn gán nhãn 2D Bounding Box trên ảnh đường phố phục vụ bài toán nhận diện chướng ngại vật giao thông cho hệ thống trợ lái nâng cao (ADAS Cấp độ 2+ đến 4) và giám sát giao thông đô thị thông minh (ITS).

- **Yêu cầu sử dụng:** Box phải bám sát ranh giới vật lý nhìn thấy (để ước lượng khoảng cách an toàn / phanh khẩn cấp AEB, đếm và phân luồng phương tiện).
- **Lỗi nguy hiểm nhất cần tránh:**
  1. **Bỏ sót đối tượng yếu thế (False Negative - VRU):** bỏ sót người đi bộ, người bán hàng rong, người đi xe đạp/xe máy dễ dẫn tới không kích hoạt phanh khẩn cấp.
  2. **Sai ranh giới hàng hóa cồng kềnh:** vẽ box chỉ ôm cabin mà bỏ qua hàng hóa vươn rộng (thanh sắt, hàng cồng kềnh) làm sai tĩnh không an toàn.
  3. **Gán nhầm vật thể giả (False Positive — "Phanh ma"):** gán nhãn xe cho bia mộ hình ô tô, phản chiếu trong gương, tranh quảng cáo hoặc mô hình đồ chơi.

---

## 2. Lớp và thuộc tính

### 2.1. Lớp (mỗi vật thuộc đúng một lớp)

Đề xuất giữ 5 lớp nền tảng (`car`, `truck`, `motorcycle`, `person`, `ignore_region`), **tách thêm** `bus`, `bicycle`, `three_wheeler` (xe ba bánh / tuk-tuk), và **gộp `van` cỡ nhỏ chở khách (7 chỗ) vào `car`** hoặc **xe tải van chở hàng thùng kín vào `truck`** (đề xuất đưa ra họp Pha 2 để thống nhất với `166` có giữ lớp `van` riêng hay không).

| Lớp | Định nghĩa (một câu) | Ví dụ trong bộ ảnh | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Phương tiện cơ giới 4 bánh chở người từ 9 chỗ trở xuống (sedan, hatchback, SUV, crossover, van chở khách 7 chỗ, bán tải tiêu chuẩn). | Ô tô con trên đường (`img_015.jpg`, `img_023.jpg`, `img_034.jpg`), các xe con trên sàn xe chở (`img_002.jpg`). | Xe tải chuyên dụng (`truck`), xe buýt mini (`bus`), xe đồ chơi trên nóc (`img_024.jpg`), bia mộ hình xe (`img_027.jpg` → `ignore_region`). |
| `truck` | Phương tiện cơ giới chuyên dùng chở hàng (xe tải thùng kín, mui bạt, đầu kéo rơ-moóc, bán tải hoán cải thùng kỹ thuật cao vượt trần cabin). | Xe lồng chở ô tô (`img_002.jpg`), xe tải thùng kín (`img_007.jpg`), xe tải mui bạt (`img_036.jpg`), bán tải thùng cao (`img_037.jpg`). | Bán tải gia đình thông thường (`car`), xe ba gác máy (`three_wheeler`). |
| `van` *(đề xuất gộp vào `car`/`truck` hoặc giữ để họp Pha 2)* | Xe thùng kín cỡ nhỏ chuyên chở người/hàng nhẹ (cần chốt ở Pha 2 ranh giới với `car` 7 chỗ và `truck` nhỏ). | Xe van thương mại cỡ nhỏ (nếu tách riêng khỏi `car`/`truck`). | Xe SUV/MPV gia đình (`car`), xe tải thùng lớn (`truck`). |
| `bus` *(đề xuất thêm)* | Phương tiện cơ giới chở khách công cộng hoặc du lịch từ 10 chỗ ngồi trở lên, thân liền khối cỡ trung đến lớn. | Xe khách du lịch County (`img_009.jpg`), xe buýt đô thị (`img_031.jpg`, `img_040.jpg`). | Xe van 7 chỗ (`car`/`van`), xe tải thùng kín (`truck`). |
| `three_wheeler` *(đề xuất thêm)* | Xe cơ giới 3 bánh (tuk-tuk, xe lam, xe ba gác máy chở hàng). | Tuk-tuk (`img_004.jpg`, `img_016.jpg`, `img_035.jpg`). | Xe đạp xích lô không động cơ (`bicycle`), mô tô sidecar 3 bánh (`motorcycle`, `img_022.jpg`). |
| `motorcycle` | Phương tiện cơ giới 2 bánh (hoặc sidecar có khung xe máy) vận hành bằng động cơ đốt trong hoặc điện, điều khiển bằng ghi-đông. | Xe máy số, tay ga đô thị (`img_006.jpg`, `img_008.jpg`, `img_021.jpg`, `img_040.jpg`), mô tô sidecar (`img_022.jpg`). | Xe đạp/xích lô đạp chân (`bicycle`), xe đồ chơi. |
| `bicycle` *(đề xuất thêm)* | Phương tiện thô sơ 2 hoặc 3 bánh vận hành chủ yếu bằng sức người qua bàn đạp (gồm cả xích lô). | Xe đạp (`img_003.jpg`, `img_006.jpg`, `img_029.jpg`), xích lô (`img_023.jpg`). | Xe máy điện tốc độ cao (`motorcycle`), xe đẩy hàng không bàn đạp. |
| `person` | Cá thể người thật ở mọi tư thế: đi bộ, đứng, ngồi, gánh hàng hoặc điều khiển phương tiện. | Người đi bộ (`img_034.jpg`), người gánh hàng rong (`img_003.jpg`), người lái xe máy/xe đạp (`img_006.jpg`, `img_029.jpg`). | Tượng người (`img_017.jpg`), ma-nơ-canh (`img_033.jpg`), hình người trên pano quảng cáo (`img_018.jpg`, `img_026.jpg`). |
| `ignore_region` | Vùng bỏ qua (không phải vật): đám đông/kẹt xe dày đặc ở xa không tách nổi từng box, gương chiếu hậu phản chiếu xe, vật phi thực cỡ lớn dễ gây nhiễu, thùng xe chủ. | Đám đông xe máy xa (`img_021.jpg`, `img_030.jpg`), gương phản chiếu (`img_020.jpg`), tủ kính đồ chơi (`img_038.jpg`), bia mộ hình xe (`img_027.jpg`), thùng xe chủ (`img_012.jpg`). | Người hoặc xe đứng độc lập ở cự ly gần có ranh giới tách biệt rõ. |

**Vật ngoài danh mục lớp:**
- Các vật phi thực cỡ nhỏ/phẳng không gây nhầm với đối tượng giao thông thật (tranh biển quảng cáo, tượng trang trí nhỏ) → **loại theo quy tắc** (không gán).
- Các vùng dễ làm mô hình nhận nhầm (gương phản chiếu xe, tủ kính mô hình, bia mộ hình xe kích thước thật, cụm kẹt xe quá dày) → khoanh Rectangle `ignore_region`.
- Xe cộ đặc thù khác (VD: xe cổ chở tượng lớn `img_028.jpg`, thanh sắt vắt ngang giữa 2 xe `img_029.jpg`) → đánh dấu `can_review` / đưa ra họp Pha 2.

### 2.2. Thuộc tính

Mỗi box đối tượng (trừ `ignore_region`) gắn đủ 3 thuộc tính sau (khớp tên trường và giá trị trong `taxonomy.json`):

| Thuộc tính | Giá trị cố định | Ý nghĩa & ngưỡng đo được |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | **`0`**: che `0%` (thấy trọn ranh giới các cạnh, VD người đi bộ `img_034.jpg`, xe tải `img_028.jpg`).<br>**`1`**: che `1% – 30%` diện tích (che nhẹ bởi lan can, cột, mép xe khác; không ảnh hưởng kết cấu chính, VD xe con trên sàn `img_002.jpg`).<br>**`2`**: che `31% – 70%` diện tích nhưng vẫn nhận dạng rõ loại vật nhờ đặc trưng mui/đèn/kính (VD taxi xanh `img_040.jpg`).<br>**`3`**: che `> 70%` diện tích, chỉ lộ góc nhỏ như gương/nóc/một phần bánh nhưng vẫn chắc chắn loại vật (VD xe máy lớp sau `img_021.jpg`). |
| `truncated` | `no`, `yes` | **`no`**: toàn bộ đối tượng nằm trọn trong khung ảnh.<br>**`yes`**: đối tượng chạm mép ảnh và bị cắt mất từ `>= 5%` diện tích thực tế hoặc mất bộ phận kết cấu chính (VD xe đỏ mép trái `img_015.jpg`, người lái xe máy mép dưới `img_040.jpg`, rơ-moóc chạm mép phải `img_002.jpg`). |
| `can_review` | `false`, `true` (checkbox) | Tick `true` khi gặp ca biên chưa chắc chắn về lớp/ranh giới box để đưa ra hiệu chỉnh/họp nhóm. |

---

## 3. Quy tắc vẽ box (Rectangle 2D trong CVAT)

1. **Độ khít & phần nhìn thấy:** Box ôm sát các điểm cực biên nhìn thấy ngoài cùng của vật thể (mép lốp, gương, mui, đỉnh mũ bảo hiểm), dung sai `<= 2 px`, không thừa khoảng trống lớn và không cắt lẹm thân vật thể.
2. **Người đi xe máy / xe đạp:** Bắt buộc **tách 2 box giao cắt**:
   - Box `person`: từ đỉnh mũ bảo hiểm (hoặc đỉnh đầu) xuống đến điểm thấp nhất của cơ thể tiếp xúc với xe/mặt đường (yên xe hoặc bàn chân chống đất), **không** kéo xuống đáy bánh xe.
   - Box `motorcycle` / `bicycle`: ôm trọn kết cấu xe (mép ngoài 2 lốp, tay lái, gương, ống xả).
3. **Xe chở xe (`img_002.jpg`):** Vẽ 01 box lớn `truck` cho toàn bộ xe đầu kéo + rơ-moóc hai tầng, đồng thời vẽ **từng box `car` riêng** cho mỗi ô tô con trên hai sàn (kèm `occluded_level = 1` hoặc `2`).
4. **Xe gắn vật trang trí nhỏ (`img_024.jpg`):** Vẽ 01 box `car` cho xe thật bên dưới; xe mô hình nhỏ gắn trên nóc coi là phụ kiện, **loại theo quy tắc** (không gán `car` riêng).
5. **Vật không thật (loại theo quy tắc vs `ignore_region`):**
   - Tượng điêu khắc người/nhân vật (`img_017.jpg`, `img_011.jpg`), ma-nơ-canh (`img_033.jpg`), ảnh in trên biển quảng cáo (`img_018.jpg`, `img_026.jpg`): **không gán** `person`/`car` (loại theo quy tắc).
   - Bia mộ hình xe kích thước lớn (`img_027.jpg`), tủ kính xe mô hình (`img_038.jpg`), gương chiếu hậu phản chiếu xe (`img_020.jpg`): không gán lớp phương tiện, khoanh Rectangle `ignore_region`.
6. **Vật quá nhỏ (`< 20 px`), đám đông & thùng xe chủ:**
   - Vật đơn lẻ có chiều cao (hoặc chiều rộng) `< 20 px` ở hậu cảnh sâu: bỏ qua không gán.
   - Cụm kẹt xe/bãi xe chen chúc ở xa không tách nổi biên (`img_021.jpg`, `img_030.jpg`): gán đủ xe hàng tiền cảnh nhìn rõ (`> 30%`), phần cụm dày đặc phía sau khoanh bằng một hoặc nhiều Rectangle `ignore_region`.
   - Thùng xe chủ phía trước camera (`img_012.jpg`): khoanh `ignore_region` để tránh nhầm hàng hóa trên xe với chướng ngại vật mặt đường.

---

## 4. Ca khó (từ khảo sát 40 ảnh)

| Ảnh | Tình huống | Rule ID (dự kiến) | Hướng xử lý (loại / quyết định / hỏi reviewer) | Lý do |
|---|---|---|---|---|
| `img_002.jpg` | Xe lồng chở 8 ô tô con hai tầng | `R-01` (dự kiến) | Quyết định gán `1 truck` + `8 car` (`occluded_level` 1–2) | Cần theo dõi từng xe tải trọng phòng sự cố và đúng quy tắc xe chở xe. |
| `img_020.jpg` | Ô tô con phản chiếu trong gương chiếu hậu trời mưa | `R-02` (dự kiến) | Loại lớp `car`; khoanh `ignore_region` mặt gương | Phản chiếu không tồn tại ở tọa độ thực phía trước xe, tránh gây phanh ma. |
| `img_027.jpg` | Bia mộ đá hoa cương điêu khắc hình xe bán tải | `R-03` (dự kiến) | Loại lớp `car`; khoanh `ignore_region` khối tượng đá | Vật tĩnh phi thực kích thước ngang xe thật dễ gây dương tính giả. |
| `img_028.jpg` | Xe bán tải cổ chở tượng ngựa thạch cao khổng lồ cao quá nóc | `R-04` (dự kiến) | Đề xuất gộp 1 box `car` ôm cả xe + tượng ngựa (**cần hỏi reviewer Pha 2** chốt `car` hay `truck`) | Không gán lớp động vật; phải ôm hết đỉnh tượng để giữ đúng tĩnh không vật lý. |
| `img_029.jpg` | Hai người đi xe máy và xe đạp cùng khiêng thanh kim loại siêu dài vắt ngang đường | `R-05` (dự kiến) | Tách cụm trước (`person`+`motorcycle`), cụm sau (`person`+`bicycle`), phần thanh kim loại ở giữa khoanh `ignore_region` (**cần hỏi reviewer Pha 2**) | Tránh mô hình tưởng khoảng trống giữa 2 xe là an toàn để lách qua. |
| `img_037.jpg` | Bán tải gắn nắp thùng kỹ thuật cao vượt trần cabin | `R-06` (dự kiến) | Xếp vào `truck` (**cần hỏi reviewer Pha 2** đối chiếu với `van`/`car`) | Hình khối và công năng chở hàng giống xe tải nhẹ hơn xe con. |

---

## 5. QC và phiên bản

- **Ngưỡng QC áp dụng (`taxonomy.json` -> `rules`):**
  - Box cao `>= 50 px`: ngưỡng IoU `>= 0.85`; box cao `< 50 px`: ngưỡng IoU `>= 0.70`; chiều cao tối thiểu đối tượng đơn lẻ: `>= 20 px`.
  - Mục tiêu IAA trước calibration: IoU trung bình `>= 0.85`, đồng thuận lớp `>= 95%` (tính trên cặp box ghép 1–1 có IoU `>= 0.50`).
  - Điều kiện Release lô: duyệt toàn bộ 40/40 ảnh, tỷ lệ ảnh lỗi `<= 2%` (tức `0/40` ảnh lỗi) và không có lỗi lặp mang tính hệ thống.
- **Người quyết định ca mơ hồ:** Thành viên đánh dấu `can_review` hoặc ghi `image_issues.csv` khi gán độc lập; hai thành viên (`175`, `166`) họp chốt ở Pha 2 / Pha 5, `166` là người review bản cuối.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| `v0 → v0.1 (175)` | Định lượng `occluded_level` (`0`–`3` theo %) và `truncated` (`>= 5%` / mất kết cấu chính); bổ sung ví dụ, phản ví dụ, đề xuất tách lớp `bus`/`bicycle`/`three_wheeler`; chốt quy tắc tách người–xe, xe chở xe, vật phi thực, đám đông và 6 ca khó. | Khảo sát độc lập 40 ảnh ở Pha 1 (`img_002`, `img_012`, `img_017`, `img_020`, `img_021`, `img_024`, `img_026`–`img_030`, `img_033`, `img_037`, `img_038`, `img_040`). |
