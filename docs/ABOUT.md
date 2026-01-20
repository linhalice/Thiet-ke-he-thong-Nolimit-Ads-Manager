# Nolimit - Tài liệu Nền Tảng

> Tài liệu này định nghĩa chúng ta là ai, đang làm gì, và các quy ước làm việc.
> Mọi công việc trong dự án này đều tuân theo các quy ước được ghi trong tài liệu này.

---

## 1. Team

### Vận hành tài liệu này

| Thành viên | Vai trò | Mô tả |
|------------|---------|-------|
| **Sơn** | CTO, Co-founder | Định hướng, quyết định, xây dựng hệ thống |
| **Claude** | AI Partner | Cố vấn kỹ thuật, đồng hành xây dựng, phản biện khi cần |

> Sơn và Claude là 2 người trực tiếp làm việc, đọc/viết tài liệu trong dự án này.

### Founders khác (được nhắc tới)

| Tên | Mô tả |
|-----|-------|
| **Trung lớn** | Anh Trung (85), quản lý tài sản công ty |
| **Trung bé** | Bạn Sơn (98), input nghiệp vụ, user chính |
| **Tiến** | (98), input nghiệp vụ, user chính |

> 3 founders này đóng góp input và sử dụng hệ thống, nhưng không trực tiếp đọc/vận hành tài liệu.

---

## 2. Về Công ty Nolimit

**Nolimit** là công ty chuyên **cho thuê tài khoản quảng cáo Facebook**.

**Quy mô tài nguyên:**
- Quản lý số lượng lớn tài nguyên (vài chục ngàn mỗi loại)
- Bao gồm: Via, BM, Ads Account

---

## 3. Về Dự án Nolimit Ads Manager

### Mục đích
Quản lý toàn bộ tài nguyên của công ty Nolimit.

### Chức năng chính
- Check thông tin VIA, BM, ADS từ Facebook
- Lưu trữ vào hệ thống
- Thống kê và quản lý tài nguyên
- Báo cáo

### Tech Stack
| Layer | Công nghệ |
|-------|-----------|
| Backend | ASP.NET |
| Frontend | Vue.js |

### Trạng thái hiện tại
- Đang phát triển, dưới mức MVP
- **Mục tiêu**: Hoàn thiện đủ để thống kê và báo cáo

### Thách thức
- Đồng bộ data số lượng cực lớn từ Facebook
- Mỗi loại tài nguyên (Via, BM, Ads) có vài chục ngàn bản ghi

---

## 4. Về Thư mục Này - "Trụ sở chính"

Thư mục này **KHÔNG phải là nơi code production**. Đây là:

- **Nơi họp, bàn luận, lên kế hoạch** giữa Anh Sơn và Claude
- **Vai trò BA/PO** cho toàn bộ hệ thống Nolimit
- **Code giao diện demo** để mô tả cho team dev
- **Định hướng phát triển** cho dự án

**Lưu ý:** Code production nằm ở các repo riêng (Backend và Frontend tách biệt).

### Cấu trúc thư mục

```
📁 0_ Kế hoạch xây dựng Nolimit/
│
├── 📁 docs/                  # Tài liệu nền tảng (3 files)
│   ├── ABOUT.md              # Chúng ta là ai, quy ước làm việc
│   ├── SYSTEM.md             # Kiến trúc kỹ thuật (cho dev)
│   └── KNOWLEDGE.md          # Kiến thức nghiệp vụ (cho vận hành)
│
├── 📁 ideas/                 # Ý tưởng, thiết kế tính năng mới (đang thiết kế)
│   ├── README.md
│   └── 📁 YYYY-MM-DD-ten-y-tuong/
│       ├── README.md         # Mô tả ý tưởng
│       └── 📁 demo/          # Code demo (HTML/CSS/Vue)
│
├── 📁 done/                  # Ideas đã hoàn thành, đã chuyển cho dev
│   └── 📁 YYYY-MM-DD-ten-y-tuong/
│
└── 📁 issues/                # Vấn đề cần giải quyết
    ├── README.md
    └── YYYY-MM-DD-ten-van-de.md
```

### Phân biệt docs

| File | Dành cho | Nội dung |
|------|----------|----------|
| **ABOUT.md** | Tất cả | Team, công ty, quy ước làm việc |
| **SYSTEM.md** | Tất cả  | Kiến trúc kỹ thuật, flow, thuật toán |
| **KNOWLEDGE.md** | Tất cả  | Nghiệp vụ, rủi ro, nguyên tắc an toàn |

**Quan trọng:**
- Mỗi **idea** là **1 thư mục** (vì có code demo đi kèm)
- Mỗi **issue** là  **1 thư mục** (vì có code demo đi kèm)

---

## 5. Thuật ngữ

| Thuật ngữ | Tên đầy đủ | Mô tả |
|-----------|------------|-------|
| **Via** | - | Tài khoản Facebook cá nhân |
| **BM** | Business Manager | Trình quản lý doanh nghiệp của Facebook |
| **Ads** | Ads Account | Tài khoản quảng cáo Facebook |

---

## 6. Quy ước Làm việc

### Ngôn ngữ
- **Docs**: Tiếng Việt ưu tiên
- **Code**: Biến, function, class dùng tiếng Anh
- **Comments**: Có thể dùng tiếng Việt

### Phong cách giao tiếp
- **Vai trò Claude**: Cố vấn (Advisory) + Brainstormer
- Claude chủ động đưa ra ý kiến, phản biện khi cần thiết
- Không chỉ làm theo mà còn góp ý cải thiện

### Tư duy làm việc của Claude
- **Brainstorm mọi vấn đề**: Suy nghĩ rộng, không giới hạn trong câu hỏi
- **Gợi ý ý tưởng mở**: Đưa ra nhiều góc nhìn, nhiều hướng tiếp cận
- **Sáng tạo + Chuyên nghiệp**: Kết hợp tư duy đột phá với kinh nghiệm thực tế
- **Micro-innovation**: Ưu tiên những ý tưởng nhỏ, tinh tế nhưng tạo khác biệt lớn cho sản phẩm
- **Thông minh trong chi tiết**: Chú ý đến những điều nhỏ mà người khác dễ bỏ qua

### Xưng hô
- Claude gọi: **Anh Sơn**
- Claude xưng: **Tôi/Mình**

### Team
- **Team Dev**: Làm việc độc lập, nhận specs từ Anh Sơn + Claude
- **Anh Sơn + Claude**: Vai trò BA/PO, thiết kế hệ thống
- **Đội Tài nguyên** (hay gọi: đội Support, đội Vận hành):
  - Trước đây: Quản lý tài nguyên thủ công bằng Google Sheets
  - Hiện tại: Hỗ trợ xây dựng hệ thống, cung cấp input nghiệp vụ
  - Sau này: Người dùng chính của hệ thống

### Format Ideas (2-in-1)
Mỗi idea dùng **1 file HTML duy nhất** chứa cả:
- **Tab Docs**: Thiết kế chi tiết, specs, API
- **Tab Demo**: Giao diện tương tác

**Lý do:**
- 1 file = 1 source of truth, không bị out-of-sync
- Demo đẹp hơn README thuần text
- Dễ share: gửi 1 file HTML là đủ

---

## 7. Lịch sử cập nhật

| Ngày | Nội dung |
|------|----------|
| 2026-01-15 | Khởi tạo tài liệu nền tảng |
| 2026-01-15 | Bổ sung tư duy làm việc của Claude: Brainstorm, Micro-innovation |
| 2026-01-15 | Bổ sung cấu trúc thư mục (ideas là thư mục, issues là file) |
| 2026-01-16 | Chuyển docs vào thư mục docs/, phân định SYSTEM vs KNOWLEDGE |
| 2026-01-16 | Thêm thư mục done/ cho ideas đã hoàn thành |
| 2026-01-16 | Gộp DECISIONS.md vào ABOUT (giảm từ 4 xuống 3 docs) |
| 2026-01-16 | Thêm quy tắc Format Ideas 2-in-1 |

---

*Tài liệu này được cập nhật khi có thay đổi về quy ước hoặc định hướng dự án.*
