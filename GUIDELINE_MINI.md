# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: Bùi Thành Long
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

Bổ sung của nhóm (nếu có): xe buýt/xe tải có gương chiếu hậu to, trồi hẳn ra ngoài thân xe — gương được coi là **một phần của xe** (phần nhìn thấy được), vẽ chung vào cùng một bbox với thân xe, không tách riêng và không bỏ qua.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | dưới 2 giây đủ ngắn để coi là cùng một xe tạm mất dấu, không phải xe mới; giữ ID giúp track không bị vỡ vụn vì occlusion ngắn hạn |
| Xe bị che lâu hơn ngưỡng trên | mở **track mới** khi xe hiện lại | che quá 2 giây có nguy cơ nhầm với một xe khác đi vào đúng vị trí đó trong lúc mất dấu; giữ ID cũ lúc này rủi ro sai nhiều hơn lợi |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | ra khỏi khung là kết thúc quãng đời track (theo GUIDE.md); xe "quay lại" không còn chắc là cùng một xe hay một xe khác đi vào |
| Hai xe cắt nhau / chồng lên nhau | mỗi xe giữ nguyên ID đã có **trước khi cắt nhau**, dựa vào hướng di chuyển và vị trí tương đối để không hoán đổi ID giữa hai xe khi chúng tách ra | đây chính là tình huống dễ gây ID switch nhất (thấy trong track 5/6 khi chấm với gold) — bám theo motion trước đó đáng tin hơn là đoán lại từ đầu sau khi tách |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: phân biệt được rõ đây là xe bốn bánh (không phải xe máy/vật thể khác) dù ảnh còn mờ — không đợi đến khi xe đủ lớn/rõ nét |
| Xe đang đỗ, không di chuyển | vẫn coi là `vehicle`, track suốt thời gian xe còn trong khung; **cẩn thận phân biệt với rung camera**: xe đứng im mà camera rung nhẹ khiến bbox tưởng cần dịch chuyển theo — không dịch bbox theo rung camera nếu xe thực sự không đổi vị trí |
| Keyframe đặt dày ở đâu | dày hơn (mỗi 3–5 frame) khi: xe đi nhanh, đang rẽ/đổi hướng, bị che một phần bởi xe khác, hoặc camera rung; thưa hơn (mỗi 20–30 frame) khi xe đi đều, không bị che, không gần biên khung hình |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01` / frame `80`  / ID `5`
- Tình huống: ô tô con bị ô tô to đè gần hết, chỉ hiện dần từ phần đuôi xe qua nhiều frame liên tiếp
- Quyết định: chỉ vẽ bbox ôm đúng phần đuôi xe đang thực sự nhìn thấy được ở từng frame, không đoán/mở rộng sang phần còn bị xe to che khuất
- Lý do: đúng luật bbox "ôm phần nhìn thấy được" — vẽ lố sang phần bị che sẽ tạo bbox sai kích thước, kéo IoU xuống khi so với gold

### Ca 2
- Clip / frame / ID: `clip_01` / frame `80`  / ID `4`
- Tình huống: xe buýt có gương chiếu hậu to, trồi hẳn ra ngoài thân xe — không rõ có nên tính gương vào bbox hay không
- Quyết định: vẽ gương vào cùng một bbox với thân xe buýt, coi gương là phần nhìn thấy được của xe
- Lý do: gương là bộ phận vật lý gắn liền với xe, đang thực sự hiện ra trong ảnh (không phải phần bị che hay ngoài khung) nên phải tính vào "phần nhìn thấy được"

### Ca 3
- Clip / frame / ID: `clip_01` / frame `1` đến `190`  / ID `1`
- Tình huống: xe đứng yên nhưng camera rung/dịch chuyển nhẹ liên tục từng chút một, khiến việc căn bbox mất nhiều thời gian hơn dự kiến
- Quyết định: giữ bbox theo đúng vị trí thật của xe trong từng frame (xe không đổi vị trí thì bbox cũng không nên "chạy" theo rung camera), kiểm tra kỹ trước khi đặt keyframe mới
- Lý do: rung camera dễ đánh lừa thành "xe đang di chuyển", nếu chỉnh bbox chạy theo rung sẽ tạo track sai lệch dù xe thực tế đứng im

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- **Thời điểm bấm Outside chưa đủ chặt chẽ.** Kết quả chấm với gold cho thấy 5 lỗi "bbox treo/thừa" (ID 4, 6, 8) — bbox tồn tại trước khi xe xuất hiện hoặc sau khi xe đã rời khung vài frame. Luật mới: **trước khi bấm Outside, tua từng frame quanh biên (±3 frame) để xác định chính xác frame xe rời/vào khung, không ước lượng theo cảm giác.**
- **Mật độ keyframe chưa đủ dày ở đoạn khó.** Track 5 bị "bbox trôi" (IoU chỉ 0.50–0.56) suốt frame 84–92 — một đoạn dài không có keyframe mới dù có thể xe đang bị che/cắt ngang. Luật mới: **khi track đi vào đoạn nghi occlusion/crossing, đặt keyframe mỗi 3–5 frame thay vì để nội suy dài, kiểm tra IoU cảm quan bằng mắt trước khi chuyển sang track khác.**
- **Thiếu bước tự kiểm phủ toàn bộ vòng đời track.** Track 6 chỉ phủ 79% số frame so với gold (44/56) — nghĩa là có đoạn giữa track bị bỏ trống dù xe vẫn còn trong khung. Luật mới: **sau khi vẽ xong một track, tua lại từ đầu đến Outside để xác nhận không có đoạn nào bị bỏ trống, trước khi chuyển sang xe tiếp theo.**
