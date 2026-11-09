# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe con, sedan, hatchback, SUV, CUV, xe bán tải (pickup), taxi. | Phương tiện cá nhân hoặc chở khách cỡ nhỏ (dưới 9 chỗ), gầm thấp hoặc gầm cao vừa phải, xe bán tải cabin đơn/đôi. |
| `truck` | Xe tải chở hàng, xe tải thùng kín/hở, xe bồn, xe đầu kéo container. | Thùng hàng lớn tách biệt hoặc gắn liền chuyên chở hàng hóa hạng trung đến nặng. Không tính xe bán tải nhỏ (thuộc `car`). |
| `bus` | Xe buýt công cộng, xe khách lớn, xe trung chuyển nhiều chỗ ngồi. | Thân xe dài, cửa kính lớn, nhiều hàng ghế chở khách (> 16 chỗ), hình hộp chữ nhật lớn. |
| `van` | Xe van chở hàng hoặc chở khách mini (minivan), xe thùng kín một khoang (MPV/Van). | Thân xe dạng hộp một khối, thường có cửa trượt bên hông, nhỏ hơn `bus` nhưng trần cao hơn `car` thông thường (ví dụ: Ford Transit, Toyota Hiace, xe giao hàng mini). |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: Đường viền bounding box phải ôm sát các mép ngoài cùng nhìn thấy được của thân xe, bánh xe; không mở rộng lấy mặt đường, vỉa hè, bầu trời hoặc khoảng trống giữa các xe.
- Vật thể bị che khuất: Vẽ bounding box bao quanh toàn bộ phần nhìn thấy được của xe bị che; không phỏng đoán hay kéo rộng box bao phủ phần bị vật thể khác che lấp.
- Vật thể bị cắt bởi rìa ảnh: Vẽ box chạm đúng mép/rìa ảnh tại điểm cắt; không cố ước lượng hoặc vẽ vượt ra ngoài tọa độ ảnh ([0, 1]).
- Vật thể rất nhỏ/mơ hồ: Chỉ gán nhãn nếu có đủ đặc điểm nhận dạng rõ ràng thuộc một trong bốn lớp. Nếu xe ở quá xa, chỉ là vệt mờ dưới vài pixel hoặc trong bãi đỗ xe nền không rõ hình dáng, không tự suy đoán gán nhãn để tránh gây nhiễu cho mô hình.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_014` | Xe buýt lớn di chuyển ở làn giữa và xe con bị cắt rìa phải ảnh (`x ~ 0.97`). | Gán `bus` (ID 2) cho xe buýt; gán `car` (ID 0) cho xe con với cạnh phải chạm mép ảnh (`x_max = 1.0`). | Đúng định nghĩa lớp `bus` chở khách; xe bị cắt bởi rìa ảnh vẽ box ôm phần nhìn thấy và chạm rìa ảnh, không đoán phần ngoài khung hình. |
| `drive_002` | Phương tiện dạng van/minivan chở hàng màu trắng ở làn trái (`x ~ 0.36, y ~ 0.42`), dễ nhầm với xe buýt nhỏ hoặc xe con gầm cao. | Gán `van` (ID 3). | Phương tiện có dáng một khoang (monobox), trần cao, dạng minivan thương mại phục vụ chở hàng/chở người quy mô nhỏ, thuộc định nghĩa `van`, không đủ kích thước thành `bus` và không phải `car`. |
| `drive_005` | Xe tải chở hàng (`x ~ 0.58`) đi cạnh xe buýt (`x ~ 0.66`), một phần thân bị che khuất bởi góc nhìn camera. | Gán 2 bounding box riêng biệt: một box cho `truck` (ID 1) và một box cho `bus` (ID 2) bám sát phần nhìn thấy của từng xe. | Quy tắc mỗi xe một box riêng biệt, ôm sát phần nhìn thấy được; không gộp hai xe dù chúng đi sát nhau hoặc che khuất nhau một phần. |
