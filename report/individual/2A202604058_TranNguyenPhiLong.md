# Báo cáo Day 13 — Trần Nguyễn Phi Long, 2A202604058

## 1. Kết luận trước
Mã số `2A202604058`, task `D13-2A202604058` (CVAT local, task #23). 10 frame: s01f39, s01f55, s02f08, s02f39, s03f09, s03f24, s04f17, s04f49, s05f14, s05f45. Windows 11, trình duyệt Chrome, CVAT server 2.74.1 (SDK 2.83.0). Bạn ngồi cạnh: PhamPhuNguyenHung-2A202604062.

Tổng **131 cuboid** (Car 60, Pedestrian 28, Cyclist 25, Truck 14, Bus 4). Frame ít nhất là **s05f45** với **8 cuboid** (xe thu dữ liệu chạy nhanh ~17–18 m/s nên ít vật quanh xe). `selfcheck` cờ **3/131** cuboid (`s01f55`, 13: Pedestrian có 25 điểm sát ngoài mép; `s02f39`, 2: Car có `length = 6,01 m` hơi vượt ngưỡng 6,0 m; `s02f39`, 4: Car có `height = 2,40 m` hơi vượt ngưỡng 2,3 m).
Phần tin nhất: **hướng và class**, vì mọi cuboid trong 30 m đều đã đối chiếu camera; đáy cũng chắc (đo theo mặt đường quanh hộp, |khe hở| ≤ 0,2 m ở hầu hết các hộp). Phần yếu nhất: **độ phủ** ở vùng xa 25–30 m (vẫn bỏ qua các cụm quá thưa hoặc không rõ hướng) và **kích thước, tâm** của xe chỉ thấy một mặt (dựng theo bảng 4.3).

Cách dùng CVAT: (1) đổi góc nhìn bằng chuột trái xoay, chuột phải dời, con lăn phóng; xoay tới khi nhìn thẳng xuống để có ảnh từ trên xuống. (2) Cuboid dựng sẵn được đưa lên task dạng **Shape** qua API, sau đó chọn từng hộp và chỉnh ở Top (tâm, dài/rộng, yaw), Side/Front (đáy, chiều cao). (3) Bật **Appearance → Cuboid orientation** để thấy vạch đỏ `+X`, đối chiếu camera để sửa đầu xe; xem cuboid trên 7 ảnh camera bằng `python scripts/lab.py preview`.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ. Ảnh: `t1-s01f39-3d`, `t1-s01f39-top`, `t1-s01f55-3d`, `t1-s01f55-top`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f39 | Phía trước là −y, ở cạnh trên của ảnh `t1-s01f39-top`; +x sang trái xe, +y ra sau | Dọc đường gần phẳng (dốc ~0 %), nghiêng ngang ~2,7 % (phía trái thấp hơn); mặt đường thấp hơn LiDAR ~1,7 m | ~17 m: xe con phía sau-trái ở (5,8; 16,4). Các cụm xa hơn ~30 m, đối chiếu camera đều là cây, cột hoặc bụi, không đọc được là vật | Xe con, xe hai bánh có người, xe đạp dựng không người, nhóm người đứng | Xe con sát bên trái ~3 m; xe hai bánh có người bên phải ~10 m, cả phía trước và phía sau; xe con phía sau-trái ~17 m | Dãy cụm cao ~2,5–2,9 m, rộng ~2 m, nằm dọc x ≈ 11 m (bên trái), lặp lại cách nhau ~6 m: chưa rõ là cây, bồn cây hay xe xếp hàng |
| s01f55 | Phía trước là −y, ở cạnh trên của ảnh `t1-s01f55-top`; +x sang trái xe, +y ra sau | Dọc đường gần phẳng, nghiêng ngang ~1,8 %; thấp hơn LiDAR ~1,7 m. Xe thu dữ liệu đang **đứng yên** (độ dời giữa hai frame kề ≈ 0 m) | ~21 m: xe con ngay phía trước (0,1; −21,1), chỉ thấy mặt đuôi | Xe con xếp hàng phía trước và phía sau, xe hai bánh | Hai xe con ngay phía sau ~7 m; bên trái ~25 m có hai mảng dài 11–12 m mà LiDAR dễ coi là xe buýt/xe tải, nhưng camera cho thấy là hàng cây và bụi | Mảng cao ~4,4 m dài ~21 m bên phải (x ≈ −6 m): tường, hàng cây hay xe lớn đỗ sát nhau, chưa đọc được |

3 điều **chỉ camera** cho biết: (1) **đầu hay đuôi** của xe đứng yên: xe sedan tối màu phía sau-trái s01f39 chỉ phân biệt được đầu nhờ thấy đèn pha; (2) cụm có người lái hay **xe đạp dựng**: cụm xe đạp công cộng trên vỉa hè bên phải s01f39 trông giống xe hai bánh trên point cloud; (3) **loại vật thật**: hai mảng dài ~11–12 m bên trái s01f55 có kích thước như xe buýt nhưng camera cho thấy là hàng cây, và màu xe (trắng, xám, tối) chỉ camera có. 3 điều **chỉ LiDAR** cho biết: (1) khoảng cách chính xác theo mét, ví dụ xe con sát bên trái s01f39 cách tâm LiDAR ~3,4 m; (2) độ cao mặt đường ~1,7 m dưới LiDAR và độ nghiêng ngang 2–3 %; (3) vật ở phía sau và hai bên ngoài vùng camera vẫn có điểm, kể cả ở xa ~40–45 m.

Tám frame lướt (phía trước luôn là −y; cạnh của ảnh tuỳ cách bạn xoay khung khi chụp):

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f08 | −y, trên | Xe thu dữ liệu chạy ~8 m/s; nhiều xe hai bánh và người quanh xe |
| s02f39 | −y, trên | Chạy nhanh ~13 m/s; vài cụm thấp nhầm thành người đứng trên bờ cao |
| s03f09 | −y, trên | Xe thu dữ liệu đứng yên; hàng xe con nối đuôi phía sau |
| s03f24 | −y, trên | ~12 m/s; dải cây phân cách bên trái (x ≈ 6–7 m) bị LiDAR tách thành 4 cụm cỡ xe con, nhưng camera cho thấy toàn là cây |
| s04f17 | −y, trên | ~2–3 m/s; nhiều xe con quanh xe, nhiều xe có phản xạ rất thấp |
| s04f49 | −y, trên | ~12 m/s; có cụm giống xe ở xa nằm cao hơn mặt đường (cầu vượt?) |
| s05f14 | −y, trên | ~2–3 m/s; bên trái ~18 m có các mảng dài 8–12 m cỡ xe tải, nhưng camera cho thấy là bụi cây |
| s05f45 | −y, trên | Chạy nhanh nhất ~17–18 m/s; ít vật gần; có điểm lạ dưới mặt đường |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car`, `t2-b-bus-truck`, `t2-c-ped-cyclist`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | s01f39, 4 (sedan trắng phía trước-trái, 15,58 m) | Car | Có: yaw 268,0°, chỉ về −y; camera thấy đuôi xe (đèn hậu, biển số) | Có: z đáy −1,847 m, khe hở 0,06 m | Camera (cam01): có. Top/Side/Front: khớp | l 4,325 m trong khoảng; w 1,596 m và h 1,389 m hơi dưới khoảng sedan (1,70–1,90 / 1,40–1,60 m) |
| (b) xe buýt hoặc xe tải | s02f08, 5 (xe buýt phía trước-trái, 17,87 m) | Bus | Có: yaw 267,0°; camera thấy đầu xe có bảng LED ở phía mũi tên | Có: z đáy −1,824 m, khe hở 0,14 m | Camera: có. Top/Side/Front: khớp | l 12,322 m, h 3,146 m trong khoảng; w 2,898 m > 2,6 m |
| (c) người đi bộ hoặc xe hai bánh | s01f39, 2 (người đi xe đạp bên phải, 10,37 m) | Cyclist | Có: yaw 268,0°; camera thấy giỏ xe ở phía trước | Có: z đáy −1,138 m trên lối đi cao hơn lòng đường, khe hở 0,07 m | Camera: có. Top/Side/Front: khớp | l 1,520 m và h 1,371 m dưới khoảng xe đạp (1,6–1,9 m); w 0,640 m trong khoảng |

Vòng hiệu chuẩn: không tham gia.

Vật bỏ qua vì không đủ cơ sở:
- Người đi bộ đứng yên và vài người đi xe máy (ví dụ nhóm người đứng ở s01f39 phía trước-phải ~25 m; người đi xe máy sau hàng rào ở s04f17): thấy là người, nhưng không xác định được hướng mặt hoặc hướng đi, nên không vẽ để tránh mũi tên ngược.
- Các cụm có kích thước như xe nhưng camera cho thấy là hàng cây, bụi, cột, xe đạp dựng không người, hoặc một phần thân xe khác (nhiều nhất ở s05f14, s01f55, s02f39): không phải vật, không vẽ.
- Các cụm "người" lơ lửng 0,5–2,3 m trên mặt đường hoặc chỉ có 2–4 điểm: UNCERTAIN_CLASS.
- Vật ở 30–50 m chỉ còn vài chục điểm và camera không nhận ra: LOW_VISIBILITY.

## 4. Bốn hiện tượng chất lượng (Bước 4)
Tất cả lấy từ frame của tôi. Vị trí ghi theo hệ VF (x trái, y sau).

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | s03f09, `t4-1-bong-che`; xe con phía sau-trái ở (3,1; 16,4), cách ~16,7 m | Giả thuyết: xe con đứng ngay trước nó ở (3,0; 9,0) chặn phần lớn tia LiDAR | Xe sau chỉ còn ~310 điểm, xe trước cùng làn ~1205 điểm; trung vị ở 10–20 m trên slide là ~718. Phần thân phía gần LiDAR bị khuyết | Bật **occluded**; cuboid vẫn bao trọn chiều dài suy ra được (~4,5 m), không chỉ bao phần điểm còn thấy |
| 2. Thưa ở xa | s05f14, `t4-2-thua-o-xa`; xe con phía trước-phải ở (−4,3; −31,3), ~31,6 m | Giả thuyết: xa thì khoảng cách giữa các tia tăng, chỉ vài tia chạm vật | 71 điểm trong hộp, gần như chỉ có mặt đuôi xe; camera xác nhận là xe con quay đuôi về phía mình. Ở s03f24 còn có cụm ~33 điểm ở ~42 m trông như xe, nhưng camera cho thấy là cây | Mặt đuôi nhìn rõ nên vẽ, dựng thân 4,5 m theo bảng 4.3 từ mặt đuôi vào. Cụm ~42 m không đủ cơ sở nên bỏ qua, ghi **LOW_VISIBILITY** |
| 3. Phản xạ thấp / ít điểm bất thường | s04f17, `t4-3-phan-xa-thap`; xe con phía trước-phải ở (−6,2; −15,9), ~17 m | Giả thuyết (chưa khẳng định): sơn tối màu, kính, hoặc góc tới xiên làm cường độ trả về rất thấp | Số điểm bình thường (~640) nhưng ~83 % điểm có intensity = 0; xe ở khoảng cách tương tự (12,6; 11,7) chỉ ~18 % | Ghi là **giả thuyết**; vẫn vẽ theo hình học cụm điểm, không đoán thêm thuộc tính từ cường độ |
| 4. Điểm lạ | s05f45, `t4-4-diem-la`; phía trước ~9 m, quanh (3,8; −9,3) và (−3,1; −8,8) | Giả thuyết: phản xạ nhiều lần (mặt đường ướt hoặc thân xe bóng) tạo "điểm ma", hoặc nhiễu cảm biến | 6 điểm ở z ≈ −4,4 đến −4,7 m, tức ~2,5 m **dưới** mặt đường (~−1,9 m); cùng kiểu thấy ở s03f09 và s01f55 | **Không** gán thành vật; không để chúng kéo đáy cuboid xuống |

## 5. Số từ cuboid của tôi (Bước 5)
| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét |
|---|---|---|---|---|---|---|---|
| s01f39, 4, Car | −1,152 m | 1,389 m | −1,847 m | 15,58 m | 15,62 m | 268,0° | Chạm đường: lệch −0,15 m so với −1,7 m, khe hở với mặt đường quanh hộp 0,06 m |
| s02f08, 5, Bus | −0,251 m | 3,146 m | −1,824 m | 17,87 m | 17,87 m | 267,0° | Chạm đường: lệch −0,12 m, khe hở 0,14 m |
| s05f45, 4, Bus | −0,534 m | 3,190 m | −2,129 m | 22,42 m | 22,43 m | 91,0° | Lệch −0,43 m so với −1,7 m nhưng khe hở với mặt đường quanh hộp chỉ 0,16 m: đường chỗ đó thấp hơn (đường nghiêng), không phải hộp chìm |

(a) Gần nhất: s01f39, 1, Car, `dist_m` 3,44 m, r 3,60 m (xe sát bên trái). Xa nhất: s05f14, 18, Pedestrian, `dist_m` 48,68 m, r 48,68 m.
(b) Xe hai bánh s01f39, 2: l 1,520 / w 0,640 / h 1,371 m so với xe đạp có người 1,6–1,9 / 0,5–0,7 / 1,6–1,9 m. Dài và cao nằm dưới khoảng vì nửa dưới bị bụi cây che, LiDAR chỉ thấy phần trên. Xe con s01f39, 4: l 4,325 / w 1,596 / h 1,389 m so với sedan 3,8–5,0 / 1,70–1,90 / 1,40–1,60 m. Rộng và cao hơi thiếu vì chỉ thấy mặt đuôi và một bên, gương và mép phía khuất không có điểm.
(c) Xe cùng chiều phía trước: s01f39, 4, yaw 268,0° (gần 270°). Xe ngược chiều: s04f17, 2 ở làn bên trái, yaw 93,0° (gần 90°).

Mật độ theo khoảng cách:

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| s04f49, 2, Car | 7,86 m | 3052 |
| s05f14, 6, Car | 15,35 m | 906 |
| s04f49, 5, Car | 21,51 m | 435 |
| s04f49, 10, Car | 31,27 m | 159 |

Quy luật: khoảng cách tăng gấp đôi (7,86 → 15,35 → 31,27 m) thì số điểm giảm nhanh theo khoảng cách (3052 → 906 → 159 điểm), gần với quy luật 1/d² (và giảm thêm khi ở xa chỉ thấy một mặt xe). So với slide: 906 điểm ở 10–20 m gần trung vị ~718; 435 điểm ở 20–30 m cao hơn trung vị ~214 vì xe đứng gần ngưỡng 20 m và lộ cả hông lẫn đuôi; xe ở 7,86 m có 3052 điểm, cao hơn trung vị 1968 vì đứng sát LiDAR.

Tự kiểm, các cuboid đã sửa sau `preview` / `selfcheck`:
- s01f39, 1, Car: đối chiếu camera thấy đèn pha ở phía ngược mũi tên, xoay 180° (yaw giờ 270,0°).
- s05f45, 4: đối chiếu camera xác nhận là xe buýt (Bus).
- s01f55, 7, Car: `selfcheck` báo nhiều điểm sát ngay ngoài mép do mặt đuôi xe lọt ra ngoài, đã dời hộp lại cho khớp cụm điểm.

Cảnh báo giữ nguyên: 3/131 (`s01f55`, 13: Pedestrian có 25 điểm sát mép; `s02f39`, 2: Car có `length = 6,01 m` sát ngưỡng trên; `s02f39`, 4: Car có `height = 2,40 m` sát ngưỡng trên).

Đổi kiểm với bạn bên cạnh (`2A202604062` — Phạm Phú Nguyên Hưng): đối chiếu hai con số của xe tải `s03f24, 7, Truck` (`yaw_deg = 88,0°` và `z đáy = −1,699 m`), kết quả khớp.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
(a) Model 3D dự đoán vị trí và kích thước vật trong không gian thật để xe lập kế hoạch tránh, phanh, nên nhãn phải đo bằng mét trong hệ toạ độ xe. Pixel thay đổi theo khoảng cách và camera, không cho biết vật cách bao xa hay lớn bao nhiêu. Vật thưa điểm cần quy tắc riêng (ngưỡng 15–20 % thể tích, LOW_VISIBILITY) vì nếu mỗi người đoán hộp tuỳ ý cho vài chục điểm, model sẽ học nhãn nhiễu và không nhất quán đúng ở vùng xa, nơi nó vốn đã yếu.

(b) Dùng AI:
- Công cụ: Claude Code (Claude Opus), chạy trên máy cá nhân.
- Việc nhờ: Sử dụng Claude để soát lại tài liệu, những mục TODO

(c) Điều còn chưa chắc:
- Hướng xe đứng yên: suy từ hình dạng chỉ đúng khoảng 2/3 khi thử trên xe đang chạy, nên đã phải dựa vào camera. Xe nằm ở góc mù camera (xe con sát bên trái s01f39) vẫn chỉ là giả định cùng chiều.
- Xe chỉ thấy một mặt (đầu hoặc đuôi) được dựng thân 4,5 m theo bảng 4.3, nên chiều dài và tâm hộp có thể lệch.
- Xe van dễ lẫn giữa Car và Truck: van trắng ở s05f14 để Car, van xanh chở hàng ở s05f45 để Truck (UNCERTAIN_CLASS).
- Các vật nhỏ hoặc xa 30–50 m (đặc biệt cụm người đi bộ/xe hai bánh ít điểm) có thể còn sót hoặc chưa chắc hoàn toàn về kích thước.
