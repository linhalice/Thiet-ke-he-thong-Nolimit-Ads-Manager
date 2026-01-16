# Decision Log

Ghi lại các quyết định quan trọng và lý do đằng sau.

> "Sau này khi ai đó hỏi 'sao lại làm thế này?', mở file này ra là có câu trả lời."

---

## Template

```
### [Ngày] Tiêu đề quyết định

**Bối cảnh:** Vấn đề gì đang gặp phải?

**Các lựa chọn:**
1. Option A - Ưu/nhược
2. Option B - Ưu/nhược

**Quyết định:** Chọn option nào?

**Lý do:** Tại sao chọn option đó?

**Hậu quả:** Điều gì sẽ thay đổi sau quyết định này?
```

---

## Decisions

### [2026-01-15] Chọn thư mục này làm "Trụ sở chính"

**Bối cảnh:** Cần một nơi để Anh Sơn và Claude họp, bàn luận, lên kế hoạch cho dự án Nolimit Ads Manager.

**Các lựa chọn:**
1. Làm việc trực tiếp trong repo code - Dễ bị lẫn với code production
2. Tạo thư mục riêng biệt - Tách biệt rõ ràng giữa planning và coding

**Quyết định:** Tạo thư mục riêng "0_ Kế hoạch xây dựng Nolimit"

**Lý do:**
- Tách biệt rõ vai trò BA/PO với dev
- Có không gian để brainstorm mà không ảnh hưởng code
- Dễ quản lý tài liệu, prototype, ideas

**Hậu quả:** Mọi kế hoạch, prototype, quyết định được lưu tập trung tại đây.

---

### [2026-01-16] Tách SYSTEM.md và KNOWLEDGE.md

**Bối cảnh:** Nội dung giữa 2 file đang bị trùng lặp, ranh giới không rõ ràng.

**Các lựa chọn:**
1. Gộp thành 1 file SYSTEM.md
2. Tách rõ: SYSTEM (kỹ thuật) vs KNOWLEDGE (nghiệp vụ)

**Quyết định:** Tách riêng theo Option 2

**Lý do:**
- SYSTEM.md cho dev team hiểu cách code hoạt động
- KNOWLEDGE.md cho team vận hành hiểu business rules và rủi ro
- Mỗi file có đối tượng đọc khác nhau

**Hậu quả:**
- Dev đọc SYSTEM.md để hiểu kiến trúc
- Team vận hành đọc KNOWLEDGE.md để hiểu nghiệp vụ
- Không còn nhập nhằng nội dung

---

### [2026-01-16] Chuyển docs vào thư mục docs/

**Bối cảnh:** Các file docs nằm rải rác ở root, cần tổ chức lại.

**Quyết định:** Tạo thư mục `docs/` chứa tất cả tài liệu nền tảng.

**Lý do:**
- Gọn gàng, dễ tìm
- Tách biệt docs với ideas và issues
- Cấu trúc chuẩn hơn

**Hậu quả:** Tất cả docs (ABOUT, SYSTEM, KNOWLEDGE, DECISIONS) nằm trong `docs/`
