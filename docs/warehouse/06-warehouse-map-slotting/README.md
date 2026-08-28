# 06 — Kho Map 2D/3D & Gợi ý vị trí (Warehouse Map & Slotting Suggestion)

## 1. Mục đích (Purpose)

Cung cấp **mô hình không gian của kho** (sơ đồ 2D và trực quan 3D) và **động cơ gợi ý vị trí lưu trữ (slotting / putaway recommendation engine)**:

- **Warehouse Map 2D/3D:** biểu diễn cấu trúc kho (khu vực → dãy → kệ → tầng → ô/bin) kèm trạng thái lấp đầy, giúp NV kho định vị nhanh và quản lý trực quan.
- **Slotting Suggestion:** với mỗi lần [Putaway](../05-putaway/README.md) hoặc [Intra-warehouse Transfer](../11-intra-warehouse-transfer/README.md), đề xuất vị trí **tối ưu** dựa trên bộ quy tắc (rules) và dữ liệu (kích thước SKU, tần suất xuất, chỗ trống, ràng buộc tương thích, FIFO/FEFO).

Mục tiêu: giảm quãng đường di chuyển, tăng mật độ lưu trữ hợp lý, đảm bảo an toàn và tuân thủ FEFO, giảm phụ thuộc kinh nghiệm cá nhân.

> ⚠️ Giả định lớn: Đề bài **không nêu tiêu chí cụ thể** của thuật toán gợi ý. Tài liệu này đề xuất một bộ tiêu chí mặc định (bên dưới) — **cần Product Owner xác nhận trọng số & thứ tự ưu tiên**. Xem [open-questions](../../open-questions/README.md).

---

## 2. Actor liên quan

| Actor | Vai trò |
|-------|---------|
| **Admin / Quản lý kho** | Khai báo cấu trúc kho, thuộc tính vị trí, cấu hình quy tắc slotting & trọng số |
| **Hệ thống (Slotting Engine)** | Tính toán và trả về gợi ý theo thời điểm putaway/transfer |
| **Nhân viên kho** | Xem map để định vị; nhận & xác nhận (hoặc ghi đè) gợi ý |
| **Quản lý kho** | Chạy phân tích tối ưu định kỳ (re-slotting), duyệt di chuyển tối ưu hoá |

---

## 3. Input / Output

### Input

| Nhóm | Dữ liệu | Nguồn |
|------|---------|-------|
| **Cấu trúc kho** | Khu vực (zone), dãy (aisle), kệ (rack), tầng (level), ô (bin/slot); toạ độ; loại vị trí (picking / bulk / cold / hazmat / oversize) | Admin khai báo |
| **Thuộc tính vị trí** | Sức chứa (thể tích, khối lượng tối đa, số ô), chiều cao lọt, quy tắc single-SKU/single-lot, độ ưu tiên picking (khoảng cách đến khu đóng gói) | Admin |
| **Thuộc tính SKU** | Kích thước, khối lượng, quy cách đóng gói, nhóm hàng, phân loại nhiệt độ, dễ vỡ/nguy hiểm | [01 Product Creation](../01-product-creation/README.md) |
| **Dữ liệu vận hành** | Tần suất xuất (order frequency / velocity), phân loại ABC/XYZ, tồn hiện tại theo vị trí, lô & hạn dùng | 03, 05, 07, tồn kho |
| **Ràng buộc tương thích** | Cặp SKU/nhóm phải để gần / cấm để chung (ví dụ thực phẩm ≠ hoá chất) | Admin |

### Output

| Kết quả | Dùng bởi |
|---------|----------|
| **Bản đồ kho 2D/3D tương tác** (heatmap độ lấp đầy, tìm vị trí, click xem chi tiết bin) | Warehouse Staff, Manager |
| **Gợi ý vị trí (ranked suggestions)** cho một putaway/transfer: danh sách vị trí + điểm số + lý do | 05 Putaway, 11 Intra-transfer |
| **Báo cáo tối ưu / đề xuất re-slotting** (di chuyển SKU bán chạy ra gần khu đóng gói…) | Manager |
| **Chỉ số sức khoẻ kho** (utilization %, honeycombing, số vị trí quá tải) | Manager |

---

## 4. Luồng nghiệp vụ từng bước (Step-by-step)

### 4.1. Thiết lập & duy trì bản đồ kho

1. Admin khai báo phân cấp không gian: Kho → Zone → Aisle → Rack → Level → Bin. Mỗi bin có mã (location barcode) duy nhất.
2. Gán thuộc tính cho từng loại vị trí: sức chứa, loại hàng cho phép, quy tắc trộn, độ ưu tiên picking.
3. Sinh mô hình 2D (mặt bằng) và 3D (khối kệ) từ toạ độ đã khai báo.
4. Duy trì: thêm/khoá vị trí khi thay đổi layout; đánh dấu vị trí bảo trì/hỏng.

### 4.2. Gợi ý vị trí khi Putaway / Transfer

1. **Nhận yêu cầu** — module 05/11 gọi Slotting Engine kèm: SKU, số lượng, lô/hạn dùng, kho, vị trí nguồn.
2. **Lọc vị trí hợp lệ (hard constraints)** — loại bỏ vị trí:
   - Không đúng loại hàng (nhiệt độ, hazmat, oversize).
   - Không đủ sức chứa còn lại.
   - Vi phạm single-SKU/single-lot.
   - Vi phạm cặp cấm-để-chung.
3. **Chấm điểm vị trí còn lại (soft scoring)** theo tiêu chí (trọng số cấu hình):
   | Tiêu chí | Ý nghĩa |
   |----------|---------|
   | Gần khu đóng gói / vị trí xuất | SKU bán chạy (A) ưu tiên vị trí gần, tầng thấp (golden zone) |
   | Gom cùng SKU | Ưu tiên bin đã chứa cùng SKU để giảm phân mảnh |
   | FEFO/FIFO | Chọn vị trí giúp lô hạn dùng gần được lấy trước |
   | Cân bằng tải | Tránh dồn quá nhiều vào một dãy gây tắc nghẽn |
   | Khối lượng/công thái học | Hàng nặng → tầng thấp |
   | Mật độ lưu trữ | Ưu tiên lấp đầy vị trí đang dùng dở trước khi mở vị trí mới |
4. **Trả về danh sách xếp hạng** (top N) kèm điểm và lý do ("gần dock, cùng SKU, còn 60% chỗ").
5. **NV kho chọn** — chấp nhận gợi ý #1, chọn phương án khác trong danh sách, hoặc **ghi đè** bằng vị trí tự nhập kèm **mã lý do**.
6. **Ghi nhận kết quả thực tế** — engine lưu (gợi ý vs thực tế, lý do lệch) để phân tích và tinh chỉnh quy tắc.

### 4.3. Tối ưu định kỳ (Re-slotting)

1. Manager chạy phân tích: SKU nào đổi hạng ABC, SKU bán chạy đang nằm xa, vị trí honeycombing.
2. Engine đề xuất danh sách di chuyển tối ưu (move list).
3. Manager duyệt → sinh các [Intra-warehouse Transfer](../11-intra-warehouse-transfer/README.md) tương ứng.

---

## 5. Trạng thái / Vòng đời

Module này **không có chứng từ vòng đời riêng**. Các thực thể liên quan:

| Thực thể | Trạng thái |
|----------|-----------|
| **Vị trí (Location/Bin)** | `Active` · `Full` (đầy) · `Blocked` (khoá kiểm kê/QC) · `Maintenance` (hỏng/bảo trì) · `Inactive` (bỏ khỏi layout) |
| **Gợi ý (Suggestion)** | `Generated` → `Accepted` / `Overridden` / `Expired` (không dùng trong thời hạn) |
| **Đề xuất re-slotting** | `Draft` → `Approved` → `Executed` (đã sinh transfer) / `Rejected` |

---

## 6. Ràng buộc nghiệp vụ (Business Rules) & Edge case

### Business rules

- **BR-01:** Hard constraints (nhiệt độ, hazmat, sức chứa, single-SKU/lot, cặp cấm) **không bao giờ bị vi phạm** kể cả khi ghi đè — nếu NV kho nhập vị trí vi phạm hard constraint, hệ thống chặn.
- **BR-02:** Soft scoring có thể bị ghi đè bởi NV kho **kèm mã lý do**.
- **BR-03:** Gợi ý phải phản ánh tồn kho & sức chứa **tại thời điểm gọi**; gợi ý cũ quá thời hạn (ví dụ > X phút) phải tính lại.
- **BR-04:** Mọi thay đổi cấu trúc/thuộc tính kho được version hoá; không xoá vị trí còn tồn > 0.
- **BR-05:** Bản đồ 3D là công cụ trực quan, **không phải nguồn sự thật của tồn** — nguồn sự thật là dữ liệu tồn kho theo vị trí.

### Edge case

| Tình huống | Hướng xử lý |
|------------|-------------|
| Không có vị trí nào thoả hard constraints | Trả kết quả rỗng + lý do; đề xuất mở overflow zone hoặc Inter-warehouse Transfer |
| Hai putaway song song cùng nhắm một bin | Khoá mềm (soft-reserve) bin khi gợi ý được `Accepted`; task sau tính lại |
| SKU mới chưa có dữ liệu velocity | Dùng mặc định theo nhóm hàng / phân hạng tạm; cập nhật sau khi có lịch sử |
| Kích thước SKU khai báo sai → gợi ý vị trí không vừa | NV kho ghi đè + lý do "kích thước sai"; hệ thống cảnh báo cập nhật lại module 01 |
| Layout kho thay đổi giữa ca | Khoá vị trí ảnh hưởng thành `Maintenance`; task đang chờ được gợi ý lại |
| Bản đồ 3D lệch thực địa | Công cụ đối soát định kỳ (cycle count theo vị trí); ưu tiên số liệu kiểm kê |

---

## 7. Liên kết với các module khác

| Module | Quan hệ |
|--------|---------|
| [05 · Putaway](../05-putaway/README.md) | Nhận gợi ý vị trí khi cất hàng; gửi lại kết quả thực tế |
| [11 · Intra-warehouse Transfer](../11-intra-warehouse-transfer/README.md) | Nhận gợi ý vị trí đích khi di chuyển nội bộ / re-slotting |
| [07 · Picking](../07-picking/README.md) | Vị trí do slotting quyết định → ảnh hưởng lộ trình & tốc độ picking; velocity từ picking là đầu vào re-slotting |
| [01 · Product Creation](../01-product-creation/README.md) | Cung cấp kích thước/khối lượng/nhóm hàng/phân loại nhiệt độ của SKU |
| [10 · Inter-warehouse Transfer](../10-inter-warehouse-transfer/README.md) | Khi kho hết chỗ, đề xuất chuyển bớt sang kho khác |
| [00 · System Overview](../../00-system-overview/README.md) | Là lớp hỗ trợ xuyên suốt luồng lưu kho |
