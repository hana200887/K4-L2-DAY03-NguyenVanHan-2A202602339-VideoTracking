# Tự review annotation — Day 3

> **Tự review, AI-assisted, thực hiện sau khi teaching reference được phát.** Nội dung là audit kỹ thuật dựa trên artifact hiện có; không phải peer review bởi một học viên khác và không được backdate thành review trước pre-gold.

| Trường | Giá trị |
| --- | --- |
| Author | Nguyễn Văn Hân — 2A202602339 |
| Reviewer | Nguyễn Văn Hân — self-review, AI-assisted |
| Pair ID | N/A — không có peer reviewer bên ngoài |
| CVAT version | Chưa ghi nhận |
| Thời điểm review | 15/09/2026, sau khi có gold |

## Danh sách finding

| # | CVAT frame | MOT frame | ID | Loại lỗi | Quan sát + rule áp dụng | Cách sửa đề xuất | Closure: fixed / not-a-defect / needs-review |
| ---: | ---: | ---: | --- | --- | --- | --- | --- |
| 1 | 64–77 | 65–78 | 5 | Endpoint bắt đầu sớm | Annotation có bbox trước khi track gold 5 xuất hiện; bbox chỉ bắt đầu khi xác định rõ xe bốn bánh. | Rà CVAT frame 65–78, đặt Outside/điểm bắt đầu đúng frame xe xuất hiện rõ. | needs-review |
| 2 | 78–99 | 79–100 | 6 | Endpoint bắt đầu sớm | ID 6 xuất hiện sớm 22 frame so với gold. | Rà visibility của xe và đặt frame đầu đúng theo guideline. | needs-review |
| 3 | 50–52, 148–150 | 51–53, 149–151 | 4 | Endpoint thừa | ID 4 có bbox trước khi xe xuất hiện và sau khi reference kết thúc. | Chỉnh keyframe đầu/cuối; dùng Outside khi xe thực sự rời khung. | needs-review |
| 4 | 80, 90, 96–97 | 81, 91, 97–98 | 5 | Interpolation drift | IoU ở các frame này chỉ 0.55–0.59; bbox cần khít phần xe nhìn thấy. | Thêm keyframe quanh đoạn xe thay đổi vị trí/kích thước. | needs-review |

## Reviewer checklist

| Hạng mục | PASS / FINDING / N/A | Frame–ID–evidence |
| --- | --- | --- |
| Có tối thiểu 6 track hợp lệ; chỉ gồm xe bốn bánh | PASS | 8 track, 628 bbox; validator 0 lỗi định dạng. |
| Một xe giữ một ID; không reuse ID cho xe khác | PASS | `eval_vs_gold.json`: IDSW 0, IDF1 0.951. |
| Occlusion ngắn giữ ID; crossing không đổi ID | N/A | IDSW 0 là tín hiệu tốt, nhưng audit không thay cho một lượt xem crossing bằng mắt. |
| Entry/exit đúng; không box treo sau khi xe rời khung | FINDING | ID 4, 5, 6, 8 có đoạn thừa theo `eval_vs_gold.json`. |
| Bbox ôm phần nhìn thấy, không đoán phần bị che/ngoài khung | FINDING | ID 5 có drift tại MOT frame 81, 91, 97, 98. |
| Frame giữa hai keyframe không bị interpolation drift | FINDING | Xem finding 4. |
| Export đúng MOT 1.1; frame bắt đầu từ 1; cột 2 là track ID | PASS | `check_mot_labels.py`: 0 lỗi, frame 1–190. |
| Mọi finding có cách sửa và closure do tác giả điền | NEEDS-REVIEW | Cần tác giả xác nhận/sửa bốn finding nếu muốn cải thiện thêm. |

## Self-QC attestation của reviewer

| Lượt | PASS / ĐÃ SỬA / NEEDS-REVIEW | Frame–ID–evidence |
| --- | --- | --- |
| 1 — identity/timeline | PASS | IDF1 0.951, IDSW 0. |
| 2 — endpoint/scope | NEEDS-REVIEW | ID 4, 5, 6, 8; chênh lệch đầu/cuối track. |
| 3 — geometry/interpolation | NEEDS-REVIEW | Track 5 tại frame 81, 91, 97, 98. |

## Exit ticket

1. Finding quan trọng nhất và rule dùng để kết luận: `ID 6 có bbox từ frame 79–100 trước khi reference xuất hiện; chỉ bắt đầu track khi xác định rõ xe bốn bánh.`
2. Một finding tác giả đóng là `not-a-defect`, kèm lý do (nếu có): `Chưa có.`
3. Một rule cần Lab Coach làm rõ (nếu có): `Không có; các finding hiện tại được xử lý bằng rule entry/exit và bbox phần nhìn thấy.`
