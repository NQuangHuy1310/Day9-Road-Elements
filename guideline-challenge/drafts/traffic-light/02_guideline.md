# Annotation guideline — đầu đèn giao thông cho xe cơ giới

**Version:** v1 — guideline đầu tiên, áp dụng cho annotation thử và calibration trên 18 ảnh JPEG trong `data/data-image/`. V2 sẽ cập nhật sau calibration; v1 chưa phải bản đã freeze hoặc gold.

## 1. Objective + scope

Gắn bbox cho **đầu đèn giao thông dành cho xe cơ giới** có mặt tín hiệu nhìn được trong ảnh đường phố và ghi **màu đang sáng tại đúng ảnh đó**. Mục tiêu là dữ liệu nhận diện đầu đèn và màu hiển thị, không phải suy ra xe camera được đi, rẽ hướng nào, đèn có hỏng hay không.

Không gắn nhãn đèn người đi bộ, đèn xe đạp, bộ đếm thời gian, biển báo, đèn hậu hoặc ánh phản chiếu. Một ảnh có thể có cả đối tượng trong và ngoài scope.

## 2. Annotation unit

Mỗi JPEG là **một ảnh tĩnh** trong CVAT; dùng Shape, không dùng Track. Một bbox ứng với **một đầu/vỏ đèn** độc lập, không phải một bóng đèn hoặc cả giàn đèn. Nhiều bóng trong một vỏ chỉ tạo một bbox; hai vỏ cạnh nhau (ví dụ đầu mũi tên và đầu tròn) là hai bbox. Không nối các ảnh thành chuỗi thời gian.

## 3. Geometry rule

Dùng rectangle ôm sát **vỏ đầu đèn nhìn thấy được** (bao gồm các bóng trong vỏ), không ôm cột, thanh ngang, bảng số đếm hoặc quầng sáng. Nếu đầu đèn bị che/cắt mép ảnh, bbox chỉ ôm phần nhìn thấy và dừng ở mép ảnh; không vẽ phần bị che theo suy đoán. Nếu không phân biệt được ranh giới đầu đèn với nền thì không vẽ bbox theo trí tưởng tượng, chuyển reviewer xem lại theo mục 7. Chưa ấn định tolerance theo pixel cho v1 vì ảnh có nhiều độ phân giải/kích thước đầu đèn; chốt sau hai người vẽ thử trên calibration.

## 4. Taxonomy

Bản v1 dùng **một class** `vehicle_signal_head` (rectangle) và tag ảnh `image_escalate`. Schema CVAT hiện có năm select trên mỗi bbox; v1 tập trung vào màu `display`, các field còn lại điền theo quy tắc đơn giản dưới đây. Bảng này phải khớp `03_ontology_and_cvat_setup.md` và `03_cvat_labels.json` đi kèm.

| Tên                   | Kiểu             | Giá trị/default                                                                              | Khi dùng                                                                                                                                                                              |
| --------------------- | ---------------- | -------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `vehicle_signal_head` | Rectangle        | Một đầu đèn / bbox                                                                           | Đầu tín hiệu cho xe cơ giới, kể cả đầu mũi tên.                                                                                                                                       |
| `signal_form`         | Select trên bbox | `circular`, `arrow`, `unknown`; mặc định `__undefined__`                                     | Chọn `arrow` nếu nhìn rõ mũi tên, `circular` nếu là đèn tròn; không phân biệt được thì `unknown`.                                                                                     |
| `display`             | Select trên bbox | `red`, `yellow`, `green`, `unlit`, `unknown`; mặc định `__undefined__`                       | Màu sáng trên **chính đầu đèn**. Mũi tên đỏ vẫn là `red`; `unlit` chỉ khi nhìn rõ các bóng đều không sáng; không chắc thì `unknown`.                                                  |
| `arrow_direction`     | Select trên bbox | `left`, `straight`, `right`, `u_turn`, `unknown`, `not_applicable`; mặc định `__undefined__` | Đèn tròn → `not_applicable`; mũi tên rõ → hướng thấy được; không đọc được hướng hoặc form → `unknown`.                                                                                |
| `ego_applicability`   | Select trên bbox | `applies`, `does_not_apply`, `unknown`; mặc định `__undefined__`                             | V1 mặc định quyết định `unknown` khi ảnh không chứng minh được quan hệ với làn ego; không suy quyền được đi từ màu. Chỉ dùng hai giá trị còn lại khi có bằng chứng làn/hướng rõ ràng. |
| `needs_review`        | Select trên bbox | `yes`, `no`; mặc định `__undefined__`                                                        | `no` nếu không có vướng mắc ở object; `yes` nếu object cần reviewer quyết định.                                                                                                       |
| `image_escalate`      | Tag trên ảnh     | Có/không                                                                                     | Cần reviewer xử lý ca chưa có rule hoặc chưa chắc có nên label; không phải kết luận thiết bị lỗi.                                                                                     |

`__undefined__` chỉ là placeholder **chưa điền**, không được còn lại ở bất kỳ select nào trên bbox khi hoàn tất; `unknown` là đã xem nhưng ảnh không đủ bằng chứng. V1 có ghi hình thức/hướng mũi tên khi nhìn rõ nhưng **không** suy quyền đi, chuyển pha hay nhấp nháy. `unlit` không có nghĩa là thiết bị hỏng. Nếu có vẻ nhiều màu sáng trong cùng một vỏ và không xác định được màu chủ đạo, chọn `display=unknown` và escalate.

## 5. Inclusion / exclusion

- **LABEL:** Đầu đèn cho xe cơ giới có mặt hướng về camera đủ để nhận diện vỏ, dù xa/nhỏ hoặc chưa đọc được màu. Gắn bbox từng đầu và chọn `display` theo ảnh. Không yêu cầu xác định đầu đó áp dụng cho làn ego.
- **IGNORE:** Đèn người đi bộ/xe đạp, bộ đếm số đứng riêng cạnh đèn, đèn hậu, đèn đường, biển báo, phản chiếu và mặt sau của đầu đèn quay khỏi camera. Không biến chấm sáng mờ không nhận diện được vỏ thành đầu đèn.
- Đầu mũi tên là đèn xe cơ giới vẫn **LABEL**, ghi hướng khi thấy rõ nhưng không suy luật ưu tiên. Không lấy màu bộ đếm, đầu bên cạnh hay đèn hậu để điền `display`.

## 6. Visibility / occlusion

Bị che một phần/cắt khung: label nếu vẫn nhận ra đầu đèn, bbox phần thấy được. Xa/nhỏ, ngược sáng, lóa, trời tối: nếu thấy vỏ nhưng không chắc màu sáng thì `display=unknown`; không ép chọn màu dựa vào vị trí bóng (trên = đỏ, v.v.). Đầu nhìn tối do ảnh tối/che/lóa là `display=unknown`; chỉ chọn `unlit` nếu thấy rõ các bóng đều không sáng, **không kết luận hỏng**. Nếu không đủ dấu hiệu xác định đó là đầu đèn xe, không dựng bbox; ca phân vân thì dùng `image_escalate`.

## 7. Ambiguity / escalation

| Quyết định | Khi nào                                                                                           | Biểu diễn trong CVAT/export                                                                                                                                                        |
| ---------- | ------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| LABEL      | Chắc chắn là đầu đèn xe, màu đọc được                                                             | Bbox `vehicle_signal_head` + `display=red/yellow/green`.                                                                                                                           |
| IGNORE     | Chắc chắn ngoài scope hoặc không nhận diện được là đầu đèn                                        | Không có bbox cho đối tượng đó (các đầu hợp lệ khác trong cùng ảnh vẫn label).                                                                                                     |
| UNKNOWN    | Chắc chắn là đầu đèn xe nhưng màu không đủ bằng chứng                                             | Bbox `vehicle_signal_head` + `display=unknown`.                                                                                                                                    |
| ESCALATE   | Không chắc thuộc scope, không thể tách vỏ, nhiều màu trong cùng đầu, hoặc tình huống chưa có rule | Gắn tag ảnh `image_escalate`. Nếu chắc là đầu đèn xe, vẽ bbox, điền các select và đặt `needs_review=yes`; nếu chưa chắc là đèn xe thì **không** vẽ bbox suy đoán, chỉ gắn tag ảnh. |

Tag chỉ cho biết **ảnh cần review**, không chỉ ra object nào: khi gửi reviewer phải ghi thêm tên file và vị trí/miêu tả đầu nghi vấn trong ghi chú QA; xử lý xong mới bỏ tag hoặc cập nhật quyết định. V1 chưa có quy tắc chấm riêng cho ca escalate.

## 8. Temporal rule

**Không áp dụng — task ảnh tĩnh.** Không suy diễn đổi pha đỏ→xanh, đèn nhấp nháy hoặc hỏng từ một JPEG hay từ hai ảnh không có thời gian đáng tin. Nếu về sau dùng video/track, cần guideline và ontology phiên bản mới.

## 9. Examples

Các ví dụ dưới đây chỉ là **minh họa v1 từ ảnh đã xem**, chưa phải gold: `data-image/*.jpg` hiện **chưa có sample_id trong `data/catalog.csv` và chưa được phân split trong `project/sample_pack.csv`**. ID `IMGxx` trong bảng là ID **tạm** (file tương ứng `data/data-image/xx.jpg`), dự kiến dành split `example`; phải đăng ký catalog/split và rà soát toàn ảnh ở độ phân giải gốc trước khi dùng trong CVAT hoặc gửi peer. Không đưa ảnh đã chọn `blind` vào ví dụ.

| sample_id (tạm; example dự kiến) | Thấy gì                                               | Expected output minh họa (không phải toàn bộ bbox của ảnh)                                                                                                                                                           | Rule áp dụng                                      |
| -------------------------------- | ----------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------- |
| IMG03 (`3.jpg`)                  | Hai đầu đèn trên thanh ngang sáng đỏ                  | Hai bbox riêng `vehicle_signal_head`, mỗi bbox `display=red`.                                                                                                                                                        | 2, 4: mỗi vỏ một bbox.                            |
| IMG08 (`8.jpg`)                  | Đèn xe đạp xanh cận cảnh; đèn xe cơ giới ở xa         | Không bbox cho **đèn xe đạp**; vẫn label các đầu đèn xe cơ giới nhận ra được ở xa (màu không rõ thì `unknown`).                                                                                                      | 5, 6: ignore theo đối tượng, không ignore cả ảnh. |
| IMG09 (`9.jpg`)                  | Đầu mũi tên trái sáng đỏ và các đầu tối bên cạnh      | Đầu mũi tên là một bbox `signal_form=arrow`, `arrow_direction=left`, `display=red`; đầu xe cơ giới tối nhưng nhận ra được là bbox khác `display=unknown` nếu không chắc bóng đều tắt; không vẽ bbox cho bảng đếm số. | 2, 4, 5: không gộp đầu/không đoán màu bóng tối.   |
| IMG17 (`17.jpg`)                 | Đầu đèn xe trên cao sáng xanh, đèn người đi bộ ở dưới | Bbox cho đầu đèn xe sáng xanh với `display=green`; không bbox cho đèn người đi bộ.                                                                                                                                   | 5: tách tín hiệu theo đối tượng.                  |

## 10. Common mistakes

- Vẽ một bbox cho cả cụm đèn, hoặc vẽ từng bóng thay vì từng **vỏ độc lập** → tách đúng theo mục 2.
- Label cả bảng đếm số, đèn xe đạp/người đi bộ hoặc bỏ qua toàn ảnh chỉ vì có các tín hiệu này → xét từng đối tượng theo mục 5.
- Điền màu từ đầu bên cạnh, từ vị trí bóng, hay coi bóng tối là đèn hỏng → chỉ ghi màu quan sát được; không chắc thì `unknown`.
- Suy xe ego được đi/rẽ, suy chuyển pha/nhấp nháy từ ảnh tĩnh → nằm ngoài phạm vi v1.
- Để bất kỳ select nào là `__undefined__` trong bbox hoặc chỉ nói miệng ca khó mà không tag `image_escalate` → hoàn tất giá trị và thể hiện quyết định trong export.
