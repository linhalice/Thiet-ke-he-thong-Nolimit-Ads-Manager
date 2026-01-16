# Flow tạo lại BM Trung gian khi die

**Trạng thái:** open
**Độ ưu tiên:** high
**Ngày tạo:** 2026-01-15

---

## Vấn đề

Khi BM Trung gian bị die/khóa:
- Cần tạo BM trung gian mới
- Share lại tất cả Ads từ BM gốc vào BM mới
- **Hiện tại đang làm THỦ CÔNG** → Mất công, dễ sót

## Flow hiện tại (Thủ công)

```
BM Trung gian A die
       │
       ▼
[THỦ CÔNG] Tạo BM trung gian B mới
       │
       ▼
[THỦ CÔNG] Vào từng BM gốc, share lại Ads sang BM B
       │
       ▼
[THỦ CÔNG] Gán Via Worker cho BM B
       │
       ▼
[THỦ CÔNG] Cập nhật mapping trong DB (?)
```

### Vấn đề của flow thủ công
- **Mất thời gian** - Đặc biệt khi có nhiều BM gốc
- **Dễ sót Ads** - Quên share một số Ads
- **Downtime** - Trong lúc làm, Ads không được check
- **Không có log** - Ai làm? Khi nào? Share những gì?

## Hướng cải tiến (đề xuất)

### Option 1: Semi-auto với checklist
```
Khi BM Trung gian die:
1. Hệ thống tự detect (hoặc user báo)
2. Hệ thống list ra tất cả Ads đang bị ảnh hưởng
3. User tạo BM mới, nhập ID vào hệ thống
4. Hệ thống generate danh sách cần share (copy-paste được)
5. User share xong, confirm
6. Hệ thống cập nhật mapping
```

### Option 2: Full auto (phức tạp hơn)
```
Khi BM Trung gian die:
1. Hệ thống tự detect
2. Tự tạo BM mới (qua API? qua Selenium?)
3. Tự share Ads (qua automation)
4. Tự gán Via Worker
5. Tự cập nhật DB
```

### Đề xuất: Bắt đầu với Option 1
- Giảm công sức thủ công
- Có checklist để không sót
- Có log để trace
- Sau này nâng lên Option 2 nếu cần

## Câu hỏi cần làm rõ

- [ ] Tần suất BM trung gian die? (tuần/tháng?)
- [ ] Mỗi lần die, mất bao lâu để recovery?
- [ ] Có API Facebook nào hỗ trợ share Ads tự động không?

---

## Ghi chú

Anh Sơn: "Cái này đang được thực hiện thủ công. Khá mất công."
