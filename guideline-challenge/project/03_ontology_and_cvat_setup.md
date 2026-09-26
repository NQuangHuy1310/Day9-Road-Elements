# Ontology + CVAT setup

Bảng ontology là **source of truth** cho schema CVAT: `03_cvat_labels.json` phải khớp từng dòng ở đây. Thay mọi
placeholder mới là xong (gate G2).

## Ontology table

| Name | Geometry | Type | Allowed values | Default | Mutable? | Rationale |
|---|---|---|---|---|---|---|
| `vehicle_signal_head` | rectangle | class | — | — | — | Mỗi vỏ đèn xe cơ giới là một instance riêng; rectangle đủ để ôm sát vỏ nhìn thấy, downstream cần bbox cho detection model. |
| `signal_form` | — | attribute of `vehicle_signal_head` | `__undefined__`, `circular`, `arrow`, `unknown` | `__undefined__` | No | Phân biệt đèn tròn vs đèn mũi tên; ảnh hưởng cách đọc `arrow_direction`. |
| `display` | — | attribute of `vehicle_signal_head` | `__undefined__`, `red`, `yellow`, `green`, `unlit`, `unknown` | `__undefined__` | No | Màu đang sáng là thông tin cốt lõi cho downstream. `unlit` = vỏ nhìn rõ nhưng không bóng nào sáng; `unknown` = không đủ bằng chứng phân biệt màu (xa, lóa, che). |
| `arrow_direction` | — | attribute of `vehicle_signal_head` | `__undefined__`, `left`, `straight`, `right`, `u_turn`, `unknown`, `not_applicable` | `__undefined__` | No | Chỉ có ý nghĩa khi `signal_form=arrow`. Nếu `signal_form=circular` → chọn `not_applicable`. |
| `ego_applicability` | — | attribute of `vehicle_signal_head` | `__undefined__`, `applies`, `does_not_apply`, `unknown` | `__undefined__` | No | Đèn có áp dụng cho xe camera (ego vehicle) hay không. Giúp downstream lọc đèn liên quan. |
| `needs_review` | — | attribute of `vehicle_signal_head` | `__undefined__`, `yes`, `no` | `__undefined__` | No | Flag cho reviewer khi annotator không chắc chắn. |
| `image_escalate` | — (tag) | class (tag) | — | — | — | Tag cấp ảnh, không gắn vào object cụ thể. Dùng khi toàn bộ ảnh có vấn đề cần reviewgier xem xét (ví dụ: ảnh quá tối, quá lóa, nhiều đèn mơ hồ). |

## Class hay attribute — lý do thiết kế

- **`vehicle_signal_head` là class (rectangle):** Downstream cần detect từng vỏ đèn riêng biệt → mỗi vỏ = 1 bbox instance. Không thể dùng attribute vì mỗi ảnh có nhiều đèn ở vị trí khác nhau.
- **`signal_form`, `display`, `arrow_direction`, `ego_applicability`, `needs_review` là attribute:** Đây là thuộc tính gắn liền với từng vỏ đèn cụ thể, không tồn tại độc lập. Một vỏ đèn = 1 bbox + N attribute.
- **`image_escalate` là class (tag):** Cần flag ở cấp ảnh, không gắn vào object nào. CVAT tag là cách duy nhất để gắn metadata cấp ảnh mà vẫn xuất hiện trong file export.
- **Default `__undefined__`:** Chọn giá trị không hợp lệ làm default để buộc annotator phải chủ động chọn. Nếu default là `circular` hoặc `no`, annotator dễ quên đổi → gây bias ngầm trong dữ liệu huấn luyện. `__undefined__` sẽ bị reject khi chấm.

## CVAT

- **Phiên bản CVAT:** `v2.74.1` (self-hosted Docker, exposed qua Cloudflare Quick Tunnel)
- **Project:** `day9_demo` (Project ID: 8)
- **Task Ground Truth:** `day9-test` (Task ID: 24, Job ID: 21) — 18 JPEG, 63 bounding boxes
- **Task cho labeller:** `day9-test-labeller2` (Task ID: 26, Job ID: 23) — cùng 18 JPEG, chưa có annotation
- **Guide của task đã dán `02_guideline.md`?** Chưa — `02_guideline_v1.md` vẫn còn TODO, cần hoàn thiện trước khi dán vào CVAT task description.
- **Nhóm dùng Shape, không dùng Track:** Task ảnh tĩnh, không có temporal sequence. Track chỉ cần cho video.

## Labels JSON

File `03_cvat_labels.json` chứa schema import trực tiếp vào CVAT project. Đã xác nhận khớp 1:1 với bảng ontology ở trên:

- 1 label `vehicle_signal_head` (rectangle, màu `#F9A825`) với 5 attribute select
- 1 label `image_escalate` (tag, màu `#8E24AA`) không có attribute

Để import: Project Settings → Raw Editor → paste nội dung `03_cvat_labels.json`.

## Setup test

Một thành viên **chưa tham gia setup** mở task và trả lời: label gì, dùng tool nào, gán attribute nào, khi nào
escalate. Ghi lại ai test và chỗ họ vấp:

| Người test | Chỗ vấp | Ghi chú |
|---|---|---|
| TODO | TODO | TODO |
