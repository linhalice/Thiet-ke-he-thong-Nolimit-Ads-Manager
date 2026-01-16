# Ideas

Nơi lưu những ý tưởng, thiết kế tính năng mới.

## Cấu trúc

```
📁 ideas/
├── README.md
├── 📁 YYYY-MM-DD-ten-y-tuong/     ← Mỗi idea là 1 thư mục
│   ├── README.md                   ← Mô tả ý tưởng
│   ├── 📁 demo/                    ← Code demo (HTML/CSS/Vue)
│   └── 📁 assets/                  ← Hình ảnh, mockup
```

## Cách sử dụng
- Mỗi ý tưởng là **1 thư mục** riêng (để chứa code demo)
- Đặt tên theo format: `YYYY-MM-DD-ten-y-tuong/`
- File `README.md` trong thư mục mô tả chi tiết ý tưởng
- Thư mục `demo/` chứa code giao diện demo cho dev

## Trạng thái ý tưởng
- **raw** - Mới nghĩ ra, chưa đánh giá
- **considering** - Đang cân nhắc
- **approved** - Sẽ làm, chờ lên kế hoạch
- **in-progress** - Đang phát triển
- **done** - Đã hoàn thành
- **rejected** - Không phù hợp (ghi rõ lý do)

## Template README.md cho idea

```markdown
# Tên Ý tưởng

**Trạng thái:** considering
**Ngày tạo:** YYYY-MM-DD
**Liên quan:** [link tới issue nếu có]

## Vấn đề
Mô tả vấn đề cần giải quyết

## Giải pháp
Mô tả giải pháp đề xuất

## Thiết kế
Chi tiết thiết kế, mockup

## Demo
Link tới code demo trong thư mục `demo/`
```
