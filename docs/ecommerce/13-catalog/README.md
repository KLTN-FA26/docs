# 13 — Catalog sản phẩm (Catalog)

## 1. Mục đích (Purpose)

Trưng bày sản phẩm cho khách hàng trên storefront: **danh mục, trang danh sách, trang chi tiết sản phẩm (PDP)**, tìm kiếm & lọc, hiển thị **giá, tình trạng còn hàng (availability)**, và điểm vào cho **thiết kế sản phẩm tùy chỉnh** ([12](../12-product-design/README.md)) hoặc **thêm vào giỏ** ([14](../14-cart-and-order/README.md)).

Catalog là **mặt tiền tiêu thụ dữ liệu** từ [Product Creation](../../warehouse/01-product-creation/README.md) và **tồn kho khả dụng** từ mảng kho vận.

---

## 2. Actor liên quan

| Actor | Vai trò |
|-------|---------|
| **Khách hàng (Customer / Guest)** | Duyệt, tìm kiếm, lọc, xem chi tiết, so sánh, thêm vào giỏ / bắt đầu thiết kế |
| **Quản trị sàn (E-commerce Admin)** | Cấu hình danh mục, thứ tự hiển thị, sản phẩm nổi bật, nội dung PDP, giá & khuyến mãi hiển thị |
| **Nhân viên Sale (Sales Staff)** | Gửi link sản phẩm cho khách qua [chat](../16-sales-chat/README.md); tư vấn lựa chọn biến thể |
| **Hệ thống (Search/Indexing, Pricing, Inventory feed)** | Đánh chỉ mục tìm kiếm, tính giá hiển thị, cập nhật tình trạng còn hàng |

---

## 3. Input / Output

### Input

| Nhóm | Dữ liệu | Nguồn |
|------|---------|-------|
| Sản phẩm & biến thể | Sản phẩm `Published`, danh sách SKU, thuộc tính biến thể, hình ảnh, mô tả, thuộc tính lọc | [01 Product Creation](../../warehouse/01-product-creation/README.md) |
| Giá bán | Giá niêm yết, giá khuyến mãi, quy tắc giá theo nhóm khách / số lượng | Cấu hình Pricing |
| Tồn kho khả dụng (ATP) | Tổng tồn `Available` (có thể theo kho / khu vực giao) | Mảng kho vận (05 Putaway, 07 Picking, 10 Transfer) |
| Cấu hình hiển thị | Danh mục, cây phân loại, bộ lọc, sắp xếp, sản phẩm nổi bật, SEO | Quản trị sàn |
| Cấu hình sản phẩm tùy chỉnh | Cờ "customizable", điểm vào trình thiết kế | [01](../../warehouse/01-product-creation/README.md) |

### Output

| Kết quả | Dùng bởi |
|---------|----------|
| Trang danh sách (PLP) & chi tiết (PDP) | Khách hàng |
| Kết quả tìm kiếm & bộ lọc | Khách hàng |
| Nút hành động: "Thêm vào giỏ" / "Bắt đầu thiết kế" | [14 Cart & Order](../14-cart-and-order/README.md), [12 Product Design](../12-product-design/README.md) |
| Chỉ báo tình trạng: Còn hàng / Sắp hết / Hết hàng / Đặt trước | Khách hàng, [14](../14-cart-and-order/README.md) |
| Link chia sẻ sản phẩm | [16 Sales Chat](../16-sales-chat/README.md) |

---

## 4. Luồng nghiệp vụ từng bước (Step-by-step)

### 4.1. Xuất bản & duy trì catalog (Quản trị sàn)

1. Sản phẩm ở [01](../../warehouse/01-product-creation/README.md) chuyển `Published` → xuất hiện trong nguồn dữ liệu catalog.
2. Quản trị sàn gán sản phẩm vào danh mục, đặt thứ tự, chọn ảnh đại diện, hoàn thiện mô tả & thuộc tính lọc.
3. Cấu hình giá hiển thị & khuyến mãi (thời hạn, điều kiện).
4. Hệ thống đánh chỉ mục tìm kiếm (search index) & cache.
5. Kiểm tra hiển thị (preview) rồi cho public.

### 4.2. Khách duyệt & chọn hàng

1. Khách vào storefront: xem trang chủ, danh mục, hoặc dùng ô tìm kiếm.
2. **Trang danh sách (PLP):** lọc theo thuộc tính (loại, màu, chất liệu, giá, đánh giá…), sắp xếp (mới nhất, giá, phổ biến).
3. **Trang chi tiết (PDP):** hình ảnh/gallery, mô tả, chọn biến thể (size/màu). Khi chọn biến thể → hiển thị **giá** và **tình trạng còn hàng** của SKU tương ứng.
4. **Tình trạng còn hàng:**
   | Hiển thị | Điều kiện (mặc định giả định) |
   |----------|------------------------------|
   | Còn hàng | ATP > ngưỡng hiển thị |
   | Sắp hết | 0 < ATP ≤ ngưỡng thấp |
   | Hết hàng | ATP ≤ 0 và không cho đặt trước |
   | Đặt trước (Pre-order / Backorder) | ATP ≤ 0 nhưng chính sách cho đặt trước |
5. **Hành động:**
   - Sản phẩm thường → **Thêm vào giỏ** (chọn số lượng).
   - Sản phẩm tùy chỉnh → **Bắt đầu thiết kế** → chuyển [12 Product Design](../12-product-design/README.md); sau khi chốt thiết kế mới thêm được vào giỏ.
6. Khách có thể lưu **wishlist**, so sánh sản phẩm, xem sản phẩm liên quan/gợi ý.

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
- **BR-02:** Giá hiển thị luôn lấy từ nguồn Pricing tại thời điểm xem; giá trong giỏ được "chốt" theo quy tắc của [14](../14-cart-and-order/README.md) (xem module đó).
- **BR-03:** Tình trạng còn hàng phản ánh **ATP** — tổng tồn `Available` trừ phần đã đặt/allocate — cập nhật gần thời gian thực.
- **BR-04:** Sản phẩm tùy chỉnh **không cho thêm thẳng vào giỏ** nếu chưa có design snapshot `Confirmed`.
- **BR-05:** Khuyến mãi hiển thị phải trong thời hạn & đúng điều kiện; hết hạn tự ẩn.
- **BR-06:** ATP có thể tính theo **kho phục vụ khu vực giao của khách** nếu bật bán theo vùng; nếu không, tính tổng toàn hệ thống.  ⚠️ Giả định — xem [open-questions](../../open-questions/README.md).

### Edge case

| Tình huống | Hướng xử lý |
|------------|-------------|
| Hết hàng ngay khi khách đang xem PDP | Hiển thị cập nhật khi refresh / khi thêm vào giỏ; chặn ở bước checkout nếu vẫn hết |
| Sản phẩm bị `Unpublished` khi khách có link trực tiếp | Trả trang "sản phẩm không còn khả dụng" + gợi ý sản phẩm tương tự |
| Giá thay đổi giữa lúc xem và lúc thêm vào giỏ | Giỏ áp giá theo quy tắc module 14; thông báo nếu lệch |
| Nhiều biến thể, một số hết hàng | Cho chọn biến thể còn hàng; biến thể hết hàng hiển thị mờ / "hết hàng" |
| Kết quả tìm kiếm rỗng | Gợi ý từ khoá liên quan, bỏ bớt bộ lọc, sản phẩm phổ biến |
| Sản phẩm tùy chỉnh nhưng model 3D lỗi | Vẫn cho thiết kế 2D (module 12 xử lý); catalog vẫn hiển thị bình thường |
| Chênh lệch index tìm kiếm vs dữ liệu thật (vừa đổi giá/ẩn) | Re-index theo sự kiện; PDP luôn đọc dữ liệu nguồn khi tải |

---

## 7. Liên kết với các module khác

| Module | Quan hệ |
|--------|---------|
| [01 · Product Creation](../../warehouse/01-product-creation/README.md) | Nguồn dữ liệu sản phẩm, biến thể, nội dung, cờ tùy chỉnh; trạng thái `Published` |
| Mảng kho vận (05, 07, 10, 11) | Cung cấp tồn khả dụng (ATP) để hiển thị tình trạng còn hàng |
| [12 · Product Design](../12-product-design/README.md) | "Bắt đầu thiết kế" từ PDP với sản phẩm tùy chỉnh |
| [14 · Cart & Order](../14-cart-and-order/README.md) | "Thêm vào giỏ"; chuyển giá & biến thể đã chọn |
| [15 · Payment](../15-payment/README.md) | Gián tiếp: giá & khuyến mãi hiển thị ảnh hưởng số tiền thanh toán |
| [16 · Sales Chat](../16-sales-chat/README.md) | Sale gửi link sản phẩm; khách hỏi về sản phẩm đang xem |
| [18 · Customer Information](../18-customer-information/README.md) | Wishlist, lịch sử xem, giá theo nhóm khách |
