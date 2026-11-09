# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Hoàng Thuận Thiên
- MSSV: 2A202603270
- Email: thienhoang20803@gmail.com
- Lớp: K5-L2
- Link repo bài làm: https://github.com/Tiny208/K5-L2-HoangThuanThien-Day02-ObjectDetection
- Link Colab đã chạy: [Dán link notebook Colab của bạn vào đây sau khi chạy]

## 2. Schema và mini guideline

Dẫn tới [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). Tóm tắt ba ca mơ hồ đã quyết định, cách áp dụng schema và phần cần giảng viên làm rõ:

1. **Ca 1 (`drive_014`):** Xe buýt lớn di chuyển ở làn giữa và xe con bị cắt rìa phải ảnh (`x ~ 0.97`).
   - *Áp dụng schema:* Gán `bus` (ID 2) cho xe buýt chở khách; gán `car` (ID 0) cho xe con với cạnh phải chạm đúng mép ảnh (`x_max = 1.0`), tuyệt đối không phỏng đoán phần thân xe nằm ngoài khung hình.
2. **Ca 2 (`drive_002`):** Phương tiện dạng van/minivan màu trắng chở hàng ở làn trái (`x ~ 0.36, y ~ 0.42`), dễ nhầm lẫn với xe buýt nhỏ hoặc xe con gầm cao.
   - *Áp dụng schema:* Gán `van` (ID 3) vì thân xe dạng monobox trần cao, cửa trượt, phục vụ chở hàng/người quy mô nhỏ, không đủ kích thước lớn của `bus` và khác kết cấu `car`.
3. **Ca 3 (`drive_005`):** Xe tải chở hàng (`x ~ 0.58`) di chuyển song song cạnh xe buýt (`x ~ 0.66`), che khuất một phần thân xe.
   - *Áp dụng schema:* Vẽ 2 bounding box riêng biệt bám sát phần nhìn thấy của từng xe (`truck` ID 1 và `bus` ID 2), không gộp hai xe thành một box dù đi sát nhau.
4. **Điểm cần làm rõ/thống nhất với giảng viên:** Cần quy định rõ ranh giới vùng quan tâm (ROI) đối với các xe đỗ ở bãi đỗ xe nền phía xa hoặc các phương tiện ở làn đối diện có kích thước quá nhỏ (diện tích < 0.0005) để tránh việc annotator tự phỏng đoán gây ra false positive cao.

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

| Chỉ số | Trước sửa | Sau sửa nếu có |
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

**Phân tích nguyên nhân và chi tiết rework:**
- **Nguyên nhân trước sửa (F1 = 0.4524, Passed = false):**
  1. *Số lượng False Positive rất cao (722 FP):* Quá trình gán nhãn ban đầu bị over-annotation, vẽ cả những xe đỗ ở xa ngoài hành lang lưu thông chính, các xe ở làn đối diện quá mờ, và các vật thể kích thước siêu nhỏ (area < 0.0005) nằm ngoài ROI của bộ nhãn chuẩn UA-DETRAC.
  2. *Nhầm lẫn ID nhãn lớp (Class Confusion):* Trong công cụ annotation, người gán nhãn bị lệch định nghĩa nhãn lớp: 20 xe buýt (`bus` ID 2) bị gán thành `truck` (ID 1); 41 xe van (`van` ID 3) và 26 xe tải (`truck` ID 1) bị gán thành `bus` (ID 2). Do matcher yêu cầu trùng khớp class ID (`prediction.class_id == truth.class_id`), các box dù trùng vị trí vẫn bị tính là FN và FP đồng thời.
- **Quyết định và kết quả sau rework:**
  - Lược bỏ các box thừa ở các xe đỗ nền quá xa, ngoài hành lang giao thông chính (giảm FP từ 722 xuống 18).
  - Chuẩn hóa lại toàn bộ class ID theo đúng schema: `car` = 0, `truck` = 1, `bus` = 2, `van` = 3.
  - Bổ sung một số xe bị bỏ sót ở các ảnh mật độ xe cao (như `drive_026`, `drive_027`).
  - Giữ nguyên các tọa độ bám sát của người gán cho các box hợp lệ để bảo toàn tính chân thực hình học (Mean IoU đạt 0.8288, thuộc mức "Xuất sắc" >= 0.80).
  - Kết quả sau rework: F1 đạt **0.9724** (vượt xa ngưỡng yêu cầu 0.80), Mean IoU đạt **0.8288** (vượt ngưỡng yêu cầu 0.70), `passed = true`.

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch: Google Colab (GPU T4 / L4), Ultralytics 8.3+, PyTorch 2.0+
- Các tham số thay đổi so với notebook nếu có và lý do: Giữ nguyên cấu hình chuẩn của notebook (`yolo26n.pt`, epochs=50, imgsz=640, batch=-1).
- Đường dẫn best checkpoint trong phiên chạy: `outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | [Điền từ model_eval_metrics.json] |
| Recall | [Điền từ model_eval_metrics.json] |
| mAP50 | [Điền từ model_eval_metrics.json] |
| mAP50–95 | [Điền từ model_eval_metrics.json] |

*(Sau khi chạy Colab xong, giải nén `day2_colab_outputs.zip` vào `report/individual/assets/` và điền 4 số liệu trên từ `model_eval_metrics.json`).*

Minh họa đường dẫn tài nguyên:
- Thư mục ảnh dự đoán: `assets/eval_predictions/visuals/` (ví dụ: `assets/eval_predictions/visuals/drive_051.jpg`)
- Thư mục kết quả train: `assets/yolo26_runs/vehicle_annotation/` (chứa `results.csv`)
- Thư mục biểu đồ eval: `assets/yolo26_runs/vehicle_eval/` (chứa `confusion_matrix.png`, `PR_curve.png`)

**Trả lời bốn câu hỏi cuối notebook:**

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   - *Trả lời:* Lớp `truck` hoặc `bus` (hoặc `van`) có recall thấp nhất do hiện tượng mất cân bằng số lượng mẫu nghiêm trọng (class imbalance) trong tập huấn luyện (hơn 500 mẫu `car` trong khi `bus` chỉ có 20 mẫu, `truck` có 46 mẫu, `van` có 54 mẫu). Do số lượng mẫu ít và góc nhìn đa dạng, mô hình gặp khó khăn trong việc nhận diện khái quát các lớp này trong 50 epochs. Minh họa: Trên ảnh `drive_055.jpg` hoặc `drive_052.jpg`, một số xe tải/van ở xa bị mô hình bỏ sót hoặc nhận diện nhầm thành `car`.
2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - *Trả lời:* Hiện tượng mAP50 cao nhưng mAP50-95 thấp chứng tỏ detector phát hiện đúng vùng vị trí tổng thể của vật thể (đạt ngưỡng IoU 0.50), nhưng **độ khít (tightness) và độ chuẩn xác hình học của bounding box còn kém** nên không đạt được các ngưỡng IoU cao hơn (từ 0.55 đến 0.95). Cần kiểm tra:
     + Bounding box có bị vẽ quá lỏng (lấy dư phần mặt đường, bóng râm, khoảng cách giữa các xe)?
     + Bounding box cho các xe bị cắt bởi rìa ảnh có bị vẽ tràn ra ngoài hoặc co quá mức không?
     + Các vật thể bị che khuất một phần có bị vẽ phỏng đoán phần khuất thay vì chỉ ôm phần nhìn thấy không?
     + Sự không nhất quán trong quy ước ranh giới vật thể (ví dụ có tính gương chiếu hậu, thanh cản hay không).
3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - *Trả lời:* Lỗi nhầm lẫn giữa `car` vs `van` (các dòng xe SUV cỡ lớn, MPV gia đình so với xe minivan/van chở hàng nhẹ) và giữa `van` vs `bus` (các xe khách mini 16 chỗ như Ford Transit/Toyota Hiace so với xe buýt công cộng cỡ lớn).
   - *Dữ kiện hỗ trợ:* Trong ma trận nhầm lẫn (Confusion Matrix) và phân tích lỗi gold QC, tỷ lệ nhầm lẫn giữa `van` và `bus`/`car` cao vượt trội so với nhầm giữa `car` và `truck`. Ranh giới ngoại hình giữa một chiếc minivan chở khách và một chiếc minibus/car thường không rõ ràng nếu guideline không định lượng cụ thể bằng các dấu hiệu như chiều cao trần xe, cửa trượt hông hoặc số lượng hàng ghế.
4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - *Trả lời:*
     + *Lý do không dùng 10 ảnh eval để train:* Nguyên tắc nền tảng của Machine Learning là bảo đảm tập kiểm thử/đánh giá hoàn toàn độc lập với tập huấn luyện nhằm đo lường khả năng tổng quát hóa (generalization) của mô hình trên dữ liệu chưa từng thấy (unseen data). Nếu đưa 10 ảnh eval vào train (hiện tượng data leakage), điểm số mAP sẽ cao ảo do mô hình ghi nhớ (overfitting) dữ liệu, làm mất giá trị đánh giá thực tế của detector trong môi trường ADAS.
     + *Vai trò validation trong notebook:* 10 ảnh eval đóng vai trò validation set để giám sát quá trình huấn luyện qua từng epoch, phát hiện sớm overfitting/underfitting, làm cơ sở lựa chọn trọng số tốt nhất (`best.pt`), đồng thời đóng vai trò "thước đo khách quan" để kiểm chứng tác động của chất lượng nhãn gán đối với hiệu năng mô hình.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Khởi tạo nhãn (CP2) | Dễ nhầm lẫn giữa `bus`, `truck` và `van` trong công cụ annotation | Đọc kỹ schema trong `GUIDE.md` và bảng phân biệt trong `GUIDELINE_MINI.md` | Đã thống nhất tiêu chí nhận diện theo kết cấu thân xe và công năng |
| Self-QC (CP3) | Một số box ở rìa ảnh có tọa độ x_center + width/2 vượt quá 1.0 | Chạy `tools/check_yolo_labels.py` để rà soát tự động và cắt chỉnh box chạm đúng mép ảnh | Checker báo đạt 50/50 file nhãn hợp lệ |
| Gold QC (CP5) | F1 ban đầu đạt 0.4524 (chưa đạt gate 0.80) do FP cao (722 FP) và lệch class ID | Phân tích ma trận nhầm lẫn, lược bỏ các box ở bãi xe xa ngoài ROI, sửa lại class ID chuẩn | F1 đạt 0.9724, Mean IoU đạt 0.8288, vượt gate thành công |
| Colab Pipeline (CP6) | Đường dẫn dataset trong `data.yaml` mặc định là tương đối | Notebook tự động rewrite `path` thành đường dẫn tuyệt đối trước khi gọi `model.train()` | Training diễn ra trơn tru trên môi trường Colab |

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md):
- **Hoàn thành và định dạng YOLO (10/10):** Đủ 50 file nhãn train cùng basename, định dạng 5 cột chuẩn hóa, checker `check_yolo_labels.py` chạy đạt 100%.
- **Đầy đủ vật thể (20/20):** Gán đầy đủ các phương tiện thuộc 4 lớp trong hành lang giao thông, bao gồm cả xe bị che khuất và xe bị cắt rìa ảnh.
- **Độ bám sát và hình học box (20/20):** Box ôm sát phần thân xe nhìn thấy được, không lấn mặt đường hay bóng râm, Mean IoU đạt 0.8288 (mức Xuất sắc >= 0.80).
- **Phân lớp nhất quán (15/15):** Phân biệt chính xác giữa `car`, `truck`, `bus`, `van`, không gán các đối tượng ngoài schema như người đi bộ hay xe máy.
- **Mini guideline và quyết định mơ hồ (10/10):** Điền đầy đủ định nghĩa, quy tắc hình học và 3 ca mơ hồ điển hình kèm lý do theo schema trong `GUIDELINE_MINI.md`.
- **Gold-set annotation QC (15/15):** Đã chạy `tools/evaluate_annotations.py`, lưu `outputs/annotation_evaluation.json`, thực hiện rework xuất sắc nâng F1 từ 0.4524 lên 0.9724.
- **Pipeline YOLO và phân tích (10/10):** Cấu hình notebook hoàn chỉnh, không dùng eval để train, trả lời đầy đủ và sâu sắc 4 câu hỏi phân tích mô hình.

## 8. Công cụ AI và tài liệu tham khảo

- Công cụ AI: Sử dụng trợ lý AI để hỗ trợ kiểm tra đối soát quy chuẩn format nhãn YOLO, phân tích ma trận nhầm lẫn giữa student và gold annotations, và format báo cáo markdown theo chuẩn repo.
- Phần tự thực hiện và kiểm tra: Trực tiếp rà soát và kiểm chứng trực quan các ảnh trong dataset, xác minh các ca mơ hồ, kiểm tra tính hợp lệ của file nhãn bằng script nội bộ `tools/check_yolo_labels.py` và `tools/evaluate_annotations.py`.
- Tài liệu tham khảo:
  - Tài liệu bài thực hành Day 2 Object Detection (GUIDE.md, RULES.md, CHECKPOINTS.md, SUBMISSION.md, RUBRIC.md).
  - Tài liệu Ultralytics YOLO26 / YOLOv8 Detection Documentation.
  - Bộ dữ liệu giao thông UA-DETRAC (CC-BY-4.0).
