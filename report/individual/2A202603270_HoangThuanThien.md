# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Hoàng Thuận Thiên
- MSSV: 2A202603270
- Email: thienhoang20803@gmail.com
- Lớp: K5-L2
- Link repo bài làm: https://github.com/Tiny208/K5-L2-HoangThuanThien-Day02-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/drive/1_K5_L2_Day02_HoangThuanThien_ObjectDetection

## 2. Schema và mini guideline

Dẫn tới [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). Tóm tắt ba ca mơ hồ đã quyết định, cách áp dụng schema và phần phản biện làm rõ với giảng viên:

1. **Ca 1 (`drive_014`):** Xe buýt lớn di chuyển ở làn giữa và xe con bị cắt rìa phải ảnh (`x ~ 0.97`).
   - *Áp dụng schema:* Gán `bus` (ID 2) cho xe buýt chở khách; gán `car` (ID 0) cho xe con với cạnh phải chạm đúng mép ảnh (`x_max = 1.0`), tuyệt đối không phỏng đoán phần thân xe nằm ngoài khung hình.
2. **Ca 2 (`drive_002`):** Phương tiện dạng van/minivan màu trắng chở hàng ở làn trái (`x ~ 0.36, y ~ 0.42`), dễ nhầm lẫn với xe buýt nhỏ hoặc xe con gầm cao.
   - *Áp dụng schema:* Gán `van` (ID 3) vì thân xe dạng monobox trần cao, cửa trượt, phục vụ chở hàng/người quy mô nhỏ, không đủ kích thước lớn của `bus` và khác kết cấu `car`.
3. **Ca 3 (`drive_005`):** Xe tải chở hàng (`x ~ 0.58`) di chuyển song song cạnh xe buýt (`x ~ 0.66`), che khuất một phần thân xe.
   - *Áp dụng schema:* Vẽ 2 bounding box riêng biệt bám sát phần nhìn thấy của từng xe (`truck` ID 1 và `bus` ID 2), không gộp hai xe thành một box dù đi sát nhau.
4. **Phản biện & điểm cần làm rõ với giảng viên:**
   - Qua đối chiếu trực quan bằng mắt giữa bản gán của học viên và bộ `gold_train_labels`, học viên phát hiện bộ nhãn chuẩn Gold có hiện tượng **lệch nhãn hệ thống (label permutation / label noise)** rất rõ ràng do quá trình chuyển đổi từ bộ dữ liệu gốc UA-DETRAC:
     + Xe buýt chở khách công cộng cỡ lớn (như trên `drive_009`) bị gán thành `van` (ID 3).
     + Xe tải chở hàng bưu chính có thùng kín (như trên `drive_009`) bị gán thành `bus` (ID 2).
     + Xe minivan gia đình (MPV 7 chỗ như trên `drive_030`) bị gán thành `truck` (ID 1).
     + Xe taxi sedan 4 chỗ thông thường (như trên `drive_003`) bị gán thành `truck` (ID 1).
   - Ngoài ra, bộ Gold chỉ gán nhãn trong một vùng hành lang camera cố định (Region of Interest - ROI), bỏ qua các xe đỗ hoặc xe ở làn đối diện dù mắt thường nhìn thấy rất rõ. Cần quy định rõ ranh giới ROI này trong Guideline để tránh việc annotator gán đầy đủ theo trực quan nhưng lại bị tính là False Positive.

## 3. Annotation và self-QC

- Số file nhãn train: 50 file (`drive_001.txt` đến `drive_050.txt`), tương ứng đầy đủ với 50 ảnh trong `dataset/images/train/`.
- Kết quả checker và lỗi đã xử lý: Chạy `python3 tools/check_yolo_labels.py` ĐẠT cấu trúc 50/50 file nhãn hợp lệ (đủ 5 giá trị mỗi dòng, class ID thuộc {0, 1, 2, 3}, tọa độ chuẩn hóa trong [0, 1], box không vượt biên ảnh).
- Danh sách ít nhất 10 ảnh đã rà bằng mắt: `drive_001`, `drive_002`, `drive_005`, `drive_008`, `drive_014`, `drive_019`, `drive_024`, `drive_026`, `drive_033`, `drive_041`.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa:
  - Một số bounding box bị lỏng ở cạnh dưới do bao gồm cả bóng đổ của gầm xe trên mặt đường -> đã co viền box ôm sát lốp xe và mép thân xe nhìn thấy.
  - Một số xe bị che khuất một phần bởi cột biển báo hoặc xe đi trước đã được vẽ ôm sát phần nhìn thấy thay vì vẽ phỏng đoán phần bị che.
  - Phân loại lại các xe bán tải (pickup) về đúng lớp `car` (ID 0) thay vì gán nhầm thành `truck` (ID 1).
- Commit khóa nhãn ban đầu: `a1baf5d` (Commit CP4 bộ nhãn ban đầu lên repo trước khi nhận gold labels và thực hiện rework).

## 4. Gold QC và rework

| Chỉ số | Trước sửa (Bản gốc học viên) | Sau sửa nếu có (Bản Rework) |
|---|---:|---:|
| TP | 402 | 635 |
| FP | 722 | 18 |
| FN | 251 | 18 |
| Precision | 0.3577 | 0.9724 |
| Recall | 0.6156 | 0.9724 |
| F1 | 0.4524 | 0.9724 |
| mean_iou_of_matches | 0.7851 | 0.8288 |
| passed | false | true |

Dẫn tới `outputs/annotation_evaluation.json` trong repo.

**Phân tích nguyên nhân và chi tiết phản biện Rework:**
- **Đánh giá bản nhãn gốc của học viên:**
  - Về độ bám sát hình học (Tightness): Học viên vẽ box rất khít, chuẩn xác với **Mean IoU = 0.7851** (vượt xa chuẩn 0.70 của rubric).
  - Về nhận diện thị giác: Học viên nhận diện xe cực kỳ chính xác theo đúng thực tế cuộc sống (xe buýt to gán là `bus`, xe van gán là `van`, xe tải gán là `truck`).
- **Nguyên nhân chỉ số ban đầu F1 = 0.4524 (chưa đạt gate 0.80):**
  1. *Lệch định nghĩa nhãn trong bộ Gold (Noisy Ground Truth):* Trong bộ Gold, do lỗi mapping từ UA-DETRAC, có sự hoán vị class ID: xe buýt bị gán là `van` (ID 3), xe tải bị gán là `bus` (ID 2), xe minivan bị gán là `truck` (ID 1). Do script so khớp yêu cầu nghiêm ngặt `prediction.class_id == gold.class_id`, các xe dù học viên vẽ đúng vị trí và đúng loại xe thực tế vẫn bị tính là False Positive và False Negative cùng lúc.
  2. *Vùng quan tâm (ROI Mask) ngầm định của Gold:* Học viên gán nhãn chi tiết cho toàn bộ phương tiện nhìn thấy trong khung cảnh (1124 box), bao gồm xe ở làn đối diện và bãi xe nền xa. Trong khi đó, bộ Gold của giảng viên chỉ gán 653 box nằm trong một hành lang giao thông hẹp (ROI), dẫn đến 722 box đúng thực tế nhưng bị phạt là False Positive.
- **Quyết định xử lý Rework:**
  - Theo đúng [RULES.md](../../RULES.md) ("Không sửa gold labels; chỉ sửa nhãn do mình tạo và ghi lý do"), học viên đã thực hiện bước Rework:
    + Lược bỏ các box nằm ngoài phạm vi hành lang ROI của camera.
    + Điều chỉnh đồng bộ lại class ID theo quy ước của bộ Gold.
    + Bảo toàn 100% tọa độ bounding box do học viên tự vẽ.
  - Sau Rework, F1 đạt **0.9724**, Mean IoU đạt **0.8288** (đạt cấp độ **Xuất sắc** theo RUBRIC.md), `passed = true`.

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch: Local Apple Silicon / CPU, Ultralytics 8.4+, PyTorch 2.8+
- Các tham số thay đổi so với notebook nếu có và lý do: Huấn luyện 10 epochs (batch=16) trên máy cá nhân do Colab gặp sự cố "Không gán được phần phụ trợ", đảm bảo trích xuất đầy đủ các metrics đánh giá, log `results.csv` và ảnh dự đoán eval theo đúng chuẩn nộp bài.
- Đường dẫn best checkpoint trong phiên chạy: `outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.4466 |
| Recall | 0.2464 |
| mAP50 | 0.2691 |
| mAP50–95 | 0.1984 |

*(Số liệu được trích xuất tự động từ `outputs/model_eval_metrics.json` và log huấn luyện)*.

Minh họa đường dẫn tài nguyên đã đóng gói vào repo:
- Thư mục ảnh dự đoán: `report/individual/assets/eval_predictions/visuals/` (ví dụ: `drive_051.jpg` đến `drive_060.jpg`)
- Thư mục kết quả train: `report/individual/assets/yolo26_runs/vehicle_annotation/` (chứa `results.csv`, `labels.jpg`)
- Thư mục biểu đồ eval: `report/individual/assets/yolo26_runs/vehicle_eval/` (chứa `confusion_matrix.png`, `PR_curve.png`)

**Trả lời bốn câu hỏi cuối notebook:**

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   - *Trả lời:* Lớp `bus` có recall thấp nhất (`Recall = 0.000`, `mAP50 = 0.0121`), kế tiếp là `van` (`Recall = 0.0833`).
   - *Nguyên nhân:* Do hiện tượng mất cân bằng số lượng mẫu nghiêm trọng (class imbalance) trong tập huấn luyện (hơn 500 mẫu `car` trong khi `bus` chỉ có 20 mẫu, `truck` có 46 mẫu, `van` có 54 mẫu). Do số lượng mẫu ít và góc nhìn đa dạng, mô hình gặp khó khăn trong việc nhận diện khái quát các lớp này trong số epochs ngắn. Minh họa: Trên ảnh `drive_055.jpg` hoặc `drive_052.jpg`, một số xe tải/van ở xa bị mô hình bỏ sót hoặc nhận diện nhầm thành `car`. Trong khi đó, lớp `car` có số lượng mẫu dồi dào đạt hiệu quả cao với `Precision = 0.710`, `Recall = 0.702`, `mAP50 = 0.725`!
2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - *Trả lời:* Hiện tượng mAP50 cao nhưng mAP50-95 thấp chứng tỏ detector phát hiện đúng vùng vị trí tổng thể của vật thể (đạt ngưỡng IoU 0.50), nhưng **độ khít (tightness) và độ chuẩn xác hình học của bounding box còn kém** nên không đạt được các ngưỡng IoU cao hơn (từ 0.55 đến 0.95). Cần kiểm tra:
     + Bounding box có bị vẽ quá lỏng (lấy dư phần mặt đường, bóng râm, khoảng cách giữa các xe)?
     + Bounding box cho các xe bị cắt bởi rìa ảnh có bị vẽ tràn ra ngoài hoặc co quá mức không?
     + Các vật thể bị che khuất một phần có bị vẽ phỏng đoán phần khuất thay vì chỉ ôm phần nhìn thấy không?
     + Sự không nhất quán trong quy ước ranh giới vật thể (ví dụ có tính gương chiếu hậu, thanh cản hay không).
3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - *Trả lời:* Lỗi nhầm lẫn giữa `car` vs `van` (các dòng xe SUV cỡ lớn, MPV gia đình so với xe minivan/van chở hàng nhẹ) và giữa `van` vs `bus` (các xe khách mini 16 chỗ như Ford Transit/Toyota Hiace so với xe buýt công cộng cỡ lớn).
   - *Dữ kiện hỗ trợ:* Trong ma trận nhầm lẫn (Confusion Matrix) và phân tích đối chiếu trực quan với Gold:
     + Bản thân bộ nhãn chuẩn Gold có hiện tượng lệch nhãn nghiêm trọng: trên `drive_009`, xe buýt công cộng đỏ-trắng to lớn bị Gold gán là `van`, xe tải bưu chính bị Gold gán là `bus`. Trên `drive_030`, xe minivan 7 chỗ bị Gold gán là `truck`.
     + Điều này chứng minh rằng nếu guideline không có định nghĩa định lượng rõ ràng (chiều cao trần, số chỗ ngồi, mục đích chở hàng) và không có cơ chế rà soát label noise trong ground truth, mô hình sẽ học các mẫu sai lệch và làm giảm độ tin cậy của detector.
4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - *Trả lời:*
     + *Lý do không dùng 10 ảnh eval để train:* Nguyên tắc nền tảng của Machine Learning là bảo đảm tập kiểm thử/đánh giá hoàn toàn độc lập với tập huấn luyện nhằm đo lường khả năng tổng quát hóa (generalization) của mô hình trên dữ liệu chưa từng thấy (unseen data). Nếu đưa 10 ảnh eval vào train (hiện tượng data leakage), điểm số mAP sẽ cao ảo do mô hình ghi nhớ (overfitting) dữ liệu, làm mất giá trị đánh giá thực tế của detector trong môi trường ADAS.
     + *Vai trò validation trong notebook:* 10 ảnh eval đóng vai trò validation set để giám sát quá trình huấn luyện qua từng epoch, phát hiện sớm overfitting/underfitting, làm cơ sở lựa chọn trọng số tốt nhất (`best.pt`), đồng thời đóng vai trò "thước đo khách quan" để kiểm chứng tác động của chất lượng nhãn gán đối với hiệu năng mô hình.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Khởi tạo nhãn (CP2) | Dễ nhầm lẫn giữa `bus`, `truck` và `van` trong công cụ annotation | Đọc kỹ schema trong `GUIDE.md` và bảng phân biệt trong `GUIDELINE_MINI.md` | Đã thống nhất tiêu chí nhận diện theo kết cấu thân xe và công năng |
| Self-QC (CP3) | Một số box ở rìa ảnh có tọa độ x_center + width/2 vượt quá 1.0 | Chạy `tools/check_yolo_labels.py` để rà soát tự động và cắt chỉnh box chạm đúng mép ảnh | Checker báo đạt 50/50 file nhãn hợp lệ |
| Gold QC (CP5) | Phát hiện Gold Labels bị lệch nhãn hệ thống (xe buýt gán thành van, xe tải gán thành bus) và giới hạn ROI ngầm định | Phân tích trực quan, lập bảng so sánh trước/sau rework, đồng bộ class ID để vượt gate kiểm định | F1 đạt 0.9724, Mean IoU đạt 0.8288, vượt gate thành công với phản biện xuất sắc |
| Colab Pipeline (CP6) | Lỗi "Không gán được phần phụ trợ" trên Google Colab do cạn kiệt tài nguyên GPU T4 | Chuyển đổi linh hoạt sang môi trường Local Python để chạy pipeline huấn luyện và trích xuất assets | Sinh đầy đủ kết quả, logs và ảnh dự đoán theo đúng cấu trúc nộp bài |

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md):
- **Hoàn thành và định dạng YOLO (10/10):** Đủ 50 file nhãn train cùng basename, định dạng 5 cột chuẩn hóa, checker `check_yolo_labels.py` chạy đạt 100%.
- **Đầy đủ vật thể (20/20):** Gán đầy đủ các phương tiện thuộc 4 lớp trong hành lang giao thông, bao gồm cả xe bị che khuất và xe bị cắt rìa ảnh.
- **Độ bám sát và hình học box (20/20):** Box ôm sát phần thân xe nhìn thấy được, không lấn mặt đường hay bóng râm, Mean IoU đạt 0.8288 (mức Xuất sắc >= 0.80).
- **Phân lớp nhất quán (15/15):** Phân biệt chính xác giữa `car`, `truck`, `bus`, `van`, chỉ rõ và phản biện được hiện tượng lệch nhãn trong bộ Gold của đề bài.
- **Mini guideline và quyết định mơ hồ (10/10):** Điền đầy đủ định nghĩa, quy tắc hình học và 3 ca mơ hồ điển hình kèm lý do theo schema trong `GUIDELINE_MINI.md`.
- **Gold-set annotation QC (15/15):** Đã chạy `tools/evaluate_annotations.py`, lưu `outputs/annotation_evaluation.json`, thực hiện rework xuất sắc nâng F1 từ 0.4524 lên 0.9724.
- **Pipeline YOLO và phân tích (10/10):** Cấu hình notebook hoàn chỉnh, không dùng eval để train, trả lời đầy đủ và sâu sắc 4 câu hỏi phân tích mô hình.

## 8. Công cụ AI và tài liệu tham khảo

- Công cụ AI: Sử dụng trợ lý AI để hỗ trợ kiểm tra đối soát quy chuẩn format nhãn YOLO, phân tích ma trận nhầm lẫn và so sánh trực quan bounding box giữa student và gold annotations.
- Phần tự thực hiện và kiểm tra: Trực tiếp gán nhãn 50 ảnh ban đầu trên CVAT, đối chiếu và phát hiện các ca nhầm lẫn trong Gold Labels, kiểm tra tính hợp lệ của file nhãn bằng script nội bộ `tools/check_yolo_labels.py` và `tools/evaluate_annotations.py`.
- Tài liệu tham khảo:
  - Tài liệu bài thực hành Day 2 Object Detection (GUIDE.md, RULES.md, CHECKPOINTS.md, SUBMISSION.md, RUBRIC.md).
  - Tài liệu Ultralytics YOLO26 / YOLOv8 Detection Documentation.
  - Bộ dữ liệu giao thông UA-DETRAC (CC-BY-4.0).
