# 01 — Tạo sản phẩm (Product Creation)

## 1. Mục đích (Purpose)

Khai báo và duy trì **dữ liệu gốc về sản phẩm (product master data)** — nguồn dữ liệu dùng chung cho toàn hệ thống: mua hàng, quản lý tồn kho, vận hành kho, và hiển thị trên catalog TMĐT.

Đây là bước **khởi tạo vòng đời sản phẩm**. Không thể lập PO, nhận hàng, hay đăng bán một mặt hàng chưa được tạo ở module này.

Với sản phẩm **tùy chỉnh / in ấn**, module này còn khai báo **khả năng cá nhân hoá** (customization capability): vùng in (print area), chất liệu, khung 3D dùng để xem trước, các tuỳ chọn biến thể.

---

## 2. Actor liên quan

| Actor | Vai trò trong module |
|-------|----------------------|
| **Quản trị hệ thống (System Admin)** | Tạo mới, phê duyệt, xuất bản (publish) sản phẩm; cấu hình danh mục, thuộc tính, quy tắc sinh SKU |
| **Quản trị sàn (E-commerce Admin)** | Nhập nội dung mô tả, hình ảnh, thông tin marketing; đề xuất tạo sản phẩm |
| **Nhân viên mua hàng (Procurement)** | Tham chiếu (read-only) để lập PO; có thể yêu cầu bổ sung thông tin nhà cung cấp/đơn vị mua |

> ⚠️ Giả định: Có quy trình phê duyệt (maker–checker) giữa người tạo và người xuất bản. Nếu tổ chức nhỏ, System Admin làm cả hai vai. Xem [open-questions](../../open-questions/README.md).

---

## 3. Input / Output

### Input

| Nhóm | Trường dữ liệu (ví dụ) | Bắt buộc |
|------|------------------------|----------|
| **Định danh** | Tên sản phẩm, mã sản phẩm nội bộ (product code), danh mục (category), thương hiệu | Có |
| **Thuộc tính biến thể (variant attributes)** | Kích cỡ, màu sắc, chất liệu… → tổ hợp sinh ra các **SKU** | Có (ít nhất 1 SKU) |
| **Thông số vật lý (logistics data)** | Khối lượng, kích thước (D×R×C), đơn vị tính (UoM), quy cách đóng gói (pack size) | Có (phục vụ slotting, packing, tính phí ship) |
| **Thông tin mua hàng** | Nhà cung cấp mặc định, giá mua tham chiếu, lead time, MOQ | Không (khuyến nghị) |
| **Kiểm soát tồn kho** | Có theo dõi lô (batch/lot)? Có hạn sử dụng? Có serial? Ngưỡng tồn tối thiểu/tối đa | Không (mặc định: không lô, không serial) |
| **Dữ liệu TMĐT** | Mô tả, hình ảnh, giá bán, thuộc tính SEO, trạng thái hiển thị | Không (bắt buộc trước khi publish lên catalog) |
| **Dữ liệu tùy chỉnh (personalization)** | Vùng in (print areas) & toạ độ, chất liệu in, template 2D, **model 3D** để live preview, ràng buộc thiết kế (kích thước file, hệ màu) | Chỉ với sản phẩm in ấn |

### Output

| Kết quả | Mô tả | Được dùng bởi |
|---------|-------|---------------|
| **Product record** (bản ghi sản phẩm) | Thực thể sản phẩm + danh sách SKU con | Toàn hệ thống |
| **SKU** | Mã hàng hoá nhỏ nhất để quản lý tồn & giao dịch | 02 PO, 03 Receipt, 05 Putaway, 07 Picking, 10/11 Transfer |
| **Cấu hình tùy chỉnh** | Bộ tham số cho trình thiết kế 2D + preview 3D | 12 Product Design |
| **Bản nháp catalog** | Nội dung hiển thị chờ publish | 13 Catalog |

---

## 4. Luồng nghiệp vụ từng bước (Step-by-step)

1. **Khởi tạo bản nháp (Create draft)** — System Admin / Quản trị sàn nhập thông tin định danh + danh mục. Sản phẩm ở trạng thái `Draft`.
2. **Khai báo thuộc tính biến thể** — chọn các trục biến thể (ví dụ: Size × Color). Hệ thống sinh **ma trận biến thể**; người dùng chọn tổ hợp hợp lệ để tạo SKU.
3. **Sinh mã SKU** — theo quy tắc cấu hình (prefix danh mục + mã thuộc tính + số thứ tự) hoặc nhập tay. Mỗi SKU phải **duy nhất toàn hệ thống**.
4. **Nhập thông số logistics** cho từng SKU (hoặc kế thừa từ sản phẩm cha): khối lượng, kích thước, pack size, UoM.
5. **Cấu hình kiểm soát tồn kho** — bật/tắt theo lô, hạn dùng, serial; đặt ngưỡng min/max phục vụ cảnh báo bổ sung tồn (reorder).
6. **(Sản phẩm in ấn) Cấu hình tùy chỉnh** — tải template 2D, model 3D, khai báo print area, ràng buộc file thiết kế.
7. **Nhập dữ liệu TMĐT** — mô tả, hình ảnh, giá bán, thuộc tính hiển thị.
8. **Gửi duyệt (Submit for review)** — chuyển sang `Pending Approval`.
9. **Phê duyệt (Approve)** — người có quyền kiểm tra tính đầy đủ & đúng đắn → `Approved`. Nếu không đạt → trả về `Draft` kèm lý do.
10. **Kích hoạt / Xuất bản (Activate / Publish)**:
    - `Active` (nội bộ): SKU sẵn sàng cho nghiệp vụ kho (PO, Receipt…).
    - `Published` (TMĐT): sản phẩm hiển thị trên catalog cho khách.
11. **Bảo trì (Maintain)** — cập nhật thông tin, thêm/bớt biến thể, điều chỉnh giá; các thay đổi nhạy cảm (thông số logistics, quy tắc tùy chỉnh) có thể yêu cầu duyệt lại.
12. **Ngừng kinh doanh (Discontinue)** — chuyển `Inactive` / `Discontinued`: ẩn khỏi catalog, chặn PO mới; vẫn cho xử lý tồn còn lại và đơn đang chạy.

---

## 5. Trạng thái / Vòng đời (Lifecycle)

### 5.1. Trạng thái sản phẩm (Product status)

| Trạng thái | Ý nghĩa | Chuyển tiếp cho phép | Ai thực hiện |
|------------|---------|----------------------|--------------|
| `Draft` | Đang soạn, chưa đầy đủ | → `Pending Approval` | Người tạo |
| `Pending Approval` | Chờ phê duyệt | → `Approved` / → `Draft` (từ chối) | Người duyệt |
| `Approved` | Đã duyệt, chưa dùng | → `Active` | Người duyệt / System Admin |
| `Active` | Dùng được cho nghiệp vụ nội bộ (kho, mua hàng) | → `Published`, → `Inactive` | System Admin |
| `Published` | Đang hiển thị trên catalog TMĐT | → `Active` (ẩn khỏi catalog), → `Inactive` | Quản trị sàn |
| `Inactive` | Tạm ngừng, không hiển thị, không PO mới | → `Active` | System Admin |
| `Discontinued` | Ngừng vĩnh viễn | *(trạng thái cuối)* | System Admin |

### 5.2. Trạng thái SKU

| Trạng thái | Ý nghĩa |
|------------|---------|
| `Active` | Cho phép giao dịch nhập/xuất/chuyển |
| `Blocked` | Tạm khoá (ví dụ nghi vấn chất lượng) — không xuất bán, vẫn có thể kiểm kê |
| `Obsolete` | Ngừng dùng — chỉ cho xuất/điều chỉnh để làm sạch tồn |

---

## 6. Ràng buộc nghiệp vụ (Business Rules) & Edge case

### Business rules

- **BR-01:** SKU là **bất biến (immutable)** sau khi đã phát sinh giao dịch tồn kho. Chỉ được đổi thuộc tính mô tả, không đổi mã hay ý nghĩa biến thể.
- **BR-02:** Không thể xoá sản phẩm/SKU đã có: giao dịch tồn, PO, hoặc đơn hàng tham chiếu. Chỉ được `Inactive` / `Discontinued`.
- **BR-03:** Thông số logistics (khối lượng, kích thước) là **bắt buộc trước khi** SKU được đưa vào Receipt/Putaway — vì module 06 Slotting và 08 Packing phụ thuộc dữ liệu này.
- **BR-04:** Chỉ sản phẩm `Published` mới hiển thị trên catalog; chỉ sản phẩm `Active`/`Published` mới được đưa vào PO.
- **BR-05:** Với sản phẩm in ấn, **bắt buộc** có ít nhất một print area và một template preview (2D hoặc 3D) trước khi `Published`.
- **BR-06:** Giá bán TMĐT và giá mua là hai trường độc lập; module này chỉ lưu, không tính margin.

### Edge case cần lưu ý

| Tình huống | Hướng xử lý đề xuất |
|------------|---------------------|
| Thêm biến thể mới vào sản phẩm đang `Published` | Biến thể mới bắt đầu ở `Draft`/`Approved`, publish riêng; không ảnh hưởng SKU hiện có |
| Gộp/tách SKU do sai khai báo ban đầu | Không sửa trực tiếp; tạo SKU mới + quy trình điều chỉnh tồn có kiểm soát, khoá SKU cũ thành `Obsolete` |
| Sản phẩm bị `Discontinued` nhưng còn tồn kho | Cho phép Picking/Transfer/Adjustment để giải phóng tồn; chặn Receipt & PO mới |
| Đổi đơn vị tính (UoM) sau khi đã có tồn | Không cho đổi; yêu cầu tạo SKU mới với UoM đúng |
| Template 3D lỗi/thiếu | Chặn `Published`; cho phép `Active` để vẫn nhập kho phục vụ kênh khác |
| Trùng mã SKU khi nhập tay | Hệ thống chặn ngay tại bước tạo (unique constraint) |

---

## 7. Liên kết với các module khác

| Module | Quan hệ |
|--------|---------|
| [02 · Purchase Order](../02-purchase-order/README.md) | PO chỉ chọn được SKU `Active`; kế thừa NCC mặc định, giá mua tham chiếu, MOQ, lead time từ đây |
| [03 · Receipt](../03-receipt/README.md) | Nhận hàng theo SKU; cấu hình lô/hạn dùng/serial quyết định thông tin phải ghi nhận khi nhận |
| [05 · Putaway](../05-putaway/README.md) & [06 · Slotting](../06-warehouse-map-slotting/README.md) | Dùng thông số kích thước/khối lượng SKU để gợi ý vị trí phù hợp |
| [07 · Picking](../07-picking/README.md) / [08 · Packing](../08-packing/README.md) | Pack size & kích thước SKU ảnh hưởng gợi ý thùng và cách đóng gói |
| [12 · Product Design](../../ecommerce/12-product-design/README.md) | Lấy template 2D, model 3D, định nghĩa print area & ràng buộc thiết kế từ đây |
| [13 · Catalog](../../ecommerce/13-catalog/README.md) | Lấy nội dung hiển thị, giá bán, trạng thái `Published` |
| [10/11 · Transfer](../10-inter-warehouse-transfer/README.md) | Giao dịch chuyển kho tham chiếu SKU |
