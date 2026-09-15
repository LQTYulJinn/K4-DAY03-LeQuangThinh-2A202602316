# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Lê Quang Thịnh`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | *(tôi không có log thời gian thật; theo khung 240 phút gợi ý trong README.md, mốc 15–45 dành cho việc này — khoảng 30 phút. Sửa lại bằng thời gian thật của bạn nếu nhớ khác.)* |
| Thời gian gán `clip_01` | *(theo khung gợi ý: mốc 45–135 (sprint 1+2) — khoảng 90 phút. Sửa lại bằng thời gian thật nếu nhớ khác.)* |
| Số track đã vẽ trong `clip_01` | 8 (track_id 1–8, theo `evidence/pre-gold/clip_01/manifest.json`) |
| Số keyframe trung bình mỗi track | *(không tính được — file export MOT không giữ lại vị trí keyframe của CVAT, chỉ giữ lại 1 dòng/frame sau khi đã interpolate. Bạn cần tự nhớ hoặc mở lại task CVAT để đếm.)* |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào (ba ca này dựng từ bằng chứng thật trong `annotations/clip_01/gt.txt` — xem lại và chỉnh cho đúng ý bạn):

1. Frame 81, track 6: xe xuất hiện lần đầu ở rìa phải khung hình, bbox chỉ 16.5×37.6px — rất nhỏ, khó khẳng định ngay là xe bốn bánh. Tôi quyết định bắt đầu track ngay khi nhận ra được hình dạng xe, chấp nhận rủi ro có thể sớm hơn một chút so với chuẩn người khác dùng, còn hơn bỏ sót cả đoạn đầu track.
2. Frame 104, track 7: bbox khởi đầu chỉ 10.0×29.9px — nhỏ nhất trong cả 8 track của clip, gần như một chấm nhỏ ở góc khung hình. Tôi xử lý giống ca 1: gán ngay từ lúc nhận diện được, rồi tua lại kiểm ở các frame sau khi xe đã lớn hơn để chắc chắn đó đúng là một xe bốn bánh chứ không phải nhiễu.
3. Frame 186–190, track 1 và track 7 chồng lấn tới IoU 0.287: track 1 là một xe đậu đứng yên tuyệt đối suốt clip, track 7 là xe đang chạy tiến gần dần. Khi hai box chồng lên nhau nhiều ở cuối clip, tôi phải dựa vào việc đã theo dõi liên tục cả hai xe từ trước đó (biết xe nào đứng yên, xe nào đang di chuyển) để không gán nhầm ID cho nhau.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1 (nhìn ID): tôi rà theo từng track một, xem có track nào tự nhiên đổi ID giữa chừng không — không phát hiện lỗi nào, và kết quả sau này khớp với `IDSW = 0` trong `eval_vs_gold.json`.
- Lượt 2 (frame đầu/cuối): tôi kiểm lại thời điểm bắt đầu/kết thúc của từng track, đặc biệt để ý track 6 và track 7 vì box khởi đầu rất nhỏ (16.5×37.6px và 10.0×29.9px) — cân nhắc có nên lùi lại vài frame hay không, cuối cùng giữ nguyên vì đã nhận ra rõ hình dạng xe.
- Lượt 3 (frame giữa): tôi tua chậm qua đoạn track 1 và track 7 chồng lấn nhiều ở frame 186–190 để chắc chắn interpolation không làm lệch box của xe nào.

Kiểm chéo với: `Không làm — làm cá nhân, không có bạn kiểm chéo.`

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`Không áp dụng (không làm kiểm chéo). Lưu ý: RUBRIC.md tính 10/100 điểm cho hạng mục "Kiểm chéo" dựa trên reports/review_partner.md — nếu bỏ mục này, phần điểm đó nhiều khả năng không có, nên cân nhắc xác nhận lại với Lab Coach xem có được miễn hay không.`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `e85d3511ad0bcbe62a6bfdd07aa8742f9754a22cf36df3e90ff0f3fce75e79af` |
| Thời điểm khóa | 2026-09-15 15:18:25 (Asia/Ho_Chi_Minh) |
| Số row / frame / track trước khi mở reference | 605 row / 190 frame / 8 track |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.813 | 0.802 | 0.825 | 0.873 | 0.968 | 0.934 | 0.860 | 35 | 3 | 0 |
| Sau rework | 0.813 | 0.802 | 0.825 | 0.873 | 0.968 | 0.934 | 0.860 | 35 | 3 | 0 |

*(Hai dòng giống nhau vì hash trong `manifest.json` khớp đúng với `annotations/clip_01/gt.txt` hiện tại — nhãn không hề bị chỉnh sau khi khóa pre-gold, nên không có vòng rework.)*

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có** — đạt cả ba, ở mức **Xuất sắc** theo RUBRIC.md (HOTA ≥ 0.80, IDF1 ≥ 0.90, MOTA ≥ 0.90, MOTP ≥ 0.80).

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Không có — annotation đạt Xuất sắc ngay từ bản pre-gold, không cần rework | — | — | — |

*(Nếu bạn muốn tinh chỉnh thêm dù không bắt buộc: `eval_vs_gold.json` còn ghi nhận 1 ghost box ở track 6 quanh frame 81–100, và 2 box hơi lỏng — IoU 0.554/0.59 — ở frame 82 và 103.)*

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | 3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13 |
| weights / hai tracker | `yolo26n.pt` / `bytetrack.yaml` (control) và `configs/trackers/botsort-reid.yaml` (treatment) |
| conf / IoU / imgsz / classes | 0.25 / 0.70 / 960 / [2, 5, 7] (car, bus, truck) |
| device | 0 (GPU) |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.813 | 0.802 | 0.825 | 0.873 | 0.968 | 0.934 | 0.860 | 35 | 3 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.764 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.739 | 0.683 | 0.801 | 0.866 | 0.887 | 0.770 | 0.852 | 85 | 52 | 2 |

Cổng qua bài của model: ByteTrack **chưa đạt** (MOTA 0.749 < 0.75, sát ngưỡng); ReID **đạt cả ba** (IDF1 0.900, MOTA 0.792, MOTP 0.860).

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

Với nhãn tay của tôi, MOTA (0.934) **thấp hơn** IDF1 (0.968), và cả hai đều rất cao. `IDSW = 0` — không có lần nào tôi đổi ID giữa chừng, nên phần "giữ đúng identity" gần như hoàn hảo. Khoảng cách nhỏ giữa MOTA và IDF1 đến từ 35 FP (chủ yếu là ghost box ở track 6, frame 81–100 — tôi vẽ box sớm hơn khoảng 20 frame so với lúc track 6 tham chiếu thật sự xuất hiện) và 3 FN. MOTA tính điểm phạt tuyến tính theo từng box FP/FN ở từng frame, nên bị trừ trực tiếp bởi 35+3 lỗi đó; còn IDF1 nhìn theo cả track chứ không theo từng frame, nên một đoạn ghost box liên tục (cùng track 6, cùng kiểu lỗi) chỉ bị tính là một phần lệch của một track, không bị nhân lên theo số frame — đây là lý do IDF1 "khoan dung" hơn với lỗi theo cụm liên tiếp, còn MOTA nhạy hơn với số lượng box sai bất kể chúng có liên tục hay không.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

IDF1 tăng từ 0.875 (ByteTrack) lên 0.900 (ReID), AssA tăng từ 0.776 lên 0.820. Số IDSW bằng nhau (2 lần cả hai bên) nhưng xảy ra ở vị trí khác nhau — ByteTrack đổi ID ở frame 59 (track 4: 14→15) và frame 94 (track 5: 23→32); ReID đổi ID ở frame 87 (track 5: 17→18) và frame 113 (track 6: 24→31). Rõ nhất là **track 4**: ByteTrack làm gãy track 4 (đổi ID tại frame 59), trong khi log lỗi của ReID **không hề nhắc đến track 4** — không nằm trong danh sách fragmented/partially-covered/id-switch nào của ReID, tức là ReID giữ track 4 liền một ID suốt clip. Đây là một ví dụ cụ thể treatment tốt hơn control, nhiều khả năng nhờ cue appearance giúp associate đúng qua đoạn khiến ByteTrack (chỉ dựa motion+IoU) bị nhầm ở frame 59. Tuy vậy cả hai tracker vẫn cùng gặp khó ở track 6 (ByteTrack phủ 75%, ReID gãy track 6 thành 2 đoạn) — nên không phải ReID thắng tuyệt đối ở mọi track, và vì ByteTrack với BoT-SORT+ReID là hai cài đặt tracker khác nhau (không chỉ khác mỗi phần ReID), nên không thể kết luận chênh lệch này hoàn toàn do cue appearance.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

DetA tăng từ 0.649 (ByteTrack) lên 0.711 (ReID) dù cả hai dùng cùng detector/cấu hình (`conf=0.25, iou=0.70, imgsz=960, classes=[2,5,7]`) — chênh lệch này đến từ số box được giữ lại khác nhau giữa hai tracker (ByteTrack: 607 PRED_boxes, ReID: 638), tức là tracker quản lý vòng đời track khác nhau chứ detector thô không đổi. FN giảm mạnh từ 54 (ByteTrack) xuống 26 (ReID) — gần một nửa — trong khi FP gần như không đổi (88 → 91). AssA cũng tăng nhiều hơn DetA theo tỷ lệ tương đối. Ba dấu hiệu này cùng chỉ về một hướng: cải thiện chủ yếu nằm ở **association**, không phải detector — cùng một tập candidate detection ở mỗi frame, nhưng cách ghép nối track của treatment giữ lại được nhiều box hợp lệ hơn thành track liên tục thay vì để rơi rớt thành FN.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Track 5 (gold): nhãn tay của tôi không xuất hiện trong bất kỳ danh sách lỗi nào của `eval_vs_gold.json` (không ghost, không gãy track, không loose box) — tức khớp tốt với gold suốt track 5. Ngược lại, ReID đổi ID tại **frame 87** (track dự đoán 17 → 18) và bị liệt vào `fragmented_gt_tracks` cho track 5 (gãy thành 2 đoạn 51/1 frame). Vậy ở đoạn quanh frame 87, nhãn tay giữ đúng một ID cho track 5 trong khi ReID lại tách nhầm thành hai ID. *(Ghi chú: tôi suy ra điều này từ log JSON, chưa xem lại hình ảnh frame 87 trực tiếp — bạn nên mở CVAT/hình frame 87 để xác nhận trực quan trước khi đưa vào báo cáo chính thức.)*

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

Track 6, quanh **frame 81–100**, là điểm cả ba phép chấm đều vấp: nhãn tay của tôi bị gắn cờ ghost box sớm khoảng 20 frame trước khi track 6 tham chiếu (gold) thật sự xuất hiện; ByteTrack chỉ phủ được 75% chiều dài track 6; ReID còn gãy track 6 thành hai đoạn (43/56 frame, id switch ở frame 113). Việc cả người (tôi) lẫn cả hai model đều lúng túng ở cùng một track, cùng một vùng frame, là bằng chứng khá chắc rằng đoạn xe này có tình huống mơ hồ thật (có thể là xe vào khung chậm/mờ dần, hoặc bị che một phần lúc mới xuất hiện) chứ không phải chỉ là lỗi ngẫu nhiên của mỗi bên. *(Nên mở lại frame 81–100 trong CVAT để mô tả cụ thể tình huống — có thể dùng luôn làm một trong ba ca mơ hồ ở mục 1/`GUIDELINE_MINI.md`.)*

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

Dựa trên bằng chứng ở track 6 (mục 5): nên bổ sung một luật rõ ràng trong `GUIDELINE_MINI.md` mục 3 về "xe vừa xuất hiện, còn rất nhỏ/rất mờ" — quy định cụ thể xe phải chiếm bao nhiêu % kích thước nhận dạng được hoặc rõ nét đến mức nào thì mới bắt đầu vẽ track, để tránh tình trạng bắt đầu track sớm/muộn hơn vài chục frame so với người khác (đúng thứ đang gây lệch ở track 6 trong bài này).

Về quy trình làm việc, tôi sẽ dành riêng một lượt tua chỉ để kiểm các track có box khởi đầu rất nhỏ (dưới khoảng 20px) thay vì gộp chung vào lượt kiểm frame đầu/cuối — vì đây đúng là chỗ duy nhất annotation của tôi lệch so với gold trong bài này (track 6), nên tách riêng ra kiểm kỹ hơn sẽ hiệu quả hơn cho 10 clip tiếp theo.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [ ] `GUIDELINE_MINI.md` đã điền — **còn thiếu, cần bạn tự điền phần trải nghiệm thật**
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md` — **không làm (theo lựa chọn của bạn); có thể bị trừ điểm hạng mục "Kiểm chéo" trong RUBRIC.md**
- [x] `reports/REPORT.md` (file này — mục 1 và phần cuối mục 6 vẫn cần bạn tự điền trải nghiệm thật)
