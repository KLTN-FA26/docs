# 07 — Lấy hàng (Picking)

## 1. Mục đích (Purpose)

Lấy đúng **SKU, số lượng, lô** từ các vị trí lưu trữ trong kho để **gom thành đơn hàng chuẩn bị giao**, theo yêu cầu từ [Order Management](../../ecommerce/17-order-management/README.md) / lệnh xuất kho.

Picking là **bước mở đầu luồng Fulfillment** (Picking → [Packing](../08-packing/README.md) → [Shipping](../09-shipping/README.md)). Chất lượng picking quyết định độ chính xác đơn hàng (order accuracy) và tốc độ giao.

---

## 2. Actor liên quan

| Actor | Vai trò |
|-------|---------|
| **Nhân viên kho (Picker)** | Nhận pick list, di chuyển theo lộ trình, lấy hàng, quét xác nhận SKU/vị trí/số lượng |
| **Hệ thống (Wave/Route Planner)** | Gom đơn thành đợt (wave/batch), tối ưu lộ trình, phân bổ (allocate) tồn theo vị trí & lô |
| **Quản lý kho (Warehouse Manager)** | Cấu hình chiến lược picking, xử lý thiếu hàng (short pick), phân công lại |
| **Nhân viên Packing** | Nhận hàng đã pick tại khu đóng gói |

---

## 3. Input / Output

### Input

| Trường | Mô tả | Bắt buộc |
|--------|-------|----------|
| Lệnh xuất / đơn cần giao | Order number + dòng hàng (SKU, số lượng) + hạn giao | Có |
| Chiến lược picking | Single-order / batch / zone / wave (cấu hình) | Có |
| Phân bổ tồn (allocation) | Hệ thống chọn vị trí & lô nguồn theo FEFO/FIFO & khoảng cách | Có |
| Thiết bị (handheld scanner, xe đẩy, tote) | | Không |
| Ưu tiên đơn | Hạn giao gần, đơn hoả tốc, khách VIP | Không |

### Output

| Kết quả | Dùng bởi |
|---------|----------|
| **Pick list / pick task** đã hoàn tất | 08 Packing |
| **Cập nhật tồn**: giảm tồn tại vị trí nguồn; chuyển sang trạng thái `Picked` / `Staged` | Tồn kho, 17 Order Management |
| **Hàng đã gom** tại khu đóng gói (staging outbound) | 08 Packing |
| **Bản ghi short pick / thiếu hàng** nếu có | Manager, 17 Order Management |
| Số liệu năng suất (lines/hour, độ chính xác) & velocity theo SKU | 06 Slotting (re-slotting) |

---

## 4. Luồng nghiệp vụ từng bước (Step-by-step)

1. **Nhận yêu cầu xuất** — [Order Management](../../ecommerce/17-order-management/README.md) chuyển các đơn `Confirmed` / `Ready to Fulfill` sang kho.
2. **Phân bổ tồn (Allocation)** — hệ thống phân bổ số lượng cần lấy tại vị trí & lô cụ thể theo FEFO/FIFO. Tồn chuyển `Available` → `Allocated`, không khả dụng cho đơn khác. Phần giữ hàng (reservation) của đơn từ [14](../../ecommerce/14-cart-and-order/README.md) được chuyển thành allocation ở bước này.
3. **Lập đợt lấy hàng (Wave / Batch planning)** — gom nhiều đơn theo tiêu chí (khu vực, hạn giao, loại đóng gói); sinh **pick list** và **lộ trình tối ưu** (giảm quãng đường).
4. **Chọn chiến lược:**
   | Chiến lược | Mô tả |
   |------------|-------|
   | Discrete / single-order | Một picker lấy trọn một đơn |
   | Batch picking | Lấy nhiều đơn cùng lúc, tách ra sau |
   | Zone picking | Mỗi picker phụ trách một khu; đơn đi qua nhiều khu |
   | Wave picking | Thả đợt theo lịch (theo chuyến xe của hãng vận chuyển) |
5. **Phân công picker** — pick task vào hàng đợi của từng NV.
6. **Đi lấy hàng theo lộ trình** — tại mỗi chặng: quét **mã vị trí** → quét **SKU/lô** → nhập/quét **số lượng lấy** → đặt vào tote/thùng. Hệ thống kiểm tra khớp phân bổ.
7. **Xử lý thiếu hàng (Short pick)** — nếu vị trí không đủ:
   - Hệ thống thử **phân bổ lại** từ vị trí/lô khác.
   - Nếu vẫn thiếu: ghi nhận short pick; đơn chuyển trạng thái chờ (backorder) hoặc giao một phần (partial) theo chính sách; thông báo Order Management & Manager; kích hoạt kiểm kê vị trí lệch.
8. **Gom về khu đóng gói (Staging)** — picker mang hàng đã lấy đến khu outbound staging; quét bàn giao. Hàng chuyển trạng thái `Picked` → `Staged`.
9. **Đóng pick task** — khi mọi dòng đã lấy đủ (hoặc chốt short). Sinh/nối tiếp **Packing task** cho đơn.

---

## 5. Trạng thái / Vòng đời (Lifecycle)

### 5.1. Trạng thái pick task

| Trạng thái | Ý nghĩa | Chuyển tiếp | Ai |
|------------|---------|-------------|----|
| `Created` | Đã sinh từ allocation, chờ lập đợt | → `Released`, → `Cancelled` | System |
| `Released` | Đã vào wave/batch, chờ phân công | → `Assigned` | System / Manager |
| `Assigned` | Đã giao cho picker | → `In Progress`, → `Released` (trả lại) | Picker / Manager |
| `In Progress` | Picker đang lấy hàng | → `Completed`, → `Short` , → `On Hold` | Picker |
| `Short` | Không lấy đủ, chờ xử lý (re-allocate / backorder) | → `In Progress` (có hàng), → `Completed` (chốt giao một phần) | Manager |
| `On Hold` | Tạm dừng | → `In Progress`, → `Cancelled` | Manager |
| `Completed` | Đã lấy đủ & gom về staging | *(cuối)* | Picker |
| `Cancelled` | Đơn bị huỷ / gộp lại | *(cuối)* | Manager |

### 5.2. Trạng thái tồn trong luồng

| Trạng thái | Ý nghĩa |
|------------|---------|
| `Available` | Khả dụng, chưa gán đơn |
| `Allocated` | Đã phân bổ cho một đơn/lệnh cụ thể tại vị trí & lô, còn ở vị trí |
| `Picked` | Đã lấy khỏi vị trí, đang trên đường ra staging |
| `Staged` | Đã ở khu đóng gói, chờ Packing |

---

## 6. Ràng buộc nghiệp vụ (Business Rules) & Edge case

### Business rules

- **BR-01:** Chỉ picking khi đơn đã `Confirmed` và thanh toán hợp lệ theo chính sách (ví dụ đơn prepaid phải `Paid`; đơn COD được phép).  Xem [15 Payment](../../ecommerce/15-payment/README.md).
- **BR-02:** Chỉ được lấy từ tồn `Available` và được phân bổ; không lấy từ hàng `Blocked` / `Quarantine` / `Allocated cho đơn khác`.
- **BR-03:** SKU FEFO → phải lấy lô hạn dùng gần nhất trước; lệch phải có lý do.
- **BR-04:** Số lượng lấy không vượt số lượng phân bổ của dòng; muốn lấy thêm phải re-allocate.
- **BR-05:** Hàng đã `Picked`/`Staged` mà đơn bị huỷ → phải làm **put-back** (trả về vị trí) qua quy trình có kiểm soát, không "biến mất".
- **BR-06:** Sản phẩm in ấn theo đơn: hạng mục cần sản xuất/in trước khi picking thành phẩm; picking chỉ áp cho vật tư/thành phẩm đã sẵn. (Xem edge case.)

### Edge case

| Tình huống | Hướng xử lý |
|------------|-------------|
| Vị trí báo có hàng nhưng thực tế trống | Short pick → re-allocate → tạo yêu cầu cycle count cho vị trí đó |
| Hàng hỏng phát hiện khi lấy | Không lấy; đánh dấu `Blocked` tại vị trí + adjustment; re-allocate lô khác |
| Đơn hoả tốc chen vào giữa wave | Cho phép tạo pick task ưu tiên, chèn lộ trình |
| Khách đổi/huỷ đơn khi đang pick | Order Management gửi tín hiệu; task `On Hold`/`Cancelled`; hàng đã lấy làm put-back |
| Một SKU nằm rải nhiều vị trí/lô | Phân bổ gộp nhiều dòng lấy; gom trong cùng tote |
| Sản phẩm tuỳ chỉnh chưa in xong | Đơn chờ ở bước sản xuất; picking chỉ release khi thành phẩm sẵn sàng (hoặc pick vật tư cho xưởng in) |
| Lấy dư do quét nhầm | Hệ thống chặn khi số lượng > phân bổ; nếu đã xảy ra → điều chỉnh + put-back |

> ⚠️ Giả định: Với sản phẩm in ấn theo đơn (print-on-demand), giữa Order và Picking có **bước sản xuất/in** (production/print job). Đề bài không mô tả module sản xuất riêng — xem [open-questions](../../open-questions/README.md).

---

## 7. Liên kết với các module khác

| Module | Quan hệ |
|--------|---------|
| [17 · Order Management](../../ecommerce/17-order-management/README.md) | Nguồn đơn cần xuất; nhận cập nhật picked/short/partial |
| [15 · Payment](../../ecommerce/15-payment/README.md) | Điều kiện thanh toán quyết định đơn có được release để pick hay không |
| [05 · Putaway](../05-putaway/README.md) / [06 · Slotting](../06-warehouse-map-slotting/README.md) | Vị trí & FEFO do putaway/slotting quyết định; velocity từ picking phản hồi lại slotting |
| [08 · Packing](../08-packing/README.md) | Nhận hàng đã pick tại staging; pick task hoàn tất → packing task |
| [11 · Intra-warehouse Transfer](../11-intra-warehouse-transfer/README.md) | Replenishment: bổ sung hàng từ vùng dự trữ (reserve) xuống vùng picking khi cạn |
| [00 · System Overview §5](../../00-system-overview/README.md) | Bước 1 của luồng Fulfillment |
