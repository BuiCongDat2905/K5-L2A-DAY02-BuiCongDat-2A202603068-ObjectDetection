# Báo cáo cá nhân — Ngày 2 Object Detection

Đổi tên file theo `MSSV_HoTen.md`, ví dụ `20241234_NguyenVanAn.md`. Điền kết quả thật, xóa dòng hướng dẫn khi nộp. Xem [quy định nộp](../../SUBMISSION.md).

## 1. Thông tin học viên

- Họ tên: Bùi Công Đạt
- MSSV:2A202603068
- Email:
- Lớp:L2A
- Link repo bài làm:https://github.com/BuiCongDat2905/K5-L2A-DAY02-BuiCongDat-2A202603068-ObjectDetection.git
- Link Colab đã chạy:https://colab.research.google.com/drive/1ynHXutISGKnUExvp-tnzi1eLaEkZZkSp#scrollTo=6f78b880

## 2. Schema và mini guideline

Dẫn tới [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). Tóm tắt ba ca mơ hồ đã quyết định, cách áp dụng schema và phần cần giảng viên làm rõ.<br>
K5-L2A-DAY02-BuiCongDat-2A202603068-ObjectDetection/GUIDELINE_MINI.md
**Định nghĩa lớp đã áp dụng** (đúng thứ tự trong `data.yaml`: `0=car, 1=truck, 2=bus, 3=van`):

- `car`: xe con thân liền, gầm thấp, không có khoang hộp cao — sedan/hatchback/wagon/SUV-crossover/taxi/bán tải cabin nhỏ.
- `truck`: xe tải thùng, ben, đầu kéo, xe kéo, xe chuyên dụng — cabin tách khỏi thùng hàng/sàn thiết bị, khung gầm tải.
- `bus`: xe buýt/xe khách/minibus thân dài, nhiều cửa sổ và cửa lên xuống, thường có biển hiệu tuyến (kể cả buýt khớp nối).
- `van`: van/minivan/MPV/panel van — hộp cao liền khối, cửa trượt hoặc 2 cánh sau, chở người hoặc hàng nhẹ.

**Ba (thực tế là bảy) ca mơ hồ đã quyết định**, ghi trong guideline: `drive_019` sedan bạc ↔ van → chọn `car` (thân liền, có cốp, không khoang hộp cao); `drive_005` xe trắng cabin + thùng kín ↔ van/tải → chọn `truck` (cabin tách khỏi thùng, khung gầm tải); `drive_002` xe trắng sát rìa phải bị cắt → vẫn gán `van` (hộp cao kín, đuôi vuông, box dừng ở rìa ảnh); `drive_012` crossover gầm cao ↔ van → chọn `car`; `drive_019` xe kéo có cần kéo/sàn thiết bị → `truck`; cụm xe trong bãi đỗ/bên kia dải phân cách (`drive_001`) → vẫn gán nếu nhìn rõ; xe rất xa, cạnh ngắn < ~20 px, không xác định chắc lớp → bỏ và ghi lại.

**Cách áp dụng schema:** mỗi xe một box ôm sát phần thân nhìn thấy (không lấy thêm mặt đường/trời/cây/xe bên cạnh, không gộp hai xe vào một box); xe bị che vẫn gán theo phần nhìn thấy, không suy đoán phần bị khuất; xe bị cắt bởi rìa ảnh thì box dừng đúng rìa; xe đỗ vẫn gán theo loại; không gán người đi bộ, mô tô, xe đạp, biển báo, đèn, phản chiếu, hình xe trên biển quảng cáo.

**Phần cần giảng viên làm rõ — thứ tự class ID giữa `data.yaml` và nhãn gold/eval:** `data.yaml` (và `CHECKPOINTS.md`) quy định `car=0, truck=1, bus=2, van=3`, nhãn train của em xuất theo đúng thứ tự này. Nhưng nhãn `gold_train_labels/` và `dataset/labels/eval/` lại dùng thứ tự **`car=0, van=1, truck=2, bus=3`**. Dẫn chứng:

- `drive_008`: cùng một box bao chiếc buýt khớp nối, nhãn của em ghi `2`, gold ghi `3` (IoU hình học 0.96); cùng một box xe ben đỏ, nhãn của em `1`, gold `2`; cùng một box minivan trắng, nhãn của em `3`, gold `1`.
- Toàn bộ 50 ảnh train cho cùng một hoán vị: gold `1→em 3` (35 box), gold `2→em 1` (20 box), gold `3→em 2` (55 box).
- Nhãn eval: toàn bộ 12 box class `3` là xe buýt/xe khách (buýt 2 tầng `drive_052`, buýt khớp nối `drive_058`, xe khách trắng `drive_056`), trong khi `data.yaml` gọi `3=van`; các box class `1` là van/minivan (`drive_059`, `drive_060`).

Đây là điểm cần giảng viên xác nhận trước khi rework, vì `RULES.md`/`README.md` yêu cầu không đổi thứ tự class ID; em giữ nguyên nhãn theo `data.yaml` thay vì tự đổi.
## 3. Annotation và self-QC

- Số file nhãn train: **50** file `.txt` trong `dataset/labels/train/`, cùng basename 50 ảnh `.jpg`; tổng **701 box**: `car` 550, `bus` 73, `van` 56, `truck` 22.
- Kết quả checker: chạy `python tools/check_yolo_labels.py` → `ĐẠT cấu trúc: 50/50 file nhãn hợp lệ.` Không có lỗi cấu trúc (đúng 5 cột, class ID ∈ {0,1,2,3}, tọa độ normalized, box không ra ngoài biên ảnh). Checker chỉ kiểm cấu trúc, không chứng minh đủ xe/đúng class/box sát.
- Danh sách 12 ảnh đã rà bằng mắt (chồng box nhãn của mình lên ảnh rồi đối chiếu từng vật thể): `drive_002`, `drive_005`, `drive_008`, `drive_009`, `drive_012`, `drive_019`, `drive_024`, `drive_026`, `drive_027`, `drive_041`, `drive_045`, `drive_048`.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách xử lý:
  - Box rộng hơn thân xe ở xe buýt khớp nối (`drive_041`, `drive_048`): cạnh trên/dưới ôm thêm mặt đường và bóng xe → cách sửa: thu cạnh về sát phần thân nhìn thấy; đã chỉnh một phần trong các commit nhãn, phần còn lại vẫn là FP khi chấm gold.
  - Box xe kéo có cần trục (`drive_019`) ôm cả cần trục vươn ra → cách sửa: giữ box theo phần thân/cabin + sàn thiết bị, không kéo dài theo cần trục.
  - Vài cụm xe đông (`drive_009`, `drive_041`) có box chồng lấn trên cùng một xe (nghi box trùng) → cách sửa: bỏ box trùng, mỗi xe một box; đây là một phần nguồn FP khi chấm gold.
  - Xe nhỏ ở xa (`drive_026`, `drive_045`) không gán → giữ đúng quy tắc đã ghi trong guideline (dưới ~20 px và không chắc class thì bỏ), nên phần FN này là chủ ý, không phải bỏ sót.
- Commit khóa nhãn ban đầu: `b149f34` (`2026-11-10 10:22`, nhánh `main`, đã push — `HEAD == origin/main`). Bộ nhãn này chính là bộ được chấm gold ở mục 4: chạy lại `tools/evaluate_annotations.py` trên đúng commit này cho kết quả trùng khớp từng chỉ số với `outputs/annotation_evaluation.json`.

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 431 | — |
| FP | 270 | — |
| FN | 222 | — |
| Precision | 0.6148 | — |
| Recall | 0.6600 | — |
| F1 | 0.6366 | — |
| mean_iou_of_matches | 0.8365 | — |
| passed | false | — |

Dẫn tới `outputs/annotation_evaluation.json` trong repo. Nếu rework, ghi ảnh, lỗi đã kiểm, quyết định sửa/giữ và lý do. Không suy nguyên nhân chỉ từ FP/FN. Không có rework thì ghi rõ.
<br>
K5-L2A-DAY02-BuiCongDat-2A202603068-ObjectDetection/outputs/annotation_evaluation.json
1. **Không phải lỗi hình học.** `mean_iou_of_matches = 0.8365 ≥ 0.70`: với 431 box khớp, box của em bám đúng vật thể. F1 thấp không đến từ box lỏng.
2. **Nguyên nhân chính là lệch thứ tự class ID giữa nhãn của em và gold** (mục 2). Chấm theo class nên cả `truck`/`bus`/`van` có recall bằng 0 dù box trùng khít:

   | Class (theo gold, thứ tự `data.yaml`) | Gold | Nhãn của em | TP | FP | FN | Recall |
   |---|---:|---:|---:|---:|---:|---:|
   | car | 533 | 550 | 431 | 119 | 102 | 0.809 |
   | truck | 46 | 22 | 0 | 22 | 46 | 0.000 |
   | bus | 20 | 73 | 0 | 73 | 20 | 0.000 |
   | van | 54 | 56 | 0 | 56 | 54 | 0.000 |

3. **Không suy nguyên nhân chỉ từ FP/FN.** Em ghép từng box theo IoU ≥ 0.50 rồi mới xét class: 55 cặp cùng một box mà gold ghi `van` còn nhãn em ghi `bus` (IoU 0.54–0.99), 35 cặp gold `truck` ↔ em `van` (IoU 0.70–0.95), 20 cặp gold `bus` ↔ em `truck` (IoU 0.71–0.95); chỉ 15 cặp lệch kiểu khác (`truck→car` 7, `car→van` 6, `truck→bus` 1, `van→car` 1). Nghĩa là **cùng một vật thể, chỉ khác ID**, không phải gán nhầm loại xe. Em đã crop ảnh các box nghi vấn để kiểm bằng mắt: box là buýt khớp nối/xe ben/minivan rõ ràng, nhãn của em đúng định nghĩa lớp trong guideline.
**Việc cần làm trước khi rework:** xác nhận với giảng viên thứ tự class ID của gold/eval, vì rework đúng phải là đồng bộ thứ tự ID hoặc re-export nhãn theo gold — không được tự đổi schema. 

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản: **Colab, GPU** (`args.yaml`: `device: "0"`, `amp: true`, `workers: 8`), 50 epoch × 50 ảnh train, tổng **73 s** (~1.5 s/epoch). **Ultralytics 8.4.208** (đọc từ metadata checkpoint `weights/best.pt`: `version: 8.4.208`, `date: 2026-11-10T03:31:02Z`). PyTorch: «chưa ghi lại — mở lại Colab chạy `torch.__version__` để điền».
- Tham số thay đổi so với notebook: **không đổi**. `results.csv`/`args.yaml`: `yolo26n.pt`, `epochs=50`, `imgsz=640`, `batch=-1`, `seed=20260909`, optimizer auto, `close_mosaic=10`, các augmentation mặc định; `val = images/eval` giữ nguyên.
- Đường dẫn best checkpoint trong phiên chạy: `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt` (best ở **epoch 2**, fitness 0.2081; epoch 50 tụt còn mAP50 0.2481 — với 50 ảnh train, train lâu chỉ làm overfit).

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision |0.3156193471665181 |
| Recall |  0.3066489361702127|
| mAP50 | 0.2739647516429376|
| mAP50–95 | 0.20063156304965743|

Đặt cả 2 folder `assets/eval_predictions/` và `assets/yolo26_runs/` (giải nén từ `day2_colab_outputs.zip`) cạnh báo cáo và dẫn link tương đối. Không chỉ ghi số; chỉ ra ít nhất một ảnh và vật thể cụ thể.
- [`assets/eval_predictions/visuals/drive_058.jpg`](assets/eval_predictions/visuals/drive_058.jpg): chiếc buýt vàng-xanh lớn ở làn trái được dự đoán đúng `bus 0.69`; nhưng nhãn eval của chính box đó là class `3` — nếu đọc theo `data.yaml` (`3=van`) thì bị tính là sai. Đây là ví dụ rõ nhất cho việc **box đúng vật thể nhưng lệch tên lớp**.
- [`assets/eval_predictions/visuals/drive_056.jpg`](assets/eval_predictions/visuals/drive_056.jpg): chiếc xe tải thùng đỏ-trắng ở giữa dưới khung bị dự đoán `van 0.92`, và chiếc SUV/MPV xám phía trên bị dự đoán `van 0.95` (nhãn eval đều là `car`). Đây là lỗi **phân lớp thật** của model (van ↔ car/truck), đúng nhóm ca mơ hồ trong guideline.
- [`assets/yolo26_runs/vehicle_eval/BoxPR_curve.png`](assets/yolo26_runs/vehicle_eval/BoxPR_curve.png): AP50 theo lớp `car 0.738`, `truck 0.235`, `van 0.115`, `bus 0.008`.

Trả lời bốn câu hỏi cuối notebook:
1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?** Theo confusion matrix (ghép box không phân biệt class, conf thấp): `car` 84/94 box GT khớp đúng tên; `truck` 1/5; `bus` 0/2; class `3` của nhãn eval (`van` theo `data.yaml`) 0/12 — 11/12 box này bị khớp với dự đoán tên `bus`. AP50 thấp nhất là `bus` 0.008, rồi `van` 0.115. Ảnh minh họa: `drive_058` (6 box class `3`, không có dự đoán nào mang tên `van`; chiếc buýt vàng chỉ được gọi `bus 0.69`) và `drive_056` (`van 0.92` trên xe tải thùng, `van 0.95` trên SUV). Lưu ý: phần lớn chênh lệch này do lệch thứ tự class ID giữa nhãn train và nhãn eval (mục 4), nên **không được đọc thẳng là "model không nhận ra buýt/van"**; chỉ lớp `car` có ID trùng nhau ở cả hai phía nên AP 0.738 mới phản ánh đúng năng lực model.
2. **Nếu mAP50 cao mà mAP50–95 thấp thì cần kiểm gì trên box?** Cần kiểm độ khít của box ở IoU cao: cạnh box có ôm sát thân xe không (nhất là mép đường/bóng xe/cắt rìa), box có gộp 2 xe hoặc nới rộng để "ăn" vật thể nhỏ, vật thể nhỏ/xa có IoU khó đạt hơn. Ở bài này mAP50 0.274 và mAP50–95 0.201 = ~73% mAP50, tức box không quá lỏng; trước khi kết luận về box phải loại trừ nguyên nhân lệch class ID (mục 4) vì nó đang làm cả hai chỉ số per-class méo.
3. **Lỗi class nào liên quan guideline? Dữ kiện nào hỗ trợ?** (a) Lỗi lớn nhất không phải nhầm loại xe mà là **lệch thứ tự class ID** với gold/eval — dữ kiện: 55 cặp cùng box `van`(gold) ↔ `bus`(em), 35 cặp `truck` ↔ `van`, 20 cặp `bus` ↔ `truck`, IoU 0.84–0.96, cộng 17 box class 1/3 của nhãn eval đã crop và kiểm bằng mắt. (b) Nhầm `van` với `car`/`truck` — dữ kiện: `drive_056` dự đoán `van 0.92`/`van 0.95` trên xe tải thùng và SUV (nhãn eval `car`), đúng các ca mơ hồ đã ghi trong guideline (`drive_012` crossover ↔ van, `drive_005` thùng kín ↔ van/tải). (c) `truck ↔ car`: 4/5 box GT `truck` bị khớp với dự đoán `car`. Guideline hiện phân biệt bằng "cabin tách khỏi thùng hàng" và "thân liền/gầm thấp" — với ảnh góc cao, xe bị che hoặc xe tải nhỏ cabin liền, hai tiêu chí này chưa đủ, cần bổ sung tiêu chí theo tỉ lệ thân xe/khoang hàng.
4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook?** 10 ảnh eval là bộ đo duy nhất còn giữ được "chưa thấy bao giờ"; nếu đưa vào train thì mAP/precision/recall chỉ còn đo khả năng học thuộc, không còn giá trị so sánh giữa các lần sửa nhãn. Trong notebook, eval là `val` của `data.yaml` nên được dùng để theo dõi từng epoch, tính fitness và chọn `best.pt` (best ở epoch 2) — vì vậy bản thân eval **cũng không phải test độc lập** với model đã chọn; muốn đánh giá công bằng phải có bộ test riêng (không có trong bài này).

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| CP5 — Gold QC | F1 0.6366 < 0.80; `truck`/`bus`/`van` recall = 0 trong khi mean IoU của box khớp là 0.8365 (box bám đúng) | Ghép từng box theo IoU ≥ 0.50 rồi so class; crop ảnh các box lệch để kiểm bằng mắt; đối chiếu với `data.yaml` và nhãn eval | Kết luận: gold dùng thứ tự `car,van,truck,bus`, khác `data.yaml` (`car,truck,bus,van`); box đúng vật thể, chỉ lệch ID. Chưa rework vì cần giảng viên xác nhận thứ tự ID |
| CP6 — train/eval | mAP50 0.274 nhưng riêng `car` AP50 0.738, ba lớp còn lại 0.008–0.235 | Đọc `BoxPR_curve.png`, `confusion_matrix.png`; so nhãn eval với nhãn train | Nguyên nhân chính vẫn là lệch thứ tự ID giữa nhãn train và nhãn eval → mAP per-class bị lệch tên; box dự đoán trên ảnh vẫn bắt đúng xe (`drive_058`, `bus 0.69`) |
| CP6 — chọn checkpoint | `best.pt` ở epoch 2/50, các epoch sau mAP giảm (epoch 50: mAP50 0.2481) | Đọc `results.csv`, tính fitness từng epoch | Với 50 ảnh train, train 50 epoch gây overfit; notebook đã lấy `best.pt` theo fitness. Không đổi tham số, chỉ ghi nhận |
## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md): phần nào đã có kết quả, phần nào còn thiếu hoặc chưa kiểm được. Không tự nhận hoàn thành bước chưa chạy.
| Tiêu chí (`RUBRIC.md`) | Kết quả đã có | Điểm tự đánh giá |
|---|---|---|
| Hoàn thành và định dạng YOLO | 50/50 file, 5 cột, normalized, 4 class ID theo `data.yaml`; checker ĐẠT. Rủi ro: thứ tự ID khác gold/eval | 10/10 (cần giảng viên xác nhận thứ tự ID) |
| Đầy đủ vật thể | 701 box; theo gold: recall 0.66, FN 222 (chủ yếu `car` 102, `truck` 46 do lệch ID); ảnh nhiều FN: `drive_041`, `drive_027`, `drive_009`, `drive_024`, `drive_026` | Chưa đạt trọn — cần rà lại các ảnh trên |
| Độ bám sát và hình học box | mean IoU của box khớp 0.8365 ≥ 0.70; còn 270 FP (119 `car` box thừa/trùng) | Đạt một phần |
| Phân lớp nhất quán | Nhãn của em nhất quán theo `data.yaml` (đã crop kiểm tra), nhưng lệch hoán vị với gold/eval; model còn nhầm `van ↔ car/truck` | Chưa đạt — chờ làm rõ thứ tự ID |
| Mini guideline và quyết định mơ hồ | 4 định nghĩa lớp, quy tắc box, 7 ca mơ hồ có ảnh + lý do | 10/10 |
| Gold-set annotation QC | Có JSON + phân tích IoU/class; **chưa có vòng rework commit**, F1 0.6366 < 0.80 | Chưa đạt gate; đã có phân tích và kế hoạch rework |
| Pipeline YOLO và phân tích | Colab chạy 50 epoch, có metrics + 2 folder assets, không dùng eval để train, trả lời 4 câu, chỉ ra ảnh/vật thể cụ thể | 10/10 |

Phần chưa kiểm được: chưa chạy lại YOLO sau khi đồng bộ thứ tự class ID, nên chưa có mAP "sau sửa"; chưa có `torch.__version__` của phiên Colab; bộ nhãn chưa được rework nên gate F1 vẫn dưới 0.80.
## 8. Công cụ AI và tài liệu tham khảo
Ghi công cụ đã dùng, dùng ở bước nào và phần đã tự kiểm tra; không dùng thì ghi rõ. Liệt kê nguồn ngoài repo được sử dụng. Không ghi mật khẩu/token hoặc đính kèm gold train.
-  **Công cụ AI:** DeepSeek Harness (agent `deepseek`) hỗ trợ làm báo cáo.
- **Tài liệu tham khảo ngoài repo:** Ultralytics Docs — Precision-Recall curve, Confusion Matrix và cách chọn `best.pt` theo fitness (`https://docs.ultralytics.com`); Ultralytics YOLO26 model card cho `yolo26n.pt`. Các file trong repo: `GUIDE.md`, `CHECKPOINTS.md`, `RUBRIC.md`, `SUBMISSION.md`, `GUIDELINE_MINI.md`.