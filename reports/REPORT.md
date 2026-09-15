# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / MSSV: `Nguyễn Lâm Bách / 2A202602143`  
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT (theo workflow của bài lab; JSON không lưu phiên bản CVAT) |
| Thời gian gán `clip_02` (warm-up) |  |
| Thời gian gán `clip_01` |  |
| Số track đã vẽ trong `clip_01` | 9 track, 686 bbox (`eval_vs_gold.json`) |
| Số keyframe trung bình mỗi track | **Không thể suy ra từ MOT/JSON vì định dạng xuất không giữ metadata keyframe** |

Ba tình huống nổi bật mà chẩn đoán JSON yêu cầu xem lại:

1. ID 3 tồn tại từ frame 1–90 nhưng không khớp track gold nào. Cần xem lại chuỗi
   ảnh để xác nhận đây có thật sự là xe bốn bánh hay là đối tượng ngoài schema;
   không giữ một track dài chỉ vì đã lỡ mở track.
2. ID 6 được mở sớm ở frame 76–78 và ID 7 được mở sớm ở frame 94–100. Luật áp
   dụng là chỉ bắt đầu ở frame đầu tiên có thể xác định chắc chắn là xe bốn bánh,
   rồi kiểm tra các frame liền kề trước khi chốt điểm bắt đầu.
3. ID 5 còn bbox ở frame 149–151 sau khi xe tham chiếu đã rời khung. Cần đặt
   `Outside` tại frame đầu tiên xe vắng mặt và tua qua ranh giới vào/ra từng frame.

## 2. Tự kiểm và kiểm chéo

Kết quả hậu kiểm có thể tái hiện từ `jsons/eval_vs_gold.json`:

- Lượt 1 — ID: không có ID switch hoặc track bị tách so với gold, nhưng có 9
  track thay vì 8; ID 3 là track dư dài 90 frame.
- Lượt 2 — frame đầu/cuối: ID 6 sớm 3 frame, ID 7 sớm 7 frame và ID 5 muộn 3
  frame so với khoảng hiện diện của track gold tương ứng.
- Lượt 3 — frame giữa/bbox: bbox ID 5 tại frame 54 có IoU 0.559 và bbox ID 6 tại
  frame 79 có IoU 0.577; đây là hai vị trí cần thêm/chỉnh keyframe.

Kiểm chéo với: **chưa có dữ liệu; `reports/review_partner.md` chưa tồn tại**.  
Số lỗi hai bên tìm được: **chưa có dữ liệu — cần người tham gia kiểm chéo bổ sung**.

Ca khác biệt cần được ghi thành luật trong `GUIDELINE_MINI.md` là: khi nào một
đối tượng đủ chắc chắn để mở track; cách chốt frame đầu/frame cuối; và cách xử lý
đối tượng nghi ngoài schema. JSON chỉ chỉ ra vị trí cần xem, không thay thế việc
hai người xem frame và adjudicate.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `ff8a80d697b7384b30112e30a08092e1f550f2ac39caea056b57006f8dec60cd` |
| Thời điểm khóa | `2026-09-15T09:49:43.544175+00:00` (16:49:43 ngày 15/09/2026, UTC+7) |
| Số row / frame / track trước khi mở reference | 686 / 190 / 9 |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.7670 | 0.7021 | 0.8428 | 0.8633 | 0.9102 | 0.8028 | 0.8473 | 113 | 0 | 0 |
| Sau rework / bản hiện tại | 0.7670 | 0.7021 | 0.8428 | 0.8633 | 0.9102 | 0.8028 | 0.8473 | 113 | 0 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có** — cả ba
chỉ số lần lượt là 0.9102, 0.8028 và 0.8473.

Hash của `annotations/clip_01/gt.txt` hiện vẫn trùng hash pre-gold; metric và
diagnostics của `outputs/eval_pre_gold.json` cũng trùng hoàn toàn
`jsons/eval_vs_gold.json`. Vì vậy artifact chưa cho thấy đã có rework sau khi mở
gold. Các finding còn cần xác minh/sửa trong CVAT là:

| Loại lỗi | Frame | ID nhãn tay | Trạng thái / hành động cần làm |
| --- | --- | --- | --- |
| Track không khớp gold | 1–90 | 3 | Chưa có bằng chứng đã sửa; xem lại class trên chuỗi frame, loại track nếu ngoài schema |
| Bắt đầu quá sớm | 76–78 | 6 | Chưa có bằng chứng đã sửa; kiểm tra và dời frame đầu về lúc xe xác định được (gold bắt đầu ở 79) |
| Bắt đầu quá sớm | 94–100 | 7 | Chưa có bằng chứng đã sửa; kiểm tra và dời frame đầu (gold bắt đầu ở 101) |
| Kết thúc quá muộn | 149–151 | 5 | Chưa có bằng chứng đã sửa; đặt `Outside` ở frame đầu xe vắng mặt |
| Bbox lỏng | 54; 79 | 5; 6 | Chưa có bằng chứng đã sửa; chỉnh bbox/thêm keyframe quanh hai frame này |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `jsons/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | 3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13 |
| weights / hai tracker | `yolo26n.pt`; ByteTrack: `bytetrack.yaml`; ReID treatment: `/content/Day3-Lab/configs/trackers/botsort-reid.yaml` |
| conf / IoU / imgsz / classes | 0.25 / 0.70 / 960 / `[2, 5, 7]` |
| device / persist / số frame | `0` / `true` / 190 |

Tất cả phép chấm dùng IoU threshold 0.5; đây là ngưỡng matching khi đánh giá,
khác với IoU 0.70 trong cấu hình inference.

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bạn vs gold | 0.7670 | 0.7021 | 0.8428 | 0.8633 | 0.9102 | 0.8028 | 0.8473 | 113 | 0 | 0 |
| ByteTrack control vs gold | 0.7085 | 0.6487 | 0.7761 | 0.8463 | 0.8746 | 0.7487 | 0.8226 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.7635 | 0.7110 | 0.8204 | 0.8721 | 0.9001 | 0.7923 | 0.8595 | 91 | 26 | 2 |
| ReID vs bạn | 0.7147 | 0.6140 | 0.8395 | 0.8738 | 0.8278 | 0.6691 | 0.8606 | 89 | 137 | 1 |

Trạng thái cổng: **bạn vs gold đạt**, **ReID vs gold đạt**, **ByteTrack vs gold
chưa đạt** vì MOTA 0.7487 thấp hơn ngưỡng 0.75 khoảng 0.0013; **ReID vs bạn chưa
đạt** vì MOTA 0.6691.

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA của bản nhãn (0.8028) **thấp hơn** IDF1 (0.9102) 0.1074. Trong trường hợp
ngược lại, MOTA cao nhưng IDF1 thấp thường báo hiệu identity bị gãy/đổi trong khi
số detection tổng thể vẫn tốt. MOTA chỉ cộng mỗi ID switch như một lỗi rời rạc,
còn IDF1 đo tính nhất quán danh tính trên toàn bộ quãng đời track nên nhạy hơn với
một lần đổi ID kéo dài nhiều frame. Ở bản nhãn này, IDSW = 0 nhưng FP = 113, vì
vậy khoảng hụt của MOTA chủ yếu đến từ bbox/track thừa chứ không phải đổi ID.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

So với ByteTrack, treatment tăng IDF1 từ 0.8746 lên 0.9001 (**+0.0255**) và AssA
từ 0.7761 lên 0.8204 (**+0.0443**), nhưng IDSW vẫn là 2. Một sequence cụ thể là
gold track 4 quanh frame 57–59: ByteTrack đổi pred ID 14 → 15 tại frame 59 và bị
ghi nhận là fragmented, trong khi danh sách lỗi của treatment không còn track 4.
Ngược lại, treatment vẫn đổi ID ở frame 87 (gold track 5) và 113 (gold track 6),
nên không thể nói ReID đã loại bỏ ID switch nói chung. Đây là **so sánh hai hệ
thống** ByteTrack và BoT-SORT + ReID; do implementation tracker khác nhau, số liệu
không cô lập tác động nhân quả của riêng appearance/ReID.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

Treatment tăng DetA 0.6487 → 0.7110 (**+0.0623**), giảm FN 54 → 26 (**−28**),
đổi lại FP tăng 88 → 91 (**+3**). PRED_boxes cũng tăng 607 → 638. Lỗi còn lại có
cả hai loại: detector vẫn còn 91 FP, 26 FN và 5 ghost track; association vẫn còn
3 track bị phân mảnh cùng 2 ID switch. Theo số đếm detection, FP là phần lớn nhất;
theo identity, các sequence quanh frame 87 và 113 vẫn cần xem riêng.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Tại **frame 113**, treatment đổi identity của gold track 6 từ pred ID 24 sang 31.
Trong phép `ReID vs bạn`, track tương ứng là ID nhãn tay 7; còn `bạn vs gold` có
IDSW = 0 và không có track phân mảnh. Vì vậy evidence định lượng ủng hộ việc nhãn
tay giữ một ID liên tục ở sequence này, còn ReID tách/đổi identity. Cần mở các
frame lân cận để xác nhận bằng hình ảnh trước khi dùng đây làm kết luận cuối.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

ReID không khớp toàn bộ **ID nhãn tay 3 ở frame 1–90**. Đây không chỉ là bất đồng
với model: phép `bạn vs gold` cũng xếp ID 3 thành ghost track 90 frame không khớp
reference nào. Hai nguồn cùng chỉ về một finding nên sequence này phải được xem
lại đầu tiên để xác định có gán nhầm đối tượng ngoài schema hay không. Model tự
nó không phải ground truth; bằng chứng quyết định vẫn là hình ảnh và luật
`vehicle`.

## 6. Nếu phải gán thêm 10 clip nữa

Đề xuất rút ra trực tiếp từ findings hiện có:

- Bổ sung vào `GUIDELINE_MINI.md` tiêu chí mở track: phải nhận ra thân xe bốn bánh
  trên frame hiện tại và frame lân cận; không nội suy bbox về trước thời điểm đó.
- Quy định `Outside` tại frame đầu tiên xe hoàn toàn vắng mặt; kiểm từng frame ở
  hai đầu track để tránh lỗi như ID 5, 6 và 7.
- Đặt keyframe dày tại entry/exit, crossing, occlusion, đổi hướng/tốc độ/kích
  thước; kiểm riêng những đoạn IoU thấp.
- Quy trình cho mỗi clip: hoàn thành từng xe → kiểm class → kiểm ID → kiểm hai đầu
  track → kiểm midpoint/keyframe → chạy validator → xem visualization → peer
  review → khóa evidence. Thời gian thực tế và kết quả peer review cần được ghi
  ngay lúc làm, không thể khôi phục đáng tin cậy từ JSON sau đó.

## 7. Tệp đã nộp

Trạng thái dưới đây phản ánh cây thư mục tại thời điểm cập nhật báo cáo:

- [x] `annotations/clip_01/gt.txt`
- [ ] `annotations/clip_02/gt.txt` — chưa có
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền từ diagnostics; các quyết định cá nhân vẫn cần xác nhận bằng hình ảnh
- [x] `outputs/eval_vs_gold.json` — kết quả hiện nằm ở `jsons/eval_vs_gold.json`
- [ ] `outputs/model_bytetrack_clip_01.txt` — chưa có
- [ ] `outputs/model_reid_clip_01.txt` — chưa có
- [x] `outputs/model_run_config.json` — cấu hình hiện nằm ở `jsons/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json` — hiện nằm trong `jsons/`
- [ ] `reports/review_partner.md` — chưa có
- [x] `reports/REPORT.md` (file này)
