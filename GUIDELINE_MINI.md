# Mini annotation guideline — Ngày 3 (tracking)

> **AI-assisted draft, cập nhật từ audit hậu gold ngày 15/09/2026.** Không coi đây là bằng chứng được ghi trước pre-gold lock.

Nhóm / tên: `Nguyễn Văn Hân — 2A202602339`
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

Bổ sung: chỉ bắt đầu bbox khi có đủ dấu hiệu là xe bốn bánh; không dùng bảng hiệu, bóng hoặc phần xe suy đoán ngoài khung.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che dưới **25 frame** | cùng hướng/chuyển động và appearance liên tục. |
| Xe bị che lâu hơn ngưỡng trên | mở track mới nếu không đủ evidence để nối identity | tránh nối nhầm hai xe khác nhau. |
| Xe rời khung hình rồi quay lại | **track mới** | xe đã rời khung là kết thúc track. |
| Hai xe cắt nhau / chồng lên nhau | giữ ID theo vị trí trước che, hướng đi và appearance; thêm keyframe hai phía vùng overlap | tránh swap ID do interpolation. |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh. |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được**. |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu từ frame đầu tiên xác định được là xe bốn bánh; nếu chưa rõ, chờ thêm frame thay vì tạo bbox sớm. |
| Xe đang đỗ, không di chuyển | vẫn giữ track khi còn trong khung; kiểm endpoint bằng Outside. |
| Keyframe đặt dày ở đâu | tại entry/exit, che khuất, đổi hướng/tốc độ và nơi bbox có nguy cơ trôi. |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

### Ca 1
- Clip / frame / ID: `clip_01 / 65–78 / 5`
- Tình huống: bbox xuất hiện trước khi xe được reference xác định là track 5.
- Quyết định: chỉ bắt đầu track khi xe bốn bánh nhận diện rõ; rà lại entry point trong CVAT.
- Lý do: audit cho thấy đoạn đầu tạo false positive, làm tăng FP.

### Ca 2
- Clip / frame / ID: `clip_01 / 79–100 / 6`
- Tình huống: ID 6 bắt đầu sớm 22 frame so với reference.
- Quyết định: đặt lại frame đầu theo visibility thực tế, không nối vào vật thể chưa chắc là xe.
- Lý do: rule entry/exit phải ưu tiên evidence nhìn thấy hơn nội suy.

### Ca 3
- Clip / frame / ID: `clip_01 / 81, 91, 97, 98 / 5`
- Tình huống: bbox trôi ở giữa đoạn track; IoU audit là 0.55–0.59.
- Quyết định: thêm keyframe quanh đoạn thay đổi vị trí/kích thước.
- Lý do: interpolation thưa làm bbox không còn ôm sát phần xe nhìn thấy.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

- Rà entry/exit của ID 4, 5, 6 và 8; không giữ bbox trước/sau khoảng xe hiện diện.
- Thêm keyframe cho ID 5 quanh frame 81, 91, 97 và 98.
- Giữ nguyên rule scope một lớp `vehicle`; không dùng model output làm đáp án annotation.
