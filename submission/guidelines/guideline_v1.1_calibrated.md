# Guideline gán nhãn — v1.1 (sau calibration) — G06

- Phiên bản: v1.1 (cập nhật diễn giải `editorial` sau họp calibration Pha 5)
- Ngày: 2026-11-17
- Nhóm: G06 (`175` — Điều phối, `166` — Người chốt calibration & Reviewer bản cuối)
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
| `car` | Phương tiện cơ giới 4 bánh chở người từ 9 chỗ trở xuống (sedan, hatchback, SUV, crossover, van chở khách ≤ 9 chỗ, bán tải tiêu chuẩn). | Ô tô con lưu thông (`img_015.jpg`, `img_034.jpg`), xe con trên giá chở (`img_002.jpg`). | Xe tải/van chở hàng (`truck`), xe buýt ≥ 10 chỗ (`bus`), xe mô hình trên nóc (`img_024.jpg`), bia mộ hình xe (`img_027.jpg` → `ignore_region`). |
| `truck` | Phương tiện cơ giới chuyên dùng chở hàng (xe tải thùng kín/mui bạt, đầu kéo rơ-moóc, van chở hàng kín, bán tải hoán cải thùng cao vượt trần cabin, xe cổ chở hàng cồng kềnh quá nóc). | Xe lồng chở ô tô (`img_002.jpg`), xe chở hàng lớn (`img_005.jpg`), xe tải thùng (`img_007.jpg`, `img_036.jpg`), bán tải nắp thùng cao (`img_037.jpg`), bán tải cổ chở tượng ngựa (`img_028.jpg`). | Bán tải gia đình tiêu chuẩn (`car`), xe ba gác máy (`three_wheeler`). |
| `bus` | Phương tiện cơ giới chở khách công cộng hoặc du lịch từ 10 chỗ ngồi trở lên, thân liền khối cỡ trung đến lớn. | Xe khách County (`img_009.jpg`), xe buýt đô thị (`img_031.jpg`, `img_040.jpg`). | Xe van chở khách ≤ 9 chỗ (`car`), xe tải thùng kín (`truck`). |
| `three_wheeler` | Phương tiện cơ giới 3 bánh chuyên chở khách hoặc hàng hóa (tuk-tuk, xe lam, xe ba gác máy và xe ba bánh đặc thù được chốt khi calibration). | Xe tuk-tuk (`img_004.jpg`, `img_016.jpg`, `img_035.jpg`, `img_040.jpg`), xe ba bánh (`img_023.jpg` chốt `three_wheeler` theo `166`). | Mô tô 2 bánh gắn sidecar (`motorcycle`, `img_022.jpg`), xe đạp 2 bánh (`bicycle`). |
| `motorcycle` | Phương tiện cơ giới 2 bánh (hoặc mô tô gắn thùng sidecar) chạy động cơ đốt trong/điện, điều khiển bằng ghi-đông. | Xe máy số/tay ga (`img_006.jpg`, `img_008.jpg`, `img_021.jpg`, `img_040.jpg`), mô tô sidecar (`img_022.jpg`). | Xe đạp 2 bánh (`bicycle`), xe tuk-tuk (`three_wheeler`). |
| `bicycle` | Phương tiện thô sơ vận hành chủ yếu bằng bàn đạp sức người (hoặc trợ lực nhẹ). | Xe đạp (`img_003.jpg`, `img_006.jpg`, `img_029.jpg`). | Xe máy điện/xe máy (`motorcycle`), xe đẩy hàng không bàn đạp. |
| `person` | Cá thể người thật ở mọi tư thế: đi bộ, đứng, ngồi, gánh hàng hoặc điều khiển phương tiện. | Người đi bộ (`img_034.jpg`), người gánh hàng (`img_003.jpg`), người lái xe máy/xe đạp (`img_006.jpg`, `img_029.jpg`). | Tượng người (`img_017.jpg`), ma-nơ-canh (`img_033.jpg`), ảnh in người trên biển quảng cáo (`img_018.jpg`, `img_026.jpg`). |
| `ignore_region` | Vùng bỏ qua (không phải vật): cụm xe dày đặc ở xa không tách được box, gương phản chiếu xe, vật phi thực cỡ lớn dễ gây nhiễu, hàng hóa nối dài nguy hiểm, thùng xe chủ. | Kẹt xe/bãi xe xa (`img_003.jpg`, `img_008.jpg`, `img_011.jpg`, `img_013.jpg`, `img_019.jpg`, `img_021.jpg`, `img_025.jpg`, `img_030.jpg`), gương phản chiếu (`img_020.jpg`), tủ kính đồ chơi (`img_038.jpg`), bia mộ hình xe (`img_027.jpg`), thanh sắt giữa 2 xe (`img_029.jpg`), thùng xe chủ (`img_012.jpg`). | Người hoặc xe đứng độc lập ở cự ly gần có biên rõ ràng. |

### 2.2. Thuộc tính (áp dụng cho mọi lớp trừ `ignore_region`)

| Thuộc tính | Giá trị | Ngưỡng đo được (làm rõ cho v1.1) |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | `0`: che `0%` (thấy trọn biên).<br>`1`: che `1% – 30%` diện tích.<br>`2`: che `31% – 70%` diện tích, vẫn nhận dạng chắc chắn loại vật.<br>`3`: che `> 70%` diện tích, chỉ lộ góc nhỏ nhưng đủ cơ sở nhận dạng.<br>*(Calibration: rà lại các box mặc định để `0` khi thực tế bị vật trước che.)* |
| `truncated` | `no`, `yes` | `no`: nằm trọn trong khung ảnh.<br>`yes`: chạm mép ảnh và bị mất `>= 5%` diện tích thực tế hoặc mất bộ phận kết cấu chính. |
| `can_review` | `false`, `true` | Chỉ dùng lúc gán độc lập; **bắt buộc chuyển về `false`** sau calibration ở bản cuối. |

---

## 3. Quy tắc vẽ box (mã quy tắc `R-01` – `R-10`)

- **`R-01` — Người đi xe hai/ba bánh:** Tách thành 2 box giao cắt: box `person` ôm từ đỉnh mũ bảo hiểm/đầu xuống điểm thấp nhất cơ thể tiếp xúc yên xe hoặc mặt đường (không kéo xuống đáy bánh xe); box phương tiện ôm trọn lốp, tay lái, gương và ống xả.
- **`R-02` — Độ khít và phần nhìn thấy:** Box Rectangle ôm khít các điểm cực biên nhìn thấy ngoài cùng (dung sai `<= 2 px`), không vẽ dư khoảng trống lớn và không suy đoán phần bị che hoàn toàn (đã rà lại ở `img_008`, `img_014`, `img_016`, `img_021`).
- **`R-03` — Xe chở xe & phụ kiện trên nóc:** Với xe lồng chở ô tô (`img_002.jpg`), vẽ `1 truck` cho toàn bộ xe chở + rơ-moóc và vẽ thêm từng `car` cho mỗi xe con trên 2 sàn (`occluded_level` 1–2). Với xe gắn mô hình đồ chơi nhỏ trên nóc (`img_024.jpg`), chỉ vẽ `1 car` cho xe thật, bỏ qua xe đồ chơi.
- **`R-04` — Vật phi thực:** Tượng người (`img_017.jpg`), ma-nơ-canh (`img_033.jpg`), biển quảng cáo (`img_018.jpg`, `img_026.jpg`) loại bỏ hoàn toàn; bia mộ hình xe (`img_027.jpg`), tủ kính đồ chơi (`img_038.jpg`), gương phản chiếu (`img_020.jpg`) khoanh `ignore_region`.
- **`R-05` — Vật nhỏ `< 20 px`, đám đông & thùng xe chủ:** Đối tượng đơn lẻ `< 20 px` bỏ qua; cụm kẹt xe/hậu cảnh dày đặc giữ nguyên các vùng `ignore_region` theo bản `166` (`img_003`, `img_008`, `img_011`, `img_013`, `img_019`, `img_021`, `img_023`, `img_025`, `img_030`).
- **`R-06` — Ranh giới `car` và `truck`:** Van chở khách ≤ 9 chỗ & bán tải tiêu chuẩn → `car`; van chở hàng kín, xe chở lớn (`img_005.jpg` chốt `truck`), bán tải nắp thùng cao (`img_037.jpg`), bán tải cổ chở tượng ngựa (`img_028.jpg` ôm trọn đến đỉnh tượng ngựa) → `truck`.
- **`R-07` — Hàng hóa siêu dài nối giữa hai xe (`img_029.jpg`):** Tách 2 cụm phương tiện và khoanh `ignore_region` phần thanh sắt nối giữa 2 xe.
- **`R-08` — Phân biệt `three_wheeler`, `motorcycle` và `bicycle`:** Xe 3 bánh tuk-tuk (`img_004`, `img_016`, `img_035`, `img_040`) và xe 3 bánh ở `img_023.jpg` chốt `three_wheeler` theo quyết định `166`; xe máy 2 bánh/sidecar (`img_022.jpg`) → `motorcycle`; xe đạp 2 bánh → `bicycle`.
- **`R-09` — Gán thuộc tính chuẩn hóa:** Gán `occluded_level` theo đúng ngưỡng `0%` / `1–30%` / `31–70%` / `>70%` và `truncated=yes` khi mất `>= 5%` hoặc mất kết cấu chính ở mép ảnh.
- **`R-10` — Xử lý `can_review` bản cuối:** Tắt toàn bộ `can_review` về `false` sau khi đã kiểm lại các ca đánh dấu từ `166`.

---

## 4. Tổng hợp quyết định Calibration (436 ca trên 38 ảnh)

| Nhóm ca | Số lượng | Người chốt | Quyết định chuẩn hóa cho bản cuối | Mã quy tắc |
|---|---:|---|---|---|
| `sai_lop` | 4 | `166` | `C051` (`img_005.jpg`) → `truck`; `C196` (`img_016.jpg`), `C280` (`img_023.jpg`), `C428` (`img_040.jpg`) → `three_wheeler` theo `166`. | `R-06`, `R-08` |
| `iou_thap` | 5 | `166` | Chỉnh ôm sát phần nhìn thấy (`img_008`, `img_014`, `img_016`, `img_021`), giữ `truck` ôm trọn đỉnh tượng ngựa ở `img_028.jpg`. | `R-02`, `R-06` |
| `bat_dong_ignore` | 13 | `166` | Giữ toàn bộ vùng `ignore_region` của `166` trên các ảnh đám đông/xa và giữ `ignore_region` các ca phi thực (`img_020`, `img_027`, `img_038`). | `R-04`, `R-05` |
| `bo_sot_thua` | 47 | `166` | Xóa box thừa/phi thực, bổ sung các vật thật bị sót (`>= 20 px`) đối chiếu theo ảnh gốc. | `R-04`, `R-05` |
| `co_can_review` | 32 | `166` | Rà soát xác nhận từng ca do `166` đánh dấu và tắt toàn bộ `can_review` về `false`. | `R-10` |
| `sai_thuoc_tinh` | 335 | `166` | Chuẩn hóa đồng nhất `occluded_level` (`0`–`3`) và `truncated` (`no`/`yes`) theo `R-09`. | `R-09` |

---

## 5. QC và lịch sử phiên bản

- Giữ nguyên toàn bộ ngưỡng QC trong `taxonomy.json` -> `rules` (`min_box_height_px=20`, `iou_thr_large=0.85`, `iou_thr_small=0.7`, `large_px=50`, `iaa_iou=0.85`, `iaa_class=0.95`, `error_rate_img_max=0.02`, `match_iou=0.5`).
- Sau khi sửa nhãn ở `work/calib/`, `166` (reviewer bản cuối) duyệt toàn bộ 40 ảnh ở Pha 6.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| `v0 → v1.0` | Chốt 8 lớp (bỏ `van`; thêm `bus`, `three_wheeler`, `bicycle`), định lượng thuộc tính, mã quy tắc `R-01`–`R-10`. | Họp thống nhất Pha 2 (`175` & `166`). |
| `v1.0 → v1.1` | `editorial`: giữ nguyên schema 8 lớp; làm rõ ghi chú áp dụng `R-04`, `R-06` (`img_005`=`truck`, `img_028` ôm đỉnh tượng), `R-08` (`three_wheeler` theo `166` ở `img_016`, `img_023`, `img_040`), `R-09` (`occluded_level`/`truncated`), `R-10` (tắt `can_review`). | Calibration 436 ca trên 38 ảnh (người chốt: `166`). |
