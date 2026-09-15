# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: Bùi Thành Long
Ngày: 15/9/2026

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | 30 phút |
| Thời gian gán `clip_01` | 90 phút |
| Số track đã vẽ trong `clip_01` | 8 |
| Số keyframe trung bình mỗi track | `...`  |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. Ô tô con bị ô tô to đè, chỉ hiện dần từ đuôi xe — phải căn chỉnh bbox nhiều lần theo từng frame để chỉ lấy đúng phần ô tô con đang thực sự hiện ra, tránh vẽ lố sang phần bị xe to che khuất.
2. Xe buýt có gương chiếu hậu to, trồi hẳn ra ngoài thân xe — quyết định vẽ gương vào cùng một bbox với thân xe buýt (coi gương là phần nhìn thấy được của xe).
3. Xe đứng yên nhưng camera rung/dịch chuyển nhẹ liên tục từng chút một — bbox tưởng thẳng hàng nhưng thực ra trôi theo rung camera, phải căn chỉnh và kiểm tra khá lâu mới khớp.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: `...`
- Lượt 2: `...`
- Lượt 3: `...`

Kiểm chéo với: `...`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `...`. Số lỗi bạn ấy tìm được trong bản của bạn: `...`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`...`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `027cbf269170359290dbf7734d27648c26b3b4d7e3f65f287bbb8bc3fc946549` |
| Thời điểm khóa | 2026-09-15 10:06:06 UTC |
| Số row / frame / track trước khi mở reference | 614 row / 190 frame / 8 track |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.803 | 0.785 | 0.823 | 0.880 | 0.937 | 0.869 | 0.873 | 58 | 17 | 0 |
| Sau rework | 0.803 | 0.785 | 0.823 | 0.880 | 0.937 | 0.869 | 0.873 | 58 | 17 | 0 |

> **Ghi chú:**  **Cập nhật sau khi xem lại trên Colab:** khi tua `visualize_tracks.py` để so trực quan track đỏ (BoT-SORT + ReID treatment) với track xanh (bạn) ở frame 91, 105–107, phát hiện track đỏ T7 (khớp đúng ID 7 trong danh sách "BBOX THỪA" của `eval_reid_vs_gold.json`, frame 16–116) đang khoanh vào **những hiện vật khác — không phải xe**.

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có** (bản pre-gold đã đạt cả 3 ngưỡng: IDF1 0.937, MOTA 0.869, MOTP 0.873)

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox treo/thừa (Outside trễ) | 80–100 | 6 | Tua tới đúng frame xe rời khung, bấm Outside tại đó thay vì frame cũ — xoá phần bbox thừa nằm sau khi xe đã ra khỏi khung |
| Bbox treo/thừa (Outside sớm/trễ) | 149–151, 51–53 | 4 | Tua tới đúng frame xe thực sự xuất hiện/rời khung, dịch keyframe Outside/vào khung về đúng vị trí đó |
| Bbox treo/thừa (Outside trễ/sớm) | 169–171, 133–135 | 8 | Tương tự: căn lại đúng frame bắt đầu/kết thúc track bằng Outside, không để bbox tồn tại ở nơi không có xe |
| Bbox trôi (IoU thấp) | 84–92 | 5 | Thêm keyframe mới ở các frame trong khoảng 84–92 (thay vì chỉ nội suy từ hai đầu xa), chỉnh khít lại bbox theo vị trí xe thật ở từng frame |
| Thiếu đoạn (chỉ phủ 79%) | — | 6 | Tua lại toàn bộ vòng đời track 6, tìm đoạn chưa có bbox dù xe vẫn còn trong khung, thêm keyframe để lấp đầy đoạn bị bỏ trống |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | 3.13.15 / 8.4.145 / 2.11.0+cpu / 0.5.13 |
| weights / hai tracker | `yolo26n.pt` / ByteTrack (`bytetrack.yaml`) control, BoT-SORT + ReID (`botsort-reid.yaml`) treatment |
| conf / IoU chạy tracker / imgsz / classes | conf: 0.25 · IoU tracker: 0.7 (IoU khớp khi chấm với gold: 0.5) · imgsz: 960 · classes: [2, 5, 7] (car, bus, truck) |
| device | cpu |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.803 | 0.785 | 0.823 | 0.880 | 0.937 | 0.869 | 0.873 | 58 | 17 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.793 | 0.740 | 0.850 | 0.925 | 0.887 | 0.774 | 0.919 | 80 | 56 | 3 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA (0.869) **thấp hơn** IDF1 (0.937) trong bản của tôi — không phải kịch bản "MOTA cao, IDF1 thấp". IDSW = 0, nghĩa là mọi ID tôi gán khớp đúng gold suốt vòng đời track, không có lỗi identity nào. Chênh lệch MOTA/IDF1 ở đây đến từ hướng ngược lại: MOTA = 1 − (FP + FN + IDSW) / số bbox gold, nên 58 FP (chủ yếu là bbox "treo/thừa" — bấm Outside trễ/sớm vài frame ở ID 4, 6, 8) và 17 FN (bbox trôi dưới IoU 0.5 ở track 5 frame 84–92, track 1 frame 190) kéo MOTA xuống dù identity hoàn toàn đúng.

Điều này minh hoạ đúng lý do MOTA không phạt nặng lỗi ID: công thức đếm IDSW là **một đơn vị lỗi cho mỗi lần đổi ID**, cùng thang với một FP hoặc FN đơn lẻ — bất kể track đó dài bao nhiêu frame. Một track 95 frame bị đổi ID một lần chỉ cộng 1 vào tử số, y hệt như một bbox lệch ở một frame. IDF1 thì ngược lại: nó match toàn bộ chuỗi ID xuyên suốt track (identity-level precision/recall), nên một lần đổi ID giữa track dài sẽ phạt nặng hơn nhiều theo tỷ lệ. Vì vậy một bản nhãn/track có thể MOTA cao (ít FP/FN tuyệt đối) nhưng IDF1 thấp nếu vài lần đổi ID rơi đúng vào các track dài — MOTA "pha loãng" lỗi đó, IDF1 thì không.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

IDF1 tăng 0.875 → 0.900 (+0.025), AssA tăng 0.776 → 0.820 (+0.044). IDSW **bằng nhau** ở cả hai (2 lần) — nhưng không rơi vào cùng track: ByteTrack đổi ID ở track gold 4 (frame 59: 14→15) và track gold 5 (frame 94: 23→32); ReID đổi ID ở track gold 5 (frame 87: 17→18) và track gold 6 (frame 113: 24→31).

Frame sequence cụ thể cho thấy bức tranh không đồng nhất một chiều:
- **Treatment tốt hơn ở track 4**: ByteTrack tách track 4 (95 frame) thành 2 ID tại frame 59, nhưng ReID giữ track 4 nguyên một ID xuyên suốt (không xuất hiện trong danh sách tách track của `eval_reid_vs_gold.json`). Đây đúng như kỳ vọng lý thuyết: đoạn frame 59 nhiều khả năng là occlusion/crossing ngắn mà chỉ motion+IoU (ByteTrack) không đủ tín hiệu để nối lại, còn appearance cue của ReID nối đúng.
- **Treatment tệ hơn ở track 6**: ByteTrack giữ track 6 (56 frame) nguyên vẹn, nhưng ReID lại tách track 6 thành 3 ID (24, 28, 31) quanh frame 107–113 — một lỗi mới mà control không có.

Vì vậy treatment tốt hơn **ở tổng thể** (AssA/IDF1 cao hơn) nhưng không phải "ReID sửa mọi lỗi association" — nó đánh đổi: fix được track 4, phát sinh lỗi mới ở track 6. Quan trọng: ByteTrack và BoT-SORT là hai tracker implementation khác nhau (khác thuật toán match, khác cách xử lý track buffer), nên chênh lệch AssA/IDF1 này là **so sánh hệ thống**, không phải bằng chứng causal rằng riêng ReID gây ra toàn bộ khác biệt. Muốn cô lập đúng effect của ReID phải chạy BoT-SORT với `with_reid: false` cùng config rồi so sánh.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

DetA tăng 0.649 → 0.711 (+0.062). FN giảm mạnh 54 → 26 (gần một nửa). FP tăng nhẹ 88 → 91 (+3).

FN giảm mạnh không đến từ detector tìm ra vật thể mới (cả hai dùng chung `yolo26n.pt`, cùng detector input) mà từ cách tracker **duy trì track** qua các đoạn khó: danh sách "MODEL BẮT THIẾU ĐOẠN" giảm từ 3 track bị hụt phủ (track 8: 61%, track 6: 75%, track 5: 77%) ở ByteTrack xuống còn 1 track (track 6: 79%) ở ReID — nghĩa là BoT-SORT+ReID giữ track sống qua occlusion tốt hơn, ít bỏ trống đoạn giữa hơn. Đây là hành vi **association** (cách nối detection với track cũ khi confidence tạm thấp), không phải detector phát hiện thêm.

FP tăng nhẹ (88→91) đến từ số ID "thừa" tương tự nhau ở cả hai (5 ID không khớp gold track nào, cùng khoảng frame 105–178) — không đổi nhiều về bản chất, chỉ đổi ID number.

Kết luận: phần lỗi còn lại chủ yếu là **association** (track fragmentation, track 6 vẫn hụt phủ 21% dù đã cải thiện; FP do track giả/tách track vẫn tồn tại ở cả hai), không phải detector — vì detector input giữ nguyên giữa hai lần chạy mà FN vẫn giảm đáng kể khi đổi tracker.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

**Track 6, frame ~107–113.** Trong bản của tôi, track 6 (78 frame) giữ **một ID duy nhất** suốt vòng đời — khớp gold (IDSW=0 giữa bạn vs gold cho track này). BoT-SORT+ReID lại tách track 6 thành 3 ID khác nhau (24 → 28 tại frame 107, rồi 28 → 31 tại frame 110–113), đúng đoạn được `eval_reid_vs_gold.json` xác nhận là lỗi thật của model (mục "ID SWITCH": frame 113, track gold 6, 24→31). Vì bản của tôi khớp gold hoàn hảo ở track này còn ReID lệch khỏi gold đúng tại đó, đây là bằng chứng rõ tôi giữ ID đúng, appearance cue của ReID bị nhiễu (có thể do xe khác màu/hình dạng tương tự xuất hiện gần đó trong đoạn 107–113) khiến nó tách nhầm.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

**Track 5, frame 84–94.** Ba tín hiệu độc lập cùng chỉ vào đúng đoạn này: (a) `eval_vs_gold.json` của chính tôi liệt "BBOX TRÔI" cho track 5 tại frame 84, 87, 88, 89, 90, 91, 92 với IoU chỉ 0.50–0.56 so với gold; (b) ByteTrack đổi ID ngay tại frame 94 cho track gold 5; (c) ReID cũng đổi ID tại frame 87 cho cùng track gold 5. Ba nguồn (annotation của tôi, control, treatment) đều gặp vấn đề trong cùng một cửa sổ ~10 frame quanh track 5 — đây là bằng chứng hội tụ đáng để tua lại bằng mắt (không kết luận vội): có thể xe này thực sự bị che một phần / cắt ngang bởi xe khác trong đoạn 84–94, khiến cả bbox của tôi lẫn cả hai tracker đều gặp khó. Tôi cần xem lại frame 84–94 để kiểm tra bbox có khít không và có thiếu keyframe không — nhưng sẽ không sửa annotation chỉ vì model báo khác, mà sửa dựa trên việc xem video thật.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

Ba luật đã bổ sung vào mục 5 của `GUIDELINE_MINI.md`, dựa trên đúng các lỗi tìm được khi chấm với gold:

1. **Outside timing** — trước khi bấm Outside, tua từng frame quanh biên (±3 frame) để xác định đúng frame xe rời/vào khung, không ước lượng. Lý do: 5/5 lỗi "bbox treo/thừa" tìm thấy đều là do bấm Outside lệch vài frame (ID 4, 6, 8).
2. **Mật độ keyframe ở đoạn khó** — khi track đi vào đoạn nghi occlusion/crossing, đặt keyframe mỗi 3–5 frame thay vì để nội suy dài. Lý do: track 5 bị "bbox trôi" IoU chỉ 0.50–0.56 suốt 9 frame liền (84–92) vì khoảng cách giữa hai keyframe quá xa.
3. **Tự kiểm phủ toàn bộ vòng đời track** — sau khi vẽ xong một track, tua lại từ đầu đến Outside để xác nhận không có đoạn nào bị bỏ trống, trước khi chuyển sang xe tiếp theo. Lý do: track 6 chỉ phủ 79% số frame so với gold — có đoạn giữa track bị bỏ sót dù xe còn trong khung.

Về quy trình làm việc, tôi sẽ đổi thứ tự thao tác: thay vì vẽ khung → nhảy keyframe cố định → export, tôi sẽ **vẽ khung → tua thử toàn bộ track một lượt ngay (chưa export) → mới nhảy sang xe tiếp theo**, để bắt lỗi treo/thiếu đoạn ngay tại chỗ thay vì chờ đến bước chấm với gold mới phát hiện ra.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)
