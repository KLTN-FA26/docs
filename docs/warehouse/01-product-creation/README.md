# 01 — Tạo sản phẩm (Product Creation)

## 1. Mục đích (Purpose)

Khai báo và duy trì **dữ liệu gốc về sản phẩm (product master data)** — nguồn dữ liệu dùng chung cho toàn hệ thống: mua hàng, quản lý tồn kho, vận hành kho, và hiển thị trên catalog TMĐT.

Đây là bước **khởi tạo vòng đời sản phẩm**. Không thể lập PO, nhận hàng, hay đăng bán một mặt hàng chưa được tạo ở module này.

Với sản phẩm **tùy chỉnh / in ấn**, module này còn khai báo **khả năng cá nhân hoá** (customization capability): vùng in (print area), chất liệu, khung 3D dùng để xem trước, các tuỳ chọn biến thể.

Dữ liệu gốc được tách thành **hai lớp**, liên kết **1:1** theo từng SKU:

| Lớp dữ liệu | Nội dung | Phục vụ chủ yếu |
|-------------|----------|-----------------|
| **Sản phẩm & Biến thể (Product / Variant)** | Thông tin để bán: định danh, danh mục, thuộc tính biến thể, nội dung TMĐT, giá bán, cấu hình tùy chỉnh | 12 Product Design, 13 Catalog, 14 Cart & Order |
| **Hàng tồn kho (Inventory Item)** | Thông tin hàng vật lý: đơn vị tính, thông số logistics, kiểm soát lô/hạn dùng/serial, ngưỡng tồn, thông tin mua hàng | 02 → 11 (mảng kho vận) |

> ⚠️ Giả định: Với sản phẩm tùy chỉnh, Inventory Item của SKU là **phôi** (hàng chưa in). Thiết kế của khách đi kèm dòng đơn dưới dạng design snapshot, không sinh SKU/Inventory Item mới; tồn và ATP hiển thị trên catalog tính theo phôi. Xem [open-questions](../../open-questions/README.md).

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

| Nhóm | Lớp dữ liệu | Trường dữ liệu (ví dụ) | Bắt buộc |
|------|-------------|------------------------|----------|
| **Định danh** | Product | Tên sản phẩm, mã sản phẩm nội bộ (product code), danh mục (category), thương hiệu | Có |
| **Thuộc tính biến thể (variant attributes)** | Variant | Dung tích (ví dụ 360 ml, 500 ml, 700 ml), chất liệu (PP / PET / giấy), đường kính miệng, màu… → tổ hợp sinh ra các **SKU** | Có (ít nhất 1 SKU) |
| **Thông số vật lý (logistics data)** | Inventory Item | Khối lượng, kích thước (D×R×C), **đơn vị cơ sở (base UoM)** — đơn vị ghi tồn, ví dụ *cái* | Có (phục vụ slotting, packing, tính phí ship) |
| **Đơn vị quy đổi (UoM conversion)** | Inventory Item | Danh sách đơn vị đóng gói kèm **hệ số quy về đơn vị cơ sở** (ví dụ *thùng* = 1.000 cái), kích thước & khối lượng của từng đơn vị; đánh dấu **đơn vị mua mặc định** dùng trên PO | Không (mặc định chỉ có đơn vị cơ sở) |
| **Đơn vị bán (sales UoM)** | Variant | Các đơn vị khách được đặt (chọn từ danh sách đơn vị quy đổi của Inventory Item, ví dụ chỉ bán theo *thùng*), giá bán theo từng đơn vị bán | Có (mặc định: đơn vị cơ sở) |
| **Thông tin mua hàng** | Inventory Item | Nhà cung cấp mặc định, giá mua tham chiếu, lead time, MOQ | Không (khuyến nghị) |
| **Cho phép xoay khi xếp xe (rotatable)** | Inventory Item | Bật → kiện chứa SKU được xoay theo 6 hướng khi lập kế hoạch xếp hàng lên xe / container; tắt → giữ nguyên hướng đặt. Là giá trị mặc định cho cờ xoay của kiện ở [08](../08-packing/README.md), dùng cho Load Planning ở [09](../09-shipping/README.md) | Không (mặc định: cho phép xoay) ⚠️ Giả định — xem [open-questions](../../open-questions/README.md) |
| **Lớp lưu trữ (storage class)** | Inventory Item | `NORMAL` / `COLD` / `HAZMAT` / `FRAGILE` / `OVERSIZE` — quyết định loại bin được phép cất (xem [06](../06-warehouse-map-slotting/README.md)) | Có (mặc định `NORMAL`) |
| **Kiểm soát tồn kho** | Inventory Item | Có theo dõi lô (batch/lot)? Có hạn sử dụng? Có serial? Ngưỡng tồn tối thiểu/tối đa | Không (mặc định: không lô, không serial) |
| **Yêu cầu QC khi nhận (QC required)** | Inventory Item | Bật → hàng nhận đi luồng 3 bước qua khu kiểm tra chất lượng trước khi putaway; tắt → luồng 2 bước (xem [03](../03-receipt/README.md)) | Không (mặc định: không yêu cầu) |
| **Dữ liệu TMĐT** | Product / Variant | Mô tả, hình ảnh, giá bán, thuộc tính SEO, trạng thái hiển thị | Không (bắt buộc trước khi publish lên catalog) |
| **Dữ liệu tùy chỉnh (personalization)** | Product / Variant | Vùng in (print areas) & toạ độ, chất liệu in, **kỹ thuật in (print method** — in lụa, in offset khô (dry offset), in UV, flexo…; dùng chọn trạm in khi có trạm in — phạm vi sau, [19](../19-production/README.md)**)**, template 2D, **model 3D** để live preview, ràng buộc thiết kế (kích thước file, hệ màu) | Chỉ với sản phẩm in ấn |

### Output

| Kết quả | Mô tả | Được dùng bởi |
|---------|-------|---------------|
| **Product record** (bản ghi sản phẩm) | Thực thể sản phẩm + danh sách SKU con | Toàn hệ thống |
| **SKU (Variant)** | Biến thể bán được; mã hàng hoá nhỏ nhất dùng trên mọi chứng từ | 02 PO, 03 Receipt, 05 Putaway, 07 Picking, 10/11 Transfer, 12 Design, 13 Catalog, 14 Order |
| **Inventory Item** | Hồ sơ hàng vật lý của SKU (1:1), hệ thống **tạo tự động** khi sinh SKU; là đối tượng được theo dõi tồn theo vị trí | 03, 05, 06, 07, 08, 10, 11; 13 Catalog (ATP) |
| **Cấu hình tùy chỉnh** | Bộ tham số cho trình thiết kế 2D + preview 3D | 12 Product Design |
| **Bản nháp catalog** | Nội dung hiển thị chờ publish | 13 Catalog |

---

## 4. Luồng nghiệp vụ từng bước (Step-by-step)

1. **Khởi tạo bản nháp (Create draft)** — System Admin / Quản trị sàn nhập thông tin định danh + danh mục. Sản phẩm ở trạng thái `Draft`.
2. **Khai báo thuộc tính biến thể** — chọn các trục biến thể (ví dụ: Dung tích × Chất liệu). Hệ thống sinh **ma trận biến thể**; người dùng chọn tổ hợp hợp lệ để tạo SKU.
3. **Sinh mã SKU** — theo quy tắc cấu hình (prefix danh mục + mã thuộc tính + số thứ tự) hoặc nhập tay. Mỗi SKU phải **duy nhất toàn hệ thống**. Với mỗi SKU, hệ thống **tự động tạo một Inventory Item** tương ứng; người dùng không tạo thủ công.
4. **Nhập thông số logistics** vào Inventory Item của từng SKU (hoặc kế thừa từ sản phẩm cha): khối lượng, kích thước, đơn vị cơ sở; khai báo **đơn vị quy đổi** (ví dụ *thùng* = 1.000 cái, kèm kích thước & khối lượng thùng) và chọn đơn vị mua mặc định.
5. **Cấu hình kiểm soát tồn kho** trên Inventory Item — bật/tắt theo lô, hạn dùng, serial; bật/tắt **Yêu cầu QC khi nhận**; đặt ngưỡng min/max phục vụ cảnh báo bổ sung tồn (reorder).
6. **(Sản phẩm in ấn) Cấu hình tùy chỉnh** — tải template 2D, model 3D, khai báo print area, kỹ thuật in, ràng buộc file thiết kế.
7. **Nhập dữ liệu TMĐT** — mô tả, hình ảnh, thuộc tính hiển thị; chọn **đơn vị bán** và giá bán theo từng đơn vị bán.
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
- **BR-03:** Thông số logistics (khối lượng, kích thước) là **bắt buộc trước khi** SKU được đưa vào Receipt/Putaway — vì module 06 Slotting, 08 Packing và Load Planning ở 09 Shipping phụ thuộc dữ liệu này.
- **BR-04:** Chỉ sản phẩm `Published` mới hiển thị trên catalog; chỉ sản phẩm `Active`/`Published` mới được đưa vào PO.
- **BR-05:** Với sản phẩm in ấn, **bắt buộc** có ít nhất một print area và một template preview (2D hoặc 3D) trước khi `Published`.
- **BR-06:** Giá bán TMĐT (thuộc Variant) và giá mua tham chiếu (thuộc Inventory Item) là hai trường độc lập; module này chỉ lưu, không tính margin.
- **BR-07:** Mỗi SKU có **đúng một** Inventory Item (quan hệ 1:1), tạo tự động khi sinh SKU. Hệ thống **không bán combo / bundle**, nên không có SKU nào ứng với nhiều Inventory Item. Do quan hệ 1:1, chứng từ kho ghi theo SKU tương đương ghi theo Inventory Item.
- **BR-08:** Inventory Item chỉ chứa dữ liệu hàng vật lý & kiểm soát tồn; **không** chứa nội dung bán hàng (mô tả, hình ảnh, SEO, giá bán).
- **BR-09:** Tồn kho và mọi chứng từ kho lưu số lượng theo **đơn vị cơ sở**. Số lượng nhập theo đơn vị quy đổi (thùng…) được hệ thống nhân hệ số trước khi ghi; hệ số phải là số nguyên dương. Mỗi SKU có thể có nhiều đơn vị quy đổi nhưng **không** sinh SKU mới cho từng quy cách.
- **BR-10:** Hệ số quy đổi đã phát sinh giao dịch **không được sửa**. Muốn đổi quy cách (ví dụ NCC đổi thùng 1.000 cái sang thùng 500 cái) phải khai báo đơn vị quy đổi mới và ngừng dùng đơn vị cũ.
- **BR-11:** Đơn vị bán của Variant chỉ được chọn trong danh sách đơn vị quy đổi của Inventory Item tương ứng.  ⚠️ Giả định: đơn vị bán cấu hình theo từng biến thể, khách chỉ đặt số nguyên đơn vị bán (không mua lẻ dưới đơn vị bán nhỏ nhất). Xem [open-questions](../../open-questions/README.md).

### Edge case cần lưu ý

| Tình huống | Hướng xử lý đề xuất |
|------------|---------------------|
| Thêm biến thể mới vào sản phẩm đang `Published` | Biến thể mới bắt đầu ở `Draft`/`Approved`, publish riêng; không ảnh hưởng SKU hiện có |
| Gộp/tách SKU do sai khai báo ban đầu | Không sửa trực tiếp; tạo SKU mới + quy trình điều chỉnh tồn có kiểm soát, khoá SKU cũ thành `Obsolete` |
| Sản phẩm bị `Discontinued` nhưng còn tồn kho | Cho phép Picking/Transfer/Adjustment để giải phóng tồn; chặn Receipt & PO mới |
| Đổi đơn vị cơ sở (base UoM) sau khi đã có tồn | Không cho đổi; yêu cầu tạo SKU mới với đơn vị cơ sở đúng |
| NCC đổi quy cách thùng (ví dụ 1.000 → 500 cái/thùng) | Khai báo đơn vị quy đổi mới; đơn vị cũ ngừng dùng cho PO/đơn mới nhưng vẫn giữ cho chứng từ cũ (BR-10) |
| Template 3D lỗi/thiếu | Chặn `Published`; cho phép `Active` để vẫn nhập kho phục vụ kênh khác |
| Trùng mã SKU khi nhập tay | Hệ thống chặn ngay tại bước tạo (unique constraint) |

---

## 7. Liên kết với các module khác

| Module | Quan hệ |
|--------|---------|
| [02 · Purchase Order](../02-purchase-order/README.md) | PO chỉ chọn được SKU `Active`; kế thừa NCC mặc định, giá mua tham chiếu, MOQ, lead time, đơn vị mua mặc định từ đây |
| [03 · Receipt](../03-receipt/README.md) | Nhận hàng theo SKU; cấu hình lô/hạn dùng/serial quyết định thông tin phải ghi nhận khi nhận; cờ Yêu cầu QC quyết định luồng nhận 2 bước hay 3 bước; đơn vị quy đổi để kiểm đếm theo thùng |
| [05 · Putaway](../05-putaway/README.md) & [06 · Slotting](../06-warehouse-map-slotting/README.md) | Dùng thông số kích thước/khối lượng SKU để gợi ý vị trí phù hợp |
| [07 · Picking](../07-picking/README.md) / [08 · Packing](../08-packing/README.md) | Đơn vị quy đổi (thùng) & kích thước SKU ảnh hưởng hiển thị số lượng lấy, gợi ý thùng và cách đóng gói; cờ cho phép xoay làm giá trị mặc định cho kiện |
| [09 · Shipping](../09-shipping/README.md) | Load Planning dùng cờ cho phép xoay (qua kiện) khi xếp hàng lên xe / container |
| [12 · Product Design](../../ecommerce/12-product-design/README.md) | Lấy template 2D, model 3D, định nghĩa print area & ràng buộc thiết kế từ đây |
| [19 · Production](../19-production/README.md) | Inventory Item của SKU tùy chỉnh là phôi được cấp cho xưởng; kỹ thuật in dùng chọn trạm in khi có trạm (phạm vi sau) |
| [13 · Catalog](../../ecommerce/13-catalog/README.md) | Lấy nội dung hiển thị, đơn vị bán & giá bán, trạng thái `Published`; ATP tính trên Inventory Item của SKU |
| [14 · Cart & Order](../../ecommerce/14-cart-and-order/README.md) | Số lượng đặt theo đơn vị bán, quy về đơn vị cơ sở bằng hệ số quy đổi khai báo ở đây |
| [10/11 · Transfer](../10-inter-warehouse-transfer/README.md) | Giao dịch chuyển kho tham chiếu SKU |
| [20 · Inventory Control](../20-inventory-control/README.md) | Dùng ngưỡng đặt lại, tồn an toàn, FEFO/FIFO, theo dõi lô / hạn dùng khai báo trên Inventory Item để sinh cảnh báo tồn và phân bổ; chỉ SKU còn mua được mới có cảnh báo đặt lại |
