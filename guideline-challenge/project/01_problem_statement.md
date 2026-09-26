# Problem statement + downstream contract

## Bài toán

Gắn nhãn đầu đèn xe cơ giới và màu đang sáng trong ảnh tĩnh, kể cả khi đầu đèn nhỏ, xa, bị che/lóa hoặc dễ nhầm tín hiệu ngoài scope.

## Downstream contract

1. **Downstream:** Huấn luyện/đánh giá mô hình phát hiện đầu đèn xe cơ giới và màu hiển thị; không suy quyền đi của xe camera.
2. **Output:** Mỗi đầu một bbox rectangle `vehicle_signal_head` ôm vỏ nhìn thấy; `display` = `red/yellow/green/unlit/unknown`. Điền thêm `signal_form`, `arrow_direction`, `ego_applicability`, `needs_review` theo schema.
3. **Failure `critical`:** Nhầm màu đỏ ↔ xanh do lấy màu đầu bên cạnh/ngoài scope hoặc suy từ vị trí bóng, gây sai nhãn huấn luyện.
4. **Escalation:** Tag ảnh `image_escalate` cho reviewer nhóm; nếu chắc thuộc scope, thêm bbox `needs_review=yes`. Gửi kèm tên ảnh và vị trí nghi vấn qua ghi chú QA (tag không định vị object).

## Scope

- **Label:** Từng vỏ đầu đèn xe cơ giới nhận diện được (kể cả đầu mũi tên, nhỏ/xa/bị che); màu không rõ → `display=unknown`.
- **Ignore:** Đèn người đi bộ/xe đạp, bộ đếm riêng, đèn hậu/đường, biển, phản chiếu, mặt sau đèn, chấm sáng không rõ vỏ.
- **Geometry tolerance:** Bbox ôm sát vỏ _nhìn thấy_, không gồm cột, bảng đếm, quầng sáng; không dựng phần bị che. Chưa có ngưỡng pixel v1, chốt sau calibration.

## Output chấm được

Blind test đối chiếu export CVAT: LABEL = bbox/class/attribute; UNKNOWN = bbox `display=unknown`; ESCALATE = tag `image_escalate` (+ `needs_review=yes` nếu có bbox); IGNORE = vắng bbox tại đối tượng ngoài scope theo candidate trong gold, không có nhãn riêng. Chấm geometry khi chốt tolerance; `__undefined__` không hợp lệ.

## Dữ liệu và giới hạn

Dự kiến 18 JPEG tĩnh `data/data-image/1.jpg`–`18.jpg`; chưa đăng ký trong `data/catalog.csv`, chưa chia split ở `project/sample_pack.csv`. Không suy trạng thái theo thời gian hay khẳng định khả năng tổng quát ngoài bộ ảnh.
