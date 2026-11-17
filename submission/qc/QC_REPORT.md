# Báo cáo QC — G06

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Đặc tả và QC nhãn 2D Bounding Box (người và phương tiện đường phố) phục vụ huấn luyện/đánh giá bài toán nhận diện giao thông; tập 40 ảnh tuyển chọn nhiều ca biên nên độ đại diện cho dữ liệu vận hành thực tế chưa được đánh giá đầy đủ.
- Nguồn và phép sử dụng dữ liệu: Toàn bộ 40 ảnh nguồn Wikimedia Commons (`CC BY`, `CC BY-SA`, `CC0`, Public domain) kèm đầy đủ ghi công tác giả và URL gốc trong `data/ATTRIBUTION.csv`.
- Manifest và checksum nhãn cuối: `submission/qc/batch_manifest.json` — SHA-256 của `submission/qc/labels_calibrated.json` là `76bcb0112655c5bd43060e513ce3e5b4e2e12277da7185ff7328ac5fe19ef76f` (`labels_calibrated.csv`: `f6052f095e1a62613925e4d3095b5d6d91a64ee66974061fc54a5a63712f632e`).
- Guideline và taxonomy khi gán / khi bàn giao: Gán độc lập bằng `guideline_v1.0_agreed.md` + `submission/taxonomy.json` (`v1.0`, 8 lớp); bàn giao theo `guideline_v1.1_calibrated.md` + `submission/guideline_change.json` (`kind: editorial`, giữ nguyên taxonomy `v1.0`).

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 80/80 lượt (2 thành viên `166` và `175`, mỗi ảnh 2 người gán độc lập trên 40 ảnh).
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 419 cặp box ghép (IoU >= 0.5); IoU trung bình = `0.986` (`>= 0.85`), đồng thuận lớp = `99.0%` (`>= 95%`), đồng thuận `occluded_level` = `23.9%`, đồng thuận `truncated` = `86.4%`.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: Chưa ghép `27/20` (`166`/`175`), loại do ignore `0`, bất đồng vùng `ignore_region`: `13` ca (`10` vùng đám đông xa từ `166`, `3` vùng phi thực từ `175`).
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: Các ca dùng pre-label trước khi chỉnh tay có thể dẫn tới đồng thuận cao giả tạo ở những box nền hoặc sai thuộc tính hệ thống (`occluded_level=0` mặc định bên `166`); do đó cần reviewer đối chiếu trực tiếp ảnh gốc ở `work/review/round-01/` để phát hiện lỗi đồng thuận sai (ví dụ tượng/biển quảng cáo/vật nhỏ `< 20 px`).

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: Đã chốt toàn bộ `436/436` ca trên `38` ảnh bị cờ trong `submission/qc/calibration_decisions.csv` (người chốt: `166`; dẫn chiếu `R-02`, `R-04`, `R-05`, `R-06`, `R-08`, `R-09`, `R-10`). Bản sửa sau calibration lưu tại `submission/qc/labels_calibrated_98f53c2a24ea0f3d.json`, gộp thành `submission/qc/labels_calibrated.json` (`40` ảnh, `479` annotation: `466` đối tượng và `13` `ignore_region`, `0` box bật `can_review`).
- Tác động v1.1 lên nhãn và ảnh: Thay đổi mức `editorial` (`guideline_change.json`), phạm vi chỉnh sửa trên `38` ảnh bị cờ (`img_017.jpg` và `img_033.jpg` giữ nguyên đồng thuận 0 đối tượng); áp dụng quyết định `C051` (`img_005.jpg` → `truck`), `C196`/`C280`/`C428` (`img_016`/`img_023`/`img_040` → `three_wheeler`), `C317` (`img_028.jpg` → `truck` ôm trọn đến đỉnh tượng ngựa), giữ đủ `13` `ignore_region`, loại các box bị phủ `>= 50%` bởi `ignore_region`, và tắt toàn bộ `can_review`.
- Tham chiếu bằng chứng khi Rework: Chưa phát sinh vòng Rework (`round-01` mới khởi tạo).

## 4. Kết quả duyệt

- Vòng QC: 1
- Người review và ngày: 166, 2026-11-17 (đang chờ duyệt phiếu `qc_review.csv`)
- Đã duyệt / cả lô: 0/40 ảnh đã duyệt trên phiếu (`40` ảnh trong `work/review/round-01/` đã xuất đầy đủ, chưa điền `submission/qc/reviews/round-01/qc_review.csv`).
- Ảnh lỗi / ảnh đã duyệt: 0/0 (N/A — chưa có dòng duyệt thực tế trong `qc_review.csv`).
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: 0/0 (N/A — `466` đối tượng trên lô đang chờ duyệt).
- Vật bỏ sót, ảnh và vùng: Chưa ghi nhận vì chưa hoàn tất duyệt mắt trên `round-01`.
- Ca khó / phần còn lại, lỗi lặp và ví dụ: Dẫn chiếu `submission/qc/reviews/round-01/qc_review.csv` và `qc_result.json` (hiện báo `HOLD` do chưa có kết quả duyệt cho `img_001.jpg` – `img_040.jpg`). Lưu ý rà lại 5 box `< 20 px` từng cảnh báo ở `166.json` (`img_009`, `img_021`, `img_025`).
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Máy (`validate_labels.py` / `qc_compare.py`) chặn được lớp `van` ngoài taxonomy ở `166.json` ban đầu và `4` ca `sai_lop` + `335` ca lệch thuộc tính; mắt người đọc ảnh cần kiểm tra độ ôm sát `<= 2 px`, vật bỏ sót ở đám đông, và xác nhận `three_wheeler` / `ignore_region` trên `work/review/round-01/`.

## 5. Quyết định bàn giao

- Quyết định: Hold
- Lý do, người nhận và phạm vi sử dụng: Tạm giữ (`Hold`) để chờ reviewer `166` kiểm đủ 40 ảnh trong `work/review/round-01/`, điền `submission/qc/reviews/round-01/qc_review.csv` và hoàn thiện các mục còn thiếu trong `submission/individual_notes/166.md`; hồ sơ và nhãn hiện chỉ dùng nội bộ nhóm G06 để hoàn tất nghiệm thu trước khi bàn giao cho coach.
- Nếu còn việc: `166` duyệt 40 ảnh tại `work/review/round-01/` điền `qc_review.csv` và hoàn tất `166.md`, sau đó chạy lại `python scripts/qc_report.py --round 1` (nếu đạt `<= 2%` lỗi và không lỗi lặp sẽ cập nhật quyết định sang `Release`; nếu vượt ngưỡng sẽ sửa Pha 5 và mở `--round 2`).
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: Truy vết ca `C051` từ báo cáo → `submission/qc/calibration_decisions.csv#C051` (`R-06`, `nguoi_chot=166`) → ảnh `data/images/img_005.jpg` (và `work/viz/img_005.png`) → box `truck` trong `submission/qc/labels_calibrated.json` (cũng truy được `C280` → `img_023.jpg` `three_wheeler` theo `R-08`, `C317` → `img_028.jpg` `truck` theo `R-06`).
