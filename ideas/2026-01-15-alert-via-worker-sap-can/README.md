# Alert khi Via Worker sắp cạn

**Trạng thái:** considering
**Ngày tạo:** 2026-01-15
**Độ ưu tiên:** Cao

---

## Vấn đề

Hiện tại hệ thống có data về:
- Via nào đang bật "làm nhiệm vụ"
- Ads nào thuộc BM nào, Via nào

Nhưng **chưa có cảnh báo** khi:
- Số Via Worker còn lại quá ít
- Một số BM/Ads không còn Via nào có thể check

---

## Giải pháp đề xuất

### Alert Level 1: Warning
```
⚠️ Số Via Worker còn lại: 3
   Đề xuất: Chuẩn bị thêm Via mới
```
- Trigger khi số Via bật "làm nhiệm vụ" < ngưỡng (ví dụ: 5)

### Alert Level 2: Critical
```
🔴 BM "ABC Corp" không có Via nào có thể check!
   - Via gốc: Die
   - Via backup: Chưa bật nhiệm vụ
```
- Trigger khi có BM/Ads không thể check được

### Alert Level 3: Orphan Detection
```
🔴 50 Ads không có đường check:
   - Không thuộc BM trung gian nào
   - Hoặc BM trung gian không có Via Worker
```
- Phát hiện Ads "mồ côi" - không có Via nào reach được

---

## Nơi hiển thị

1. **Dashboard Web** - Widget cảnh báo
2. **Tool Master** - Notification
3. **Email/Telegram** - Alert nghiêm trọng

---

## Ghi chú

Anh Sơn: "Cái này tôi nghĩ rất quan trọng và nên ghi lại để sau này triển khai."

---

## Tasks khi triển khai

- [ ] Định nghĩa ngưỡng cảnh báo (configurable)
- [ ] Query đếm Via Worker còn hoạt động
- [ ] Query tìm BM/Ads không có Via check được
- [ ] UI hiển thị cảnh báo trên Web
- [ ] Notification trên Tool Master
- [ ] (Optional) Tích hợp alert external (email/telegram)

---

## Demo

Code demo sẽ được thêm vào thư mục `demo/`
