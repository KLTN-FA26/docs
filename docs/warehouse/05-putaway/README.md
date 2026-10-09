# 05 — Chuyển hàng vào kho (Putaway)

## 1. Mục đích (Purpose)

Di chuyển hàng hoá từ **khu chờ cất** sau khi [nhận hàng](../03-receipt/README.md) — **khu kiểm tra chất lượng (Area `QUALITY_CONTROL`)** với hàng đã đạt QC, hoặc **khu nhận hàng (Area `RECEIVING`)** với SKU không yêu cầu QC — đến **vị trí lưu trữ chính thức (storage location / bin / slot)** trong kho, và **cập nhật tồn kho theo vị trí** để hàng trở nên **khả dụng cho picking / bán hàng**.

Putaway quyết định *hàng nằm ở đâu* — ảnh hưởng trực tiếp đến hiệu suất [Picking](../07-picking/README.md), mật độ lưu trữ, và an toàn kho. Module dựa vào **[Gợi ý vị trí (Slotting Suggestion)](../06-warehouse-map-slotting/README.md)** để đề xuất vị trí tối ưu.

Về nghiệp vụ, Putaway được thiết kế theo mô hình WMS phổ biến như Odoo/MISA: **Receipt/nhận hàng** đưa hàng vào khu chờ (và qua khu QC nếu SKU yêu cầu), sau đó **Putaway/nhập kho nội bộ** mới đưa hàng lên bin chính thức. Putaway là **chặng cuối** của luồng nhập kho 3 bước (WH/Input → WH/Quality Control → WH/Stock) hoặc 2 bước (WH/Input → WH/Stock). Thuật toán chỉ là lớp gợi ý; quyết định cuối cùng vẫn được xác nhận bằng thao tác quét thực tế tại kho.

> Nguyên tắc dữ liệu: thao tác vật lý thường xử lý theo **thùng/kiện/pallet** (handling unit), nhưng tồn kho hệ thống vẫn ghi nhận theo **đơn vị cơ sở của SKU** (hệ số quy đổi khai báo ở [01](../01-product-creation/README.md)). Ví dụ: nhận 4 thùng × 1.000 cái thì task có thể hướng dẫn theo 4 thùng, còn `InventoryStock` lưu 4.000 cái.

---

## 2. Actor liên quan

| Actor | Vai trò |
|-------|---------|
| **Nhân viên kho (Warehouse Staff)** | Nhận hàng, bàn giao phần hàng `Accepted` ở khu QC (hoặc hàng không cần QC ở khu nhận hàng) cho luồng putaway, nhận nhiệm vụ putaway, di chuyển hàng, quét mã hàng/kiện/vị trí, xác nhận số lượng thực cất |
| **Nhân viên QC** (nếu có) | Xác nhận hàng `Accepted` / `Quarantine` / `Rejected`; chỉ hàng `Accepted` được đưa vào vùng lưu trữ khả dụng |
| **Quản lý kho (Warehouse Manager / Thủ kho)** | Phân công, ưu tiên task, xử lý hết chỗ/lệch tồn, duyệt override đặc biệt, mở/khoá vị trí |
| **Admin / System Config** | Khai báo kho & bản đồ kho (zone, kệ, tầng, bin, khu vực), sức chứa, mã vị trí, lớp lưu trữ — chi tiết ở [06](../06-warehouse-map-slotting/README.md) |
| **Hệ thống (WMS / Slotting Engine)** | Sinh gợi ý vị trí, kiểm tra ràng buộc cứng, khoá mềm vị trí khi task được nhận, tạo movement log |
| **Inventory Controller** (nếu có) | Kiểm kê vị trí, xử lý sai lệch tồn phát hiện trong/ sau putaway |

> Trong MVP, `Warehouse Staff` là role vận hành chính cho cả nhận hàng và putaway. `QC` và `Inventory Controller` chỉ tách riêng nếu kho cần kiểm soát chất lượng/kiểm kê độc lập.

---

## 3. Input / Output

### Input

| Trường | Mô tả | Bắt buộc |
|--------|-------|----------|
| Nhiệm vụ putaway (putaway task) | Sinh từ Receipt: khi QC kết luận `Accepted` (SKU cần QC), hoặc khi Receipt `Confirmed` (SKU không cần QC) | Có |
| SKU, số lượng cơ sở, lô/hạn dùng/serial | Từ Receipt; số lượng lưu theo đơn vị cơ sở của SKU | Có |
| Quy cách xử lý (handling unit) | Thùng/kiện/pallet, số lượng mỗi kiện, kích thước, khối lượng | Có nếu quản lý theo kiện |
| Vị trí nguồn (from location) | Khu QC (Area `QUALITY_CONTROL`) với hàng đạt QC; khu nhận hàng (Area `RECEIVING`) với SKU không cần QC; khu cách ly (Area `QUARANTINE`) với hàng được kết luận lại `Accepted` | Có |
| Vị trí đích gợi ý (suggested to location) | Từ Slotting Engine | Không (nếu tắt gợi ý thì nhập tay) |
| Vị trí đích thực tế (actual to location) | NV kho quét khi đặt hàng | Có |
| Ràng buộc lưu trữ | Lớp lưu trữ (storage class) của SKU & bin; sức chứa bin theo đơn vị; tải trọng tối đa & chiều cao lọt của tầng kệ | Có nếu SKU/vị trí có cấu hình |
| Thiết bị nâng hạ / ràng buộc thao tác | Ví dụ hàng nặng chỉ để tầng thấp, cần xe nâng, không đưa vào lối hẹp | Không |
| Mã lý do override | Bắt buộc khi vị trí thực tế khác vị trí gợi ý | Có nếu override |

### Output

| Kết quả | Dùng bởi |
|---------|----------|
| **Cập nhật tồn kho theo vị trí** (location inventory) | 06 Map, 07 Picking, báo cáo tồn |
| **Tồn khả dụng để bán (available stock)** tăng lên | 13 Catalog / 14 Order (ATP) |
| **Putaway task `Completed`** + lịch sử di chuyển (StockMovement / movement log) | Truy vết, 03 Receipt (chuyển `In Putaway` → `Closed`) |
| **Cảnh báo chênh lệch gợi ý vs thực tế** | 06 Slotting (feedback loop) |
| **Exception / On Hold reason** nếu không thể cất hàng | Manager, Inventory Controller |

---

## 4. Luồng nghiệp vụ từng bước (Step-by-step)

1. **Sinh nhiệm vụ putaway** — hệ thống tạo putaway task khi phần hàng sẵn sàng cất: SKU cần QC → khi QC kết luận `Accepted` tại khu `QUALITY_CONTROL`; SKU không cần QC → ngay khi [Receipt](../03-receipt/README.md) `Confirmed`. Task có thể tách theo SKU, lô, serial, kiện hoặc pallet để khớp thao tác thực tế.
2. **Chuẩn hoá số lượng xử lý** — hệ thống quy đổi giữa đơn vị vận hành và đơn vị tồn kho: ví dụ `quantityBase = 4.000 cái`, `packageQty = 1.000 cái/thùng`, `handlingUnits = 4 thùng`.
3. **Gợi ý vị trí (slotting suggestion)** — [Slotting Engine](../06-warehouse-map-slotting/README.md) đề xuất vị trí đích dựa trên:
   - Ràng buộc cứng: bin cho phép cất & đang `Active`, lớp lưu trữ, sức chứa, kích thước, tải trọng & chiều cao lọt của tầng.
   - Tiêu chí mềm: gom cùng SKU/lô, velocity/ABC, quãng đường đi thực tế đến khu đóng gói, FEFO/FIFO, cân bằng tải, độ lấp đầy.
4. **Phân công & ưu tiên** — task được đẩy vào hàng đợi hoặc gán cho Warehouse Staff. Manager có thể ưu tiên hàng bán chạy, hàng sắp hết hạn, hàng đang chặn dock/lối đi.
5. **Nhận task & khoá mềm vị trí** — khi nhân viên nhận task hoặc chấp nhận gợi ý, hệ thống giữ tạm sức chứa của vị trí trong một khoảng thời gian để tránh hai task song song cùng dùng một bin.
6. **Lấy hàng ở khu chờ cất** — nhân viên quét mã task, mã vị trí nguồn (khu `QUALITY_CONTROL` hoặc `RECEIVING`), mã SKU/lô/kiện và xác nhận số lượng lấy khỏi khu đó.
7. **Di chuyển đến vị trí đích** — đi theo vị trí/lộ trình được gợi ý; với kho lớn có thể qua nhiều chặng trung gian.
8. **Đặt hàng & quét xác nhận vị trí** — quét mã vị trí (location barcode). Hệ thống kiểm tra:
   - Bin có cho phép cất, đang `Active` và đúng lớp lưu trữ của SKU không.
   - Còn sức chứa (capacity) không; tầng kệ có vượt tải trọng / chiều cao lọt không.
9. **Xử lý sai lệch vị trí** — nếu đặt khác vị trí gợi ý, hệ thống chỉ cho xác nhận khi vị trí mới không vi phạm ràng buộc cứng. Nhân viên phải chọn mã lý do như: vị trí thực tế đã đầy, kệ đang bảo trì, hàng không vừa, ưu tiên gom lô, hoặc chỉ đạo từ Manager.
10. **Cập nhật tồn kho bằng movement** — hệ thống không sửa trực tiếp số `onHand`; phải tạo movement từ vị trí nguồn (Area `QUALITY_CONTROL` / `RECEIVING`) sang bin lưu trữ; tồn chuyển `Inbound` → `Available`. Tồn khả dụng để bán tăng sau khi hàng đã nằm ở vị trí cho phép picking.
11. **Hoàn tất task** — putaway task `Completed`. Khi mọi task của một Receipt đã xử lý xong, Receipt có thể chuyển `Closed`.
12. **Putaway nhiều chặng (multi-step)** — với kho lớn: chặng 1 đưa từ khu nhận hàng ra một khu vực trung gian gần kệ đích, chặng 2 cất lên bin cuối. Mỗi chặng nên là task con hoặc `PutawayTaskLeg` để truy vết người làm, thời gian và vị trí.

### Các luồng biến thể

| Luồng | Khi dùng | Cách xử lý |
|-------|----------|------------|
| **Luồng 2 bước** | SKU không bật cờ Yêu cầu QC | Receipt vào khu `RECEIVING` → Putaway lên bin chính thức |
| **Luồng 3 bước (có QC)** | SKU bật cờ Yêu cầu QC | Receipt vào khu `RECEIVING` → chuyển sang khu `QUALITY_CONTROL` và kiểm QC (module 03) → `Accepted` sinh putaway từ khu QC; `Quarantine` / `Rejected` chuyển sang khu cách ly (Area `QUARANTINE`), không putaway |
| **Override vị trí** | Gợi ý không đúng thực tế | Cho chọn vị trí khác nếu không vi phạm ràng buộc cứng, bắt buộc mã lý do |
| **Không có bin hợp lệ** | Kho đầy, sai cấu hình, hàng quá khổ | Task `On Hold`, cảnh báo Manager; Manager chỉ định khu tràn (Area `OVERFLOW`) hoặc tạo transfer |
| **Split nhiều vị trí** | Một bin không chứa đủ toàn bộ kiện | Hệ thống có thể đề xuất tách qua nhiều bin hợp lệ; đây là năng lực nâng cao của hệ thống, không nên coi là mặc định của mọi WMS |

---

## 5. Trạng thái / Vòng đời (Lifecycle)

### Trạng thái nhiệm vụ putaway (Putaway Task status)

| Trạng thái | Ý nghĩa | Chuyển tiếp | Ai |
|------------|---------|-------------|----|
| `Pending` | Đã sinh, chờ phân công | → `Assigned`, → `Cancelled` | System |
| `Assigned` | Đã giao cho NV kho | → `In Progress`, → `Pending` (trả lại) | Warehouse Staff / Manager |
| `In Progress` | NV kho đang xử lý; vị trí có thể đã được khoá mềm | → `Partially Completed`, → `Completed`, → `On Hold` | Warehouse Staff |
| `Partially Completed` | Đã cất xong một phần dòng/kiện, còn phần chưa xử lý | → `In Progress`, → `On Hold`, → `Completed` | Warehouse Staff |
| `On Hold` | Tạm dừng do exception: không còn chỗ, chờ QC, lệch số lượng, chờ Manager | → `In Progress`, → `Cancelled` | Warehouse Staff / Manager |
| `Completed` | Đã đặt hàng vào vị trí, tồn đã cập nhật | *(cuối)* | Warehouse Staff |
| `Cancelled` | Huỷ (hàng chuyển sang quy trình khác, ví dụ trả NCC) | *(cuối)* | Manager |

### Trạng thái hàng trong quá trình

| Trạng thái tồn | Ý nghĩa |
|----------------|---------|
| `Inbound` | Đã nhận, đang ở khu nhận hàng (Area `RECEIVING`) hoặc khu QC (Area `QUALITY_CONTROL`), **chưa khả dụng** để bán |
| `In Putaway` | Đang di chuyển (in-transit nội bộ) |
| `Available` | Đã ở vị trí lưu trữ, **khả dụng** để picking/bán |
| `Blocked` | Ở vị trí nhưng bị khoá (QC lại, kiểm kê) |

### Dữ liệu tối thiểu cần lưu

| Thực thể | Trường chính |
|----------|--------------|
| `PutawayTask` | `taskNo`, `receiptId`, `warehouseId`, `status`, `priority`, `assignedTo`, `startedAt`, `completedAt`, `holdReason` |
| `PutawayTaskLine` | `skuId`, `lotId`, `serialIds`, `quantityBase`, `handlingUnitType`, `handlingUnitQty`, `fromLocationId`, `suggestedLocationId`, `actualLocationId`, `overrideReason` |
| `StockMovement` | `sourceDocument`, `movementType = PUTAWAY`, `fromLocationId`, `toLocationId`, `skuId`, `lotId`, `quantityBase`, `performedBy`, `performedAt` |
| `SlottingFeedback` | `suggestedLocationId`, `actualLocationId`, `score`, `reasonCode`, `acceptedSuggestion`, `createdAt` |

Các trường `*LocationId` trỏ tới một **Bin** hoặc một **Area chứa hàng** (xem [06](../06-warehouse-map-slotting/README.md)).

---

## 6. Ràng buộc nghiệp vụ (Business Rules) & Edge case

### Business rules

- **BR-01:** Chỉ hàng `Accepted` (đạt QC) hoặc hàng của SKU không yêu cầu QC mới sinh putaway task. Hàng chờ QC, `Quarantine`, `Rejected` không putaway vào vùng picking.
- **BR-02:** Tồn kho **luôn gắn với một vị trí** — một **Bin** hoặc một **Area chứa hàng**. Không tồn tại tồn "không vị trí" ngoài trạng thái quá cảnh (`In Putaway` / `In-Transit`).
- **BR-03:** Không được đặt hàng vượt sức chứa khai báo: số đơn vị tối đa của bin (`capacity_units`); tải trọng tối đa & chiều cao lọt của tầng kệ. Trường nào không khai báo thì không kiểm tra.
- **BR-04:** Chỉ cất vào bin **cho phép cất hàng** (`is_putaway_bin`), đang `Active`, có **lớp lưu trữ hiệu lực trùng lớp lưu trữ của SKU**. Một bin có thể chứa nhiều SKU/lô nếu còn sức chứa.
- **BR-05:** Với SKU FEFO (First-Expired-First-Out), gợi ý vị trí phải giúp lô hạn dùng gần được picking trước; không "chôn" lô cũ sau lô mới.
- **BR-06:** Putaway task đã `Completed` không sửa; điều chỉnh vị trí sau đó phải dùng [Intra-warehouse Transfer](../11-intra-warehouse-transfer/README.md).
- **BR-07:** Sai lệch giữa vị trí gợi ý và vị trí thực tế phải kèm lý do (mã lý do chuẩn hoá).
- **BR-08:** Khi nhân viên chấp nhận gợi ý, sức chứa vị trí nên được khoá mềm trong thời hạn cấu hình; quá hạn hoặc huỷ task thì giải phóng.
- **BR-09:** Override chỉ được phép với tiêu chí mềm. Ràng buộc cứng như sai lớp lưu trữ (nhiệt độ, hàng nguy hiểm, dễ vỡ, quá khổ), quá tải, bin không cho phép cất hoặc không `Active` phải bị chặn.
- **BR-10:** Split một task qua nhiều bin được phép nếu cấu hình hỗ trợ, nhưng mỗi phần split vẫn phải có dòng/movement riêng để truy vết.
- **BR-11:** Nếu phát hiện lệch số lượng tại khu nhận hàng hoặc tại bin đích, không tự sửa task đã chốt; tạo adjustment/cycle count có phê duyệt.
- **BR-12:** Available-to-promise chỉ tăng khi hàng đã putaway vào vị trí được phép xuất bán; hàng ở inbound, quarantine, blocked hoặc đang di chuyển không được bán.

### Đối chiếu chuẩn tham chiếu

| Chủ đề | Cách thiết kế trong hệ thống | Ghi chú so với Odoo/MISA |
|--------|------------------------------|---------------------------|
| Nhập kho nhiều bước | 3 bước: khu nhận hàng → khu QC → bin; 2 bước: khu nhận hàng → bin; chọn theo cờ Yêu cầu QC của SKU | Khớp mô hình nhận hàng 2/3 bước (Input → Quality Control → Stock) của Odoo; khác Odoo ở chỗ số bước chọn theo SKU thay vì theo kho; MISA cũng hỗ trợ quy trình nhập/xuất nhiều bước |
| Gợi ý vị trí | Slotting Engine lọc ràng buộc cứng, sau đó chấm điểm tiêu chí mềm | Tương tự tinh thần Odoo Putaway Rules/Storage Categories; công thức điểm là thiết kế riêng |
| Capacity | Kiểm tra số đơn vị tối đa của bin, tải trọng & chiều cao lọt của tầng kệ, kích thước bin | Odoo có storage category/capacity; hệ thống cần tự định nghĩa rõ cách tính phần còn trống |
| Override thủ công | Cho phép đổi vị trí nếu không vi phạm ràng buộc cứng và có lý do | Phù hợp thực tế vận hành barcode/WMS: người làm kho xác nhận theo thực địa |
| Split nhiều bin | Có thể hỗ trợ khi một bin không đủ chứa | Đây là năng lực nâng cao của hệ thống; không nên mô tả là hành vi mặc định của Odoo |
| FIFO/FEFO | Putaway phải hỗ trợ picking đúng FIFO/FEFO về sau | Khớp chiến lược xuất kho phổ biến trong Odoo/MISA |

### Edge case

| Tình huống | Hướng xử lý |
|------------|-------------|
| Vị trí gợi ý đã bị chiếm (do task song song) | Slotting Engine gợi ý lại vị trí thay thế; hoặc NV kho chọn vị trí hợp lệ khác + lý do |
| Không còn chỗ cho SKU (kho đầy) | Task `On Hold` kèm reason; cảnh báo Manager; xem xét [Inter-warehouse Transfer](../10-inter-warehouse-transfer/README.md) hoặc mở khu vực tràn (overflow) |
| Hàng cồng kềnh không vừa vị trí tiêu chuẩn | Cất vào bin lớp lưu trữ `OVERSIZE`; nếu không có, Manager chỉ định khu tràn (Area `OVERFLOW`) |
| Kiện hỗn hợp nhiều SKU | Tách task theo SKU; các SKU có thể cất chung một bin nếu cùng lớp lưu trữ và còn sức chứa |
| Mất mát/hư hỏng phát hiện khi putaway | Điều chỉnh giảm số lượng task + inventory adjustment + ghi log; thông báo Manager |
| Đặt nhầm vị trí và đã confirm | Sửa bằng Intra-warehouse Transfer, không sửa trực tiếp task |
| Hàng Quarantine sau đó `Accepted` | Sinh putaway task mới từ khu cách ly (Area `QUARANTINE`) vào vùng lưu trữ |
| Gợi ý hết hạn trước khi nhân viên tới kệ | Tính lại gợi ý theo tồn/capacity hiện tại trước khi cho xác nhận |
| Hàng phải putaway theo pallet nhưng app đang chia theo cái | Hiển thị task theo handling unit, movement vẫn ghi `quantityBase` để tồn kho chính xác |

---

## 7. Liên kết với các module khác

| Module | Quan hệ |
|--------|---------|
| [03 · Receipt](../03-receipt/README.md) | Kết luận QC `Accepted` (hoặc Receipt `Confirmed` với SKU không cần QC) sinh putaway task; putaway xong → Receipt `Closed` |
| [06 · Warehouse Map & Slotting](../06-warehouse-map-slotting/README.md) | Cung cấp gợi ý vị trí, kiểm tra hard constraints, khoá mềm capacity, nhận feedback khi thực tế lệch gợi ý; khu `QUALITY_CONTROL` / `RECEIVING` là vị trí nguồn |
| [01 · Product Creation](../01-product-creation/README.md) | Kích thước/khối lượng/quy cách SKU là đầu vào cho quy tắc slotting |
| [07 · Picking](../07-picking/README.md) | Chất lượng putaway (vị trí hợp lý, FEFO đúng) quyết định hiệu suất & độ chính xác picking |
| [11 · Intra-warehouse Transfer](../11-intra-warehouse-transfer/README.md) | Công cụ để sửa/tối ưu vị trí sau khi putaway đã hoàn tất |
| [13 · Catalog](../../ecommerce/13-catalog/README.md) / [14 · Order](../../ecommerce/14-cart-and-order/README.md) | Putaway hoàn tất làm tăng tồn khả dụng (available-to-promise) hiển thị/bán trên sàn |
