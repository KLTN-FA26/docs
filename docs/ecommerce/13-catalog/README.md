# 13 — Catalog sản phẩm trên cổng khách hàng (Catalog)

> Đã chốt 2026-10-06: hệ thống **chỉ bán sỉ (B2B)**. Catalog nằm trong **cổng khách hàng**, chỉ khách sỉ đã đăng nhập mới xem được, với **giá riêng** của từng khách.

## 1. Mục đích (Purpose)

Trưng bày ly và bao bì cho **khách sỉ đã đăng nhập** trên cổng khách hàng: **danh mục, trang danh sách, trang chi tiết sản phẩm (PDP)**, tìm kiếm & lọc, hiển thị **giá riêng của khách (kèm bậc số lượng / MOQ), tình trạng còn hàng (availability)**, và điểm vào cho **thiết kế** ([12](../12-product-design/README.md)), **yêu cầu báo giá** hoặc **đặt nhanh / đặt lại** ([14](../14-cart-and-order/README.md)).

Catalog là **mặt tiền tiêu thụ dữ liệu** từ [Product Creation](../../warehouse/01-product-creation/README.md) và **tồn kho khả dụng** từ mảng kho vận.

---

## 2. Actor liên quan

| Actor | Vai trò |
|-------|---------|
| **Khách hàng sỉ (đã đăng nhập)** | Duyệt, tìm kiếm, lọc, xem chi tiết & giá riêng, yêu cầu báo giá, đặt nhanh / đặt lại, bắt đầu thiết kế |
| **Quản trị bán hàng (E-commerce Admin)** | Cấu hình danh mục, thứ tự hiển thị, nội dung PDP, bảng giá theo khách / bậc số lượng |
| **Nhân viên Sale (Sales Staff)** | Gửi link sản phẩm cho khách qua [chat](../16-sales-chat/README.md); tư vấn lựa chọn biến thể |
| **Hệ thống (Search/Indexing, Pricing, Inventory feed)** | Đánh chỉ mục tìm kiếm, tính giá hiển thị, cập nhật tình trạng còn hàng |

---

## 3. Input / Output

### Input

| Nhóm | Dữ liệu | Nguồn |
|------|---------|-------|
| Sản phẩm & biến thể | Sản phẩm `Published`, danh sách SKU, thuộc tính biến thể, hình ảnh, mô tả, thuộc tính lọc | [01 Product Creation](../../warehouse/01-product-creation/README.md) |
| Giá bán | **Bảng giá riêng của khách** theo **đơn vị bán** và bậc số lượng; nếu khách chưa có bảng giá riêng thì dùng giá theo nhóm khách | Cấu hình Pricing ([18](../18-customer-information/README.md)); đơn vị bán & hệ số quy đổi từ [01](../../warehouse/01-product-creation/README.md) |
| Tồn kho khả dụng (ATP) | ATP = tồn `Available` − phần đang giữ hàng (reservation), tính trên Inventory Item của SKU; theo kho hoặc toàn hệ thống (xem BR-06) | Mảng kho vận (05 Putaway, 07 Picking, 10 Transfer) |
| Cấu hình hiển thị | Danh mục, cây phân loại, bộ lọc, sắp xếp | Quản trị bán hàng |
| Cấu hình sản phẩm tùy chỉnh | Cờ "customizable", điểm vào trình thiết kế | [01](../../warehouse/01-product-creation/README.md) |

### Output

| Kết quả | Dùng bởi |
|---------|----------|
| Trang danh sách (PLP) & chi tiết (PDP) | Khách hàng |
| Kết quả tìm kiếm & bộ lọc | Khách hàng |
| Nút hành động: "Yêu cầu báo giá" / "Đặt nhanh" / "Bắt đầu thiết kế" | [14 Quote & Order](../14-cart-and-order/README.md), [12 Product Design](../12-product-design/README.md) |
| Chỉ báo tình trạng: Còn hàng / Sắp hết / Hết hàng / Đặt trước | Khách hàng, [14](../14-cart-and-order/README.md) |
| Link chia sẻ sản phẩm | [16 Sales Chat](../16-sales-chat/README.md) |

---

## 4. Luồng nghiệp vụ từng bước (Step-by-step)

### 4.1. Xuất bản & duy trì catalog (Quản trị sàn)

1. Sản phẩm ở [01](../../warehouse/01-product-creation/README.md) chuyển `Published` → xuất hiện trong nguồn dữ liệu catalog.
2. Quản trị sàn gán sản phẩm vào danh mục, đặt thứ tự, chọn ảnh đại diện, hoàn thiện mô tả & thuộc tính lọc.
3. Cấu hình bảng giá theo khách / nhóm khách và bậc số lượng.
4. Hệ thống đánh chỉ mục tìm kiếm (search index) & cache.
5. Kiểm tra hiển thị (preview) rồi cho public.

### 4.2. Khách duyệt & chọn hàng

1. Khách sỉ **đăng nhập cổng khách hàng**: xem danh mục, hoặc dùng ô tìm kiếm. Người chưa đăng nhập không xem được giá và tồn.
2. **Trang danh sách (PLP):** lọc theo thuộc tính (nhóm hàng — ly / bao bì mang đi; dung tích, chất liệu, đường kính miệng, giá…), sắp xếp (mới nhất, giá, phổ biến).
3. **Trang chi tiết (PDP):** hình ảnh/gallery, mô tả, chọn biến thể (dung tích / kích thước / chất liệu) và **đơn vị bán** (ví dụ *thùng 1.000 cái*). Khi chọn → hiển thị **giá riêng theo đơn vị bán và bậc số lượng** (kèm đơn giá quy ra đơn vị cơ sở), **MOQ** và **tình trạng còn hàng** của phôi tương ứng.
4. **Tình trạng còn hàng:**
   | Hiển thị | Điều kiện (mặc định giả định) |
   |----------|------------------------------|
   | Còn hàng | ATP > ngưỡng hiển thị |
   | Sắp hết | 0 < ATP ≤ ngưỡng thấp |
   | Hết hàng | ATP ≤ 0 và không cho đặt trước |
   | Đặt trước (Pre-order / Backorder) | ATP ≤ 0 nhưng chính sách cho đặt trước |
5. **Hành động:**
   - Hàng sẵn hoặc thiết kế đã có mẫu duyệt → **Đặt nhanh** (chọn số lượng ≥ MOQ) → [14](../14-cart-and-order/README.md).
   - Thiết kế mới → **Bắt đầu thiết kế** ([12](../12-product-design/README.md)) rồi **Yêu cầu báo giá** — Sale báo giá và làm mẫu trước khi đặt hàng.
6. Khách có thể xem **lịch sử đặt hàng** và **đặt lại** các thiết kế đã có mẫu duyệt.

---

## 5. Trạng thái / Vòng đời (Lifecycle)

Catalog không có "chứng từ". Trạng thái hiển thị của một sản phẩm/biến thể trên catalog:

| Trạng thái hiển thị | Ý nghĩa | Nguồn |
|---------------------|---------|-------|
| `Visible – In stock` | Hiển thị, mua ngay được | `Published` + ATP đủ |
| `Visible – Low stock` | Hiển thị kèm cảnh báo sắp hết | `Published` + ATP thấp |
| `Visible – Out of stock` | Hiển thị nhưng không cho mua (hoặc chỉ cho đặt trước) | `Published` + ATP ≤ 0 |
| `Visible – Pre-order` | Cho đặt trước, giao sau | Chính sách pre-order bật |
| `Hidden` | Không hiển thị (ẩn thủ công / hết hàng lâu / hết mùa) | Admin hoặc rule |
| `Unpublished` | Sản phẩm rời khỏi catalog | `Active`/`Inactive` ở [01](../../warehouse/01-product-creation/README.md) |

---

## 6. Ràng buộc nghiệp vụ (Business Rules) & Edge case

### Business rules

- **BR-01:** Chỉ sản phẩm `Published` mới hiển thị. Biến thể (SKU) `Blocked`/`Obsolete` bị ẩn khỏi lựa chọn.
- **BR-02:** Giá hiển thị là **giá riêng của khách đang đăng nhập** tại thời điểm xem; giá được "chốt" khi tạo đơn theo quy tắc của [14](../14-cart-and-order/README.md). Chỉ khách sỉ đã đăng nhập mới xem được giá và tồn.
- **BR-03:** Tình trạng còn hàng phản ánh **ATP = tồn `Available` − phần đang giữ hàng (reservation)** (định nghĩa ở [Glossary](../../glossary/README.md)). Hàng đã `Allocated` không thuộc tồn `Available` nên không bị trừ lại. Cập nhật gần thời gian thực.
- **BR-04:** Dòng in chỉ đặt nhanh được với **thiết kế đã có mẫu duyệt**; thiết kế mới phải qua báo giá và làm mẫu ([19](../../warehouse/19-production/README.md)).
- **BR-05:** Hiển thị MOQ và bậc số lượng; không cho đặt dưới MOQ.
- **BR-06:** ATP có thể tính theo **kho phục vụ khu vực giao của khách** nếu bật bán theo vùng; nếu không, tính tổng toàn hệ thống.  ⚠️ Giả định — xem [open-questions](../../open-questions/README.md).
- **BR-07:** ATP tính theo đơn vị cơ sở; số lượng còn hàng hiển thị theo đơn vị bán = **phần nguyên của (ATP ÷ hệ số quy đổi)**. Ví dụ ATP 2.500 cái, đơn vị bán *thùng* = 1.000 cái → còn 2 thùng.

### Edge case

| Tình huống | Hướng xử lý |
|------------|-------------|
| Hết hàng ngay khi khách đang xem PDP | Hiển thị cập nhật khi refresh / khi đặt; chặn ở bước checkout nếu vẫn hết |
| Sản phẩm bị `Unpublished` khi khách có link trực tiếp | Trả trang "sản phẩm không còn khả dụng" + gợi ý sản phẩm tương tự |
| Giá thay đổi giữa lúc xem và lúc đặt | Đơn áp giá theo quy tắc module 14; thông báo nếu lệch |
| Nhiều biến thể, một số hết hàng | Cho chọn biến thể còn hàng; biến thể hết hàng hiển thị mờ / "hết hàng" |
| Tồn còn lẻ, không đủ một đơn vị bán (ví dụ còn 600 cái, bán theo thùng 1.000) | Đơn vị bán đó hiển thị "hết hàng"; nếu biến thể có đơn vị bán nhỏ hơn thì vẫn cho chọn đơn vị nhỏ hơn |
| Kết quả tìm kiếm rỗng | Gợi ý từ khoá liên quan, bỏ bớt bộ lọc, sản phẩm phổ biến |
| Sản phẩm tùy chỉnh nhưng model 3D lỗi | Vẫn cho thiết kế 2D (module 12 xử lý); catalog vẫn hiển thị bình thường |
| Chênh lệch index tìm kiếm vs dữ liệu thật (vừa đổi giá/ẩn) | Re-index theo sự kiện; PDP luôn đọc dữ liệu nguồn khi tải |

---

## 7. Liên kết với các module khác

| Module | Quan hệ |
|--------|---------|
| [01 · Product Creation](../../warehouse/01-product-creation/README.md) | Nguồn dữ liệu sản phẩm, biến thể, đơn vị bán & hệ số quy đổi, nội dung, cờ tùy chỉnh; trạng thái `Published` |
| Mảng kho vận (05, 07, 10, 11) | Cung cấp tồn khả dụng (ATP) để hiển thị tình trạng còn hàng |
| [12 · Product Design](../12-product-design/README.md) | "Bắt đầu thiết kế" từ PDP với sản phẩm tùy chỉnh |
| [14 · Quote & Order](../14-cart-and-order/README.md) | "Yêu cầu báo giá" / "Đặt nhanh"; chuyển giá & biến thể đã chọn |
| [15 · Payment](../15-payment/README.md) | Gián tiếp: giá riêng của khách ảnh hưởng số tiền thanh toán |
| [16 · Sales Chat](../16-sales-chat/README.md) | Sale gửi link sản phẩm; khách hỏi về sản phẩm đang xem |
| [18 · Customer Information](../18-customer-information/README.md) | Bảng giá riêng / nhóm khách; lịch sử đặt hàng để đặt lại |
