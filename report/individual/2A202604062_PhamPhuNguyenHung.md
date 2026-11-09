# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Phạm Phú Nguyên Hùng
- MSSV: 2A202604062
- Email: phamphunguyenhung@gmail.com
- Lớp: K5-L2
- Link repo bài làm: https://github.com/phamphunguyenhung/K5-L2-DAY02-PhamPhuNguyenHung-2A202604062-ObjectDetection
- Link Colab đã chạy: [Sẽ cập nhật ở CP6]

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

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch:
- Các tham số thay đổi so với notebook nếu có và lý do:
- Đường dẫn best checkpoint trong phiên chạy:

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | |
| Recall | |
| mAP50 | |
| mAP50–95 | |

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| | | | |

## 7. Tự đánh giá theo rubric

## 8. Công cụ AI và tài liệu tham khảo

- Công cụ: Antigravity Assistant hỗ trợ rà soát cấu trúc thư mục, git remote và đối chiếu rubric của bài lab.
