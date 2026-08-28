# 05 — Chuyển hàng vào kho (Putaway)

## 1. Mục đích (Purpose)

Di chuyển hàng hoá từ **khu vực chờ cất (inbound / staging area)** sau khi [nhận hàng](../03-receipt/README.md) đến **vị trí lưu trữ chính thức (storage location / bin / slot)** trong kho, và **cập nhật tồn kho theo vị trí** để hàng trở nên **khả dụng cho picking / bán hàng**.

Putaway quyết định *hàng nằm ở đâu* — ảnh hưởng trực tiếp đến hiệu suất [Picking](../07-picking/README.md), mật độ lưu trữ, và an toàn kho. Module dựa vào **[Gợi ý vị trí (Slotting Suggestion)](../06-warehouse-map-slotting/README.md)** để đề xuất vị trí tối ưu.

---

## 2. Actor liên quan

| Actor | Vai trò |
|-------|---------|
| **Nhân viên kho (Warehouse Staff / Putaway Operator)** | Nhận nhiệm vụ putaway, di chuyển hàng, quét xác nhận vị trí đặt |
| **Hệ thống (Slotting Engine)** | Sinh gợi ý vị trí dựa trên quy tắc |
| **Quản lý kho (Warehouse Manager)** | Ghi đè gợi ý, xử lý trường hợp không còn chỗ, cấu hình quy tắc slotting |

---

## 3. Input / Output

### Input

| Trường | Mô tả | Bắt buộc |
|--------|-------|----------|
| Nhiệm vụ putaway (putaway task) | Sinh từ Receipt `Confirmed` (phần hàng `Accepted`) | Có |
| SKU, số lượng, lô/hạn dùng/serial | Từ Receipt | Có |
| Vị trí nguồn (from location) | Thường là inbound/staging bin | Có |
| Vị trí đích gợi ý (suggested to location) | Từ Slotting Engine | Không (nếu tắt gợi ý thì nhập tay) |
| Vị trí đích thực tế (actual to location) | NV kho quét khi đặt hàng | Có |
| Thiết bị nâng hạ / ràng buộc khu vực | (ví dụ hàng nặng chỉ để tầng thấp) | Không |

### Output

| Kết quả | Dùng bởi |
|---------|----------|
| **Cập nhật tồn kho theo vị trí** (location inventory) | 06 Map, 07 Picking, báo cáo tồn |
| **Tồn khả dụng để bán (available stock)** tăng lên | 13 Catalog / 14 Order (ATP) |
| **Putaway task `Completed`** + lịch sử di chuyển (movement log) | Truy vết, 03 Receipt (đóng phiếu) |
| **Cảnh báo chênh lệch gợi ý vs thực tế** | 06 Slotting (feedback loop) |

---

## 4. Luồng nghiệp vụ từng bước (Step-by-step)

1. **Sinh nhiệm vụ putaway** — sau khi [Receipt](../03-receipt/README.md) `Confirmed`, hệ thống tạo putaway task cho từng dòng hàng `Accepted` (có thể gộp/tách theo SKU, lô, kiện).
2. **Gợi ý vị trí (slotting suggestion)** — [Slotting Engine](../06-warehouse-map-slotting/README.md) đề xuất vị trí đích tối ưu dựa trên: nhóm hàng, tần suất xuất (velocity/ABC), kích thước & khối lượng SKU, chỗ trống, quy tắc gần nhau/tránh xa (compatibility), FIFO/FEFO theo lô.
3. **Phân công & ưu tiên** — task được đẩy vào hàng đợi của NV kho; ưu tiên hàng bán chạy, hàng có hạn dùng gần, hàng chặn lối đi.
4. **Lấy hàng ở inbound area** — NV kho quét mã task / SKU / lô tại vị trí nguồn để xác nhận đúng hàng và đúng số lượng.
5. **Di chuyển đến vị trí đích** — theo gợi ý hoặc theo lộ trình.
6. **Đặt hàng & quét xác nhận vị trí** — quét mã vị trí (location barcode). Hệ thống kiểm tra:
   - Vị trí có phù hợp SKU/khu vực không.
   - Còn sức chứa (capacity) không.
   - Có vi phạm quy tắc (ví dụ trộn lô khác hạn dùng trong cùng bin nếu cấm) không.
7. **Xử lý sai lệch vị trí** — nếu NV kho đặt khác vị trí gợi ý: cho phép kèm **lý do** (hết chỗ, hàng cồng kềnh…); ghi nhận để Slotting Engine học.
8. **Cập nhật tồn kho** — số lượng chuyển từ *inbound* sang *storage location*; tồn khả dụng để bán tăng tương ứng (trừ phần cấu hình cần bước kích hoạt thêm).
9. **Hoàn tất task** — putaway task `Completed`. Khi mọi task của một Receipt xong → Receipt có thể `Closed`.
10. **Putaway nhiều chặng (multi-step)** — với kho lớn: chặng 1 đưa từ dock ra khu tạm gần khu lưu trữ, chặng 2 cất lên kệ; mỗi chặng là một task con.

---

## 5. Trạng thái / Vòng đời (Lifecycle)

### Trạng thái nhiệm vụ putaway (Putaway Task status)

| Trạng thái | Ý nghĩa | Chuyển tiếp | Ai |
|------------|---------|-------------|----|
| `Pending` | Đã sinh, chờ phân công | → `Assigned`, → `Cancelled` | System |
| `Assigned` | Đã giao cho NV kho | → `In Progress`, → `Pending` (trả lại) | Warehouse Staff / Manager |
| `In Progress` | NV kho đang di chuyển hàng | → `Completed`, → `On Hold` | Warehouse Staff |
| `On Hold` | Tạm dừng (không còn chỗ, chờ QC, chờ Manager) | → `In Progress`, → `Cancelled` | Warehouse Staff / Manager |
| `Completed` | Đã đặt hàng vào vị trí, tồn đã cập nhật | *(cuối)* | Warehouse Staff |
| `Cancelled` | Huỷ (hàng chuyển sang quy trình khác, ví dụ trả NCC) | *(cuối)* | Manager |

### Trạng thái hàng trong quá trình

| Trạng thái tồn | Ý nghĩa |
|----------------|---------|
| `Inbound` | Đã nhận, đang ở staging, **chưa khả dụng** để bán |
| `In Putaway` | Đang di chuyển (in-transit nội bộ) |
| `Available` | Đã ở vị trí lưu trữ, **khả dụng** để picking/bán |
| `Blocked` | Ở vị trí nhưng bị khoá (QC lại, kiểm kê) |

---

## 6. Ràng buộc nghiệp vụ (Business Rules) & Edge case

### Business rules

- **BR-01:** Chỉ hàng `Accepted` (đạt QC) ở Receipt mới sinh putaway task. Hàng `Quarantine`/`Rejected` không putaway vào vùng picking.
- **BR-02:** Tồn kho **luôn gắn với một vị trí**. Không tồn tại tồn "không vị trí" ngoài trạng thái quá cảnh (`In Putaway` / `In-Transit`).
- **BR-03:** Không được đặt hàng vào vị trí vượt sức chứa khai báo (capacity: theo thể tích, khối lượng, hoặc số ô).
- **BR-04:** Nếu bin cấu hình **single-SKU** hoặc **single-lot**, không được đặt SKU/lô khác vào khi bin chưa trống.
- **BR-05:** Với SKU FEFO (First-Expired-First-Out), gợi ý vị trí phải giúp lô hạn dùng gần được picking trước; không "chôn" lô cũ sau lô mới.
- **BR-06:** Putaway task đã `Completed` không sửa; điều chỉnh vị trí sau đó phải dùng [Intra-warehouse Transfer](../11-intra-warehouse-transfer/README.md).
- **BR-07:** Sai lệch giữa vị trí gợi ý và vị trí thực tế phải kèm lý do (mã lý do chuẩn hoá).

### Edge case

| Tình huống | Hướng xử lý |
|------------|-------------|
| Vị trí gợi ý đã bị chiếm (do task song song) | Slotting Engine gợi ý lại vị trí thay thế; hoặc NV kho chọn vị trí hợp lệ khác + lý do |
| Không còn chỗ cho SKU (kho đầy) | Task `On Hold`; cảnh báo Manager; xem xét [Inter-warehouse Transfer](../10-inter-warehouse-transfer/README.md) hoặc mở khu vực tràn (overflow) |
| Hàng cồng kềnh không vừa vị trí tiêu chuẩn | Đặt khu vực bulk/oversize; ghi đè gợi ý |
| Kiện hỗn hợp nhiều SKU | Tách task theo SKU; hoặc dùng vị trí "mixed" nếu cho phép |
| Mất mát/hư hỏng phát hiện khi putaway | Điều chỉnh giảm số lượng task + inventory adjustment + ghi log; thông báo Manager |
| Đặt nhầm vị trí và đã confirm | Sửa bằng Intra-warehouse Transfer, không sửa trực tiếp task |
| Hàng Quarantine sau đó `Accepted` | Sinh putaway task mới từ vùng cách ly vào vùng lưu trữ |

---

## 7. Liên kết với các module khác

| Module | Quan hệ |
|--------|---------|
| [03 · Receipt](../03-receipt/README.md) | Receipt `Confirmed` sinh putaway task; putaway xong → Receipt `Closed` |
| [06 · Warehouse Map & Slotting](../06-warehouse-map-slotting/README.md) | Cung cấp gợi ý vị trí; nhận feedback khi thực tế lệch gợi ý |
| [01 · Product Creation](../01-product-creation/README.md) | Kích thước/khối lượng/quy cách SKU là đầu vào cho quy tắc slotting |
| [07 · Picking](../07-picking/README.md) | Chất lượng putaway (vị trí hợp lý, FEFO đúng) quyết định hiệu suất & độ chính xác picking |
| [11 · Intra-warehouse Transfer](../11-intra-warehouse-transfer/README.md) | Công cụ để sửa/tối ưu vị trí sau khi putaway đã hoàn tất |
| [13 · Catalog](../../ecommerce/13-catalog/README.md) / [14 · Order](../../ecommerce/14-cart-and-order/README.md) | Putaway hoàn tất làm tăng tồn khả dụng (available-to-promise) hiển thị/bán trên sàn |
