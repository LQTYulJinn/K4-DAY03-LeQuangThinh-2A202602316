# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Lê Quang Thịnh (làm cá nhân)`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm (nếu có): `Không có — làm cá nhân, theo đúng schema đề bài.`

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | Trong bài nộp thật, cả 8 track (`track_id` 1–8) đều **liên tục, không có track nào bị đứt đoạn frame** (kiểm bằng script), tức mọi lần bị che trong clip này đều ngắn hơn ngưỡng nên không cần tách ID. |
| Xe bị che lâu hơn ngưỡng trên | tách thành track mới, không cố giữ ID cũ | Che quá lâu thì tôi không còn đủ tin cậy để khẳng định lúc xe hiện lại vẫn là đúng chiếc xe đó — thà tách ID còn hơn gán nhầm, vì gán nhầm mới là lỗi bị chấm nặng (AssA/IDF1 chiếm 25 điểm trong rubric). |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | Xe đã rời hẳn khung hình rồi quay lại thì tôi không có căn cứ gì để biết chắc đó là cùng một chiếc xe cũ hay một chiếc xe khác trông giống — tách track mới an toàn hơn là đoán bừa. |
| Hai xe cắt nhau / chồng lên nhau | giữ nguyên ID của từng xe theo vị trí/hướng di chuyển trước đó, không hoán đổi ID khi hai box chồng lên nhau | Minh chứng: `track 1` (xe đậu, bbox đứng yên tuyệt đối từ frame 1–190) và `track 7` (xe chạy tiến gần, lớn dần) chồng lấn IoU tới 0.287 ở frame 186–190 — nhãn của bạn **giữ nguyên cả hai ID** trong suốt đoạn này (xem Ca 3 ở mục 4). |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track ngay khi nhận ra được là xe bốn bánh, dù box còn rất nhỏ; ngưỡng quan sát được trong bài nộp: nhỏ nhất là **10×30 px** (`track 7`, frame 104) và **16.5×37.6 px** (`track 6`, frame 81) trên khung 960×540 |
| Xe đang đỗ, không di chuyển | vẫn gán track suốt thời gian xuất hiện; giữ nguyên tọa độ nếu xe thật sự không dịch chuyển pixel nào (minh chứng: `track 1` giữ đúng một tọa độ `(210.27, 249.67, 94.59, 44.14)` suốt 190 frame) |
| Keyframe đặt dày ở đâu | đặt dày hơn ở đoạn xe đổi hướng, tăng/giảm tốc rõ rệt, hoặc lúc bắt đầu/kết thúc bị che — những chỗ interpolation tuyến tính dễ vẽ sai quỹ đạo thật của xe nhất |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

> Ba ca dưới đây tôi tìm và xác minh trực tiếp từ `annotations/clip_01/gt.txt` và ảnh frame gốc (frame size, tọa độ, độ chồng lấn IoU đều là số thật, đã tính toán) — nhưng phần "Quyết định" suy luận từ dữ liệu, còn "Lý do" thật sự trong đầu bạn lúc đó thì chỉ bạn nhớ được. Đọc và sửa lại cho đúng ý bạn trước khi nộp.

### Ca 1
- Clip / frame / ID: `clip_01 / frame 81 / track 6`
- Tình huống: Xe xuất hiện lần đầu ở rìa phải khung hình, bbox chỉ **16.5×37.6 px** (rất nhỏ, xa camera). Theo `outputs/eval_vs_gold.json`, track 6 của bạn bị gắn cờ "ghost box" — bắt đầu sớm hơn khoảng 20 frame so với track tham chiếu (gold) bắt đầu tính.
- Quyết định: Bắt đầu vẽ track ngay khi nhận ra được là một xe bốn bánh, dù box rất nhỏ — sớm hơn mốc mà gold coi là "đủ rõ".
- Lý do: Quy định của đề nói "mọi xe bốn bánh nhìn thấy được đều phải có bbox", nên khi tôi đã khẳng định được đó là một xe bốn bánh (dù còn nhỏ) là tôi bắt đầu track ngay, không đợi xe lớn hơn — tôi sợ nếu chờ thì dễ quên mất đoạn đầu, còn nếu sai thì có thể sửa lại sau khi tua kiểm.

### Ca 2
- Clip / frame / ID: `clip_01 / frame 104 / track 7`
- Tình huống: Xe vừa xuất hiện ở rìa phải, bbox **10.0×29.9 px** — nhỏ nhất trong cả 8 track của clip. Bốn frame sau (105–108) box lớn dần đều (13.5→26 px chiều rộng) khi xe tiến lại gần.
- Quyết định: Vẫn bắt đầu track ngay từ frame vật thể còn cực nhỏ, không đợi đến khi box đủ lớn mới gán.
- Lý do: Cùng nguyên tắc như Ca 1 — tôi ưu tiên "gán sớm, sửa sau" hơn là bỏ sót, vì bỏ sót một đoạn đầu track sẽ làm hụt hẳn một phần chiều dài track đó, còn nếu gán hơi sớm thì chỉ lệch nhẹ vài chục frame, mức thiệt hại nhỏ hơn nhiều so với thiếu hẳn một đoạn.

### Ca 3
- Clip / frame / ID: `clip_01 / frame 186–190 / track 1 (xe đậu) và track 7 (xe đang chạy)`
- Tình huống: `track 1` là xe đậu, bbox giữ nguyên y hệt `(210.27, 249.67, 94.59, 44.14)` suốt 190 frame. Cuối clip, `track 7` (đã lớn dần khi tiến gần, box ~143.8×75.3 px) đi ngang/che một phần vị trí xe đậu, hai box chồng lấn tới **IoU 0.287** ở frame 190.
- Quyết định: Giữ nguyên ID và tọa độ của `track 1` không đổi, đồng thời tiếp tục `track 7` theo đúng ID cũ của nó — không để hai ID bị hoán đổi hay gộp lại dù box chồng nhau nhiều.
- Lý do: Tôi theo dõi track 1 từ đầu clip nên đã biết chắc đó là xe đậu, không di chuyển; còn track 7 tôi theo dõi liên tục thấy nó đang tiến gần dần theo một hướng rõ ràng. Vì đã bám theo cả hai xe qua nhiều frame trước đó, lúc hai box chồng lên nhau tôi vẫn phân biệt được nhờ biết trước "xe nào đang đứng yên, xe nào đang chạy" — không chỉ nhìn một frame đơn lẻ rồi đoán.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Nên bổ sung một ngưỡng kích thước tối thiểu rõ ràng (vd. bề rộng ≥ 10–15 px) cho "khi nào đủ nhỏ để bắt đầu track" — đây là lỗi duy nhất trong `eval_vs_gold.json` (ghost box ở track 6, frame 81–100), cho thấy ngưỡng "đủ rõ để bắt đầu" của bạn và của gold không khớp nhau dù không được viết thành luật cụ thể trước đó.
- `[bạn tự điền thêm nếu có, đặc biệt sau khi làm peer review]`
