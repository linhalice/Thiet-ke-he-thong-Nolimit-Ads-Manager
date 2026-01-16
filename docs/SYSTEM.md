# Nolimit Ads Manager - Kiến trúc Kỹ thuật

> Tài liệu mô tả kiến trúc hệ thống và cách các component hoạt động.
> **Dành cho:** Dev team, technical reference.

---

## 1. Tổng quan Hệ thống

```
┌─────────────────────────────────────────────────────────────────┐
│                         HỆ THỐNG NOLIMIT                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌─────────────┐         ┌─────────────┐         ┌──────────┐  │
│   │   WEB APP   │ ◄─────► │  DATABASE   │ ◄─────► │  MASTER  │  │
│   │  (Vue.js)   │         │             │         │ (Desktop)│  │
│   └─────────────┘         └─────────────┘         └────┬─────┘  │
│         │                                              │        │
│         │ Tra cứu, báo cáo                    Điều khiển│        │
│         ▼                                              ▼        │
│   ┌─────────────┐                          ┌─────────────────┐  │
│   │    USER     │                          │  Chrome + Via   │  │
│   │  (Browser)  │                          │  + Worker Ext   │  │
│   └─────────────┘                          └─────────────────┘  │
│                                                     │           │
│                                              Call FB API        │
│                                                     ▼           │
│                                            ┌─────────────┐      │
│                                            │  FACEBOOK   │      │
│                                            └─────────────┘      │
└─────────────────────────────────────────────────────────────────┘
```

| Thành phần | Tên | Tech | Mô tả |
|------------|-----|------|-------|
| **Web** | Nolimit Web | Vue.js | Tra cứu, xem data, báo cáo |
| **Backend** | Nolimit API | ASP.NET | API server, business logic |
| **Desktop** | Tool Master | Windows App | Quản lý Via, điều khiển Chrome |
| **Extension** | Worker | Chrome Extension | Check data từ Facebook |

---

## 2. Tool Master (Desktop App)

### Tên gọi
- Tên chính thức: **Nolimit Ads Manager Client**
- Tên nội bộ: **Tool Master** (hoặc Master)

### Chức năng chính

| Chức năng | Mô tả |
|-----------|-------|
| CRUD Via | Thêm, sửa, xóa, xem Via |
| Phân nhóm Via | Tổ chức Via theo nhóm |
| Mở Chrome Selenium | Đăng nhập Via, thao tác thủ công |
| Auto Selenium | Tự động đăng nhập, các tác vụ cơ bản |
| **Quản lý Workers** | Điều khiển các Chrome có gắn Extension Worker |

### Điểm đặc biệt
> **"Thứ làm nên tên tuổi của phần mềm này là khả năng tự động quản lý các Chrome"**

---

## 3. Kiến trúc Master - Worker

```
┌─────────────────────────────────────────────────────────┐
│                      TOOL MASTER                         │
│  (Phần mềm Windows - Quản lý & Điều phối)               │
└─────────────────────────────────────────────────────────┘
                            │
                            │ Quản lý
                            ▼
    ┌───────────┐    ┌───────────┐    ┌───────────┐
    │  Chrome   │    │  Chrome   │    │  Chrome   │
    │  + Via A  │    │  + Via B  │    │  + Via C  │
    │  + Worker │    │  + Worker │    │  + Worker │
    └───────────┘    └───────────┘    └───────────┘
         │                │                │
         └────────────────┼────────────────┘
                          │
                          ▼
                    ┌───────────┐
                    │  DATABASE │
                    └───────────┘
                          │
                          ▼
                    ┌───────────┐
                    │    WEB    │
                    └───────────┘
```

### Worker (Extension)
- Extension Chrome gắn vào mỗi trình duyệt
- Tự động check thông tin Via/BM/Ads
- Gửi data về Database
- Nhận nhiệm vụ từ hệ thống

---

## 4. Flow: Check theo Yêu cầu

```
┌──────────────────────────────────────────────────────────────────┐
│ BƯỚC 1: User yêu cầu check trên Web                              │
│ ─────────────────────────────────────                            │
│ User chọn 1 Ads → Bấm "Check" → Web tạo Nhiệm vụ                │
└──────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│ BƯỚC 2: Hệ thống truy ngược quan hệ                              │
│ ─────────────────────────────────────                            │
│ Ads cần check → Thuộc BM nào? → Via nào có quyền BM đó?         │
│ → Chọn Via phù hợp để thực hiện check                           │
└──────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│ BƯỚC 3: Master nhận nhiệm vụ                                     │
│ ─────────────────────────────────────                            │
│ Master đang ở trạng thái chờ → Phát hiện nhiệm vụ mới           │
│ → Xác định Via cần mở → Mở Chrome cho Via đó                    │
└──────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│ BƯỚC 4: Worker thực hiện                                         │
│ ─────────────────────────────────────                            │
│ Extension Worker call API → Tìm nhiệm vụ của mình               │
│ → Phát hiện Ads cần check → Thực hiện check                     │
│ → Đẩy kết quả về Server                                          │
└──────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│ BƯỚC 5: Kết quả hiển thị                                         │
│ ─────────────────────────────────────                            │
│ Data mới được lưu vào DB → Web hiển thị thông tin cập nhật      │
└──────────────────────────────────────────────────────────────────┘
```

### Áp dụng cho
- ✅ Check Via
- ✅ Check BM
- ✅ Check Ads

---

## 5. Thuật toán Chọn Via Worker

### Vấn đề
Khi cần check Ads, hệ thống phải chọn Via nào để thực hiện? Nhiều Via có quyền truy cập cùng 1 BM/Ads.

### Giải pháp: "Sacrificial Worker" (Thợ Hy Sinh)

#### Bước 1: Flag "Làm nhiệm vụ"
```
Via A [✅ Làm nhiệm vụ] ──► Tham gia làm Worker
Via B [❌ Không làm]     ──► Không bao giờ bị gọi, dù có quyền BM/Ads
Via C [✅ Làm nhiệm vụ] ──► Tham gia làm Worker
```

- Chỉ Via được **bật flag "Làm nhiệm vụ"** mới tham gia check
- Các Via khác được **bảo vệ** - không làm worker dù có quyền

#### Bước 2: Thuật toán "Thợ Lành Nghề"
```
Via đã hoàn thành 100 task ──► Ưu tiên cao (thợ quen việc)
Via đã hoàn thành 10 task  ──► Ưu tiên thấp hơn
Via mới, chưa làm task     ──► Ưu tiên thấp nhất
```

- Ưu tiên Via **đã từng hoàn thành nhiều nhiệm vụ**
- Via đang "quen việc" → cứ cho làm tiếp
- **Die thì mới đổi** sang Via khác

#### Bước 3: Auto Retry (Via Rotation)
```
Nhiệm vụ check Ads X
    │
    ▼
Via A thử check ──► Thất bại (die/checkpoint)
    │
    ▼
Via B thử check ──► Thất bại
    │
    ▼
Via C thử check ──► Thành công ✅
```

- Hệ thống **tự động retry** tìm Via khác khi Via hiện tại fail
- **Tối đa 3 lần** thử
- Nếu hết 3 lần vẫn fail → Nhiệm vụ **báo thất bại**

### Tại sao chiến lược này?

| Cách tiếp cận | Rủi ro |
|---------------|--------|
| ❌ Chia đều cho tất cả Via | Tất cả đều có nguy cơ bị khóa |
| ✅ Tập trung vào vài Via | Chỉ vài Via "hy sinh", đại quân an toàn |

> **Triết lý:** Càng dùng nhiều Via → rủi ro chia đều → tỷ lệ khóa cao hơn.

---

## 6. Dự kiến: Health Score (Chưa triển khai)

```
Via Health Score = f(success_rate, avg_time, warning_count)
```

- Tỷ lệ task thành công/thất bại
- Thời gian trung bình hoàn thành
- Số lần bị warning/checkpoint
- *Chưa triển khai vì hiện tại chưa cấp bách*

---

## 7. Trạng thái Hiện tại

| Tính năng | Trạng thái |
|-----------|------------|
| Lưu trữ Via/BM/Ads | ✅ Cơ bản hoạt động |
| Đồng bộ data từ FB | ✅ Hoạt động |
| Cơ chế Check theo yêu cầu | ✅ Đã triển khai |
| Chiến lược chọn Via Worker | ✅ Đã triển khai |
| Web tra cứu | ✅ Cơ bản |
| Tool Master | ✅ Hoạt động |

**Mục tiêu tiếp theo:** Hoàn thiện để thống kê và báo cáo

---

## 8. Lịch sử cập nhật

| Ngày | Nội dung |
|------|----------|
| 2026-01-15 | Khởi tạo tài liệu kiến trúc hệ thống |
| 2026-01-16 | Tách riêng kiến trúc kỹ thuật, chuyển nghiệp vụ sang KNOWLEDGE.md |

---

*Tài liệu này được cập nhật khi có thay đổi về kiến trúc hoặc tính năng mới.*
