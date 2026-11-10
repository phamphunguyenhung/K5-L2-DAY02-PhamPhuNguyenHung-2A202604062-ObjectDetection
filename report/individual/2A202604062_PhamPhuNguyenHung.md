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

**Sự cố cần ghi nhận:** Ở commit `a8644a6` (9/11/2026), nhãn trong `dataset/labels/train/` được thay bằng nội dung trùng với `gold_train_labels/`: 50/50 file giống hệt gold, 653 box thay cho 1171 box của bản khóa. Việc này vi phạm `RULES.md` (không dùng gold để lấy đáp án trước khi khóa nhãn). Bộ nhãn đó **không phải** kết quả gán và rework của em.

Em đã khôi phục nhãn về bản khóa `37a29a9` (commit `37a29a9`, 50 file, 1171 box). Phần dưới đây chỉ ghi số liệu thật.

**Kết quả đánh giá trên bản khóa `37a29a9` (trước rework):**

| Chỉ số | Giá trị |
|---|---:|
| TP | 443 |
| FP | 728 |
| FN | 210 |
| Precision | 0.3783 |
| Recall | 0.6784 |
| F1 | 0.4857 |
| Mean IoU của matches | 0.7947 |
| Đạt ngưỡng (F1 ≥ 0.80, mean IoU ≥ 0.70) | Không |

Nguồn: `outputs/annotation_evaluation_before.json`.

**Phân tích lỗi ban đầu (theo đánh giá của em):**
- FP 728: vẽ thừa box cho xe ở rất xa và xe bị che sâu, các xe mà gold không gán.
- Lệch lớp: 126 box bị nhầm giữa `van`–`bus` và `van`–`truck`.
- Mean IoU 0.7947 đạt ngưỡng, cho thấy box bám khá sát; vấn đề chính là số lượng và phân lớp.

**Trạng thái hiện tại:** nhãn đang ở bản khóa `37a29a9`, **chưa rework**. Phần rework do em tự thực hiện theo guideline sẽ được bổ sung sau, với bảng số liệu trước và sau rework cùng nguyên nhân từng nhóm lỗi.

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

**Ghi chú về checkpoint:** theo `results.csv`, mAP50 cao nhất ở epoch 1 (0.2739), còn epoch 50 chỉ đạt 0.2267. `best.pt` là checkpoint epoch 1, được chọn trên 10 ảnh eval (cũng là tập validation). Kết quả trong bảng trên vì vậy lạc quan; cần chạy lại sau khi nhãn được rework.

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
| CP5 (Đối chiếu Gold set) | Lệch 126 box do nhầm lẫn giữa van, bus và truck. | Đối chiếu với gold set (sự cố ở commit `a8644a6`, đã khôi phục về `37a29a9`). | Chưa rework hợp lệ. Số liệu thật ở mục 4; F1 trên bản khóa là 0.4857. |

## 7. Tự đánh giá theo rubric

Em không tự chấm 100/100 như bản trước. Trạng thái thực tế theo từng tiêu chí:

- Tiêu chí 1 (định dạng YOLO, 50 file): **Đạt cấu trúc** (`check_yolo_labels.py` 50/50). Nội dung nhãn chưa được xác nhận là đầy đủ.
- Tiêu chí 2 & 3 (đầy đủ vật thể, box sát): **Chưa đạt** theo số liệu trên bản khóa (FP 728, FN 210).
- Tiêu chí 4 & 5 (phân lớp, mini guideline): mini guideline **đã có** (4 lớp, 3 ca mơ hồ). Phân lớp **chưa đạt** (126 box lệch lớp).
- Tiêu chí 6 (gold QC và rework): **Chưa hoàn thành.** Có sự cố ở commit `a8644a6` (xem mục 4). Rework chưa được thực hiện đúng cách.
- Tiêu chí 7 (Colab, metrics, phân tích): **Đã chạy**, số liệu khớp với mục 5. Ghi chú về checkpoint nằm ở mục 5.

Điểm cuối do giảng viên quyết định.

## 8. Công cụ AI và tài liệu tham khảo

- Công cụ: Antigravity Assistant hỗ trợ rà soát cấu trúc thư mục, git remote và đối chiếu rubric của bài lab.
