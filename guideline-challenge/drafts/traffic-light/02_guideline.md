# BẢN NHÁP — guideline đèn giao thông (chưa dùng để label/nộp)

**Version:** v0 — đề xuất để nhóm xem xét; KHÔNG phải guideline đã chốt. Không tự sửa `project/02_guideline.md` từ file này.

## Những gì đã kiểm tra và những gì chưa biết

- Đã xem trực tiếp `data/lisa/LISA01.jpg` và `LISA23.jpg`: có đầu đèn mũi tên trái đỏ và một đầu đèn tròn riêng bên cạnh; ở LISA23 đầu tròn sáng xanh. Đây là **quan sát trên ảnh**, không khẳng định cùng một đầu đèn chuyển pha hoặc quyền rẽ trái của xe ego.
- Đã xem `data/bdd100k/BDD12.jpg`: có đèn hình người đi bộ; dùng làm ca phân biệt loại tín hiệu, không tuyên bố cả ảnh là negative.
- Chưa xác nhận danh sách ảnh sẽ dùng, khả năng đọc tín hiệu ở từng ảnh, split example/calibration/blind, tiêu chuẩn pháp lý về quyền đi, hay CVAT trên máy nhóm. Do đó **chưa ấn định ví dụ chính thức hoặc gold**.
- BDD là ảnh tĩnh; LISA trong repo là các ảnh trích từ một clip. Chưa kiểm tra video gốc hoặc tốc độ lấy mẫu: **không thể kết luận** sự kiện `red→green`, đèn chớp, đèn hỏng, chu kỳ bất thường chỉ từ một ảnh.

## 1. Objective + scope — đề xuất

Đề xuất bài toán hẹp: gắn bbox cho **từng đầu tín hiệu đèn dành cho xe cơ giới nhìn về phía camera**, mô tả hình thức tín hiệu, màu sáng đang quan sát, hướng mũi tên và mức độ có thể áp dụng cho làn/hướng ego **nếu ảnh đủ bằng chứng**. Output là annotation thị giác để phân tích chất lượng dữ liệu, **không phải quyết định lái xe hay kết luận được phép rẽ/đi thẳng**.

Không label xe ưu tiên, biển báo, đèn người đi bộ, phản chiếu hoặc đèn hậu. Nếu mục tiêu thực sự là xác định "rẽ được/không được đi thẳng", nhóm phải cung cấp quy định địa phương, loại tín hiệu và lane mapping được kiểm chứng; schema hiện tại không mã hoá quyền đi.

## 2. Annotation unit — đề xuất cần thử trên ảnh

Một bbox cho **một đầu/mặt đèn hoạt động độc lập**: các bóng trong cùng vỏ là một object, mũi tên trong một đầu riêng cạnh đầu đèn tròn là hai object. Chưa định nghĩa ca cùng một vỏ có nhiều chỉ thị sáng cùng lúc; gặp ca đó ghi `display=unknown`, `needs_review=yes`, tag cả ảnh, rồi bổ sung rule sau khi kiểm chứng.

Nếu dùng `sample_pack.csv` để upload các JPEG rời, dùng CVAT **Shape** cho từng ảnh; không tự ghép Track giữa các frame LISA. Nếu nhóm chuyển sang video gốc, cần thiết kế quy trình Track và version schema **mới**.

## 3. Geometry rule — đề xuất cần đo trên calibration

Rectangle ôm phần **vỏ/mặt đầu đèn thực sự nhìn thấy**, không ôm thanh treo, cột hoặc quầng sáng. Vật bị che/cắt khung: dừng bbox ở vùng nhìn thấy/mép ảnh, không vẽ phần bị che theo trí tưởng tượng. Ngưỡng sai số pixel **chưa chốt**; chỉ chốt sau khi xem kích thước các đầu đèn ở tập ảnh đã chọn và hai người vẽ thử.

## 4. Taxonomy — bản thử để tạo task CVAT

Chỉ một class rectangle `vehicle_signal_head`, một tag `image_escalate`. Bảng chi tiết/default và file Raw mẫu: `03_ontology_and_cvat_setup.md`, `03_cvat_labels.json` cùng thư mục `drafts/traffic-light/`. Đây là **đề xuất** chứ không phải schema đã xác nhận dùng được trên CVAT của nhóm.

- `signal_form`: `circular` / `arrow` / `unknown`.
- `display`: `red` / `yellow` / `green` / `unlit` / `unknown`. `unlit` nghĩa là quan sát được các bóng không sáng, **không phải đèn hỏng**.
- `arrow_direction`: `left` / `straight` / `right` / `u_turn` / `unknown` / `not_applicable`.
- `ego_applicability`: `applies` / `does_not_apply` / `unknown`, chỉ điền hai giá trị chắc chắn nếu thấy rõ làn/hướng liên quan.
- `needs_review`: `yes` / `no`.

Default các select = `__undefined__` nghĩa là **chưa trả lời**; `unknown` = đã xem nhưng không đủ bằng chứng. Không export object còn `__undefined__`. Đèn tròn → `arrow_direction=not_applicable`; đèn mũi tên → chọn hướng nhìn thấy hoặc `unknown`. Hình mũi tên trái đỏ và đầu tròn xanh **phải là hai observation riêng**, không sao chép màu hoặc suy ra quyền đi của cả giao lộ.

## 5. Inclusion / exclusion — đề xuất

- LABEL khi nhận ra đây là đầu đèn cho xe cơ giới hướng về phía camera, kể cả màu mờ hoặc xa nhưng còn xác định được đầu đèn. Nếu màu không đọc được, chọn `display=unknown` thay vì bỏ object.
- IGNORE tín hiệu người đi bộ/xe đạp, đèn hậu, đèn trang trí/phản chiếu, chấm sáng không xác định được là đầu đèn, và mặt đèn quay hẳn khỏi camera. Không gán một chấm sáng mờ thành "đèn lỗi".
- Với đầu đèn trông có thể điều khiển nhánh/làn khác nhưng không thấy lane mapping: LABEL đầu đèn, `ego_applicability=unknown` — không bỏ qua chỉ vì chưa biết liên quan ego.

## 6. Visibility / occlusion — đề xuất

Đầu bị che một phần: bbox phần thấy được và dùng `unknown` riêng cho thông tin không đọc được. Màu lẫn do chói/LED/camera, đầu quá nhỏ hoặc mưa/đêm: nếu không xác định được màu, `display=unknown`; không lấy màu phản chiếu hoặc tín hiệu bên cạnh thế chỗ. Không dùng `unlit` khi chỉ không thấy bóng vì bị che hoặc ảnh tối.

## 7. Ambiguity / escalation — đề xuất

| Quyết định | Điều kiện                                                                                                            | Trong export CVAT                                                                        |
| ---------- | -------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| LABEL      | Đầu đèn cho phương tiện thấy đủ để nhận diện                                                                         | Bbox `vehicle_signal_head` và các attribute đã chọn.                                     |
| IGNORE     | Chắc chắn ngoài scope; không nhận ra đầu đèn                                                                         | Không vẽ bbox.                                                                           |
| UNKNOWN    | Nhận ra đầu đèn nhưng thiếu màu/hướng/lane mapping                                                                   | Bbox và `unknown` ở **đúng field**.                                                      |
| ESCALATE   | Không thể xác định tín hiệu có khả năng liên quan ego, chỉ thị nhìn như mâu thuẫn, hoặc nghi sự cố cần dữ liệu nguồn | Object: `needs_review=yes`; ảnh: tag `image_escalate`. Tag không phải kết luận đèn hỏng. |

Từng ca ESCALATE cần nhóm ghi lý do ngoài export cho reviewer (sample_id, object, ảnh/chứng cứ cần xem) trong tài liệu QA. Chưa có bản mẫu hoặc quy tắc chấm cho ca hỏng đã được kiểm chứng.

## 8. Temporal rule

**Bản nháp này áp dụng cho ảnh tĩnh.** Không thêm `transition`, `flashing`, `malfunction` vào JSON: chúng cần chuỗi frame có thời gian/chu kỳ lấy mẫu đáng tin và rule review riêng. Nếu về sau có video phù hợp: cùng đầu vật lý là một Track; `display` phải mutable, đánh keyframe khi **quan sát được** trạng thái mới, kết thúc bằng `outside` khi rời khung; chỉ ghi sự kiện đổi pha khi có ít nhất hai trạng thái rõ trên cùng track theo thời gian. Đây là hướng mở rộng, **chưa triển khai/kiểm chứng** trong task ảnh.

## 9. Examples — chưa chốt split

Các ca khảo sát `LISA01`, `LISA23` (đầu mũi tên và đầu tròn) và `BDD12` (đèn người đi bộ) là **ứng viên** để viết ví dụ, chưa phải ví dụ gửi peer. Chỉ đưa `sample_id` vào đây **sau khi** nhóm đã gán nó vào split `example` hoặc `calibration` trong `project/sample_pack.csv` và kiểm lại ảnh ở độ phân giải gốc. Không dùng ảnh blind làm ví dụ.

## 10. Common mistakes cần thử trong calibration

- Gộp đầu mũi tên với đầu tròn hoặc áp màu của một đầu cho đầu còn lại.
- Xem `green` của một đầu là "ego được rẽ/đi thẳng" khi lane mapping chưa rõ.
- Gán `unlit` thành "hỏng"; gán chuyển pha/nhấp nháy từ một frame.
- Vẽ đèn người đi bộ hay quầng sáng/phản chiếu như đèn phương tiện; quên gán `unknown` và để `__undefined__` trong export.

## Việc nhóm phải xác nhận trước khi nâng v1

1. Chốt downstream task: chỉ nhận dạng đầu đèn hay bắt buộc phán đoán quyền đi theo hướng? Nếu quyền đi là mục tiêu, schema này **chưa đủ**.
2. Kiểm kê ảnh đủ positive/negative/edge và 4–5 ảnh blind có case critical thực sự; chọn split không rò rỉ (LISA là một clip liên tiếp).
3. Review bbox unit, ngưỡng geometry, tên/allowed values cùng hai annotator; thử Raw JSON và **export XML thật** từ CVAT.
4. Chốt ai review ca `image_escalate`; viết ví dụ từ split example/calibration và soạn gold trước freeze.
