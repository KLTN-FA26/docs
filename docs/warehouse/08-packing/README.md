# 08 — Đóng gói (Packing)

## 1. Mục đích (Purpose)

Kiểm tra lần cuối hàng đã [pick](../07-picking/README.md) và thành phẩm in do [xưởng](../19-production/README.md) bàn giao, **đóng gói thành kiện (package/parcel)** phù hợp, gắn nhãn, và chuẩn bị chứng từ để bàn giao [Shipping](../09-shipping/README.md).

Packing đảm bảo: **đúng đơn – đúng hàng – đủ số lượng – đóng gói an toàn – có nhãn vận chuyển hợp lệ**. Đây là **chốt kiểm soát chất lượng cuối** trước khi hàng rời kho.

---

## 2. Actor liên quan

| Actor | Vai trò |
|-------|---------|
| **Nhân viên đóng gói (Packer)** | Quét kiểm đơn, chọn thùng, đóng gói, cân, in & dán nhãn, xác nhận hoàn tất |
| **Hệ thống (Cartonization)** | Gợi ý loại thùng, ước tính khối lượng quy đổi |
| **Quản lý kho** | Xử lý sai lệch khi kiểm đơn, cấu hình vật liệu đóng gói & quy tắc |

---

## 3. Input / Output

### Input

| Trường | Mô tả | Bắt buộc |
|--------|-------|----------|
| Hàng đã pick (staged) theo đơn | SKU, số lượng, lô/serial | Có (với dòng hàng thường) |
| Thành phẩm in đạt QC (với dòng tùy chỉnh) | Xưởng bàn giao tại khu `PACKING`, gắn lệnh sản xuất & design snapshot (xem [19](../19-production/README.md)) | Có (nếu áp dụng) |
| Thông tin đơn hàng | Địa chỉ giao, phương thức vận chuyển, yêu cầu đặc biệt (quà tặng, hoá đơn kèm) | Có |
| Vật liệu đóng gói khả dụng | Danh mục thùng/túi + kích thước + khối lượng bì | Có |
| **Design artifact** (với sản phẩm in ấn) | Bản in thành phẩm / nhãn kiểm cho đúng thiết kế khách chốt | Có (nếu áp dụng) |
| Quy tắc đóng gói | SKU dễ vỡ, hàng cần tách kiện, giới hạn khối lượng/kiện | Không |

### Output

| Kết quả | Dùng bởi |
|---------|----------|
| **Package(s)** — một hoặc nhiều kiện: nội dung, kích thước, khối lượng thực & quy đổi, **cờ cho phép xoay** | 09 Shipping (kể cả Load Planning khi giao nguyên chuyến) |
| **Packing slip** (phiếu đóng gói) & hoá đơn bán hàng kèm (nếu chính sách yêu cầu) | Khách hàng, 17 Order Management |
| **Nhãn kiện (package label)** — nhãn nội bộ in tại bàn đóng gói | Dán lên kiện; 09 Shipping (quét khi xếp xe, bàn giao) |
| **Cập nhật trạng thái đơn** → `Packed` | 17 Order Management |
| Bản ghi sai lệch / hư hỏng khi đóng gói | Manager, 07 Picking / 19 Production (điều tra) |

---

## 4. Luồng nghiệp vụ từng bước (Step-by-step)

1. **Nhận hàng tại bàn đóng gói** — Packer quét mã đơn / tote để nạp danh sách dòng hàng cần đóng. Packing task chỉ sinh khi **mọi dòng** của đơn đã ở khu `PACKING`: dòng hàng thường đã pick (`Staged`) và dòng tùy chỉnh đã có thành phẩm đạt QC do xưởng bàn giao ([19](../19-production/README.md)). Đơn chỉ gồm hàng tùy chỉnh đi thẳng từ sản xuất sang packing, không qua picking thành phẩm.
2. **Kiểm đơn (verification / check)** — quét từng SKU & số lượng đối chiếu đơn:
   - Với sản phẩm in: thành phẩm nhận từ xưởng ([19](../19-production/README.md)) ở khu `PACKING`; đối chiếu **thành phẩm với thiết kế đã chốt và mẫu khách đã duyệt** (mã design, hình ảnh preview) — đảm bảo đúng nội dung khách đặt.
   - Với SKU serial: quét serial để gắn vào đơn.
3. **Gợi ý đóng gói (cartonization)** — hệ thống đề xuất loại thùng/túi & cách chia kiện dựa trên kích thước, khối lượng và quy tắc dễ vỡ.
4. **Đóng gói vật lý** — cho hàng vào thùng, chèn lót, thêm phiếu đóng gói / hoá đơn / quà tặng theo yêu cầu.
5. **Cân & đo** — nhập khối lượng thực và kích thước kiện (hoặc lấy từ cân tích hợp). Hệ thống tính **khối lượng quy đổi (dimensional weight)**. Kiện nhận **cờ cho phép xoay** mặc định từ SKU bên trong; Packer được ghi đè (ví dụ kiện phải giữ chiều đứng).
   > ⚠️ Giả định: kiện chứa nhiều SKU có cờ khác nhau lấy giá trị chặt nhất — chỉ cần một SKU không cho xoay thì kiện không cho xoay. Xem [open-questions](../../open-questions/README.md).
6. **In nhãn kiện** — hệ thống sinh **nhãn kiện nội bộ**: mã đơn, mã kiện (package ID), người nhận, địa chỉ giao, địa chỉ hoàn hàng, số tiền thu hộ (phần còn lại của đơn đặt cọc) nếu có. In & dán lên kiện. Hệ thống **không gọi API hãng vận chuyển** (xem [09 Shipping](../09-shipping/README.md)). Kiện của đơn **giao nguyên chuyến** chuyển sang lập kế hoạch chuyến xe ([09 §4.2](../09-shipping/README.md#42-giao-nguyên-chuyến--lập-kế-hoạch-xếp-hàng-3d-load-planning)).
7. **Niêm phong & xác nhận** — dán nhãn, niêm phong; xác nhận `Packed`. Mỗi kiện có mã kiện (package ID) riêng.
8. **Chuyển sang khu chờ giao (Area `DISPATCH`)** — bàn giao cho bước [Shipping](../09-shipping/README.md).
9. **Xử lý đơn nhiều kiện (multi-package)** — một đơn có thể tách nhiều kiện; mỗi kiện có nội dung & mã kiện riêng, cùng thuộc một shipment.

---

## 5. Trạng thái / Vòng đời (Lifecycle)

### Trạng thái packing task / package

| Trạng thái | Ý nghĩa | Chuyển tiếp | Ai |
|------------|---------|-------------|----|
| `Pending` | Hàng đã staged / thành phẩm đã bàn giao, chờ đóng gói | → `In Progress`, → `Cancelled` | System |
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
- **BR-02:** Với sản phẩm in, thành phẩm phải khớp **design snapshot đã chốt** và **mẫu đã duyệt** của đơn; không khớp → `Verification Failed`; thành phẩm lỗi được xuất huỷ và lệnh sản xuất tương ứng quay về `Ready` để in bù (xem [19](../19-production/README.md)).
- **BR-03:** Kiện phải có khối lượng & kích thước thực đo — đây là đầu vào của bàn giao và kế hoạch xếp hàng ở [09 Shipping](../09-shipping/README.md); kiện quá khổ / quá nặng so với mọi loại phương tiện → tách kiện.
- **BR-04:** Mỗi kiện chỉ thuộc một đơn (không gộp nhiều đơn vào một kiện) — trừ khi có tính năng "consolidated shipment" được bật.  ⚠️ Giả định — xem [open-questions](../../open-questions/README.md).
- **BR-05:** Đơn bị huỷ sau khi đã in nhãn kiện → nhãn kiện bị huỷ cùng shipment (`Cancelled`); kiện chưa bàn giao nên không phát sinh gì với hãng.
- **BR-06:** Hàng đã `Packed` mà cần rút ra (đổi hàng, huỷ) → mở kiện có kiểm soát, put-back tồn, huỷ nhãn kiện.

### Edge case

| Tình huống | Hướng xử lý |
|------------|-------------|
| Kiểm đơn thiếu hàng (picking sót) | `Verification Failed` → yêu cầu re-pick phần thiếu; nếu hết hàng → partial ship / backorder theo chính sách |
| Hàng bị hư khi đóng gói | Loại ra, `Blocked` + adjustment, re-pick hàng thay thế |
| Hết loại thùng gợi ý | `On Hold`; chọn thùng thay thế lớn hơn (cảnh báo tăng phí) |
| Địa chỉ giao không hợp lệ (thiếu phường/xã, sai định dạng) | Không in được nhãn kiện → chuyển Order Management liên hệ khách sửa địa chỉ; task `On Hold` |
| Đơn có sản phẩm cấm vận chuyển bằng hãng đã chọn (hàng đặc thù) | Đổi hãng/dịch vụ phù hợp; nếu không có → thông báo Order Management |
| Khách yêu cầu không kèm hoá đơn giá trị trong kiện (giao làm quà) | Theo cờ "gift order": in packing slip ẩn giá |
| Đơn nhiều kiện, một kiện lỗi | Chỉ đóng lại kiện lỗi; các kiện khác vẫn tiến hành; shipment chờ đủ kiện hoặc cho giao lệch chuyến theo chính sách |

---

## 7. Liên kết với các module khác

| Module | Quan hệ |
|--------|---------|
| [07 · Picking](../07-picking/README.md) | Nhận hàng đã pick tại staging; báo ngược khi kiểm đơn phát hiện lệch |
| [19 · Production](../19-production/README.md) | Nhận thành phẩm in đạt QC tại khu `PACKING`; báo ngược khi thành phẩm sai thiết kế / mẫu đã duyệt (lệnh quay về `Ready` để in bù) |
| [09 · Shipping](../09-shipping/README.md) | Bàn giao kiện đã đóng (kèm khối lượng, kích thước, nhãn kiện nội bộ) cho bước giao vận; cung cấp kích thước, khối lượng & cờ cho phép xoay của kiện cho Load Planning |
| [17 · Order Management](../../ecommerce/17-order-management/README.md) | Cập nhật trạng thái `Packed`; nhận yêu cầu sửa địa chỉ / xử lý huỷ |
| [12 · Product Design](../../ecommerce/12-product-design/README.md) | Cung cấp design snapshot để đối chiếu thành phẩm in |
| [01 · Product Creation](../01-product-creation/README.md) | Kích thước/khối lượng/quy cách & cờ dễ vỡ của SKU cho cartonization; cờ cho phép xoay của SKU làm giá trị mặc định cho kiện |
| [00 · System Overview §5](../../00-system-overview/README.md) | Bước 2 của luồng Fulfillment |
