# 03 — Nhận hàng (Receipt / Goods Receipt)

## 1. Mục đích (Purpose)

Ghi nhận việc **hàng hoá thực tế về đến kho** theo một [PO](../02-purchase-order/README.md): kiểm đếm số lượng, kiểm tra chất lượng sơ bộ, ghi nhận thông tin lô/hạn dùng/serial (nếu SKU yêu cầu), và đưa hàng vào **khu vực chờ cất (staging / inbound area)**.

Receipt là **điểm hàng hoá chính thức thuộc quyền kiểm soát của kho** và là căn cứ để:
- Cập nhật tiến độ PO (đã nhận / còn chờ).
- Đối chiếu [hoá đơn NCC](../04-invoice/README.md) (three-way matching).
- Sinh nhiệm vụ [Putaway](../05-putaway/README.md) để cất hàng lên vị trí lưu trữ.

> ⚠️ Giả định: Tồn kho khả dụng để bán (available-to-promise) chỉ được cộng **sau khi Putaway hoàn tất**, không phải ngay khi Receipt. Hàng ở inbound area coi là "đã nhận, chưa khả dụng". Xem [open-questions](../../open-questions/README.md).

---

## 2. Actor liên quan

| Actor | Vai trò |
|-------|---------|
| **Nhân viên kho (Warehouse Staff / Receiver)** | Kiểm đếm, quét mã, nhập số lượng thực nhận, ghi lô/hạn dùng, chụp ảnh bằng chứng |
| **Nhân viên QC** (nếu có) | Kiểm tra chất lượng, đánh dấu đạt/không đạt (accept / reject / quarantine) |
| **Quản lý kho (Warehouse Manager)** | Xử lý chênh lệch (discrepancy), duyệt nhận vượt/thiếu ngoài dung sai |
| **Nhân viên mua hàng (Procurement)** | Được thông báo khi có chênh lệch để làm việc với NCC |

---

## 3. Input / Output

### Input

| Trường | Mô tả | Bắt buộc |
|--------|-------|----------|
| PO tham chiếu | Một hoặc nhiều PO của cùng NCC | Có (trừ trường hợp nhận không PO — xem edge case) |
| Kho & khu vực nhận (dock / inbound zone) | | Có |
| Phiếu giao hàng NCC (delivery note / packing slip) | Số phiếu, ngày | Không (khuyến nghị) |
| **Dòng nhận (receipt lines)** | SKU, số lượng thực nhận, đơn vị | Có |
| Thông tin lô (batch/lot), hạn sử dụng, serial | Bắt buộc nếu SKU được cấu hình theo dõi (module 01) | Có/Không tuỳ SKU |
| Kết quả kiểm tra chất lượng | Đạt / Không đạt / Cách ly | Không (mặc định: Đạt) |
| Ảnh / chứng từ đính kèm | | Không |

### Output

| Kết quả | Dùng bởi |
|---------|----------|
| **Receipt number** (mã phiếu nhận) | 04 Invoice, 05 Putaway, truy vết |
| **Cập nhật PO**: SL đã nhận, open quantity, trạng thái PO | 02 PO |
| **Tồn ở inbound area** (theo SKU + lô + trạng thái QC) | 05 Putaway |
| **Nhiệm vụ Putaway** (putaway task) | 05 Putaway |
| **Bản ghi chênh lệch (discrepancy record)** nếu có | Procurement, Warehouse Manager |

---

## 4. Luồng nghiệp vụ từng bước (Step-by-step)

1. **Tạo phiên nhận (Create receipt)** — NV kho chọn PO đang `Confirmed` / `Partially Received`. Hệ thống nạp danh sách dòng hàng còn *open* làm dòng nhận dự kiến.
2. **Đối chiếu phiếu giao NCC** — so số kiện, số phiếu giao với thực tế trên xe.
3. **Kiểm đếm & quét mã** — với mỗi SKU: quét barcode, nhập **số lượng thực nhận**. Hệ thống so với số lượng dự kiến.
4. **Ghi nhận lô / hạn dùng / serial** — nếu SKU yêu cầu. Một dòng nhận có thể tách nhiều lô.
5. **Kiểm tra chất lượng (QC)** — đánh dấu từng phần: `Accepted`, `Rejected`, hoặc `Quarantine` (cách ly chờ xử lý). Phần `Rejected` không tạo Putaway.
6. **Xử lý chênh lệch (discrepancy)**:
   - Nhận **thiếu**: ghi nhận số thực nhận; phần còn lại vẫn open trên PO.
   - Nhận **thừa** trong dung sai: chấp nhận.
   - Nhận **thừa ngoài dung sai** / sai SKU: cần Warehouse Manager duyệt ngoại lệ hoặc từ chối phần vượt.
7. **Xác nhận phiếu nhận (Confirm receipt)** — chốt số liệu. Hệ thống:
   - Cập nhật *SL đã nhận* & trạng thái PO (`Partially Received` / `Received`).
   - Ghi tồn vào **inbound area** theo (SKU, lô, trạng thái QC).
   - Sinh **nhiệm vụ Putaway** cho phần `Accepted`.
8. **Bàn giao cho Putaway** — hàng nằm ở inbound area chờ NV kho thực hiện [Putaway](../05-putaway/README.md).
9. **Thông báo Invoice matching** — Receipt trở thành một vế trong three-way matching với [Invoice](../04-invoice/README.md).
10. **Đóng phiếu nhận** — khi toàn bộ dòng đã xử lý (Putaway xong / Rejected đã trả NCC / Quarantine đã có kết luận) → `Closed`.

---

## 5. Trạng thái / Vòng đời (Lifecycle)

### 5.1. Trạng thái phiếu nhận (Receipt status)

| Trạng thái | Ý nghĩa | Chuyển tiếp | Ai |
|------------|---------|-------------|----|
| `Draft` | Đang kiểm đếm, chưa chốt số | → `Confirmed`, → `Cancelled` | Warehouse Staff |
| `Confirmed` | Đã chốt số lượng thực nhận; tồn đã vào inbound area; đã sinh Putaway task | → `In Putaway`, → `Closed` | Warehouse Staff |
| `In Putaway` | Đang cất hàng lên vị trí lưu trữ | → `Closed` | Warehouse Staff |
| `Closed` | Mọi dòng đã xử lý xong | *(cuối)* | System / Manager |
| `Cancelled` | Huỷ khi chưa Confirmed | *(cuối)* | Warehouse Staff |

### 5.2. Trạng thái chất lượng của hàng nhận (per lot)

| Trạng thái | Ý nghĩa | Hệ quả |
|------------|---------|--------|
| `Accepted` | Đạt QC | Sinh Putaway task; sẽ thành tồn khả dụng sau Putaway |
| `Quarantine` | Cách ly chờ xử lý | Không Putaway vào vùng picking; chờ kết luận `Accepted` hoặc `Rejected` |
| `Rejected` | Không đạt | Trả NCC / huỷ; không nhập tồn khả dụng; có thể phát sinh trả hàng NCC (return to vendor) |

---

## 6. Ràng buộc nghiệp vụ (Business Rules) & Edge case

### Business rules

- **BR-01:** Không tạo Receipt cho PO chưa `Confirmed`.
- **BR-02:** Tổng SL nhận luỹ kế của một dòng PO ≤ *SL đặt × (1 + over-receipt tolerance)*. Vượt → chặn hoặc yêu cầu duyệt.
- **BR-03:** SKU cấu hình theo lô/hạn dùng/serial → **bắt buộc** nhập đủ thông tin tương ứng mới cho `Confirmed`.
- **BR-04:** Hàng `Rejected` / `Quarantine` **không** được tính vào tồn khả dụng bán.
- **BR-05:** Receipt đã `Confirmed` không được xoá; sai sót xử lý bằng **điều chỉnh tồn có kiểm soát (inventory adjustment)** hoặc **receipt reversal** kèm lý do và phê duyệt.
- **BR-06:** Hạn sử dụng nhập vào phải > ngày nhận (cảnh báo nếu quá gần hạn theo ngưỡng cấu hình).

### Edge case

| Tình huống | Hướng xử lý |
|------------|-------------|
| Hàng về không có PO (nhận khẩn, hàng mẫu, quà kèm) | Cho phép "Receipt không PO" với lý do; cần duyệt; sau đó gắn PO bổ sung nếu có |
| Một chuyến giao nhiều PO | Một phiên nhận xử lý nhiều PO cùng NCC; mỗi dòng vẫn map đúng PO gốc |
| Sai SKU trên phiếu giao NCC | Nhận theo SKU thực tế; nếu SKU không có trong PO → xử lý như thừa/ngoại lệ |
| Thùng hỏng, nghi hụt bên trong | Ghi nhận có điều kiện (conditional receipt), chuyển `Quarantine`, mở khiếu nại NCC |
| NCC giao lô có hạn dùng ngắn hơn cam kết | Cảnh báo; Manager quyết định nhận/từ chối |
| Kiểm đếm lại sau khi Confirmed phát hiện lệch | Inventory adjustment tại inbound area, ghi log, thông báo Procurement |
| Hàng Quarantine cuối cùng bị Rejected | Chốt trả NCC; điều chỉnh giảm tồn inbound; cập nhật hồ sơ chất lượng NCC |

---

## 7. Liên kết với các module khác

| Module | Quan hệ |
|--------|---------|
| [02 · Purchase Order](../02-purchase-order/README.md) | Receipt tham chiếu PO; cập nhật SL đã nhận & trạng thái PO |
| [04 · Invoice](../04-invoice/README.md) | Receipt là vế "hàng thực nhận" trong three-way matching PO ↔ Receipt ↔ Invoice |
| [05 · Putaway](../05-putaway/README.md) | Receipt `Confirmed` sinh nhiệm vụ Putaway cho phần hàng `Accepted` |
| [01 · Product Creation](../01-product-creation/README.md) | Cấu hình theo lô/hạn dùng/serial của SKU quyết định dữ liệu phải ghi khi nhận |
| [06 · Warehouse Map & Slotting](../06-warehouse-map-slotting/README.md) | Sau Putaway, hàng nằm ở vị trí do slotting gợi ý |
| [10 · Inter-warehouse Transfer](../10-inter-warehouse-transfer/README.md) | Nhận hàng đến từ kho khác cũng dùng luồng Receipt (nguồn là Transfer Order thay vì PO) |
