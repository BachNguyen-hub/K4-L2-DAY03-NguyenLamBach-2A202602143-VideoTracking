# Mini annotation guideline — Ngày 3 (tracking)

Nhóm / tên: `Nguyễn Lâm Bách`
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
| xe đang đỗ nhưng còn nhìn thấy | đốm mờ, bóng hoặc vật thể chưa thể xác định là xe bốn bánh |

Bổ sung của nhóm: chỉ mở track khi xác định chắc chắn vật thể là xe bốn bánh và
có thể đặt bbox ổn định. Kiểm tra frame hiện tại cùng các frame lân cận; không suy
đoán class từ một đốm mờ hoặc từ output của tracker. Mọi xe hợp lệ dùng chung
nhãn `vehicle`.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | Nếu xe còn nhìn thấy một phần, tiếp tục gán bbox và giữ ID. Nếu mất hoàn toàn **không quá 25 frame**, dùng `Outside` trong đoạn vắng mặt rồi nối lại cùng ID khi quỹ đạo/hình dáng vẫn phù hợp | Che ngắn không kết thúc vòng đời của xe; giữ ID tránh phân mảnh track |
| Xe bị che lâu hơn ngưỡng trên | Nếu mất hoàn toàn **quá 25 frame**, mở track mới khi xe xuất hiện lại | Sau khoảng mất dấu dài, bằng chứng nối identity không còn đủ chắc chắn theo mặc định lab |
| Xe rời khung hình rồi quay lại | Tạo **track mới**, kể cả khi hình dáng giống xe cũ | Đi qua rìa ảnh là kết thúc track; không suy đoán identity ngoài khung |
| Hai xe cắt nhau / chồng lên nhau | Giữ hai ID riêng và mỗi xe một bbox cho phần còn nhìn thấy; đối chiếu hướng đi, vị trí và hình dáng trước/sau giao cắt; đặt keyframe sát hai phía và kiểm thêm 3–5 frame sau khi tách | Tránh ID switch do chọn xe gần nhất tại đúng một frame |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | Bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | Bbox ôm phần **nhìn thấy được**; bật `Occluded` khi phù hợp và giữ ID theo luật ở mục 2 |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | Bắt đầu ở frame đầu tiên có thể phân biệt chắc chắn xe bốn bánh với xe máy/nhiễu và đặt bbox ổn định; xác nhận bằng frame lân cận, không nội suy ngược về đoạn chỉ có đốm mờ |
| Xe đang đỗ, không di chuyển | Vẫn gán `vehicle` và giữ cùng ID trong toàn bộ quãng xe còn nhìn thấy |
| Frame đầu và frame cuối | Mở track ở frame đầu tiên xác nhận được xe; bbox cuối ở frame cuối xe còn nhìn thấy và đặt `Outside` tại frame kế tiếp. Không giữ bbox trước khi xe xuất hiện hoặc sau khi xe đã rời |
| Keyframe đặt dày ở đâu | Tại entry/exit; ngay trước/sau che khuất hoặc giao cắt; khi xe đổi hướng, tốc độ hoặc kích thước; và ở đoạn nội suy bị trôi. Luôn kiểm frame giữa hai keyframe xa nhau; riêng clip này phải xem lại frame 54 và 79 |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Các quyết định dưới đây là **kết luận cần áp dụng khi rework sau khi đã mở lại
frame để xác nhận**, không phải bằng chứng rằng file MOT hiện tại đã được sửa.

### Ca 1

- Clip / frame / ID: `clip_01`, frame `1–90`, ID nhãn tay `3`
- Tình huống: Track kéo dài 90 frame nhưng không khớp track gold nào; ReID cũng
  không khớp toàn bộ track này.
- Quyết định: Xem lại cả sequence để xác định class. Nếu không phải xe bốn bánh
  thật thì loại ID 3; không giữ track chỉ vì nó kéo dài nhiều frame.
- Lý do: Đây là ghost track lớn nhất và là ứng viên chính gây FP; model không
  phải ground truth nên vẫn phải xác nhận bằng hình ảnh.

### Ca 2

- Clip / frame / ID: `clip_01`, frame `76–79`, ID nhãn tay `6` / ID gold `5`
- Tình huống: ID 6 có bbox ở frame 76–78 trước khi track gold xuất hiện; bbox tại
  frame 79 còn lỏng, IoU 0.577.
- Quyết định: Nếu video xác nhận chưa thấy xe ở 76–78 thì dời frame đầu về 79;
  chỉnh bbox hoặc thêm keyframe tại 79.
- Lý do: Không nội suy về trước thời điểm có bằng chứng nhìn thấy, đồng thời bbox
  ở frame đầu phải ôm sát phần xe hiện hữu.

### Ca 3

- Clip / frame / ID: `clip_01`, frame `94–101`, ID nhãn tay `7` / ID gold `6`
- Tình huống: ID 7 bắt đầu sớm 7 frame; ở đầu ra ReID, identity tương ứng còn bị
  tách từ pred ID 24 sang 31 tại frame 113.
- Quyết định: Nếu video xác nhận xe chỉ đủ rõ từ frame 101 thì bỏ bbox 94–100,
  nhưng vẫn giữ một ID liên tục qua frame 113 nếu cùng một xe.
- Lý do: Frame bắt đầu và tính liên tục identity là hai quyết định độc lập; sửa
  điểm bắt đầu không có nghĩa phải tách ID giữa track.

### Ca 4

- Clip / frame / ID: `clip_01`, frame `149–151`, ID nhãn tay `5` / ID gold `4`
- Tình huống: Bbox còn tồn tại 3 frame sau khi xe tham chiếu đã rời khung; bbox
  của cùng ID tại frame 54 cũng lỏng, IoU 0.559.
- Quyết định: Nếu chuỗi ảnh xác nhận xe đã rời, kết thúc tại frame cuối còn nhìn
  thấy và đặt `Outside` từ frame kế tiếp; thêm/chỉnh keyframe tại frame 54.
- Lý do: Bbox treo làm tăng FP, còn keyframe thiếu khiến bbox nội suy bị trôi.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

- Bản nhãn hiện đạt cổng so với gold (`IDF1 = 0.9102`, `MOTA = 0.8028`,
  `MOTP = 0.8473`) và không có ID switch, fragmentation hay FN. Vấn đề chính là
  113 FP: 686 bbox/9 track so với 573 bbox/8 track của gold.
- Đã bổ sung vào guideline quy tắc xác nhận đúng class trước khi mở track, dựa
  trên finding ID 3 ở frame 1–90.
- Đã bổ sung quy tắc frame đầu/cuối và `Outside`, dựa trên ID 6 ở frame 76–78,
  ID 7 ở frame 94–100 và ID 5 ở frame 149–151.
- Đã bổ sung quy tắc đặt keyframe ở đoạn bbox trôi, dựa trên frame 54 và 79.
- Không sao chép nhãn model vào annotation. BoT-SORT + ReID đạt cổng khi so với
  gold nhưng không đạt khi so với nhãn tay; mọi bất đồng vẫn phải quyết định bằng
  video, guideline và teaching reference.
- **Chưa có dữ liệu kiểm chéo** vì `reports/review_partner.md` chưa tồn tại. Tên
  reviewer, finding hai chiều và quyết định adjudication phải được người tham gia
  bổ sung sau khi review thật.
- Hash annotation hiện vẫn trùng pre-gold và metric trước/sau giống hệt nhau; vì
  vậy các chỉnh sửa bbox/track nêu ở mục 4 vẫn là việc cần rework trong CVAT,
  không được coi là đã hoàn tất chỉ vì guideline đã cập nhật.
