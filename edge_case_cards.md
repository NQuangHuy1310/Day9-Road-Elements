# Edge-case library

Tối thiểu **8 card**, khuyến nghị 10–12. Một edge case tốt là case mà hai annotator hợp lý có thể làm khác nhau nếu
guideline chưa rõ. Tám ảnh dễ có label rõ ràng không được tính là edge-case library.

Cần có đủ độ đa dạng: occlusion / truncation / small-far · ambiguous semantics · conflicting road elements · **một case
critical-risk** · **một case guideline cho phép escalation**.

File này là kho nội bộ của nhóm, **không gửi cho peer**. Card dùng ảnh example/calibration thì chép rule + ví dụ sang
`02_guideline.md` (mục 7 và 9) để peer đọc được. Card về ảnh blind chỉ nằm ở đây, và decision của nó phải có trong
`gold_decisions.csv` trước `make freeze`.

`make status` đếm số dòng `CASE ID:` đã điền (đã thay placeholder). Copy khối dưới cho mỗi case.

---

CASE ID: EC-01
Sample: 16.jpg
Scene: Giao lộ đô thị ban ngày có vạch qua đường cho người đi bộ (crosswalk); cột tín hiệu gắn cụm đèn người đi bộ bên cạnh đèn phương tiện
Observation: Xuất hiện cụm đèn tín hiệu dành riêng cho người đi bộ (biểu tượng hình người qua đường) gắn ở tầm thấp cạnh vỉa hè
Decision: IGNORE
Expected: Không vẽ bounding box (bỏ qua). Lớp đối tượng quy định của dự án chỉ là vehicle_signal_head (đèn tín hiệu dành cho phương tiện cơ giới)
Rationale: Downstream planner/control của xe tự hành (ego) chỉ nhận tín hiệu đèn xe. Nếu gán nhầm đèn người đi bộ thành đèn xe, xe có thể khởi hành sai nhịp hoặc phanh gấp bất thường khi đèn người đi bộ đổi màu, gây nguy cơ va chạm nghiêm trọng
Common mistake: Annotator thấy hộp đèn có màu xanh/đỏ là vẽ box mà không phân biệt đèn người đi bộ hay đèn phương tiện
Diversity: negative

---

CASE ID: EC-02
Sample: 15.jpg
Scene: Giao lộ lúc ban ngày hoặc chập tối; cụm đầu đèn tín hiệu phương tiện gắn trên cột/giá treo nhưng toàn bộ các mặt đèn đều không sáng bóng nào
Observation: Nhìn thấy rõ hộp đèn 3 khoang vật lý nhưng cả 3 khoang đều tối, không phát sáng màu đỏ, vàng hay xanh (đèn tắt do mất điện, hỏng bóng, hoặc pha rẽ chưa kích hoạt)
Decision: LABEL
Expected: Bounding box ôm sát vỏ đầu đèn. Attribute: signal_form = circular (hoặc unknown nếu quá mờ), display = unlit, arrow_direction = not_applicable, ego_applicability = applies, needs_review = yes
Rationale: Xe tự hành cần biết sự hiện diện của đầu đèn giao thông tại giao lộ ngay cả khi đèn tắt để chuyển sang chế độ dự phòng "ngã tư mất tín hiệu" (nhường đường/dừng quan sát 4-way stop theo luật), tránh lao qua giao lộ mất an toàn
Common mistake: Bỏ qua không gán nhãn vì nghĩ đèn tắt là nền/vô hiệu; hoặc tự ý đoán màu đèn đang hoạt động
Diversity: edge

---

CASE ID: EC-03
Sample: 11.jpg
Scene: Giao lộ góc nhìn từ xa; ánh sáng phức tạp, đầu đèn rẽ ở nhánh phụ bị mờ, chói lóa hoặc độ phân giải thấp
Observation: Nhìn thấy kết cấu đầu đèn có bóng sáng mờ nhưng không thể xác định chắc chắn màu sắc pha đèn (đỏ/vàng/xanh) hoặc hình dạng thấu kính mũi tên
Decision: UNKNOWN
Expected: Bounding box vẽ khít vỏ đèn. Attribute: signal_form = arrow, display = unknown, arrow_direction = left, ego_applicability = unknown, needs_review = yes
Rationale: Thực thi nghiêm ngặt nguyên tắc "Gán theo bằng chứng nhìn thấy, không đoán mò". Việc gán display = unknown và bật needs_review = yes giúp pipeline downstream lọc dữ liệu nghi ngờ, tránh hiện tượng mô hình hallucinate do nhãn suy đoán
Common mistake: Zoom quá mức vào pixel bị vỡ rồi tự suy đoán màu theo các đèn xung quanh, gây nhiễu ground truth
Diversity: ambiguity

---

CASE ID: EC-04
Sample: 8.jpg
Scene: Tuyến đường nhiều làn; có làn rẽ trái riêng biệt và các làn đi thẳng. Ego đang di chuyển ở làn đi thẳng
Observation: Đầu đèn mũi tên rẽ trái đang sáng màu đỏ rực (display = red, arrow_direction = left), nằm trên cột nhánh rẽ riêng. Làn đi thẳng của ego không bị chi phối bởi đèn này
Decision: LABEL
Expected: Bounding box ôm sát đầu đèn rẽ trái. Attribute: signal_form = arrow, display = red, arrow_direction = left, ego_applicability = does_not_apply, needs_review = no
Rationale: CASE CRITICAL-RISK: Nếu annotator gán nhầm ego_applicability = applies, xe tự hành sẽ hiểu nhầm làn mình bị cấm và phanh khẩn cấp (phantom braking) giữa dòng xe đang chạy thẳng tốc độ cao, dẫn đến nguy cơ đâm va dây chuyền từ phía sau
Common mistake: Gán mặc định ego_applicability = applies cho mọi đèn đỏ nhìn thấy mà không xét hướng đi thực tế của làn ego
Diversity: critical

---

CASE ID: EC-05
Sample: 14.jpg
Scene: Giao lộ có tán cây râm mát hoặc biển báo che khuất một phần thân đèn tín hiệu
Observation: Khoảng 30%–50% vỏ hộp đèn bị cành cây/lá cây che khuất, nhưng bóng đèn đỏ phát sáng vẫn lộ rõ
Decision: LABEL
Expected: Bounding box chỉ ôm trọn phần nhìn thấy được (visible box), không vẽ phóng đại hộp bao trùm cả phần cây che (không vẽ amodal). Attribute: signal_form = circular, display = red, arrow_direction = not_applicable, ego_applicability = applies, needs_review = yes
Rationale: Đảm bảo độ chính xác hình học (IoU) cho mô hình 2D object detection. Nếu vẽ bao cả cành lá, model sẽ học sai đặc trưng (feature) và nhận diện tán cây là đèn giao thông
Common mistake: Vẽ bounding box to bao gồm cả cành lá bên ngoài để cố tái tạo hình dạng nguyên vẹn của hộp đèn
Diversity: occlusion

---

CASE ID: EC-06
Sample: 2.jpg
Scene: Xe đang tiến sát vạch dừng giao lộ; đèn tín hiệu vừa dứt pha xanh và chuyển sang màu vàng (yellow transition)
Observation: Mặt đèn tròn ở giữa cụm 3 đèn phát sáng màu vàng cam rõ rệt
Decision: LABEL
Expected: Bounding box ôm sát vỏ đèn vàng. Attribute: signal_form = circular, display = yellow, arrow_direction = not_applicable, ego_applicability = applies, needs_review = no
Rationale: Pha đèn vàng quyết định hành vi dừng an toàn trước vạch dừng hay tiếp tục qua giao lộ (Dilemma Zone). Gán nhầm thành đỏ hoặc xanh sẽ làm sai lệch nghiêm trọng chính sách điều khiển của xe tự hành
Common mistake: Gán nhầm màu vàng thành đỏ (do tâm lý chuẩn bị dừng) hoặc coi đèn vàng là unknown
Diversity: ambiguity

---

CASE ID: EC-07
Sample: 4.jpg
Scene: Giao lộ lớn nhiều nhánh; tầm quan sát thấy các đầu đèn ở đường nhánh cắt ngang và các đèn ở cự ly xa (> 80m)
Observation: Xuất hiện nhiều đầu đèn nhỏ (chiều cao khoảng 15–25 px) trên ảnh, nhìn rõ hình dạng khối hộp đèn nhưng chi tiết mặt đèn khá nhỏ
Decision: LABEL
Expected: Bounding box vẽ khít từng đầu đèn riêng biệt (đối với đèn ≥ 20px). Attribute: signal_form = circular, display = unknown (nếu không rõ màu) hoặc red/green, ego_applicability theo đúng hướng làn
Rationale: Giữ tính nhất quán cho mô hình phát hiện vật thể ở cự ly xa để hệ thống tự hành chuẩn bị kế hoạch giảm tốc từ sớm, đồng thời không gom cụm các đèn rời rạc
Common mistake: Vẽ một bounding box khổng lồ ôm trọn cả giàn 3-4 đầu đèn treo ngang thay vì tách riêng từng đầu đèn
Diversity: small_far

---

CASE ID: EC-08
Sample: 6.jpg
Scene: Ảnh chụp ngược sáng mặt trời hoặc đèn pha đối diện chiếu trực tiếp vào cảm biến camera gây hiện tượng chói lóa cực mạnh (lens flare/blooming)
Observation: Một vùng sáng xanh khổng lồ lan rộng hàng trăm pixel, mất hoàn toàn viền vỏ đèn vật lý, không thể phân biệt đâu là bóng đèn thật và đâu là ánh sáng tán xạ quang học
Decision: ESCALATE
Expected: Gán tag image_escalate cho ảnh trong CVAT. Attribute đèn: signal_form = unknown, display = green, arrow_direction = not_applicable, ego_applicability = applies, needs_review = yes; chuyển lên QA Lead/Reviewer thẩm định loại bỏ ảnh
Rationale: CASE ESCALATION: Khi dữ liệu suy hao quang học nghiêm trọng không thể xác định ranh giới vật lý, guideline cho phép kích hoạt quy trình Escalation để tránh ép annotator vẽ nhãn ảo làm hỏng gradient khi huấn luyện
Common mistake: Cố vẽ một hộp bao khổng lồ bao quanh toàn bộ vệt chói lóa 340x327 px mà không kích hoạt escalation
Diversity: escalation

---

CASE ID: EC-09
Sample: 12.jpg
Scene: Giá long môn treo song song hai đầu đèn khác nhau phục vụ hai luồng xe: 1 đèn mũi tên rẽ trái và 1 đèn tròn đi thẳng
Observation: Đèn mũi tên đang sáng xanh (green arrow left), đèn tròn đang sáng vàng (yellow circular). Hai đầu đèn gắn cạnh nhau trên cùng một thanh xà ngang
Decision: LABEL
Expected: Tách thành 2 bounding box riêng biệt:
- Box 1: signal_form = arrow, display = green, arrow_direction = left, ego_applicability = applies, needs_review = no
- Box 2: signal_form = circular, display = yellow, arrow_direction = not_applicable, ego_applicability = applies, needs_review = yes
Rationale: Mỗi đầu đèn vật lý mang chỉ lệnh độc lập cho từng luồng xe. Gộp chúng lại sẽ tạo ra xung đột trạng thái (vừa xanh vừa vàng) làm hệ thống downstream không thể diễn giải ngữ nghĩa
Common mistake: Vẽ một hộp bao chung cho cả hai đầu đèn vì thấy chúng nằm liền kề trên cùng một khung treo
Diversity: conflict

---

CASE ID: EC-10
Sample: 17.jpg
Scene: Xe tiến sát bên dưới giá treo đèn trên cao; đầu đèn tín hiệu bắt đầu trôi dần ra khỏi góc nhìn phía trên của kính lái
Observation: Đáy hộp đèn và bóng đèn xanh vẫn nằm trọn trong ảnh, nhưng phần đỉnh hộp đèn bị mép trên của khung hình cắt ngang (ytl = 0)
Decision: LABEL
Expected: Bounding box đặt cạnh trên bám sát mép ảnh (ytl = 0), không kéo tọa độ ra ngoài khung hình. Attribute: signal_form = circular, display = green, arrow_direction = straight, ego_applicability = applies, needs_review = no
Rationale: Duy trì bám vết (tracking) và nhận diện đèn tín hiệu ở cự ly gần nhất trước khi xe vượt qua giao lộ. Loại bỏ đèn cắt mép sẽ tạo khoảng trống mù tín hiệu (blind spot) ngay trước vạch dừng
Common mistake: Bỏ qua không gán vì cho rằng vật thể bị khuyết tật không nguyên vẹn
Diversity: occlusion
