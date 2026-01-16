# Kiến thức Nghiệp vụ - Nolimit

> Tài liệu lưu trữ kiến thức về mô hình dữ liệu, rủi ro cần tránh, và nguyên tắc vận hành.
> **Dành cho:** Team vận hành, người mới cần onboard, Sơn, Claude AI trợ lý Sơn
> **Bắt buộc đọc** trước khi làm việc với hệ thống.

---

## 1. Mô hình Dữ liệu: Via - BM - Ads

### Quan hệ cơ bản

```
Via (Tài khoản Facebook cá nhân)
 └── BM (Business Manager)
      └── Ads (Tài khoản quảng cáo)
```

**Quy tắc:**
- Via tạo ra BM
- BM tạo ra Ads
- **Via là gốc của mọi thứ**

---

### BM có thể chia sẻ cho nhiều Via

```
Via A (chủ sở hữu) ──┐
Via B (được share)  ──┼──► BM ──► Ads
Via C (được share)  ──┘
```

**Tại sao share?** Phòng rủi ro - nếu Via chủ bị khóa, vẫn còn Via khác truy cập BM.

---

### BM Gốc vs BM Trung gian

| Loại | Mô tả | Vai trò |
|------|-------|---------|
| **BM Gốc** | BM sở hữu Ads, nơi Ads được tạo ra | Quan trọng nhất, cần bảo vệ tuyệt đối |
| **BM Trung gian** | BM được share quyền sử dụng Ads từ BM Gốc | Lớp đệm bảo vệ, dùng để check hàng ngày |

### Kiến trúc BM Trung gian (Proxy)

```
┌─────────────────────────────────────────────────────────────────┐
│                         BM GỐC (Quan trọng)                      │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐            │
│  │ BM gốc 1│  │ BM gốc 2│  │ BM gốc 3│  │ BM gốc N│            │
│  │ 50 Ads  │  │ 50 Ads  │  │ 50 Ads  │  │ 50 Ads  │            │
│  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘            │
│       │            │            │            │                  │
│       └────────────┴─────┬──────┴────────────┘                  │
│                          │ Share quyền sử dụng Ads              │
│                          ▼                                      │
│              ┌───────────────────────┐                          │
│              │    BM TRUNG GIAN      │ ◄── CHECK TỪ ĐÂY        │
│              │   (Proxy/Buffer)      │                          │
│              │    ~1000 Ads gom      │                          │
│              └───────────┬───────────┘                          │
│                          │                                      │
│                          ▼                                      │
│                   ┌────────────┐                                │
│                   │ Via Worker │ ◄── 1 Via check được 1000 Ads  │
│                   └────────────┘                                │
└─────────────────────────────────────────────────────────────────┘
```

**Lợi ích:**

| Aspect | BM Gốc | BM Trung gian |
|--------|--------|---------------|
| Rủi ro khi check | ❌ Cao - die = mất Ads | ✅ Thấp - die chỉ mất proxy |
| Số Via cần | Nhiều (mỗi BM 1 Via) | Ít (1 Via cho nhiều Ads) |
| Quản lý | Phân tán | Tập trung |

---

## 2. Các Tình huống Nguy hiểm

### 💀 FATAL - Mất trắng, KHÔNG THỂ phục hồi

| Tình huống | Hậu quả | Cách phòng tránh |
|------------|---------|------------------|
| **BM Gốc die** | GAME OVER - Mất toàn bộ Ads trong BM đó vĩnh viễn | Không bao giờ check trực tiếp từ BM gốc |
| **Tất cả Via cầm BM Gốc đều die** | Mất quyền truy cập BM Gốc vĩnh viễn | Luôn có ≥3 Via cầm mỗi BM Gốc |

> ⚠️ **CẢNH BÁO NGHIÊM TRỌNG:**
>
> **Trường hợp 1: BM Gốc die**
> - Tài sản trong BM **VÔ HIỆU LỰC HOÀN TOÀN**
> - Không thao tác được, không chạy Ads được
> - **Mất trắng, không thể khôi phục**
>
> **Trường hợp 2: Tất cả Via cầm BM Gốc die**
> - **NGHIÊM TRỌNG KHÔNG KÉM** - dù BM còn sống
> - Không còn ai có quyền truy cập BM
> - Coi như **mất trắng BM và toàn bộ Ads** trong đó
> - Không thể khôi phục quyền truy cập
>
> **Kết luận:** Cả 2 trường hợp đều = **GAME OVER**

---

### 🔴 CRITICAL - Cần xử lý ngay lập tức

| Tình huống | Hậu quả | Cách xử lý |
|------------|---------|------------|
| **0 Via live cho 1 Ads** | Không thể check Ads đó | Bổ sung Via có quyền BM liên quan |
| **0 BM Trung gian live** | Không có đường check an toàn | Tạo BM trung gian mới, share từ gốc |
| **Via Worker sắp cạn** | Hết Via để làm nhiệm vụ | Chuẩn bị thêm Via mới |

---

### ⚠️ WARNING - Cần chú ý, xử lý sớm

| Tình huống | Hậu quả | Cách xử lý |
|------------|---------|------------|
| **Chỉ 1 Via live cho Ads** | Rủi ro cao nếu Via đó die | Bổ sung Via backup |
| **Chỉ 1 BM Trung gian** | Ít dự phòng | Tạo thêm BM trung gian |
| **BM Gốc chưa sync (0G)** | Không biết trạng thái thực | Sync Via liên quan để lấy thông tin |
| **Via cầm trực tiếp BM Gốc làm Worker** | Rủi ro nếu Via đó bị khóa | Tắt "làm nhiệm vụ" cho Via này |

---

## 3. Ma trận Rủi ro - Bảng tra cứu nhanh

### Via live cho 1 Ads

| Số Via live | Trạng thái | Ý nghĩa |
|-------------|------------|---------|
| 0 | 🔴 **CRITICAL** | Không thể check |
| 1 | 🟡 **WARNING** | Rủi ro cao |
| 2+ | 🟢 **OK** | An toàn |

### BM Trung gian live cho 1 Ads

| Số BM TG live | Trạng thái | Ý nghĩa |
|---------------|------------|---------|
| 0 | 🔴 **CRITICAL** | Không đường check an toàn |
| 1 | 🟡 **WARNING** | Ít dự phòng |
| 2+ | 🟢 **OK** | Đủ dự phòng |

### BM Gốc

| Trạng thái | Hiển thị | Ý nghĩa |
|------------|----------|---------|
| Live | `+1G` | Bình thường |
| Chưa sync | `+⚠️0G` | Cần sync để xác định |
| Die | `💀GỐC DIE` | **GAME OVER** |

---

## 4. Nguyên tắc Vận hành An toàn

### Nguyên tắc 1: Bảo vệ BM Gốc tuyệt đối
- ❌ **KHÔNG BAO GIỜ** check trực tiếp từ BM Gốc
- ❌ **KHÔNG** bật "làm nhiệm vụ" cho Via cầm BM Gốc
- ✅ Luôn check qua BM Trung gian

### Nguyên tắc 2: Dự phòng đa lớp
- ✅ Mỗi BM Gốc có ≥2 Via cầm
- ✅ Mỗi Ads có ≥2 Via live có thể check
- ✅ Mỗi nhóm Ads có ≥2 BM Trung gian

### Nguyên tắc 3: Hy sinh có kiểm soát
- ✅ Chỉ định một số Via làm "Worker" (hy sinh)
- ✅ Tập trung risk vào nhóm nhỏ, bảo vệ đại quân
- ✅ Via die thì thay, không để ảnh hưởng toàn hệ thống

### Nguyên tắc 4: Monitor liên tục
- ✅ Theo dõi số Via Worker còn hoạt động
- ✅ Phát hiện sớm Ads "mồ côi" (không Via nào reach được)
- ✅ Alert khi xuống dưới ngưỡng an toàn

---

## 5. Quy trình Phục hồi khi BM Trung gian Die

```
BM Trung gian die
       │
       ▼
┌─────────────────────────┐
│ 1. Tạo BM Trung gian mới │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│ 2. Share Ads từ BM Gốc  │
│    vào BM TG mới        │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│ 3. Gán Via Worker mới   │
│    vào BM TG            │
└───────────┬─────────────┘
            │
            ▼
       ✅ Phục hồi xong
```

> **Lưu ý:** BM Gốc die thì KHÔNG THỂ phục hồi bằng cách này hay bất kỳ cách nào.

---

## 6. Checklist Kiểm tra Sức khỏe Hệ thống

### Hàng ngày
- [ ] Số Via Worker còn hoạt động > ngưỡng tối thiểu?
- [ ] Có Ads nào 0 Via live không?
- [ ] Có BM Trung gian nào die không?

### Hàng tuần
- [ ] Tất cả Ads có ≥2 Via live?
- [ ] Tất cả BM Gốc có ≥2 Via cầm?
- [ ] Có Ads "mồ côi" (chưa thuộc BM trung gian nào)?

### Khi có Via die
- [ ] Via đó có cầm BM Gốc nào không?
- [ ] BM đó còn Via nào khác cầm không?
- [ ] Cần bổ sung Via thay thế không?

---

## 7. Lịch sử cập nhật

| Ngày | Nội dung |
|------|----------|
| 2026-01-16 | Khởi tạo tài liệu kiến thức nghiệp vụ |
| 2026-01-16 | Tách riêng từ SYSTEM.md, focus vào nghiệp vụ & vận hành |

---

*Tài liệu này được cập nhật khi có kiến thức mới hoặc thay đổi quy trình vận hành.*
