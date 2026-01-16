# Orphan Ads - Ads lạc trong điểm mù

**Trạng thái:** open
**Độ ưu tiên:** critical
**Ngày tạo:** 2026-01-15

---

## Vấn đề

Nhiều Ads đang **lạc trong điểm mù** - hệ thống không thể check được:

### Loại 1: Ads chỉ có trong BM gốc
```
BM Gốc ──► Ads X
           (chưa share vào BM trung gian nào)
           → Không ai check được (vì không check trực tiếp từ BM gốc)
```

### Loại 2: Ads trong BM trung gian "chết"
```
BM Trung gian (không có Via Worker) ──► Ads Y
           → Via không reach được
           → Ads bị "mù"
```

### Loại 3: Ads mất kết nối
```
Ads Z tồn tại trong DB
       nhưng không thuộc BM nào có Via Worker
       → Orphan hoàn toàn
```

## Tại sao nghiêm trọng

- **Không đồng bộ 100%** - Luôn có Ads không được check
- **Không biết Ads nào đang mù** - Không có cảnh báo
- **Rủi ro tài chính** - Ads có thể đang chạy tiền mà không ai biết status

## Hướng giải quyết (đề xuất)

### Bước 1: Query phát hiện Orphan Ads
```sql
-- Pseudo query
SELECT ads.*
FROM ads
WHERE ads.id NOT IN (
  SELECT DISTINCT ads_id
  FROM ads_bm_mapping
  WHERE bm_id IN (
    SELECT bm_id FROM bm WHERE has_active_via_worker = true
  )
)
```

### Bước 2: Dashboard hiển thị
- Số lượng Orphan Ads
- Danh sách chi tiết
- Lý do tại sao orphan (chưa share? BM chết? Via die?)

### Bước 3: Action
- Tự động share vào BM trung gian?
- Alert để xử lý thủ công?

## Mức độ tự tin hiện tại

> Anh Sơn: "Hệ thống tôi chưa tự tin về việc đồng bộ chuẩn 100%"

Đây là vấn đề **nhức nhối** cần giải quyết để nâng độ tin cậy của hệ thống.

---

## Liên quan

- [Mapping BM gốc vs trung gian](./2026-01-15-mapping-bm-goc-vs-trung-gian.md)
- [Alert Via Worker sắp cạn](../ideas/2026-01-15-alert-via-worker-sap-can.md)
