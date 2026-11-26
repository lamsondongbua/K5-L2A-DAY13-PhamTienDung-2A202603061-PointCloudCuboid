# Báo cáo Day 13 — Phạm Tiến Dũng, 2A202603061

## 1. Kết luận trước
Mã số `2A202603061`, task `D13-2A202603061`, 10 frame: s01f31, s01f54, s02f21, s02f38, s03f22, s03f51, s04f04, s04f53, s05f15, s05f34. Hệ điều hành Windows 11. Trình duyệt: Microsoft Edge, phiên bản CVAT (2.74.1), MSSV bạn ngồi cạnh: 2A202603051.

Tổng cộng 95 cuboid; frame ít nhất là s01f31 (5 cuboid), nhiều nhất là s01f54 (14). `selfcheck` cờ 10 trên 95 cuboid. s01f31 ít nhất vì frame chỉ có 5 xe rõ: ba xe ở gần và hai xe ở khoảng 44 m, thưa điểm (61 và 56 điểm). Tôi tin nhất phần hướng xe (mũi tên yaw dọc thân xe, `yaw_deg` của xe cùng chiều gần 270°) và yếu nhất phần đáy xe (3 cuboid vẫn bị cờ đáy lệch) và vật xa.


Cách dùng CVAT:
    + Đổi góc nhìn (nhìn từ trên xuống, xoay, dời): em sẽ vẽ theo thứ tự top, front, side sẽ chỉnh lại nếu front nhìn không rõ 
    + Vẽ và chỉnh một cuboid (Shape, nháy đúp, Top/Side/Front): căn sát viền pointcloud và vẽ sát mặt đất
    + Bật mũi tên hướng (Cuboid orientation) và xem cuboid trên ảnh camera (`preview`): bật ở Cuboid Orientation, khi cần xem ảnh 2D, dùng Setting ở ảnh 2D để xem bao quát các vật

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ: s01f31 và s01f54. Ảnh: `t1-s01f31-3d`, `t1-s01f31-top`, `t1-s01f54-3d`, `t1-s01f54-top` (ảnh nhìn từ trên xuống, có đánh dấu phía trước, +x, +y).

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f31 | Phía trước (cam01) là cạnh dưới của ảnh, hướng −y. +x chỉ sang trái, +y chỉ lên trên | Phẳng trong vùng gần (vòng điểm mặt đường đồng tâm đều); theo z đáy cuboid xe, tức thấp hơn LiDAR khoảng 1,7 m. | ~44,5 m (Car #4, #5) | Chỉ Car (5 hộp) | Hai xe phía sau (cạnh trên ảnh), một xe phía trước | Vật xa hơn ~45 m chỉ còn vài điểm, chưa đọc được |
| s01f54 | Như trên: phía trước là −y (cạnh dưới), +x trái, +y lên | z đáy xe ≈ −1,7 m (ví dụ #13: −1,677 m). | ~42,9 m (Car #13) | Car (11), Pedestrian (2), Cyclist (1) | Đoàn xe nối đuôi phía trước (cạnh dưới), hai xe phía sau, người đi bộ và xe hai bánh ở bên phải, gần xe thu dữ liệu | Các xe xếp sát nhau thành hàng ở xa, ranh giới từng xe khó tách |

Chỉ camera cho biết (nhìn cam01, cam03 của s01f54): (1) màu xe, ví dụ nhiều xe trắng xếp hàng phía trước; (2) biển báo, đèn tín hiệu trên khung cổng; (3) chi tiết bề mặt như đèn, kính để nhận ra loại xe. Chỉ LiDAR cho biết: (1) khoảng cách và vị trí 3D chính xác (mét) của từng xe; (2) vật phía sau xe thu dữ liệu, nơi cam01, cam03, cam05 không thấy; (3) chiều cao, mặt đường và kích thước thật của vật.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f21 | Cạnh dưới (−y) | 8 Car, 1 Pedestrian, 1 Cyclist; xa nhất 41,3 m |
| s02f38 | Cạnh dưới (−y) | Có hai Car nằm ngang ở bên phải (vuông góc đường); xa nhất 38,8 m |
| s03f22 | Cạnh dưới (−y) | 10 Car, 3 Pedestrian; Pedestrian #5 bị cờ "điểm sát ngoài mép" |
| s03f51 | Cạnh dưới (−y) | Có 1 Bus dài 11,9 m ở 30,5 m bên trái |
| s04f04 | Cạnh dưới (−y) | 7 Car, không có người; xa nhất 24,6 m |
| s04f53 | Cạnh dưới (−y) | 7 Car, 1 Pedestrian (#1 bị cờ "sát mép") |
| s05f15 | Cạnh dưới (−y) | 12 Car, 1 Truck; Car #12 ở 60,6 m, ngoài 50 m |
| s05f34 | Cạnh dưới (−y) | 6 Car, 1 Truck dài 10,6 m ở 26,4 m |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car`, `t2-b-bus-truck`, `t2-c-ped-cyclist`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | s04f04 #1 | Car | Đúng: mũi tên dọc thân xe, yaw 269,5° ≈ 270° (xe cùng chiều xe thu dữ liệu) | Chạm: z đáy −1,593 m, Side thấy đáy sát điểm xe | Có: Top, Side, Front khớp, đã đối chiếu cam01 (`t2-a-car`) | l 4,93 / w 1,85 / h 1,75 m: trong khoảng Car (3,0–6,0 / 1,4–2,3 / 1,2–2,3 m) |
| (b) xe buýt hoặc xe tải | s05f34 #4 | Truck | Đúng: mũi tên dọc thân xe, yaw 91,6° ≈ 90° (xe ngược chiều) | Chạm: z đáy −1,700 m | Có: Top, Side, Front (`t2-b-bus-truck`) | l 10,64 / w 2,69 / h 2,51 m: l trong khoảng xe tải nặng 6–12 m; h 2,51 m thấp hơn 3,0–4,0 m; w 2,69 m vượt 2,40–2,60 m khoảng 3,5%, trong sai số ±5% |
| (c) người đi bộ hoặc xe hai bánh | s02f38 #4 | Cyclist | Đúng: mũi tên dọc xe, yaw 267,8° ≈ 270° | Chạm: z đáy −1,752 m (lệch 0,05 m) | Có: Top, Side, Front (`t2-c-ped-cyclist`) | l 1,84 / w 0,89 / h 1,62 m: cả ba chiều trong khoảng Cyclist (1,0–2,4 / 0,3–1,2 / 1,0–2,2 m) |


## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân ghi là **giả thuyết**. Frame mượn thì ghi rõ nguồn: task chung, hoặc MSSV của bạn cho mượn.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | s03f51, `t4-1-bong-che` (frame của tôi) | Xe buýt (hộp vàng, cao 2,6 m, 30,5 m) chặn tia LiDAR nên vùng ngay sau nó chưa quan sát được. Giả thuyết | Đếm điểm mặt đường trong dải 10 m sau Bus: 0 điểm, trong khi hai bên cùng khoảng cách có 9 và 40 điểm (dải hai bên rộng gấp đôi). Vùng sau Bus gần như trống | Bật occluded, cuboid bao phần suy luận được; vùng trống sau vật che là chưa quan sát được, không phải đường trống |
| 2. Thưa ở xa | s01f54, `t4-2-thua-o-xa` (frame của tôi) | Xa nên tia LiDAR 40 tia thưa dần theo khoảng cách. Giả thuyết | Car #13 ở 42,9 m chỉ có 46 điểm trong hộp (`points_in`), so với 5839 điểm của xe ở 3,9 m | Vẽ nếu thấy khoảng 15–20% thể tích; không đủ thì bỏ qua, ghi LOW_VISIBILITY |
| 3. Phản xạ thấp / ít điểm bất thường | s03f51 #5 Car, ảnh `t4-3-phan-xa-thap` (frame của tôi; cần chụp lại đúng cuboid này, ảnh hiện tại là s04f04 #1) | Xe tối màu hoặc kính, ít phản xạ tia LiDAR. Giả thuyết, không khẳng định (cũng có thể bị che một phần) | Xe ở 22,1 m chỉ có 65 điểm trong hộp, trong khi trung vị Car ở 20–30 m là 230 điểm; intensity trung bình trong hộp xấp xỉ 0 | Ghi giả thuyết, không khẳng định; vẫn vẽ vì hộp có điểm và nhìn thấy trên ảnh camera, và không nới hộp theo suy đoán |
| 4. Điểm lạ | s01f31, ảnh `t4-4-diem-la` (frame của tôi; cần chụp lại: xoay nghiêng khung 3D về phía trước, ~53 m, ảnh hiện tại là s02f38) | Phản xạ sai hình học (ví dụ tia bị phản xạ qua bề mặt kính, hoặc nhiễu). Giả thuyết | Chỉ có 3 điểm ở z ≈ −24,6 m (thấp hơn mặt đường khoảng 23 m) tại x ≈ −1,5, y ≈ −52,7 m, tách hẳn khỏi các điểm xung quanh | Không gán thành vật, không vẽ cuboid |

## 5. Số từ cuboid của tôi (Bước 5)
Lấy từ `submission/D13-2A202603061-bangA.csv`; số điểm lấy từ cột `points_in` của `-tukiem.csv`. Mặt đường khoảng −1,7 m: z đáy lệch hơn 0,3 m là lơ lửng hoặc chìm.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| s04f04 #1 Car | −0,715 | 1,754 | −1,593 | 3,92 | 3,99 | 269,5 | chạm đường (lệch 0,11 m so với −1,7) |
| s02f38 #4 Cyclist | −0,944 | 1,615 | −1,752 | 5,25 | 5,34 | 267,8 | chạm đường (lệch 0,05 m) |
| s05f34 #4 Truck | −0,445 | 2,510 | −1,700 | 26,35 | 26,35 | 91,6 | z đáy đúng −1,7 m; `selfcheck` vẫn cờ "lơ lửng 0,49 m" (xem phần tự kiểm) |

(a) Gần nhất: s04f04 #1 Car, `dist_m` = 3,92 m, r = 3,99 m. Xa nhất: s05f15 #12 Car, `dist_m` = 60,60 m, r = 60,61 m; hộp này vượt giới hạn 50 m của đề (quyết định giữ). Xa nhất trong 50 m: s03f22 #11 Car, `dist_m` = 45,62 m. Mỗi `dist_m` chỉ lệch r vài phần trăm mét vì z nhỏ so với x, y.

(b) Xe con s04f04 #1: l 4,93 / w 1,85 / h 1,75 m, cả ba chiều nằm trong khoảng Car. Người đi bộ s01f54 #11: l 0,38 / w 0,40 / h 1,43 m.

(c) Xe cùng chiều phía trước ở s04f04 (#2, #4, #5, #6, #7) có `yaw_deg` từ 268,0° đến 274,5°, gần 270°. Xe ngược chiều s04f04 #3 có `yaw_deg` = 87,5°, gần 90°.

Mật độ theo khoảng cách (4 cuboid ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| s04f04 #1 Car | 3,92 | 5839 |
| s02f21 #1 Car | 13,25 | 631 |
| s05f34 #3 Car | 25,69 | 217 |
| s01f54 #13 Car | 42,90 | 46 |

Trung vị `points_in` của Car theo khoảng cách (từ phiếu tự kiểm): 0–10 m: 1503 điểm (15 xe); 10–20 m: 631 (25 xe); 20–30 m: 230 (19 xe); 30–40 m: 117 (12 xe); 40–50 m: 62 (7 xe).

Quy luật: số điểm giảm rất nhanh theo khoảng cách, mỗi khi xa thêm 10 m thì số điểm còn khoảng 1/2 đến 1/2,7 (1503 → 631 → 230 → 117 → 62). Gần giống đường cong ở slide (1968, 718, 214, 72 điểm ở 0–10, 10–20, 20–30, 30–40 m): vùng 0–10 m của tôi thấp hơn một chút (1503 so với 1968), vùng 30–40 m cao hơn (117 so với 72). Nguyên nhân: tia LiDAR 40 tia tỏa ra, nên số tia trúng một xe giảm nhanh theo khoảng cách.

Phần tự kiểm.

Cảnh báo sau lần `sync` mới nhất (10/95): s03f22 #5, s03f22 #7, s03f22 #11, s03f51 #6, s04f53 #1, s05f15 #11, s05f15 #12, s05f34 #1, s05f34 #3, s05f34 #4. Cuboid đã sửa: s05f34 #4 (đổi class sang Truck, sau đó đặt đáy về −1,7 m), s03f51 #6, s05f34 #3, s03f22 #7 (đặt đáy về −1,7 m). Cảnh báo giữ nguyên và lý do: s03f22 #11 (cao 2,34 m) và s05f15 #11 (rộng 3,24 m) chỉ vượt ngưỡng 0,04 m; s05f15 #12 (rộng 2,41 m, cao 2,57 m) và s05f34 #1 (cao 2,79 m) có thể là xe cao như SUV hoặc van; s03f22 #5 và s04f53 #1 (Pedestrian bị cờ "sát mép") có thể là vật nằm cạnh người. Xác nhận giữ lại do đủ điểm pointcloud để vẽ. Lưu ý: ba cuboid s05f34 #3, s03f22 #7, s03f51 #6 vẫn bị cờ đáy lệch 0,41–0,63 m so với mặt đường cục bộ ước lượng từ vòng điểm quanh hộp, trong khi z đáy = −1,7 m; mặt đường ở đó có thể không đúng −1,7 m.

2A202603051 đã kiểm chéo.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
(a) Nhãn 3D đo bằng mét vì model cần biết vị trí và kích thước thật của vật để lái xe an toàn, còn pixel đổi theo khoảng cách và góc camera. Vật thưa điểm cần quy tắc riêng (15–20% thể tích mới vẽ, LOW_VISIBILITY) vì vẽ theo suy đoán sẽ cho model học từ nhãn không có điểm hỗ trợ.

(b) Dùng AI:
- Công cụ: Claude Code (Anthropic), theo thông báo của giảng viên trên lớp cho phép dùng AI hỗ trợ sau khi gán nhãn xong.
- Câu hỏi chính: cách đọc cờ `selfcheck` và ý nghĩa `#idx`; chỉnh `z` của 4 cuboid bị cờ đáy (s05f34 #3, s05f34 #4, s03f22 #7, s03f51 #6); script chụp màn hình; viết khung báo cáo, điền số từ bảng A và phiếu tự kiểm, và viết nhận xét từ ảnh chụp và số điểm LiDAR (bóng che, phản xạ thấp, điểm lạ).
- Cách dùng kết quả: tôi tự gán toàn bộ cuboid ban đầu trên CVAT; phần AI viết tôi đọc lại, đối chiếu với bảng A và ảnh, và sửa chỗ không khớp trước khi nộp.

(c) Điều còn chưa chắc: mặt đường cục bộ ở s05f34 #3, s03f22 #7, s03f51 #6 (đáy đặt −1,7 m nhưng `selfcheck` vẫn cờ); hộp xa s05f15 #12 ở 60,6 m, ngoài giới hạn 50 m, tôi quyết định giữ; phân loại Truck hay Bus của hộp dài khoảng 10–12 m; hai Pedestrian bị cờ "sát mép" (s03f22 #5, s04f53 #1).
