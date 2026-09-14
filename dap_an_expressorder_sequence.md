# THIẾT KẾ BIỂU ĐỒ TUẦN TỰ CHO PHÂN HỆ ĐẶT ĐƠN GIAO HỎA TỐC RIKKEI LOGISTICS EXPRESS

## Phần 1 — Xác định đối tượng và thông điệp

### Đối tượng tham gia

| Loại | Tên |
|---|---|
| Actor | Người gửi |
| Object | ExpressAppUI |
| Object | DispatchManager |
| Object | SMSThread |

### Danh sách thông điệp theo thứ tự thời gian

| # | Bước | Từ → Đến | Loại thông điệp | Giải thích |
|---|---|---|---|---|
| 1 | Bấm đặt đơn hỏa tốc | Người gửi → ExpressAppUI | **Sync** | Người gửi bấm nút và chờ ứng dụng xử lý xong mới có phản hồi |
| 2 | create_express_order() | ExpressAppUI → DispatchManager | **Sync** | ExpressAppUI cần chờ kết quả khóa vận đơn thì mới có mã vận đơn để hiển thị, nên đây bắt buộc là lời gọi đồng bộ |
| 3 | Gửi SMS cho người nhận | DispatchManager → SMSThread | **Async** | Gửi SMS có thể mất 8–10 giây và tuyệt đối không được chặn phản hồi cho Người gửi → bắt buộc dùng thông điệp Bất đồng bộ (gửi đi rồi tiếp tục ngay, không chờ SMSThread xử lý xong) |
| 4 | Khóa vận đơn thành công | DispatchManager → ExpressAppUI | **Return** | Kết quả trả về hoàn tất lời gọi Sync ở bước 2 |
| 5 | Hiển thị mã vận đơn | ExpressAppUI → Người gửi | **Return** | Kết quả trả về hoàn tất lời gọi Sync ở bước 1 |

### Lưu ý tránh bẫy chọn sai loại thông điệp

Bước 3 (gửi SMS) **phải** vẽ bằng thông điệp **Async** (đường liền nét, đầu mũi tên hở/không tô đặc). Nếu vẽ bằng Sync, DispatchManager sẽ bị buộc phải chờ SMSThread xử lý xong (8–10 giây) rồi mới được trả kết quả khóa vận đơn cho ExpressAppUI — trực tiếp vi phạm quy tắc *"Gửi SMS cho Người nhận tuyệt đối không được làm chậm phản hồi cho Người gửi"*.

## Phần 2 — Sơ đồ hoàn chỉnh

File đính kèm: `expressorder_sequence_daydu.drawio`

Điểm thiết kế then chốt trong sơ đồ:

- Thanh kích hoạt (activation bar) của **DispatchManager** kết thúc ngay sau khi gửi thông điệp Async (bước 3) và trả Return (bước 4) — **không kéo dài chờ** SMSThread.
- Thanh kích hoạt của **SMSThread** được vẽ **kéo dài hơn**, thể hiện việc gửi SMS vẫn đang tiếp tục xử lý ở hậu trường sau khi Người gửi đã nhận được mã vận đơn (bước 5 đã hoàn tất trước khi SMSThread xử lý xong).
- Ba loại mũi tên được phân biệt rõ theo đúng chuẩn UML:
  - **Sync**: đường liền nét, đầu mũi tên tô đặc (filled/block).
  - **Async**: đường liền nét, đầu mũi tên hở (open, không tô).
  - **Return**: đường nét đứt, đầu mũi tên hở (open).
