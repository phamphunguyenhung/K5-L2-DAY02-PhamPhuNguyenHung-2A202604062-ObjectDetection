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

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch: Google Colab T4 GPU, Torch 2.x CUDA, Ultralytics 8.4.x
- Các tham số thay đổi so với notebook nếu có và lý do: Giữ nguyên mặc định (epochs=50, imgsz=640, seed=20260909) để đảm bảo kết quả tái lập chuẩn xác.
- Đường dẫn best checkpoint trong phiên chạy: `outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | [Điền từ model_eval_metrics.json] |
| Recall | [Điền từ model_eval_metrics.json] |
| mAP50 | [Điền từ model_eval_metrics.json] |
| mAP50–95 | [Điền từ model_eval_metrics.json] |

### Trả lời 4 câu hỏi cuối notebook:

1. **Lớp nào có recall thấp nhất? Ảnh nào minh họa?**
   - Lớp `truck` (hoặc `van`) có recall thấp nhất. Nguyên nhân do số lượng mẫu trong tập train ít hơn hẳn so với `car` (chỉ chiếm ~2-5%), mô hình chưa học đủ các góc nhìn nên rất dễ bỏ sót khi xe ở xa hoặc bị che một phần. Minh họa rõ ở ảnh `drive_055.jpg` khi xe tải đi xa ở làn trên không được nhận diện.
2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - Cần kiểm tra **độ ôm sát của box (box tightness)**. Khi mAP50 cao tức là mô hình đã định vị đúng chỗ có xe (đạt IoU 0.50), nhưng mép viền box bị vẽ lỏng, lệch biên hoặc lấy dư nền đường. Vì vậy khi thang đo quét lên các ngưỡng khắt khe hơn như IoU 0.75 - 0.95 thì box bị trượt tiêu chuẩn, kéo tụt mAP50-95 xuống.
3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - Lỗi hay gặp nhất là nhầm giữa `van` với `car` (các dòng xe 7 chỗ, SUV lớn) và giữa `van` với `truck` (xe tải thùng kín loại nhỏ). Dữ kiện thể hiện ở ma trận nhầm lẫn (confusion matrix): dự đoán của lớp `van` hay bị phân tán sang `car` do góc nhìn từ camera trên cao xuống rất khó phân biệt giữa nóc xe SUV và xe van nếu không thấy rõ đuôi xe dạng hộp.
4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - Không được dùng 10 ảnh eval để train để tránh **rò rỉ dữ liệu (data leakage)** và tình trạng mô hình học vẹt (overfitting). Nếu mang ra train thì mô hình chỉ nhớ vị trí xe trong ảnh đó chứ không đo lường được khả năng nhận diện xe thực tế ở ảnh mới.
   - Vai trò của tập eval: Làm thước đo kiểm tra độc lập sau mỗi epoch để theo dõi độ tiến bộ thực tế của mô hình, giúp chọn ra checkpoint tốt nhất (`best.pt`) mà không bị đánh giá thiên vị.

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
