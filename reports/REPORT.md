# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: `Nguyễn Văn Hân — 2A202602339`
Ngày: `15/09/2026`

> Báo cáo này dùng artifact và metric đã chạy trong repo. Thời gian CVAT không được ghi tự động nên được đánh dấu trung thực là không ghi nhận. Review trong repo là tự review annotation, AI-assisted, thực hiện hậu gold; không phải peer review bởi một học viên khác.

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT export MOT 1.1 |
| Thời gian gán `clip_02` (warm-up) | Không ghi nhận |
| Thời gian gán `clip_01` | Không ghi nhận |
| Số track đã vẽ trong `clip_01` | 8 |
| Số keyframe trung bình mỗi track | Không ghi nhận từ export MOT |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. `ID 5, frame 65–78`: kiểm lại entry point để không tạo bbox khi xe chưa rõ.
2. `ID 6, frame 79–100`: không bắt đầu track sớm hơn evidence nhìn thấy.
3. `ID 5, frame 81/91/97/98`: thêm keyframe để giảm interpolation drift.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: Không có log thao tác tua bằng mắt; audit tự động cho IDSW 0 và IDF1 0.951.
- Lượt 2: Audit tự động ghi endpoint thừa của ID 4, 5, 6, 8 để rà lại trong CVAT.
- Lượt 3: Audit tự động phát hiện track 5 có IoU 0.55–0.59 ở frame 81, 91, 97, 98; cần keyframe dày hơn.

Tự review: `Nguyễn Văn Hân — AI-assisted`. Chi tiết ở `reports/review_partner.md`.
Số finding kỹ thuật: `4`. Không có peer reviewer bên ngoài.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`Không có peer pair bên ngoài. Guideline đã bổ sung rule entry/exit, occlusion và keyframe.`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `57d5401e96c59fa7d73c627bc5d39414e55d6a423772429b06813e7f94741000` |
| Thời điểm khóa | `2026-09-15T10:04:48.441932+00:00` |
| Số row / frame / track trước khi mở reference | `237 / 60 / 7` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.002 | 0.001 | 0.005 | 0.905 | 0.000 | -0.414 | 0.000 | 237 | 573 | 0 |
| Sau rework | 0.734 | 0.715 | 0.758 | 0.818 | 0.951 | 0.897 | 0.791 | 57 | 2 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Coverage | 61–190 | toàn clip | Export cuối phủ đủ 190 frame, 8 track, 628 bbox. |
| Endpoint | 65–78 | 5 | Audit hậu gold đánh dấu rà lại frame bắt đầu. |
| Endpoint | 79–100 | 6 | Audit hậu gold đánh dấu rà lại frame bắt đầu. |
| Geometry | 81, 91, 97, 98 | 5 | Audit hậu gold đề xuất thêm keyframe. |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | 3.12.13 / 8.4.145 / 2.14.0+cpu / 0.5.13 |
| weights / hai tracker | `yolo26n.pt` / `bytetrack.yaml` và `configs/trackers/botsort-reid.yaml` |
| conf / IoU / imgsz / classes | 0.25 / 0.70 / 960 / car, bus, truck (2, 5, 7) |
| device | CPU |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.734 | 0.715 | 0.758 | 0.818 | 0.951 | 0.897 | 0.791 | 57 | 2 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.682 | 0.622 | 0.753 | 0.834 | 0.866 | 0.733 | 0.810 | 88 | 78 | 2 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

`MOTA 0.897 thấp hơn IDF1 0.951. Bản này không có trường hợp MOTA cao/IDF1 thấp; cả detection và identity đều đạt. Nếu MOTA cao nhưng IDF1 thấp, coverage/geometry vẫn tốt nhưng identity bị đổi/tách; MOTA chủ yếu tổng hợp FP, FN và ID switch nên không phản ánh mọi lỗi gán identity mạnh bằng IDF1.`

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

`ReID tăng IDF1 từ 0.875 lên 0.900 và AssA từ 0.776 lên 0.820; IDSW vẫn là 2. Tại đoạn track 5 quanh frame 87, ReID đổi ID 17 sang 18, còn ByteTrack đổi track 5 ở frame 94 (ID 23 sang 32); treatment tốt hơn tổng thể nhưng không loại hết fragmentation. Đây là system comparison, không cô lập causal effect của ReID vì hai tracker implementation khác.`

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

`Từ ByteTrack sang ReID, DetA tăng 0.649 lên 0.711 và FN giảm 54 xuống 26, đổi lại FP tăng 88 lên 91. Vẫn còn lỗi detector/coverage (FN, FP), đồng thời còn association error vì cả hai có 2 ID switch; không thể quy toàn bộ thay đổi cho ReID.`

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

`ReID track 7 tồn tại frame 16–116 nhưng không khớp track reference hay annotation; dưới schema chỉ nhận xe bốn bánh, đây là false-positive candidate cần soi bằng overlay trước khi khẳng định loại vật thể.`

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

`ReID tách gold track 6 ở frame 113 (ID 24 sang 31) và chỉ phủ 44/56 frame. Evidence cho thấy model vẫn lỗi association/coverage ở đoạn này; annotation tay không nên bị thay thế theo output model.`

## 6. Nếu phải gán thêm 10 clip nữa

`Giữ rule 25 frame cho occlusion, kiểm entry/exit trước khi export, và thêm keyframe ở vùng đổi kích thước/hướng. Chạy validator, visualization và review trước pre-gold lock; model chỉ dùng để chẩn đoán sau gold.`

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền (AI-assisted draft hậu gold)
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [x] `reports/review_partner.md` (tự review annotation, AI-assisted)
- [x] `reports/REPORT.md` (file này)
