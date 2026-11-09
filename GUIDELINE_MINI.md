# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Ô tô con chở người thông dụng (sedan, hatchback, SUV, crossover, xe 4-7 chỗ gia đình). | Thân xe thấp hoặc gầm cao vừa phải, khoang hành khách liền khối, kính bao quanh. |
| `truck` | Xe tải chuyên chở hàng hóa các loại (thùng hở, thùng kín, xe ben, xe bồn, đầu kéo container). | Có cabin lái tách rời hoặc khoang chở hàng riêng biệt phía sau. Xe bán tải (pickup) có thùng hàng cũng tính là truck. |
| `bus` | Xe buýt công cộng, xe khách vận tải hành khách liên tỉnh cỡ lớn. | Xe dài, thân to khối hộp đồ sộ, nhiều hàng kính cửa sổ hai bên, sức chứa từ 16 chỗ trở lên. |
| `van` | Xe tải nhẹ thân kín, minivan, xe van thương mại giao hàng hoặc chở khách cỡ trung (Ford Transit, Starex...). | Thân xe liền khối dạng hộp chữ nhật đứng, đuôi bằng phẳng; to hơn xe con car nhưng ngắn hơn xe khách bus. |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: Kéo viền box sát khít phần vỏ xe và bánh xe chạm mặt đường. Không lấy dư bóng đổ trên mặt đường, không lấy lấn sang dải phân cách hay xe đi bên cạnh.
- Vật thể bị che khuất: Chỉ vẽ hộp quanh phần thân xe thực tế nhìn thấy bằng mắt. Phần bị xe khác hay chướng ngại vật che lấp thì bỏ qua, tuyệt đối không tự đoán vẽ bù.
- Vật thể bị cắt bởi rìa ảnh: Cạnh box kéo chạm khít mép ảnh (giá trị 0 hoặc 1). Phần thân xe nằm ngoài khung hình không được vẽ lấn ra ngoài.
- Vật thể rất nhỏ/mơ hồ: Xe ở quá xa tít chân trời (ước lượng dưới 15 pixel) mờ nhòe không rõ nét hoặc không phân biệt nổi đèn/bánh xe thì bỏ qua không gán, tránh tạo nhãn nhiễu cho mô hình.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_001.jpg` | Một chiếc bán tải (pickup) đi ở làn giữa, dáng xe nhìn qua khá giống SUV lớn nhưng có thùng hàng lộ thiên phía sau. | Gán nhãn `truck` | Theo schema, xe có kết cấu thùng chở hàng riêng biệt phía sau được ưu tiên xếp vào nhóm xe tải (truck) thay vì ô tô con (car). |
| `drive_008.jpg` | Xe con ở sát lề trái đang chạy vào khung hình, chỉ nhìn thấy nửa đầu xe và bánh trước, nửa sau bị mép ảnh cắt mất. | Gán nhãn `car` | Vẫn nhận dạng rõ là xe con, áp dụng đúng luật cắt khung (truncated): vẽ box ôm sát nửa đầu xe nhìn thấy, cạnh trái chạm sát viền ảnh x=0, không tự vẽ đoán phần đuôi. |
| `drive_022.jpg` | Một xe tải lớn che khuất xe con đi ngay sau, xe con chỉ lộ phần nắp capo và kính chắn gió trước. | Vẽ 2 box tách biệt (`truck` và `car`) | Xe nào tính xe đó; box xe con chỉ ôm đúng mảng thân xe còn nhìn thấy được, không vẽ đè trùm lên thùng xe tải đi trước. |
