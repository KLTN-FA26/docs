# 08 — Đóng gói (Packing)

## 1. Mục đích (Purpose)

Kiểm tra lần cuối hàng đã [pick](../07-picking/README.md), **đóng gói thành kiện (package/parcel)** phù hợp, gắn nhãn, và chuẩn bị chứng từ để bàn giao [Shipping](../09-shipping/README.md).

Packing đảm bảo: **đúng đơn – đúng hàng – đủ số lượng – đóng gói an toàn – có nhãn vận chuyển hợp lệ**. Đây là **chốt kiểm soát chất lượng cuối** trước khi hàng rời kho.

---

## 2. Actor liên quan

| Actor | Vai trò |
|-------|---------|
| **Nhân viên đóng gói (Packer)** | Quét kiểm đơn, chọn thùng, đóng gói, cân, in & dán nhãn, xác nhận hoàn tất |
| **Hệ thống (Cartonization / Rate engine)** | Gợi ý loại thùng, ước tính khối lượng quy đổi, chọn dịch vụ vận chuyển & phí |
| **Quản lý kho** | Xử lý sai lệch khi kiểm đơn, cấu hình vật liệu đóng gói & quy tắc |
| **Đơn vị vận chuyển (Carrier)** | Cung cấp nhãn vận đơn qua tích hợp (xem [09 Shipping](../09-shipping/README.md)) |

---

## 3. Input / Output

### Input

| Trường | Mô tả | Bắt buộc |
|--------|-------|----------|
| Hàng đã pick (staged) theo đơn | SKU, số lượng, lô/serial | Có |
| Thông tin đơn hàng | Địa chỉ giao, phương thức vận chuyển, yêu cầu đặc biệt (quà tặng, hoá đơn kèm) | Có |
| Vật liệu đóng gói khả dụng | Danh mục thùng/túi + kích thước + khối lượng bì | Có |
| **Design artifact** (với sản phẩm in ấn) | Bản in thành phẩm / nhãn kiểm cho đúng thiết kế khách chốt | Có (nếu áp dụng) |
| Quy tắc đóng gói | SKU dễ vỡ, hàng cần tách kiện, giới hạn khối lượng/kiện | Không |

### Output

| Kết quả | Dùng bởi |
|---------|----------|
| **Package(s)** — một hoặc nhiều kiện: nội dung, kích thước, khối lượng thực & quy đổi | 09 Shipping |
| **Packing slip** (phiếu đóng gói) & hoá đơn bán hàng kèm (nếu chính sách yêu cầu) | Khách hàng, 17 Order Management |
| **Yêu cầu tạo nhãn vận đơn** (shipping label request) | 09 Shipping / Carrier |
| **Cập nhật trạng thái đơn** → `Packed` | 17 Order Management |
| Bản ghi sai lệch / hư hỏng khi đóng gói | Manager, 07 Picking (điều tra) |

---

## 4. Luồng nghiệp vụ từng bước (Step-by-step)

1. **Nhận hàng tại bàn đóng gói** — Packer quét mã đơn / tote để nạp danh sách dòng hàng cần đóng.
2. **Kiểm đơn (verification / check)** — quét từng SKU & số lượng đối chiếu đơn:
   - Với sản phẩm in ấn: đối chiếu **thành phẩm với thiết kế đã chốt** (mã design, hình ảnh preview) — đảm bảo đúng nội dung khách đặt.
   - Với SKU serial: quét serial để gắn vào đơn.
3. **Gợi ý đóng gói (cartonization)** — hệ thống đề xuất loại thùng/túi & cách chia kiện dựa trên kích thước, khối lượng, quy tắc dễ vỡ, và giới hạn của hãng vận chuyển.
4. **Đóng gói vật lý** — cho hàng vào thùng, chèn lót, thêm phiếu đóng gói / hoá đơn / quà tặng theo yêu cầu.
5. **Cân & đo** — nhập khối lượng thực và kích thước kiện (hoặc lấy từ cân tích hợp). Hệ thống tính **khối lượng quy đổi (dimensional weight)**.
6. **Chọn dịch vụ vận chuyển & tạo nhãn** — theo phương thức khách chọn / rule chọn hãng rẻ-nhanh; gọi [Shipping](../09-shipping/README.md) để lấy **tracking number + shipping label**. In & dán nhãn lên kiện.
7. **Niêm phong & xác nhận** — dán nhãn, niêm phong; xác nhận `Packed`. Mỗi kiện có mã kiện (package ID) và tracking riêng.
8. **Chuyển sang khu chờ giao (Area `DISPATCH`)** — bàn giao cho bước [Shipping](../09-shipping/README.md).
9. **Xử lý đơn nhiều kiện (multi-package)** — một đơn có thể tách nhiều kiện; mỗi kiện có nội dung & tracking riêng, cùng thuộc một shipment.

---

## 5. Trạng thái / Vòng đời (Lifecycle)

### Trạng thái packing task / package

| Trạng thái | Ý nghĩa | Chuyển tiếp | Ai |
|------------|---------|-------------|----|
| `Pending` | Hàng đã staged, chờ đóng gói | → `In Progress`, → `Cancelled` | System |
| `In Progress` | Packer đang kiểm & đóng | → `Verification Failed`, → `Packed`, → `On Hold` | Packer |
| `Verification Failed` | Kiểm đơn lệch (thiếu/thừa/sai SKU/sai thiết kế) | → `In Progress` (đã bổ sung/sửa), → `On Hold` | Packer / Manager |
| `On Hold` | Chờ xử lý (hết thùng, chờ re-pick, chờ Manager) | → `In Progress`, → `Cancelled` | Manager |
| `Packed` | Đã đóng gói, có nhãn, sẵn giao | → `Handed to Shipping` | Packer |
| `Handed to Shipping` | Đã chuyển khu chờ giao | *(chuyển sang module 09)* | Packer |
| `Cancelled` | Đơn huỷ trước khi đóng xong | *(cuối)* | Manager |

---

## 6. Ràng buộc nghiệp vụ (Business Rules) & Edge case

### Business rules

- **BR-01:** Không `Packed` nếu kiểm đơn chưa khớp 100% (đúng SKU, đủ số lượng, đúng serial, đúng thiết kế).
- **BR-02:** Với sản phẩm in ấn, thành phẩm phải khớp **design snapshot đã chốt** của đơn; không khớp → `Verification Failed`, chuyển điều tra sản xuất.
- **BR-03:** Khối lượng & kích thước kiện phải nằm trong giới hạn của dịch vụ vận chuyển được chọn; vượt → tách kiện hoặc đổi dịch vụ.
- **BR-04:** Mỗi kiện chỉ thuộc một đơn (không gộp nhiều đơn vào một kiện) — trừ khi có tính năng "consolidated shipment" được bật.  ⚠️ Giả định — xem [open-questions](../../open-questions/README.md).
- **BR-05:** Nhãn vận đơn đã tạo mà đơn bị huỷ → phải **void label** với hãng vận chuyển (tránh bị tính phí).
- **BR-06:** Hàng đã `Packed` mà cần rút ra (đổi hàng, huỷ) → mở kiện có kiểm soát, put-back tồn, void label.

### Edge case

| Tình huống | Hướng xử lý |
|------------|-------------|
| Kiểm đơn thiếu hàng (picking sót) | `Verification Failed` → yêu cầu re-pick phần thiếu; nếu hết hàng → partial ship / backorder theo chính sách |
| Hàng bị hư khi đóng gói | Loại ra, `Blocked` + adjustment, re-pick hàng thay thế |
| Hết loại thùng gợi ý | `On Hold`; chọn thùng thay thế lớn hơn (cảnh báo tăng phí) |
| Địa chỉ giao không hợp lệ (hãng vận chuyển từ chối) | Không tạo được nhãn → chuyển Order Management liên hệ khách sửa địa chỉ; task `On Hold` |
| Đơn có sản phẩm cấm vận chuyển bằng hãng đã chọn (hàng đặc thù) | Đổi hãng/dịch vụ phù hợp; nếu không có → thông báo Order Management |
| Khách yêu cầu không kèm hoá đơn giá trị trong kiện (giao làm quà) | Theo cờ "gift order": in packing slip ẩn giá |
| Đơn nhiều kiện, một kiện lỗi | Chỉ đóng lại kiện lỗi; các kiện khác vẫn tiến hành; shipment chờ đủ kiện hoặc cho giao lệch chuyến theo chính sách |

---

## 7. Liên kết với các module khác

| Module | Quan hệ |
|--------|---------|
| [07 · Picking](../07-picking/README.md) | Nhận hàng đã pick tại staging; báo ngược khi kiểm đơn phát hiện lệch |
| [09 · Shipping](../09-shipping/README.md) | Gửi yêu cầu tạo nhãn vận đơn; bàn giao kiện đã đóng cho bước giao vận |
| [17 · Order Management](../../ecommerce/17-order-management/README.md) | Cập nhật trạng thái `Packed`; nhận yêu cầu sửa địa chỉ / xử lý huỷ |
| [12 · Product Design](../../ecommerce/12-product-design/README.md) | Cung cấp design snapshot để đối chiếu thành phẩm in ấn |
| [01 · Product Creation](../01-product-creation/README.md) | Kích thước/khối lượng/quy cách & cờ dễ vỡ của SKU cho cartonization |
| [00 · System Overview §5](../../00-system-overview/README.md) | Bước 2 của luồng Fulfillment |
