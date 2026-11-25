# Báo cáo Day 13 — Đặng Minh Huy, 2A202603162

## 1. Kết luận trước

Mã số 2A202603162, task `D13-2A202603162`, 10 frame của tôi: s01f04, s01f28, s02f15, s02f54, s03f34, s03f56, s04f10, s04f28, s05f03, s05f24. Máy Windows, trình duyệt Chrome, CVAT v2.83.0 (đăng nhập bằng tài khoản của tôi, xem trong About); MSSV bạn ngồi cạnh: chưa ghi nhận để kiểm chéo.

Tổng cộng tôi vẽ **73 cuboid** trên 10 frame. Frame ít cuboid nhất là **s02f54, s03f34, s03f56 (mỗi frame 5)** vì cả ba đoạn ghi này chỉ có vài cụm điểm chắc chắn trong 50 m; frame nhiều nhất là s01f04 và s04f28 (10 mỗi frame) do hai làn xe nối dài. `selfcheck` cờ **4/73** cuboid (1 lỗi kích thước, 3 lỗi đáy so với mặt đường). Tôi tin nhất phần **hướng yaw** (xe cùng chiều sát ~270°, ngược chiều sát ~90°, mũi tên bật đúng chiều đầu xe) và yếu nhất phần **đáy hộp ở vật xa ≥ 30 m** — ở đó gần như không có điểm mặt đường dưới hộp nên phải suy bằng class template.

Cách dùng CVAT (3 dòng): đổi góc nhìn bằng con lăn (zoom), giữ Ctrl + chuột trái để xoay, chuột phải để dời, phím tắt chuyển 3D/Top view. Vẽ cuboid: chọn **Shape** Car trong bảng cuboid, kéo khung 3D, nháy đúp vào hộp để chỉnh từng cạnh ở view Top/Side/Front. Bật **Cuboid orientation** để thấy mũi tên yaw, và dùng `python scripts/lab.py preview` để chiếu cuboid lên 7 ảnh camera vì bản CVAT lớp không có nút tự chiếu.

## 2. Đọc frame (Bước 2)

Hai frame đọc kỹ: **s01f04** và **s01f28**. Ảnh: `t1-s01f04-3d`, `t1-s01f04-top`, `t1-s01f28-3d`, `t1-s01f28-top`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f04 | Trong ảnh Top của CVAT phía trước xe thu nằm ở đáy ảnh; +x chỉ sang trái thân xe, +y chỉ về phía sau | Phẳng, mặt đường thấp hơn LiDAR khoảng 1,7–1,8 m | ~35 m (1 ô con phía sau, 106 điểm) | Ô tô con ở hai bên + phía sau, 1 truck bên trái | Dải 5–6 ô cách đều 10–35 m phía sau (đỗ/dừng); xe ngược chiều bên phải | Vệt điểm thấp dọc mép đường hai bên, không rõ là rào hay cây |
| s01f28 | Tương tự: phía trước ở đáy ảnh, +x sang trái, +y ra sau | Dốc nhẹ (z đáy dao động −1,72 đến −2,06 giữa các hộp) | ~39 m (ô con 0,31; 39,09 m) | Chủ yếu ô tô con, 1 truck hai bên | Hàng ô đỗ dọc hai bên ở 12–17 m | Phía sau trên 40 m không đọc được, chỉ còn vệt thưa |

Ba điều **chỉ camera** cho biết (s01f04, s01f28): (1) vật phía sau là **ô tô đang đỗ hay đang chạy** (đèn/tài xế qua cửa kính), point cloud không phân biệt được; (2) **loại chính xác** sedan hay SUV/pickup — cụm điểm phía sau hai xe giống nhau; (3) **vạch kẻ đường và hướng làn**, để chắc mũi tên yaw chỉ đúng đầu xe. Ba điều **chỉ LiDAR** cho biết: (1) **khoảng cách thật theo mét** đến từng cụm (camera chỉ cho thấy vật gần hay xa tương đối); (2) **mặt đường thấp hơn LiDAR bao nhiêu** — cụm điểm đất ở −1,75 m; (3) **vật bị che vẫn tồn tại**: sau ô tô bên trái có một cụm điểm riêng ở 15–20 m mà ảnh camera cùng lúc không nhìn thấy.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f15 | đáy ảnh (+x trái, +y sau) | Dải 6 ô song song đều nhau; một ô cách 4,2 m ngay bên trái phải dùng kích thước template |
| s02f54 | đáy ảnh | Xe thưa; một ô 32,7 m chỉ còn 149 điểm |
| s03f34 | đáy ảnh | Có xe buýt đọc được ở **64 m**, cao đáy −1,44 m (dốc lên); thêm 1 Cyclist và 1 Pedestrian gần 10 m |
| s03f56 | đáy ảnh | Một Cyclist bị xe trước che gần hết, tôi đặt occluded=1, chỉ 106 điểm |
| s04f10 | đáy ảnh | Bên phải dày 9 hộp ở 4,7–39 m; một Cyclist/xe hai bánh khó tách khỏi bóng xe buýt |
| s04f28 | đáy ảnh | Hai chiều xe đan xen: nhóm yaw ~272° và nhóm ~89° xen kẽ ở cùng hành lang x≈−6…+6 |
| s05f03 | đáy ảnh | Xe gần nhất chỉ 3,43 m, box phải kéo dài l=4,48 để ôm trọn cụm 5151 điểm |
| s05f24 | đáy ảnh | Mặt đường tương đối phẳng, tất cả hộp trong 26,4 m, không có bus/pickup |

## 3. Ba cuboid có hướng dẫn (Bước 3)

Ảnh: `t2-a-car`, `t2-b-bus-truck`, `t2-c-ped-cyclist`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | s05f03, 6 | Car | Có, 272,5° (cùng chiều) | Gần chạm, hở +0,37 m (chưa quá ngưỡng 0,3 m? tự kiểm ghi 0,37 → hơi lơ lửng, giữ vì mép đường cao hơn mặt chính) | Có, cả ba view + `preview` | 4,48 × 1,96 × 1,49 — trong khoảng sedan, sai số < ±5% |
| (b) xe buýt | s03f56, 5 | Bus | Có, 88,8° (ngược chiều, đầu xe hướng về phía trước ảnh) | Không — đáy −2,12 m, cờ **chìm 1,02 m** | Có; view Side cho thấy hộp lún sâu xuống đám điểm mặt đường | 12,0 × 2,5 × 3,0 — đúng khung buýt thường, nhưng h chưa sửa vì chưa chắc mặt dốc |
| (c) người / hai bánh | s03f34, 4 | Pedestrian | Có, 91,8° | Có, hở −0,05 m (sát đường) | Có + ảnh camera (nhìn thấy người đi bộ bên trái) | 0,73 × 0,98 × 1,53 — w sát mép trên khoảng 0,5–0,9, trong dung sai ±10% của Pedestrian |

Vòng hiệu chuẩn: cuboid của tôi lệch ở **chiều cao h của xe con phía trước** — tôi để h ≈ 1,6–1,84 trong khi bạn bên cạnh để 1,5; cả lớp chốt theo guideline là h phải **khớp đỉnh cụm điểm ở view Side**, không lấy số đẹp, và bán tải/xe chở hàng phải là **Truck** chứ không phải Car.

Vật bỏ qua vì không đủ cơ sở: (1) s04f28 — cụm hai bánh dựng bên đường không có người, guideline nói **không vẽ**; (2) s02f54 — cụm điểm mờ phía sau xe buýt ở ~45 m, nghi **UNCERTAIN_CLASS** (bus hay truck nối toa); (3) s01f28 — vệt thấp sau dải ô đỗ, nghi **LOW_VISIBILITY** của rào chắn, không đủ 5 điểm đặc trưng.

## 4. Bốn hiện tượng chất lượng (Bước 4)

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | s03f56, ảnh `t4-1-bong-che.png` | Xe con phía trước chắn tia LiDAR | Cyclist chỉ còn 106 điểm ở rìa hộp, phần thân không điểm; camera thấy rõ người trên xe | Đặt `occluded=1`, dùng kích thước Cyclist template, không đoán sâu vào vùng không điểm |
| 2. Thưa ở xa | s03f34 (Bus 64 m, 92 điểm), ảnh `t4-2-thua-o-xa.png` | Tia phân kỳ, công suất phản hồi giảm theo bình phương khoảng cách | Đám điểm vỡ từng mảng ngang, không thấy輪/rìa đáy | Ôm cả cụm bằng kích thước bảng 4.3 và đánh dấu vật ở `dist_bin` ≥ 50 |
| 3. Phản xạ thấp | s01f28, idx 8 (39,09 m, 72 điểm), ảnh `t4-3-phan-xa-thap.png` | Sơn tối/vật nhựa phía sau hấp thụ tia | Cụm điểm chỉ thấy mặt trên, thiếu mặt bên, đáy hộp phải suy | Tăng dung sai kích thước, ưu tiên khớp camera ở `preview` thay vì bám cụm điểm |
| 4. Điểm lạ | s05f03, idx 3 (chìm 0,41 m), ảnh `t4-4-diem-la.png` | Phản xạ nhiều lần từ mặt đường/xe cạnh (ghost) | Hai vệt điểm nằm thấp hơn mặt đường khoảng 0,4 m ngay dưới hộp | Không kéo đáy theo điểm lạ; giữ đáy theo mặt đường ước lượng và ghi cờ tự kiểm |

## 5. Số từ cuboid của tôi (Bước 5)

Lấy từ `submission/D13-2A202603162-bangA.csv`; số điểm lấy từ cột `points_in` của `-tukiem.csv`. Mặt đường khoảng −1,7 m: z đáy lệch hơn 0,3 m là lơ lửng hoặc chìm.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| s05f03, 6, Car | −0,671 | 1,492 | −1,417 | 3,43 | 3,49 | 272,5 | Hở +0,28 so với −1,7 → hơi **lơ lửng** (tự kiểm ghi hở 0,37 m) |
| s01f04, 1, Car | −1,016 | 1,438 | −1,735 | 6,96 | 7,04 | 270,8 | Đáy −1,735 → **chạm đường** |
| s03f56, 5, Bus | −0,618 | 3,000 | −2,118 | 28,54 | 28,55 | 88,8 | Thấp hơn −1,7 tới 0,42 m; tự kiểm cờ **chìm 1,02 m** |
| s03f34, 5, Bus | 0,056 | 3,000 | −1,444 | 64,02 | 64,02 | 271,7 | **Lơ lửng** 0,26 m — khả năng mặt đường dốc lên ở cuối frame |

(a) Cuboid gần nhất: **s05f03 idx 6**, `dist_m` = 3,43, r = 3,49. Cuboid xa nhất: **s03f34 idx 5 (Bus)**, `dist_m` = 64,02, r = 64,02 (vượt phạm vi 50 m nên là hộp phải cân nhắc nhất).

(b) Pedestrian s03f34, 4: l 0,734; w 0,981; h 1,534. **w 0,981 vượt khoảng 0,5–0,9 của bảng 4.3** (~9%) — vì hộp ôm cả vạt áo/túi xách, nằm trong dung sai ±10% của Pedestrian nên tôi giữ. Cyclist s03f56, 4: l 1,600; w 0,500; h 1,600 — sát mép dưới ba chiều, khớp "xe đạp có người". Car s02f15, 4: 4,83 × 1,80 × 1,60 — **h 1,60 trên mép trên sedan (1,40–1,60)**, tôi để theo SUV/minivan (1,65–2,10 không đủ) nên chấp nhận 1,60 với dung sai ±5%.

(c) Xe cùng chiều phía trước/sau: s01f04, 1 `yaw_deg` = 270,8 ≈ 270°. Xe ngược chiều: s01f04, 4 = 92,9°; s02f15, 4 = 91,4° ≈ 90°. Nhóm yaw frame nào cũng bám chặt hai giá trị này → hướng xe lưu thông chỉ hai chiều dọc theo đường.

Mật độ theo khoảng cách (4 cuboid ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| s04f10, 2, Car | 4,71 | 4544 |
| s01f28, 3, Car | 12,56 | 904 |
| s05f24, 3, Car | 20,39 | 335 |
| s04f28, 3, Car | 31,88 | 129 |

Quy luật: khoảng cách **gấp ~6,8 lần (4,71 → 31,88 m) thì số điểm còn ~1/35 (4544 → 129)**, gần với tỉ lệ nghịch bình phương khoảng cách (6,8² ≈ 46, thực tế giảm ít hơn vì tia quét dày dần xuống mặt xe). So với đường cong bài giảng (xe con: 1968 điểm ở 0–10 m, 718 ở 10–20 m, 214 ở 20–30 m, 72 ở 30–40 m) thì số của tôi **cao hơn nhẹ ở mọi bin** (904 vs 718; 335 vs 214; 129 vs 72) — có thể vì hộp tôi vẽ hơi rộng hơn thân xe thật, một phần vì các xe này là SUV to.

Phần tự kiểm — hai cuboid tôi **đã sửa** sau `preview`/`selfcheck`: (1) **s03f56, 4 (Cyclist)** — sau `preview` thấy hộp lấn sang xe con phía trước, tôi thu gọn dài 1,600 và đánh dấu `occluded=1`; (2) **s01f04, 5 (Car)** — sau `selfcheck` hạ thấp đỉnh 2,42 → **2,374** để không vượt hẳn trần SUV, dù vẫn còn cờ "ngoài khoảng 1,2–2,3". Cảnh báo tôi **giữ nguyên**: (a) s01f28, 1 (chìm 0,33 m) — cụm đỗ trên lề thấp hơn mặt chính, hạ thêm sẽ cắt mất điểm thân; (b) s03f56, 5 (Bus chìm 1,02 m) — hai view Side không cho mặt đường đáng tin dưới gầm xe buýt; (c) s05f03, 3 (chìm 0,41 m) — nghi điểm ma phía dưới, không muốn kéo đáy theo.

Đổi kiểm: tôi chưa ghi được MSSV bạn kiểm chéo, nên tự đối chiếu hai con số với bảng A: **tổng cuboid 73** và **khoảng cách hộp xa nhất 64,02 m (s03f34, 5)** — cả hai khớp bảng A, không lệch.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

(a) Nhìn từ phía model sẽ học trên nhãn này: nhãn 3D phải đo bằng **mét** vì model dự đoán vị trí thật để lập kế hoạch tránh va chạm — pixel chỉ cho vị trí trên ảnh, không suy ra được khoảng cách nếu vật nằm ngoài tầm hiệu chuẩn; còn vật thưa điểm cần quy tắc riêng (template theo class, `occluded`, giới hạn 50 m) để nhà cung cấp nhãn không buộc model học từ hộp suy đoán, giúp đánh giá độ chính xác công bằng giữa vật gần/xa.

(b) AI: dùng **GitHub Copilot trong VS Code** để hỏi lệnh `lab.py sync`/`selfcheck`, ý nghĩa cột `z_bottom` và cách đặt tên ảnh theo mẫu; ngoài ra nhờ AI rà lại tên file và định dạng bảng theo danh sách SUBMISSION. Cách dùng: mọi con số và nhận xét trong báo cáo tôi lấy từ bảng A/phiếu tự kiểm và từ quan sát của chính mình trong CVAT.

(c) Điều còn chưa chắc: hộp Bus ở **s03f34, 5 (64,02 m)** vượt quá giới hạn 30–50 m đề bài nêu — nếu coach tính ngoài phạm vi thì tôi sẽ xoá. Đáy của **s03f56, 5** và **s05f03, 3** còn nghiêng giữa hai giả thuyết "mặt đường dốc xuống" và "điểm ma". Ngoài ra, vì phần lớn cluster đọc được nằm phía sau (yaw nhóm 270°), tôi chưa chắc mình hiểu nhất quán chiều dương yaw trong CVAT so với quy ước 0° chỉ +x — phát hiện sau khi đã hết hạn nộp nên vẫn giữ nguyên hộp như bảng A.
