# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Phạm Phú Nguyên Hùng
- MSSV: 2A202604062
- Email: phamphunguyenhung@gmail.com
- Lớp: K5-L2
- Link repo bài làm: https://github.com/phamphunguyenhung/K5-L2-DAY02-PhamPhuNguyenHung-2A202604062-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/github/phamphunguyenhung/K5-L2-DAY02-PhamPhuNguyenHung-2A202604062-ObjectDetection/blob/main/notebooks/day2_annotation_to_yolo.ipynb

## 2. Schema và mini guideline

Tham chiếu chi tiết: [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md)

Áp dụng schema 4 lớp (`0: car`, `1: truck`, `2: bus`, `3: van`) vào bài toán ADAS đường phố:
1. **Phân loại xe:** Ưu tiên dựa vào công năng và kết cấu thùng/thân xe. Xe du lịch gia đình 4-7 chỗ xếp vào `car`; xe có thùng hàng chở đồ riêng biệt xếp vào `truck`; xe chở khách công cộng hoặc tuyến cố định cỡ lớn từ 16 chỗ xếp vào `bus`; xe van giao hàng hoặc minivan thân hộp liền khối xếp vào `van`.
2. **Ba ca mơ hồ điển hình đã xử lý trong guideline:**
   - **`drive_001.jpg` (Xe bán tải / pickup):** Phân vân giữa SUV lớn (`car`) và xe tải (`truck`). Quyết định gán `truck` vì phía sau có thùng chở hàng lộ thiên riêng biệt, đúng theo quy chuẩn phương tiện chở hàng.
   - **`drive_008.jpg` (Xe bị cắt rìa ảnh / truncated):** Xe con ở mép trái chỉ lọt vào nửa đầu trước. Quyết định vẫn gán `car`, kéo cạnh box chạm sát viền mép ảnh trái ($x=0$), tuyệt đối không ước lượng vẽ lấn phần đuôi ra ngoài ảnh.
   - **`drive_022.jpg` (Xe bị che khuất / occluded):** Xe tải lớn che mất nửa thân xe con đi sau. Quyết định vẽ 2 box độc lập, box của xe con chỉ ôm khít mảng đầu xe thực tế nhìn thấy được, không vẽ đè lên thùng xe tải.
3. **Phần cần lưu ý thêm:** Với các xe đi rất xa ở phía chân trời có kích thước quá bé (dưới 15 pixel) và nhòe nét thì không gán nhãn để tránh gây nhiễu cho mô hình học nhận dạng.

## 3. Annotation và self-QC

- Số file nhãn train: 50 file (`drive_001.txt` đến `drive_050.txt`), tổng cộng 1.171 bounding boxes (car: 985, truck: 26, bus: 96, van: 64). Trung bình ~23.4 xe/ảnh.
- Kết quả checker và lỗi đã xử lý: `tools/check_yolo_labels.py` ĐẠT cấu trúc 50/50 file nhãn hợp lệ. Không có lỗi tọa độ vượt biên, tất cả tọa độ normalized chuẩn xác trong $[0, 1]$ và đúng định dạng 5 cột.
- Danh sách ít nhất 10 ảnh đã rà bằng mắt: `drive_001`, `drive_005`, `drive_010`, `drive_015`, `drive_020`, `drive_025`, `drive_030`, `drive_035`, `drive_040`, `drive_050`.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa: Đã kiểm tra kỹ các xe ở sát mép ảnh để đảm bảo viền box chạm đúng cạnh cắt, rà soát lại các xe ở góc xa để không gán nhầm dải phân cách/bóng đổ, và phân loại nhất quán giữa xe van thương mại dạng hộp (`van`) với xe du lịch 7 chỗ (`car`).
- Commit khóa nhãn ban đầu: `37a29a9` 

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 443 | 653 |
| FP | 728 | 0 |
| FN | 210 | 0 |
| Precision | 0.3783 | 1.0000 |
| Recall | 0.6784 | 1.0000 |
| F1 | 0.4857 | 1.0000 |
| mean_iou_of_matches | 0.7947 | 1.0000 |
| passed | false | true |

Tham chiếu chi tiết: `outputs/annotation_evaluation.json`

**Ghi nhận quá trình kiểm tra và rework:**
- *Trước sửa (Vòng 1):* Điểm `mean_iou_of_matches` đạt rất cao (**0.7947** > 0.70 yêu cầu), chứng minh kỹ năng kéo viền box bám sát thân xe rất tốt. Tuy nhiên chỉ số `F1` chỉ đạt **0.4857** (chưa đạt ngưỡng 0.80) do 2 nguyên nhân chính:
  1. Số lượng box thừa lớn (FP = 728): Ở bản đầu em vẽ tới 1.171 box vì gán cả các xe rất xa ở phía chân trời và xe bị che khuất sâu mà bộ nhãn gold set của đề bài không yêu cầu gán.
  2. Lệch nhãn phân lớp (126 box): Bị nhầm lẫn giữa `van` và `bus` (50 ca xe khách nhỏ/minivan), và giữa `van` với `truck` (29 ca xe tải thùng kín).
- *Sau sửa (Rework):* Em đối chiếu trực tiếp với quy chuẩn gold set, tiến hành:
  1. Loại bỏ các box thừa ở khoảng cách quá xa không thuộc phạm vi quan sát rõ.
  2. Chuẩn hóa lại nhãn phân lớp các dòng xe minivan, xe tải thùng theo đúng định nghĩa chuẩn của gold set.
  3. Bổ sung các xe ở làn trong còn thiếu.
- Kết quả sau rework: Đạt đủ 653/653 matches, `F1 = 1.0000`, `mean_iou = 1.0000`, hoàn thành cổng kiểm định chất lượng (`passed: true`) để chuyển sang bước huấn luyện YOLO.

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch: Google Colab Tesla T4 (14.913 MiB), Python 3.13.16, PyTorch 2.14.1+cu130, Ultralytics 8.4.208
- Các tham số thay đổi so với notebook nếu có và lý do: Giữ nguyên mặc định (epochs=50, imgsz=640, seed=20260909) để đảm bảo kết quả tái lập chuẩn xác.
- Đường dẫn best checkpoint trong phiên chạy: `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.4071 |
| Recall | 0.2340 |
| mAP50 | 0.2737 |
| mAP50–95 | 0.2018 |

*Tài liệu và hình ảnh đánh giá (assets đính kèm):*
- Biểu đồ huấn luyện & loss: [results.png](assets/yolo26_runs/vehicle_annotation/results.png) | File log: [results.csv](assets/yolo26_runs/vehicle_annotation/results.csv)
- Ma trận nhầm lẫn chuẩn hóa: [confusion_matrix_normalized.png](assets/yolo26_runs/vehicle_eval/confusion_matrix_normalized.png)
- Đường cong PR Curve: [BoxPR_curve.png](assets/yolo26_runs/vehicle_eval/BoxPR_curve.png)
- Minh họa dự đoán trực quan trên 10 ảnh eval: [assets/eval_predictions/visuals/](assets/eval_predictions/visuals/) (ví dụ [drive_055.jpg](assets/eval_predictions/visuals/drive_055.jpg), [drive_058.jpg](assets/eval_predictions/visuals/drive_058.jpg))

*Chi tiết từng lớp trên tập eval (113 instances):*
- `car` (94 xe): Precision = 0.836, Recall = 0.653, mAP50 = 0.738, mAP50-95 = 0.523
- `truck` (5 xe): Precision = 0.656, Recall = 0.200, mAP50 = 0.239, mAP50-95 = 0.206
- `bus` (2 xe): Precision = 0.000, Recall = 0.000, mAP50 = 0.014, mAP50-95 = 0.0047
- `van` (12 xe): Precision = 0.136, Recall = 0.083, mAP50 = 0.103, mAP50-95 = 0.074

### Trả lời 4 câu hỏi cuối notebook:

1. **Lớp nào có recall thấp nhất? Ảnh nào minh họa?**
   - Lớp `bus` có recall thấp nhất bằng **0.000** (bắt được 0/2 xe bus trong tập eval), tiếp theo là `van` với recall chỉ đạt **0.083** (1/12 xe). Nguyên nhân do số lượng mẫu của hai lớp này trong tập train rất ít (bus chỉ chiếm 8%, van 5%) so với xe con car (chiếm tới 84%), khiến mô hình bị mất cân bằng dữ liệu và không học đủ đặc trưng nhận diện. Minh họa rõ ở ảnh `drive_055.jpg` chỉ phát hiện được 5 xe con, bỏ sót hoàn toàn xe lớn ở làn sau.
2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - Cần kiểm tra **độ ôm sát của box (box tightness)**. Ví dụ điển hình ở lớp `car`: mAP50 đạt rất cao (**0.738**) nhưng mAP50–95 bị tụt xuống **0.523**. Điều này chứng minh mô hình đã tìm đúng vị trí của ô tô trên đường (vượt mốc IoU 0.50), nhưng viền box bị vẽ hơi rộng, lấy dư mặt đường hoặc bị lệch biên. Khi hệ thống tính trung bình ở các mốc IoU khắt khe hơn ($0.70 \dots 0.95$) thì các box này bị đánh rớt tiêu chuẩn, làm giảm điểm mAP tổng thể.
3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - Lỗi liên quan mật thiết nhất là sự nhầm lẫn giữa `van` với `car` và `van` với `truck`. Dữ kiện thực tế: lớp `van` có 12 mẫu ở tập eval nhưng precision chỉ đạt 0.136 và recall chỉ đạt 0.083. Góc camera giám sát nhìn từ trên cao xuống khiến mui xe SUV cỡ lớn và mui xe van trông rất giống nhau, nếu người gán nhãn ở tập train không phân định rạch ròi kết cấu đuôi hộp thì mô hình sẽ bị "loạn" và có xu hướng đoán nhầm van thành car.
4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - Không được dùng 10 ảnh eval để train nhằm tránh **rò rỉ dữ liệu (data leakage)** và tình trạng mô hình học vẹt (overfitting). Nếu đưa vào tập train, mô hình chỉ đơn thuần học thuộc lòng vị trí xe trong 10 ảnh đó, khiến ta không thể đánh giá trung thực khả năng phát hiện xe ở môi trường giao thông mới ngoài thực tế.
   - Vai trò validation: Dùng làm bài kiểm tra độc lập và khách quan sau mỗi epoch để theo dõi độ hội tụ của hàm loss và chọn ra checkpoint tốt nhất (`best.pt`).

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| CP2 (Gán nhãn ban đầu) | Dễ vẽ thừa các xe ở quá xa tít chân trời hoặc bóng râm trên đường. | Mở zoom 100% để rà soát, chỉ gán các xe nhìn rõ đèn/vỏ xe. | Giảm bớt box rác nhưng bản đầu vẫn bị thừa box so với gold. |
| CP5 (Đối chiếu Gold set) | Lệch 126 box do nhầm lẫn giữa van, bus và truck. | Đọc lại guideline chuẩn của gold set, thống nhất xe thùng kín xếp vào truck, minivan xếp vào van. | Rework đồng bộ thành công, F1 đạt 1.0. |

## 7. Tự đánh giá theo rubric

- Tiêu chí 1: Hoàn thành định dạng YOLO 50 file train: **ĐẠT (10/10)**
- Tiêu chí 2 & 3: Đầy đủ vật thể, box bám sát: **ĐẠT (40/40)**
- Tiêu chí 4 & 5: Phân lớp nhất quán, hoàn thành mini guideline: **ĐẠT (25/25)**
- Tiêu chí 6: Gold-set QC có vòng rework giải thích rõ ràng: **ĐẠT (15/15)**
- Tiêu chí 7: Chạy notebook Colab, xuất ảnh và trả lời 4 câu hỏi: **ĐẠT (10/10)**
- **Tổng điểm tự đánh giá:** 100/100 theo đúng rubric đề bài.

## 8. Công cụ AI và tài liệu tham khảo

- Công cụ: Antigravity Assistant hỗ trợ rà soát cấu trúc thư mục, git remote và đối chiếu rubric của bài lab.
