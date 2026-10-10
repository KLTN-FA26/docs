# 03 — Nhận hàng (Receipt / Goods Receipt)

## 1. Mục đích (Purpose)

Ghi nhận việc **hàng hoá thực tế về đến kho** theo một [PO](../02-purchase-order/README.md): kiểm đếm số lượng, ghi nhận thông tin lô/hạn dùng/serial (nếu SKU yêu cầu), đưa hàng vào **khu nhận hàng (WH/Input — Area `RECEIVING`)**, và với SKU bắt buộc kiểm tra chất lượng thì **chuyển hàng sang khu kiểm tra chất lượng (WH/Quality Control — Area `QUALITY_CONTROL`)** để QC kết luận.

Nhập kho được tổ chức theo **luồng nhiều bước**, số bước phụ thuộc cờ **Yêu cầu QC khi nhận** của SKU ([01](../01-product-creation/README.md)):

| Luồng | Áp dụng cho | Các chặng |
|-------|-------------|-----------|
| **3 bước** | SKU bật cờ Yêu cầu QC | NCC → WH/Input (`RECEIVING`) → WH/Quality Control (`QUALITY_CONTROL`) → WH/Stock (bin) |
| **2 bước** | SKU không bật cờ | NCC → WH/Input (`RECEIVING`) → WH/Stock (bin) |

Module này bao phủ các chặng **đến hết kết luận QC**; chặng cuối lên bin là [Putaway](../05-putaway/README.md).

Receipt là **điểm hàng hoá chính thức thuộc quyền kiểm soát của kho** và là căn cứ để:
- Cập nhật tiến độ PO (đã nhận / còn chờ).
- Đối chiếu [hoá đơn NCC](../04-invoice/README.md) (three-way matching).
- Sinh nhiệm vụ [Putaway](../05-putaway/README.md) để cất hàng lên vị trí lưu trữ.

Hàng **gia công ngoài** trả về cũng nhận theo luồng này, tham chiếu **PO gia công**: đếm thành phẩm, và nếu mình đã cấp phôi thì đếm thêm phế phẩm và phôi chưa dùng trả về để **đối soát phôi** ([19 §4.7](../19-production/README.md)). Thành phẩm gia công không đi Putaway mà sang QC của LSX (trạng thái `QC`) rồi tới khu `PACKING`; phôi chưa dùng trả về đi luồng nhận thường.

> ⚠️ Giả định: Tồn kho khả dụng để bán (available-to-promise) chỉ được cộng **sau khi Putaway hoàn tất**, không phải ngay khi Receipt. Hàng ở khu nhận hàng và khu QC coi là "đã nhận, chưa khả dụng" (`Inbound`). Xem [open-questions](../../open-questions/README.md).

---

## 2. Actor liên quan

| Actor | Vai trò |
|-------|---------|
| **Nhân viên kho (Warehouse Staff / Receiver)** | Kiểm đếm, quét mã, nhập số lượng thực nhận, ghi lô/hạn dùng, chụp ảnh bằng chứng; chuyển hàng từ khu nhận hàng sang khu QC |
| **Nhân viên QC (QC Staff)** | Kiểm tra chất lượng hàng tại khu QC, kết luận đạt/cách ly/không đạt (accept / quarantine / reject) cho từng phần hàng |
| **Quản lý kho (Warehouse Manager)** | Xử lý chênh lệch (discrepancy), duyệt nhận vượt/thiếu ngoài dung sai |
| **Nhân viên mua hàng (Procurement)** | Được thông báo khi có chênh lệch hoặc hàng không đạt QC để làm việc với NCC |

---

## 3. Input / Output

### Input

| Trường | Mô tả | Bắt buộc |
|--------|-------|----------|
| PO tham chiếu | Một hoặc nhiều PO của cùng NCC | Có (trừ trường hợp nhận không PO — xem edge case) |
| Kho & khu nhận hàng | Area loại `RECEIVING` của kho (WH/Input, xem [06](../06-warehouse-map-slotting/README.md)) | Có |
| Khu kiểm tra chất lượng | Area loại `QUALITY_CONTROL` của kho (WH/Quality Control) | Có nếu phiếu có SKU bật cờ Yêu cầu QC |
| Phiếu giao hàng NCC (delivery note / packing slip) | Số phiếu, ngày | Không (khuyến nghị) |
| **Dòng nhận (receipt lines)** | SKU, số lượng thực nhận theo đơn vị quy đổi (ví dụ số thùng nguyên) và/hoặc đơn vị cơ sở (cái lẻ); hệ thống quy về đơn vị cơ sở | Có |
| Thông tin lô (batch/lot), hạn sử dụng, serial | Bắt buộc nếu SKU được cấu hình theo dõi (module 01) | Có/Không tuỳ SKU |
| Cờ Yêu cầu QC khi nhận | Lấy từ SKU (module 01); quyết định dòng nhận đi luồng 3 bước hay 2 bước | Có (mặc định: không yêu cầu) |
| Kết quả kiểm tra chất lượng | Số lượng `Accepted` / `Quarantine` / `Rejected` của từng phần hàng, lý do nếu không đạt; ghi tại khu QC | Có với dòng cần QC |
| Ảnh / chứng từ đính kèm | | Không |

### Output

| Kết quả | Dùng bởi |
|---------|----------|
| **Receipt number** (mã phiếu nhận) | 04 Invoice, 05 Putaway, truy vết |
| **Cập nhật PO**: SL đã nhận, open quantity, trạng thái PO | 02 PO |
| **Tồn `Inbound` ở khu nhận hàng** (Area `RECEIVING`) theo SKU + lô ngay khi Receipt `Confirmed` | 05 Putaway, báo cáo tồn |
| **Nhiệm vụ chuyển sang khu QC** (QC transfer task) + movement `RECEIVING` → `QUALITY_CONTROL` | NV kho, truy vết |
| **Kết quả QC** theo từng phần hàng (`Accepted` / `Quarantine` / `Rejected`) | 04 Invoice, 05 Putaway, hồ sơ chất lượng NCC |
| **Nhiệm vụ Putaway** (putaway task) cho hàng `Accepted` hoặc hàng không cần QC | 05 Putaway |
| **Tồn ở khu cách ly** (Area `QUARANTINE`) cho hàng `Quarantine` / `Rejected` | Warehouse Manager, Procurement (RTV) |
| **Bản ghi chênh lệch (discrepancy record)** nếu có | Procurement, Warehouse Manager |

---

## 4. Luồng nghiệp vụ từng bước (Step-by-step)

**Chặng 1 — Nhận hàng vào WH/Input**

1. **Tạo phiên nhận (Create receipt)** — NV kho chọn PO đang `Confirmed` / `Partially Received`. Hệ thống nạp danh sách dòng hàng còn *open* làm dòng nhận dự kiến.
2. **Đối chiếu phiếu giao NCC** — so số kiện, số phiếu giao với thực tế trên xe.
3. **Kiểm đếm & quét mã** — với mỗi SKU: quét barcode, nhập **số lượng thực nhận** (theo thùng nguyên và/hoặc cái lẻ). Hệ thống quy về đơn vị cơ sở rồi so với số lượng dự kiến của PO.
4. **Ghi nhận lô / hạn dùng / serial** — nếu SKU yêu cầu. Một dòng nhận có thể tách nhiều lô.
5. **Xử lý chênh lệch (discrepancy)**:
   - Nhận **thiếu**: ghi nhận số thực nhận; phần còn lại vẫn open trên PO.
   - Nhận **thừa** trong dung sai: chấp nhận.
   - Nhận **thừa ngoài dung sai** / sai SKU: cần Warehouse Manager duyệt ngoại lệ hoặc từ chối phần vượt.
6. **Xác nhận phiếu nhận (Confirm receipt)** — chốt số liệu. Hệ thống:
   - Cập nhật *SL đã nhận* & trạng thái PO (`Partially Received` / `Received`).
   - Ghi tồn `Inbound` vào **khu nhận hàng** (Area `RECEIVING`) theo (SKU, lô) bằng movement nhận hàng.
   - Với dòng **cần QC**: sinh **nhiệm vụ chuyển sang khu QC**.
   - Với dòng **không cần QC**: sinh **nhiệm vụ Putaway** từ khu `RECEIVING` (luồng 2 bước — bỏ qua chặng 2).

**Chặng 2 — Chuyển sang WH/Quality Control và kiểm QC** *(chỉ dòng cần QC)*

7. **Chuyển hàng sang khu QC** — NV kho nhận nhiệm vụ, quét mã vị trí nguồn (`RECEIVING`), mã SKU/lô và mã khu QC (`QUALITY_CONTROL`). Hệ thống ghi movement `RECEIVING` → `QUALITY_CONTROL`; tồn vẫn ở trạng thái `Inbound`.
8. **Kiểm tra chất lượng (QC)** — QC Staff kiểm hàng tại khu QC và chia số lượng của từng phần hàng thành: `Accepted`, `Quarantine` (cách ly chờ xử lý), `Rejected` (không đạt). Tổng ba phần bằng số lượng đã chuyển sang khu QC; phần `Quarantine` / `Rejected` bắt buộc có lý do.
9. **Xử lý theo kết luận QC**:
   - `Accepted` → sinh **nhiệm vụ Putaway** từ khu `QUALITY_CONTROL`.
   - `Quarantine` → chuyển sang **khu cách ly** (Area `QUARANTINE`), tồn chuyển trạng thái `Quarantine`; chờ kết luận lại.
   - `Rejected` → chuyển sang **khu cách ly** (Area `QUARANTINE`), tồn chuyển trạng thái `Blocked`; chờ trả NCC (RTV). Không sinh Putaway.

**Chặng 3 — Bàn giao và đóng phiếu**

10. **Bàn giao cho Putaway** — hàng `Accepted` (hoặc hàng không cần QC) chờ NV kho thực hiện [Putaway](../05-putaway/README.md) lên bin.
11. **Thông báo Invoice matching** — Receipt trở thành một vế trong three-way matching với [Invoice](../04-invoice/README.md); phần được đối chiếu thanh toán là phần không bị `Rejected`.
12. **Đóng phiếu nhận** — khi toàn bộ dòng đã xử lý (Putaway xong / `Rejected` đã trả NCC / `Quarantine` đã có kết luận) → `Closed`.

---

## 5. Trạng thái / Vòng đời (Lifecycle)

### 5.1. Trạng thái phiếu nhận (Receipt status)

| Trạng thái | Ý nghĩa | Chuyển tiếp | Ai |
|------------|---------|-------------|----|
| `Draft` | Đang kiểm đếm, chưa chốt số | → `Confirmed`, → `Cancelled` | Warehouse Staff |
| `Confirmed` | Đã chốt số lượng thực nhận; tồn `Inbound` đã vào khu nhận hàng; đã sinh nhiệm vụ chuyển QC và/hoặc Putaway task | → `In QC`, → `In Putaway` | Warehouse Staff |
| `In QC` | Còn ít nhất một dòng cần QC chưa có kết luận (đang chờ chuyển sang khu QC hoặc đang kiểm) | → `In Putaway`, → `Closed` | Warehouse Staff / QC Staff |
| `In Putaway` | Mọi dòng cần QC đã có kết luận; đang cất hàng lên vị trí lưu trữ | → `Closed` | Warehouse Staff |
| `Closed` | Mọi dòng đã xử lý xong | *(cuối)* | System / Manager |
| `Cancelled` | Huỷ khi chưa Confirmed | *(cuối)* | Warehouse Staff |

`Confirmed` đi thẳng `In Putaway` khi phiếu không có dòng nào cần QC. `In QC` đi thẳng `Closed` khi không còn phần hàng `Accepted` nào phải putaway.

### 5.2. Trạng thái chất lượng của hàng nhận (per lot)

| Trạng thái | Ý nghĩa | Hệ quả |
|------------|---------|--------|
| `Not Required` | SKU không bật cờ Yêu cầu QC | Sinh Putaway task từ khu `RECEIVING` ngay khi Receipt `Confirmed` |
| `Pending` | Cần QC, chưa có kết luận (chờ chuyển sang khu QC hoặc đang kiểm) | Tồn `Inbound`; không Putaway |
| `Accepted` | Đạt QC | Sinh Putaway task từ khu `QUALITY_CONTROL`; sẽ thành tồn khả dụng sau Putaway |
| `Quarantine` | Cách ly chờ xử lý | Chuyển sang Area `QUARANTINE`, tồn `Quarantine`; chờ kết luận `Accepted` hoặc `Rejected` |
| `Rejected` | Không đạt | Chuyển sang Area `QUARANTINE`, tồn `Blocked`; trả NCC (return to vendor); không nhập tồn khả dụng |

### 5.3. Trạng thái nhiệm vụ chuyển sang khu QC (QC transfer task)

| Trạng thái | Ý nghĩa | Chuyển tiếp | Ai |
|------------|---------|-------------|----|
| `Pending` | Đã sinh khi Receipt `Confirmed`, chờ NV kho nhận | → `In Progress`, → `Cancelled` | System |
| `In Progress` | NV kho đang chuyển hàng | → `Completed` | Warehouse Staff |
| `Completed` | Hàng đã nằm ở khu QC, movement đã ghi; phần hàng chờ QC Staff kiểm | *(cuối)* | Warehouse Staff |
| `Cancelled` | Huỷ (ví dụ Receipt reversal) | *(cuối)* | Manager |

---

## 6. Ràng buộc nghiệp vụ (Business Rules) & Edge case

### Business rules

- **BR-01:** Không tạo Receipt cho PO chưa `Confirmed`.
- **BR-02:** Tổng SL nhận luỹ kế của một dòng PO ≤ *SL đặt × (1 + over-receipt tolerance)*. Vượt → chặn hoặc yêu cầu duyệt.
- **BR-03:** SKU cấu hình theo lô/hạn dùng/serial → **bắt buộc** nhập đủ thông tin tương ứng mới cho `Confirmed`.
- **BR-04:** Hàng chưa qua Putaway (`Inbound` ở khu nhận hàng hoặc khu QC), hàng `Quarantine` và hàng `Rejected` **không** được tính vào tồn khả dụng bán.
- **BR-05:** Receipt đã `Confirmed` không được xoá; sai sót xử lý bằng **điều chỉnh tồn có kiểm soát (inventory adjustment)** hoặc **receipt reversal** kèm lý do và phê duyệt.
- **BR-06:** Hạn sử dụng nhập vào phải > ngày nhận (cảnh báo nếu quá gần hạn theo ngưỡng cấu hình).
- **BR-07:** Dòng nhận của SKU bật cờ Yêu cầu QC **không** được sinh Putaway task trước khi có kết luận QC; hàng chỉ được putaway từ khu `QUALITY_CONTROL`, không đi tắt từ khu `RECEIVING` lên bin.
- **BR-08:** QC chỉ được kết luận cho hàng **đã nằm ở khu QC** (nhiệm vụ chuyển sang khu QC `Completed`). Tổng `Accepted` + `Quarantine` + `Rejected` phải bằng số lượng đã chuyển sang khu QC; phần `Quarantine` / `Rejected` bắt buộc có lý do.
- **BR-09:** Mọi lần hàng đổi chỗ trong luồng nhận (khu nhận hàng → khu QC → khu cách ly / bin) đều phải ghi **movement** có chứng từ nguồn là Receipt; không sửa tồn trực tiếp.
- **BR-10:** Phiếu nhận của PO gia công mà mình đã cấp phôi chỉ `Confirmed` khi đã ghi đủ thành phẩm, phế phẩm và phôi trả về; hao hụt vượt mức cho phép phải được Quản lý kho duyệt ([19](../19-production/README.md) BR-17).

### Edge case

| Tình huống | Hướng xử lý |
|------------|-------------|
| Hàng về không có PO (nhận khẩn, hàng mẫu, quà kèm) | Cho phép "Receipt không PO" với lý do; cần duyệt; sau đó gắn PO bổ sung nếu có |
| Một chuyến giao nhiều PO | Một phiên nhận xử lý nhiều PO cùng NCC; mỗi dòng vẫn map đúng PO gốc |
| Sai SKU trên phiếu giao NCC | Nhận theo SKU thực tế; nếu SKU không có trong PO → xử lý như thừa/ngoại lệ |
| Thùng hỏng, nghi hụt bên trong | Ghi nhận có điều kiện (conditional receipt); dòng đó **bắt buộc đi qua khu QC** dù SKU không bật cờ Yêu cầu QC (⚠️ giả định — [open-questions C15](../../open-questions/README.md)); mở khiếu nại NCC |
| NCC giao lô có hạn dùng ngắn hơn cam kết | Cảnh báo; Manager quyết định nhận/từ chối |
| Kiểm đếm lại sau khi Confirmed phát hiện lệch | Inventory adjustment tại khu đang chứa hàng (khu nhận hàng hoặc khu QC), ghi log, thông báo Procurement |
| Số lượng đếm được tại khu QC khác số đã chuyển | QC Staff không kết luận; báo Warehouse Manager xử lý bằng inventory adjustment trước khi kiểm |
| Khu QC chưa được khai báo nhưng phiếu có SKU cần QC | Chặn `Confirmed`; Admin phải khai báo Area `QUALITY_CONTROL` cho kho ([06](../06-warehouse-map-slotting/README.md)) |
| Hàng Quarantine cuối cùng bị Rejected | Tồn chuyển `Blocked` tại khu cách ly; chốt trả NCC; cập nhật hồ sơ chất lượng NCC |
| Hàng Quarantine sau đó `Accepted` | Sinh Putaway task từ khu `QUARANTINE` lên bin |
| Hàng `Rejected` đã trả NCC | Movement xuất khỏi kho làm giảm tồn `Blocked` tại khu cách ly; dòng nhận được coi là đã xử lý |
| Nhà gia công trả thiếu thành phẩm / thiếu phôi | Ghi số thực nhận; đối soát phôi ra hao hụt; vượt mức → Quản lý kho duyệt, trừ vào hoá đơn gia công |

---

## 7. Liên kết với các module khác

| Module | Quan hệ |
|--------|---------|
| [02 · Purchase Order](../02-purchase-order/README.md) | Receipt tham chiếu PO; cập nhật SL đã nhận & trạng thái PO |
| [04 · Invoice](../04-invoice/README.md) | Receipt là vế "hàng thực nhận" trong three-way matching PO ↔ Receipt ↔ Invoice; phần `Rejected` không được thanh toán |
| [05 · Putaway](../05-putaway/README.md) | Sinh nhiệm vụ Putaway cho hàng `Accepted` (nguồn: khu `QUALITY_CONTROL`) và hàng không cần QC (nguồn: khu `RECEIVING`) |
| [01 · Product Creation](../01-product-creation/README.md) | Cấu hình theo lô/hạn dùng/serial và cờ Yêu cầu QC của SKU quyết định dữ liệu phải ghi khi nhận và luồng 2 bước hay 3 bước |
| [06 · Warehouse Map & Slotting](../06-warehouse-map-slotting/README.md) | Cung cấp khu nhận hàng (`RECEIVING`), khu QC (`QUALITY_CONTROL`), khu cách ly (`QUARANTINE`); sau Putaway, hàng nằm ở vị trí do slotting gợi ý |
| [19 · Production](../19-production/README.md) | Nhận hàng gia công về; phiếu nhận `Confirmed` đưa LSX con sang `QC` |
| [10 · Inter-warehouse Transfer](../10-inter-warehouse-transfer/README.md) | Nhận hàng đến từ kho khác cũng dùng luồng Receipt (nguồn là Transfer Order thay vì PO) |
