# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe tự hành dùng động cơ (xăng, dầu, điện, hybrid...) để di chuyển chủ yếu trên đường bộ: sedan, hatchback, wagon, SUV/crossover, taxi, xe bán tải cabin nhỏ | `drive_012` #9 crossover trắng, `drive_019` #0 sedan bạc: thân liền khối, gầm thấp, không có khoang hộp cao |
| `truck` | Loại phương tiện cơ giới được thiết kế chuyên dụng để vận chuyển hàng hóa, vật liệu hoặc thực hiện các nhiệm vụ đặc thù: xe tải thùng, tải ben, đầu kéo, xe kéo (tow truck), xe chuyên dụng | `drive_005` #9 tải thùng trắng; `drive_019` #1 xe kéo: cabin tách khỏi thùng hàng/sàn thiết bị, khung gầm tải |
| `bus` | Xe buýt, xe khách, minibus chở khách thân dài, nhiều cửa sổ và cửa lên xuống, thường có biển hiệu tuyến | `drive_019` #15/#16/#17 (buýt trắng-xanh); `drive_040` xe buýt khớp nối (articulated) |
| `van` | Xe van/minivan/MPV/panel van/microvan: hộp cao liền khối, cửa trượt hoặc 2 cánh sau, chở người hoặc hàng nhẹ | `drive_002` #7 panel van trắng; `drive_008` #9 MPV trắng; `drive_019` #14 minivan |

Phân biệt nhanh: `bus` = thân dài, nhiều cửa sổ + cửa lên xuống; `van` = hộp cao cỡ xe con, một khoang liền; `truck` = có thùng hàng/sàn thiết bị tách khỏi cabin; `car` = thân liền, gầm thấp, không khoang hộp cao.

## Quy tắc chung

- Box bám sát vật thể nghĩa là:Box ôm sát phần thân xe nhìn thấy được, không lấy thêm mặt đường, trời hay xe bên cạnh.
- Vật thể bị che khuất: vẫn gán box quanh phần nhìn thấy được nếu nhận diện được lớp.
- Vật thể bị cắt bởi rìa ảnh: khi vật thể bị cắt khung; không đoán phần nằm ngoài ảnh.
- Vật thể rất nhỏ/mơ hồ:  không tự đoán. Ghi vào mini guideline và hỏi giảng viên.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_019` |  sedan bạc chạy trên đường ướt, ảnh hơi mờ dễ nhầm với van | `car` | Thân liền, gầm thấp, có cốp sau; không có khoang hộp cao như van |
| `drive_005` | xe màu trắng, cabin trước và phần sau liền nhau, khó phân biệt van chở hàng hay tải thùng | `truck` | Cabin tách khỏi thùng hàng kín phía sau, khung gầm tải; van phải là một khoang liền |
| `drive_002` |  xe trắng sát rìa phải, bị cắt một phần, hình dạng hộp cao | `van` | Hộp cao kín, cửa trượt bên hông, đuôi vuông; không có thùng hàng rời |
| `drive_012` |  xe trắng gầm cao, dễ bị gán van | `car` | Crossover/SUV thuộc `car`: thân liền, gầm cao hơn sedan nhưng không có khoang hộp cao của van |
| `drive_019` |  xe kéo màu trắng, có cần kéo và sàn thiết bị | `truck` | Xe chuyên dụng: cabin riêng + sàn thiết bị/cần kéo, khung gầm tải |
| `drive_001` | Cụm xe nằm trong bãi đỗ/bên kia dải phân cách, không chạy trên làn đang quan sát | vẫn gán box | Xe đỗ vẫn thuộc schema; box chỉ ôm phần nhìn thấy, không gộp cả cụm |
| nhiều ảnh | Xe rất xa, cạnh ngắn dưới ~20 px, không phân biệt được van/car/truck | bỏ, không gán | Không tự đoán khi không đủ dữ kiện xác định class |

