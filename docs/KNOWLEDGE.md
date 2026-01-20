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

### Via cầm BM vs Admin trong BM (Vấn đề ID không khớp)

> ⚠️ **Vấn đề kỹ thuật quan trọng**

**2 nguồn data không match được:**

| Nguồn | Cách lấy | ID |
|-------|----------|-----|
| **Via cầm BM** | Sync từ Via → thấy BM | Via ID (ID Facebook cá nhân) |
| **Admin trong BM** | API call từ BM → list admins | **Admin ID (ID khác!)** |

**Nguyên nhân:** Khi user join BM làm admin, Facebook sinh ra một **ID riêng** cho user trong context BM đó. ID này khác với Via ID gốc.

```
Via "Nguyễn Văn A" (Via ID: 100001234567890)
    └── Join BM "ABC Corp" làm admin
            └── Facebook sinh Admin ID: 200009876543210 (KHÁC!)
    └── Join BM "XYZ Ltd" làm admin
            └── Facebook sinh Admin ID: 200001111222233 (KHÁC NỮA!)
```

**Hậu quả - Điểm mù (Blind Spot):**

| So sánh | Ý nghĩa | Rủi ro |
|---------|---------|--------|
| Admin trong BM > Via cầm | Có admin ta không biết là Via nào | **Không phát hiện hacker gắn Via lạ** |
| Admin trong BM < Via cầm | Via bị kick nhưng data chưa sync | Data cũ, không phản ánh thực tế |
| Admin = Via cầm nhưng không match | 2 list riêng biệt, không link được | Không verify được 1-1 |

### Cơ chế Matching Admin ↔ Via

**Quy ước đặt tên hiện tại (đội vận hành):**
- Khi Via join BM làm admin, đặt tên có **5-6 số cuối của Via ID** ở đuôi
- Email thường dùng format chung: `via2@nolimitnotifyaccount.com`

**Ví dụ thực tế:**
```
Admin name: "1.12 XMDN 30333"
Email: via2@nolimitnotifyaccount.com
       └─────────────────────┘
       Email chung, không chứa UID

Via ID: 100001230333
                └────┘
                30333 = 5 số cuối → MATCH với tên admin
```

**Logic auto-match (parse từ tên):**

```
Admin name: "1.12 XMDN 30333"
                       └────┘
                       Extract: 30333
                            │
Via list:   100001230333 ◄──┘ MATCH! (ends with 30333)
            100009876543     (không match)
```

> 💡 **Đề xuất cải tiến: Dùng Email chứa UID**
>
> Thay vì parse số từ tên (dễ sai, format không chuẩn), dùng email doanh nghiệp với UID:
> ```
> Email mới: 100001230333@nolimit.company
>            └──────────┘
>            UID của Via → Auto-match 100% chính xác
> ```
>
> **Lợi ích:**
> - ✅ Email doanh nghiệp tạo được bao nhiêu tùy ý, đặt tên gì cũng được
> - ✅ Match bằng email = exact match, không cần parse/regex
> - ✅ Giảm lỗi do đội vận hành đặt tên sai format
> - ✅ Dễ audit, dễ trace hơn

**Trạng thái matching:**

| Status | Ý nghĩa | Icon |
|--------|---------|------|
| ✅ **Auto-matched** | Đuôi Via ID khớp với đuôi trong tên admin | 🟢 |
| 🔗 **Manual-matched** | Đội tài nguyên đã confirm thủ công | 🔵 |
| ⚠️ **Unmatched** | Không tìm được Via khớp → Cần giải thích | 🟡 |

**Khi Unmatched - Đội tài nguyên phải chọn lý do:**

| Lý do | Mô tả | Mức độ |
|-------|-------|--------|
| 📦 **Via thất lạc** | Via của ta nhưng chưa link được | Cần tìm |
| 🔄 **Chưa đồng bộ** | Via mới, chưa sync vào hệ thống | Chờ sync |
| 👤 **Via của khách** | Khách được add admin để thao tác Ads | Bình thường (phổ biến ở BM Trung gian) |
| ❓ **Không rõ** | Chưa xác định được | Cần điều tra |
| 🚨 **Nghi ngờ hacker** | Admin lạ, không thuộc ta hoặc khách | **ALERT BẢO MẬT!** |

**Flow xử lý:**

```
┌─────────────────────────────────────────────────────────────────────┐
│  1. Sync admin từ BM                                                │
│     └─► Lấy danh sách admin (ID, tên, email)                       │
├─────────────────────────────────────────────────────────────────────┤
│  2. Auto-match                                                      │
│     └─► Parse số cuối từ tên → Tìm Via LIVE có ID khớp đuôi       │
├─────────────────────────────────────────────────────────────────────┤
│  3. Kết quả                                                         │
│     ├─► Matched: Gắn link Admin ↔ Via                              │
│     └─► Unmatched: ⚠️ Warning + Gợi ý Via candidates              │
├─────────────────────────────────────────────────────────────────────┤
│  4. Xử lý Unmatched có số trong tên                                │
│     ├─► Hệ thống tìm Via candidates có đuôi ID khớp               │
│     ├─► Hiển thị gợi ý: Via live + Via die                        │
│     └─► Đội kỹ thuật kiểm tra và đồng bộ                          │
├─────────────────────────────────────────────────────────────────────┤
│  5. Nếu Via đã die (không sync BM được nữa)                        │
│     └─► Cho phép gắn thủ công quan hệ Admin ↔ Via                 │
├─────────────────────────────────────────────────────────────────────┤
│  6. Nếu không tìm được Via phù hợp                                 │
│     └─► Đội tài nguyên chọn lý do (Via khách, hacker, etc.)       │
└─────────────────────────────────────────────────────────────────────┘
```

### Gợi ý Via cho Admin Unmatched

Khi phát hiện admin có số trong tên nhưng chưa match được, hệ thống **tự động gợi ý** Via candidates:

**Ví dụ:**
```
Admin "1.12 XMDN 30333" - Unmatched
       ┌──────────────────────────────────────────────────────┐
       │ 🔍 Phát hiện số: 30333                               │
       │                                                      │
       │ 📋 Gợi ý Via có đuôi khớp:                          │
       │ ┌────────────────────────────────────────────────┐  │
       │ │ 🟢 100001230333 - Via A (Live)                 │  │
       │ │    └─ Chưa sync BM này → [Yêu cầu đồng bộ]    │  │
       │ ├────────────────────────────────────────────────┤  │
       │ │ 💀 100009930333 - Via B (Die - 2025-12-20)    │  │
       │ │    └─ Không thể sync → [Gắn thủ công]         │  │
       │ └────────────────────────────────────────────────┘  │
       │                                                      │
       │ Không phải Via nào ở trên? [Chọn lý do khác ▼]      │
       └──────────────────────────────────────────────────────┘
```

**Logic xử lý theo trạng thái Via:**

| Via Status | Hành động | Nút |
|------------|-----------|-----|
| 🟢 **Live** | Yêu cầu đội kỹ thuật sync Via → BM | `[Yêu cầu đồng bộ]` |
| 💀 **Die** | Via không thể sync nữa → Gắn quan hệ thủ công | `[Gắn thủ công]` |
| ❌ **Không tìm thấy** | Chọn lý do: Via khách, hacker, etc. | `[Chọn lý do ▼]` |

> **Tại sao Via die vẫn cần gắn thủ công?**
> - Via die không thể login → không sync được danh sách BM nữa
> - Nhưng admin đó vẫn tồn tại trong BM (chưa bị kick)
> - Cần gắn thủ công để hệ thống biết admin đó là Via nào (dù đã die)
> - Giúp trace được lịch sử và đảm bảo không phải hacker

**Hiển thị trong UI:**

```
BM Root ABC
├── 👤 Via cầm BM: 3 (từ sync Via)
├── 👥 Admin trong BM: 5 (từ API BM)
│   ├── ✅ Admin A (234567) → Via 100001234567 🟢
│   ├── ✅ Admin B (345678) → Via 100002345678 🟢
│   ├── 🔗 Admin C (456789) → Via 100003456789 💀 (gắn thủ công)
│   ├── ⚠️ Admin D (30333) → [Gợi ý: Via 100001230333 🟢] [Đồng bộ]
│   └── ⚠️ Admin E (không có số) → [Chọn lý do ▼]
└── ⚠️ Chênh lệch: +2 admin cần review
```

---

### BM Gốc vs BM Trung gian

| Loại | Mô tả | Vai trò |
|------|-------|---------|
| **BM Gốc** | BM sở hữu Ads, nơi Ads được tạo ra | Quan trọng nhất, cần bảo vệ tuyệt đối |
| **BM Trung gian** | BM được share quyền sử dụng Ads từ BM Gốc | Lớp đệm bảo vệ, dùng để check hàng ngày |

### Cách xác định BM Gốc vs Trung gian

> **Lưu ý quan trọng:** "BM Gốc" và "BM Trung gian" là khái niệm **tự định nghĩa** của Nolimit, không phải field có sẵn từ Facebook API.

**Nguyên tắc phân loại: Dựa vào TỶ TRỌNG Ads**

```
Nếu số Ads OWNED > số Ads SHARED
    → BM GỐC

Nếu số Ads SHARED > số Ads OWNED (hoặc chỉ có SHARED)
    → BM TRUNG GIAN
```

**Tại sao không phân tuyệt đối?**

Thực tế BM Gốc không chỉ có Ads gốc, đôi khi vẫn có một số Ads được share vào với mục đích:
- **Kích BM nâng giới hạn** - share Ads vào để tăng limit tạo thêm Ads
- **Trick nâng cấp BM** - các kỹ thuật để upgrade BM lên tier cao hơn
- **Backup tạm thời** - share Ads qua BM khác trong thời gian chờ xử lý

**Ví dụ phân loại:**

| BM | Ads Owned | Ads Shared | Tỷ trọng | Phân loại |
|----|-----------|------------|----------|-----------|
| BM Alpha | 50 | 5 | 91% owned | 🏠 **BM Gốc** |
| BM Beta | 0 | 120 | 100% shared | 🔗 **BM Trung gian** |
| BM Gamma | 30 | 10 | 75% owned | 🏠 **BM Gốc** |
| BM Delta | 5 | 80 | 94% shared | 🔗 **BM Trung gian** |

**Ngưỡng đề xuất:**
- ≥50% Ads owned → BM Gốc
- <50% Ads owned (hoặc 0) → BM Trung gian

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
| 2026-01-16 | Bổ sung cách phân loại BM Gốc vs Trung gian theo tỷ trọng Ads owned/shared |
| 2026-01-16 | Ghi nhận vấn đề Via ID vs Admin ID không khớp - điểm mù bảo mật |
| 2026-01-16 | Bổ sung cơ chế Matching Admin ↔ Via: quy ước đặt tên 5-6 số cuối, auto-match logic, các trạng thái unmatched và flow xử lý |
| 2026-01-16 | Bổ sung tính năng Gợi ý Via cho Admin Unmatched: auto-detect số từ tên, gợi ý Via candidates (live + die), hỗ trợ gắn thủ công cho Via die |

---

*Tài liệu này được cập nhật khi có kiến thức mới hoặc thay đổi quy trình vận hành.*
