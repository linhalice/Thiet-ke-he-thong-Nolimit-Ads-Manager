# Mapping BM Gốc vs BM Trung gian bị lẫn lộn

**Trạng thái:** open
**Độ ưu tiên:** high
**Ngày tạo:** 2026-01-15
**Cần:** Họp với đội dev

---

## Vấn đề

Hiện tại trong DB, quan hệ giữa Ads và BM **đang lẫn lộn**:
- Không phân biệt rõ Ads thuộc BM gốc hay BM trung gian
- Khó truy vết: Ads này gốc ở BM nào? Đang được share vào BM trung gian nào?

## Tại sao quan trọng

1. **Không biết Ads gốc ở đâu** → Không biết mất BM nào thì mất Ads nào
2. **Khó tính toán rủi ro** → BM gốc chứa bao nhiêu Ads?
3. **Khó làm báo cáo** → Thống kê theo BM gốc vs BM trung gian

## Hướng giải quyết (đề xuất)

### Option 1: Thêm field phân loại BM
```
BM {
  id,
  name,
  type: 'root' | 'proxy',  // <-- Thêm field này
  parent_bm_id: null | id  // <-- BM trung gian trỏ về BM gốc
}
```

### Option 2: Bảng mapping riêng
```
AdsMapping {
  ads_id,
  root_bm_id,      // BM gốc sở hữu Ads
  proxy_bm_id,     // BM trung gian đang share
  shared_at        // Thời điểm share
}
```

## Câu hỏi cần làm rõ với dev

- [ ] Hiện tại DB đang lưu quan hệ Ads-BM như thế nào?
- [ ] Có cách nào truy ngược từ Ads ra BM gốc không?
- [ ] Khi share Ads sang BM trung gian, có log lại không?

---

## Ghi chú

Anh Sơn: "Hiện tại đang khá là lẫn lộn. Cái này tôi phải họp lại với đội dev sau."
