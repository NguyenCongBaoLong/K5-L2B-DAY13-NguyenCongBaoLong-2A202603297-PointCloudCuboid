# Báo cáo Day 13 — <Nguyễn Công Bảo Long>, <2A202603297>

> **Cách dùng mẫu này:** đổi tên file thành `<MSSV>_<HoTen>.md` (họ tên không dấu, viết liền), ví dụ `2A202600123_NguyenVanAn.md`. Đổi tên, đừng chép thành file thứ hai.
>
> Xoá mọi dấu TODO và dòng gợi ý (dòng bắt đầu bằng `>`). Mỗi con số lấy từ **bảng A** hoặc **phiếu tự kiểm** trong `submission/`, kèm đơn vị (m, °). Ảnh để trong `report/individual/images/` (nộp LMS trong file zip, không commit); trong báo cáo chỉ ghi **tên file ảnh**. Báo cáo này nằm trên repo public: không chép biển số, chữ trên biển hiệu hay biển chỉ đường, tên đường, địa danh; nhắc tới bạn khác thì ghi MSSV, không ghi họ tên. Cả báo cáo ~2 trang.

## 1. Kết luận trước
TODO(học viên): mã số, task `D13-2A202603297`, 10 frame của bạn; hệ điều hành, trình duyệt, phiên bản CVAT (tên tài khoản → About); MSSV bạn ngồi cạnh. Rồi 2–3 câu: tổng bao nhiêu cuboid, frame nào ít nhất và vì sao; `selfcheck` cờ bao nhiêu cuboid trên tổng số; bạn tin phần nào nhất và phần nào còn yếu nhất (hướng, đáy, kích thước, class, vật xa).

TODO(học viên): cách dùng CVAT, 3 dòng: đổi góc nhìn (nhìn từ trên xuống, xoay, dời); vẽ và chỉnh một cuboid (Shape, nháy đúp, Top/Side/Front); bật mũi tên hướng (Cuboid orientation) và xem cuboid trên ảnh camera (`preview`).

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ. Ảnh: `t1-<s01f20>-3d`, `t1-s01f20-top` (ảnh nhìn từ trên xuống, có đánh dấu phía trước, +x, +y).

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f20 | Phía trước hướng về cạnh dưới (`-y` trên khung ảnh chuẩn), trục `+x` chỉ sang phải, trục `+y` chỉ lên trên | Mặt đường phẳng, thấp hơn cảm biến LiDAR khoảng `1,7 m` (`z_bottom` các xe quanh `-1,70 m` đến `-1,90 m`) | `40,12 m` (`Car` idx 8) | `Car`, `Bus`, `Pedestrian` | Có xe chạy ngược chiều ở làn bên cạnh (`idx 5`), các xe cùng chiều phía trước/sau (`idx 1, 2, 4, 6, 8`) và người đi bộ bên lề (`idx 3`) | Các cụm điểm rải rác ở xa trên `40 m` và sau hàng rào/vật cản hai bên đường |
| s01f46 | Phía trước hướng về cạnh dưới (`-y` trên khung ảnh chuẩn), trục `+x` chỉ sang phải, trục `+y` chỉ lên trên | Mặt đường phẳng, thấp hơn cảm biến LiDAR khoảng `1,7 m` (`z_bottom` các xe quanh `-1,75 m` đến `-1,85 m`) | `14,18 m` (`Pedestrian` idx 7) | `Truck`, `Car`, `Cyclist`, `Pedestrian` | Có xe tải nhỏ/bán tải bên trái (`idx 1`), các xe con phía trước và sau (`idx 2, 3, 4`), cùng nhóm người đi xe đạp và người đi bộ bên lề trái (`idx 5, 6, 7`) | Các cụm điểm nhỏ sát công trình/cây cối bên lề đường và các điểm thưa ở xa |

TODO(học viên): 3 điều **chỉ camera** cho biết và 3 điều **chỉ LiDAR** cho biết, lấy từ chính hai frame trên.

- **3 điều chỉ camera cho biết (từ `s01f20` và `s01f46`):**
  1. Màu sắc, kiểu dáng thân xe, kính lái và đèn xe giúp phân biệt chính xác đâu là đầu xe hay đuôi xe.
  2. Phân biệt rõ người đi xe đạp (`Cyclist` idx 5) với hai người đi bộ (`Pedestrian` idx 6, 7) đứng gần nhau trên `s01f46`.
  3. Nhận diện được các vật thể ở khoảng cách xa hoặc bị che khuất một phần mà trên point cloud chỉ còn vài chục điểm rời rạc.
- **3 điều chỉ LiDAR cho biết (từ `s01f20` và `s01f46`):**
  1. Tọa độ không gian 3D, khoảng cách thực tế tính bằng mét (`dist_m`) và góc phương vị (`az_deg`) của từng vật quanh xe thu dữ liệu.
  2. Chiều cao thực tế của phương tiện (ví dụ phân biệt `Bus` cao `3,324 m` với `Car` cao `1,48–1,56 m` trên `s01f20`) và độ cao mặt đường (`~ -1,7 m`).
  3. Mức độ dày/thưa của đám mây điểm phản xạ theo khoảng cách và góc che khuất.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f16 | Cạnh dưới (`-y`) | Nhiều xe chạy ngược chiều ở làn bên phải (`yaw_deg` quanh `90°–93°`) |
| s02f40 | Cạnh dưới (`-y`) | Có xe đỗ/chạy ngang hoặc chéo góc ở bên trái (`idx 1` góc `0,0°`, `idx 5` góc `358,5°`) |
| s03f04 | Cạnh dưới (`-y`) | Các xe di chuyển khá sát nhau quanh phạm vi dưới `10 m` |
| s03f58 | Cạnh dưới (`-y`) | Xuất hiện xe buýt lớn (`Bus` idx 2 dài `12,058 m`) ở phía sau và xe hai bánh ở xa (`22,14 m`) |
| s04f28 | Cạnh dưới (`-y`) | Có xe buýt ngược chiều ở xa (`38,44 m`) kèm nhiều điểm lề đường sát mép hộp |
| s04f49 | Cạnh dưới (`-y`) | Các xe phân bố trải dài cả phía trước gần (`7,62 m`) lẫn phía sau xa (`32,89 m`) |
| s05f00 | Cạnh dưới (`-y`) | Có xe tải cao `3,500 m` (`Truck` idx 4) ở phía sau gần (`9,13 m`) |
| s05f19 | Cạnh dưới (`-y`) | Mật độ phương tiện tập trung chủ yếu ở dải trung bình `10–26 m` |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car`, `t2-b-bus-truck`, `t2-c-ped-cyclist`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f20, idx 1` | `Car` | Có (`269,0°`, chỉ thẳng theo chiều xe chạy) | Có (`z_bottom = -1,795 m`, `ground_gap_m = 0,05 m`) | Đã kiểm | Có (`4,715 × 1,901 × 1,561 m`) |
| (b) xe buýt hoặc xe tải | `s01f20, idx 2` | `Bus` | Có (`270,8°`, dọc thân xe buýt) | Có (`z_bottom = -1,896 m`, `ground_gap_m = 0,09 m`) | Đã kiểm | Có (`11,775 × 3,200 × 3,324 m`) |
| (c) người đi bộ hoặc xe hai bánh | `s01f46, idx 5` | `Cyclist` | Có (`271,3°`, theo hướng di chuyển) | Có (`z_bottom = -1,436 m`, `ground_gap_m = 0,26 m`) | Đã kiểm | Có (`1,766 × 0,707 × 1,798 m`) |

- **Vòng hiệu chuẩn và vật bỏ qua:** Ở vòng hiệu chuẩn, cuboid của tôi lệch nhẹ so với các bạn ở độ cao mặt đáy (`z_bottom`) và độ ôm sát biên chiều rộng của xe; cả lớp thống nhất lấy mốc mặt đường quanh `-1,7 m`, ưu tiên bao trọn thân vật nhưng không ôm cả điểm nhiễu/lề đường bên cạnh. Các vật tôi bỏ qua vì không đủ cơ sở gồm các cụm điểm rời rạc nằm ngoài `40–50 m` trên `s01f20` và `s04f49` (`LOW_VISIBILITY` do quá ít điểm để xác định hướng và ranh giới hộp).

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân ghi là **giả thuyết**. Frame mượn thì ghi rõ nguồn: task chung, hoặc MSSV của bạn cho mượn.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s01f46, idx 6` (`Pedestrian`) / task cá nhân | Người đi xe đạp (`idx 5`) và người đi bộ (`idx 7`) đứng gần che một phần tia LiDAR | Cụm điểm chỉ còn `40` điểm ở `13,7 m`, sát ngay bên cạnh có thêm `32` điểm của vật kế bên | Dễ vẽ hộp quá chật hoặc ôm nhầm điểm người bên cạnh; cần đối chiếu camera để tách riêng từng người |
| 2. Thưa ở xa | `s01f20, idx 8` (`Car` ở `40,12 m`) / task cá nhân | Khoảng cách xa làm tia quét LiDAR giãn rộng, mật độ điểm giảm mạnh | Chỉ còn `101` điểm tạo thành khung mờ ở cạnh dưới và mặt sau xe | Khó căn chính xác bề rộng (`width = 2,33 m` hơi dư nhẹ) và chiều dài đầu xe; không gán nếu xa hơn `50 m` |
| 3. Phản xạ thấp / ít điểm bất thường | `s03f58, idx 5` (`Cyclist` ở `22,14 m`) / task cá nhân | Diện tích phản xạ nhỏ của xe hai bánh và bề mặt tối màu hấp thụ tia laser | Chỉ thu được `59` điểm ở khoảng cách `22,1 m`, phần bánh sát đất rất ít điểm | Dễ làm đáy hộp bị nâng cao hơn mặt đường (`ground_gap_m = 0,39 m`); cần kéo đáy hộp xuống sát mặt đường |
| 4. Điểm lạ | `s04f28, idx 5` (`Bus` ở `38,44 m`) / task cá nhân | Điểm phản xạ từ dải phân cách/lề đường hoặc tán xạ ở rìa thân xe dài | Có tới `159` điểm nằm sát ngay ngoài mép hộp dù hộp đã rộng `3,20 m` | Không mở rộng hộp quá giới hạn vật lý để ôm điểm nhiễu lề đường nhằm tránh sai lệch kích thước thật |

## 5. Số từ cuboid của tôi (Bước 5)
Lấy từ `submission/D13-2A202603297-bangA.csv`; số điểm lấy từ cột `points_in` của `-tukiem.csv`. Mặt đường khoảng −1,7 m: z đáy lệch hơn 0,3 m là lơ lửng hoặc chìm.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s01f20, 1, Car` | `-1,015 m` | `1,561 m` | `-1,796 m` | `3,67 m` | `3,81 m` | `269,0°` | Chạm đường (`ground_gap_m = 0,05 m`, lệch `0,096 m` so với `-1,7 m`) |
| `s01f20, 7, Car` | `-0,426 m` | `1,000 m` | `-0,926 m` | `33,47 m` | `33,47 m` | `0,0°` | Lơ lửng (`ground_gap_m = 0,70 m`, lệch `0,774 m` so với `-1,7 m`) |
| `s01f46, 5, Cyclist` | `-0,537 m` | `1,798 m` | `-1,436 m` | `12,67 m` | `12,68 m` | `271,3°` | Chạm đường trong ngưỡng (`ground_gap_m = 0,26 m`, lệch `0,264 m` so với `-1,7 m`) |

- **Phân tích số liệu từ bảng A:**
  - **(a) Gần nhất và xa nhất:** Cuboid gần nhất là `s01f46, idx 1` (`Truck`) với `dist_m = 3,09 m` và `r = 3,15 m`; cuboid xa nhất là `s01f20, idx 8` (`Car`) với `dist_m = 40,12 m` và `r = 40,14 m`.
  - **(b) Kích thước l, w, h:** Một `Pedestrian` chuẩn (`s01f20, idx 3`) có `l = 0,431 m`, `w = 0,613 m`, `h = 1,411 m` (đều nằm trong khoảng tham chiếu bảng 4.3); một `Car` chuẩn (`s01f20, idx 1`) có `l = 4,715 m`, `w = 1,901 m`, `h = 1,561 m` (nằm gọn trong khoảng `3,0–6,0 m`, `1,4–2,3 m`, `1,2–2,3 m`). Riêng `Car` ở xa `s02f40, idx 3` (`36,79 m`) có `w = 2,630 m` và `h = 2,327 m` vượt nhẹ ngưỡng `2,3 m` do điểm ở xa bị nhòe biên nên hộp bị kéo rộng hơn thân xe thật.
  - **(c) Hướng `yaw_deg`:** Xe chạy cùng chiều phía trước (`s01f20, idx 1`) có `yaw_deg = 269,0°` (rất gần `270°`); xe chạy ngược chiều (`s01f20, idx 5` hoặc `s02f16, idx 4`) có `yaw_deg = 91,4°` và `92,2°` (rất gần `90°`).

Mật độ theo khoảng cách (4 cuboid ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| `s01f20, 1, Car` | `3,67 m` | `2798` |
| `s01f46, 4, Car` | `10,41 m` | `876` |
| `s05f19, 5, Car` | `25,47 m` | `349` |
| `s01f20, 8, Car` | `40,12 m` | `101` |

- **Quy luật mật độ và tự kiểm:** Khi khoảng cách tăng lên gấp khoảng `3 lần` (từ `3,67 m` lên `10,41 m`), số điểm giảm còn khoảng `1/3` (`2798` xuống `876` điểm), và khi xa gấp hơn `10 lần` (`40,12 m`) thì số điểm chỉ còn khoảng `1/28` (`101` điểm); xu hướng này khớp chặt với đường cong suy giảm mật độ theo khoảng cách ở slide bài giảng (`1968` điểm ở `0–10 m`, `718` ở `10–20 m`, `214` ở `20–30 m`, `72` ở `30–40 m`).
  - **Đã sửa sau `preview` / `selfcheck`:** Đã điều chỉnh lại `s01f20, idx 1` (`Car`) và `s01f20, idx 2` (`Bus`) để xoay mũi tên `yaw_deg` thẳng dọc thân xe (`269,0°` và `270,8°`) và hạ đáy hộp sát mặt đường (`ground_gap_m` chỉ còn `0,05 m` và `0,09 m`).
  - **Cảnh báo giữ nguyên và lý do:** Giữ nguyên cảnh báo sát mép ngoài trên `s01f46, idx 6` (`32 điểm sát ngay ngoài mép`) và `s04f28, idx 5` (`159 điểm sát ngay ngoài mép`) vì các điểm đó thuộc người đi bộ đứng kế bên hoặc dải phân cách lề đường, nếu nới rộng hộp để bao vào sẽ làm sai lệch kích thước thật của đối tượng.

TODO(học viên): đổi kiểm: MSSV người đã kiểm chéo với bạn, hai con số nào được đối chiếu với bảng A, khớp hay lệch (lệch thì đã sửa gì).

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
- **(a) Phía mô hình học trên nhãn:** Nhãn 3D đo bằng mét giúp mô hình nhận thức tự hành học đúng kích thước vật lý, vị trí không gian và hướng di chuyển thực tế của vật thể thay vì bị biến dạng theo góc nhìn phối cảnh 2D. Các vật thưa điểm cần quy tắc gán nhãn riêng để tránh đưa vào tập huấn luyện những hộp suy đoán thiếu chính xác, giúp mô hình học được ranh giới tin cậy giữa tín hiệu thật và nhiễu.
- **(b) Cách dùng AI:**
  - Công cụ: Trợ lý AI hội thoại.
  - Câu hỏi chính: Cách căn hướng đầu xe với `Cuboid orientation`, kiểm tra và xử lý lỗi kết nối CVAT trên WSL/Docker, cách chụp ảnh minh họa Bước 2 và tổng hợp bảng số liệu từ `bangA.csv` và `tukiem.csv`.
  - Cách dùng kết quả: Dùng để hướng dẫn thao tác kỹ thuật và đối chiếu công thức kiểm tra, sau đó tự thực hiện gán nhãn trên CVAT, kiểm tra lại từng cuboid với ảnh camera và chạy lệnh `sync` để xác nhận số liệu thực tế.
- **(c) Điều còn chưa chắc:** Một số xe ở khoảng cách trên `30 m` (như trường hợp xe bị xoay ngang hoặc dư bề rộng do điểm thưa) vẫn cần kiểm tra kỹ hơn trên nhiều góc nhìn `Top`/`Front` và ảnh camera để đảm bảo không bị nhầm chiều dài với chiều rộng.
