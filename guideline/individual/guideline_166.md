# Guideline gán nhãn — <phiên bản> — <mã hoặc nhóm>

> Copy file này (hoặc `guideline_v0_starter.md`) rồi viết. Một guideline đủ gồm **5 phần**. Câu nào của v0
> làm hai người đọc ra hai cách khác nhau thì phải viết lại cho một cách hiểu duy nhất.

- Phiên bản: v1
- Ngày: 16/11/2026
- Người viết / duyệt: 166

## 1. Mục tiêu
Nhãn được sử dụng để huấn luyện và đánh giá mô hình Object Detection trên ảnh đường phố. Mô hình có nhiệm vụ phát hiện và xác định vị trí của người và phương tiện giao thuộc những lớp được quy định trong dataset.

Kết quả phát hiện các đối tượng có thể được sử dụng trong các hệ thống hỗ trợ giám sát, điều tiết giao thông cũng như các agent nhận thức cho xe tự lái

Việc gán nhãn sai, đặc biệt là bỏ sót người hoặc phương tiện, phân loại sai label và xác định sai vị trí có thể làm giảm độ tin cậy của mô hình, gây ảnh hưởng thực tế đến khả năng nhận diện chướng ngại vật và an toàn giao thông
## 2. Lớp và thuộc tính
### Lớp

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Phương tiện cơ giới 4 bánh, thiết kế chủ yếu để chở người | Sedan, SUV, hatchbach, coupe | Xe bán tải (truck), minivan (van) |
| `truck` | Phương tiện cơ giới được thiết kế chủ yếu để vận chuyển hàng hóa. | Container truck, pickup truck, dump truck, flatbed truck | Passenger car (car), cargo van (van) |
| `van` | Phương tiện cơ giới có khoang thân kín, được thiết kế để chở người hoặc hàng hóa. | Cargo van, passenger van, minivan | Box truck (truck), SUV (car) |
| `motorcycle` | Phương tiện cơ giới hai bánh được vận hành bằng động cơ. |Xe máy, mô tô, xe máy điện | Bicycle (bicycle), tuk-tuk (three_wheeler) |
| `three_wheeler` | Phương tiện giao thông có cấu hình ba bánh tiếp xúc mặt đường, không phụ thuộc vào nguồn năng lượng vận hành. | Tuk-tuk, auto-rickshaw, xích lô, xe ba bánh chở hàng | Motorcycle hai bánh (motorcycle), bicycle hai bánh (bicycle) |
| `bicycle`| Phương tiện hai bánh được vận hành chủ yếu bằng sức người thông qua bàn đạp. | City bicycle, mountain bicycle, road bicycle, xe đạp điện có bàn đạp | Motorcycle (motorcycle), xe ba bánh (three_wheeler) |
| `bus`| 	Phương tiện cơ giới có thiết kế chuyên chở nhiều hành khách. | 	City bus, coach bus, school bus | Minivan (van), passenger car (car) |
| `person` | Cá thể con người xuất hiện trong ảnh, bất kể tư thế hoặc hoạt động. | Người đi bộ, người đi xe đạp, người đi xe máy | Ma-nơ-canh, hình người trên áp phích |
| `ignore_region` | Vùng ảnh được đánh dấu để loại trừ khỏi việc gán nhãn đối tượng thông thường theo quy tắc ignore. |

Vật **không thuộc lớp nào** thì làm gì (không gán? vùng ignore? lớp mới?).
- Không gán

### Thuộc tính
A. occluded_level
| Tên thuộc tính | Giá trị cố định | Tỷ lệ bị che khuất | Ý nghĩa |
|---|---|---|---|
| occluded_level | 0 | 0% | Đối tượng hoàn toàn không bị che khuất. |
| occluded_level | 1 | Trên 0% đến 25% | Đối tượng bị che khuất nhẹ. |
| occluded_level | 2 | Trên 25% đến 50% | Đối tượng bị che khuất một phần đáng kể. |
| occluded_level | 3 | Trên 50% đến dưới 100% | Đối tượng bị che khuất nhiều nhưng vẫn nhận diện được. |

B. truncated
| Tên thuộc tính |	Giá trị cố định | Điều kiện | Ý nghĩa |
|---|---|---|---|
| truncated | no | 0% |	Toàn bộ đối tượng nằm trong khung ảnh. |
| truncated | yes | Có bất kỳ phần nào của đối tượng nằm ngoài khung ảnh do ranh giới ảnh cắt qua. | Đối tượng bị cắt ở mép ảnh. |

C. can_review
| Tên thuộc tính |	Giá trị cố định | Điều kiện | Ý nghĩa |
|---|---|---|---|
| can_review | yes | Không thể xác định chắc chắn class, vị trí box hoặc thuộc tính. | Cần người khác kiểm tra lại. |
| can_review | no |	Có thể đưa ra quyết định theo guideline hiện hành. | Không cần yêu cầu review. |
## 3. Quy tắc vẽ box

| Rule | Tình huống | Quy tắc |
|---|---|---|
| BOX-01 | Box thông thường |Vẽ box bao sát toàn bộ phần đối tượng nhìn thấy được, không lấy thêm quá nhiều nền xung quanh.|
| BOX-02 | Đối tượng bị che khuất | Vẽ box theo phần đối tượng còn nhìn thấy, không ước lượng hoặc vẽ thêm phần bị vật khác che. |
| BOX-03 | Đối tượng bị cắt ở mép ảnh | Vẽ box đến sát ranh giới ảnh tại vị trí object bị cắt, không mở rộng box ngoài ảnh|
| BOX-04 | Phản chiếu | Không gán ảnh phản chiếu của người hay phương tiện |
| BOX-05 | Hình in, tranh vẽ, ảnh trên màn hình | 	Không gán nhãn người hoặc phương tiện chỉ xuất hiện dưới dạng hình ảnh hai chiều. |
| BOX-06 | Tượng, mô hình, đồ chơi | Không gán nhãn các object mô phỏng người/phương tiện nhưng không phải object thật thuộc dataset |
| BOX-07 | Xe chủ (ego vehicle) | Không dán nhãn phương tiện ghi chụp hình nếu guideline xác định nó là xe chủ và không thuộc đối tượng cần phát hiện |
| BOX-08 | Đối tượng quá nhỏ | Chỉ gán nhãn khi có thể xác định đối tượng và vị trí bõ đủ tin cậy. Cần thống nhất ngưỡng kích thước tối thiểu|
| BOX-09 | Đám đông |Gán nhãn từng người riêng biệt nếu có thể phân biệt được từng cá thể. Không gộp nhiều người vào một box chung. |
| BOX-10 | Người và phương tiện | Nếu một người đang đi xe, gán nhãn người và phương tiện bằng hai box riêng biệt khi cả hai đều nhìn thấy và thuộc phạm vi dataset. |

## 4. Ca khó
| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| img_0xx | | | | |

Nếu chưa chốt, đánh dấu `can_review` trên box hoặc ghi vấn đề cấp ảnh. Dùng mã quy tắc trong nhật ký QC.

## 5. QC và phiên bản
Ngưỡng QC áp dụng, ai quyết ca mơ hồ, lịch sử thay đổi:

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| | | |
