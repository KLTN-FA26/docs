# 09 — Giao hàng cho đơn vị vận chuyển (Shipping)

## 1. Mục đích (Purpose)

Bàn giao các kiện hàng đã [đóng gói](../08-packing/README.md) cho **đơn vị vận chuyển ngoài (external carrier / 3PL)**, tạo **vận đơn (shipping label / AWB)**, ghi nhận **bàn giao (handover / manifest)**, và **theo dõi hành trình giao** đến khi đơn được giao thành công hoặc xử lý sự cố (giao thất bại, hoàn hàng).

Đây là **bước cuối của luồng Fulfillment trong phạm vi kho**. Sau khi giao thành công, [Order Management](../../ecommerce/17-order-management/README.md) đóng đơn.

Module hỗ trợ **hai hình thức giao**:

| Hình thức | Áp dụng | Đặc trưng |
|-----------|---------|-----------|
| **Gửi kiện qua hãng vận chuyển (Parcel)** | Đơn khách thông thường | Mỗi kiện có nhãn vận đơn & tracking từ API hãng; bàn giao cho tài xế hãng theo manifest ([§4.1](#41-gửi-kiện-qua-hãng-vận-chuyển-parcel)) |
| **Giao nguyên chuyến (Full Truckload – FTL)** | Đơn sỉ khối lượng lớn được chỉ định giao nguyên chuyến; lệnh chuyển kho ([10](../10-inter-warehouse-transfer/README.md)) đi xe tải / container | Thuê nguyên xe tải hoặc container; trước khi thuê xe, **lập kế hoạch xếp hàng 3D (Load Planning)** để biết cần loại xe nào, bao nhiêu chiếc, mỗi kiện đặt ở đâu trong thùng xe ([§4.2](#42-giao-nguyên-chuyến--lập-kế-hoạch-xếp-hàng-3d-load-planning)) |

> ⚠️ Giả định: đơn khách giao nguyên chuyến do **Điều phối đơn hàng (Order Coordinator)** chỉ định khi release đơn, khách không tự chọn hình thức này trên storefront; phí vận chuyển và cách thu COD cho đơn giao nguyên chuyến chưa xác định. Xem [open-questions](../../open-questions/README.md).

---

## 2. Actor liên quan

| Actor | Vai trò |
|-------|---------|
| **Nhân viên kho / Dispatch Staff** | Quét kiện, gom chuyến, bàn giao cho tài xế hãng vận chuyển, ký nhận manifest; với giao nguyên chuyến: lập & chốt kế hoạch chuyến xe, kiểm tra / chỉnh tay trên mô hình 3D, đặt xe, giám sát xếp hàng lên xe theo sơ đồ |
| **Quản lý kho (Warehouse Manager)** | Khai báo **danh mục loại xe** (kích thước lòng thùng, tải trọng, số chiếc sẵn có, giá thuê); xử lý kế hoạch chuyến xe bị huỷ / phải lập lại |
| **Hệ thống (Carrier Integration)** | Kết nối API hãng vận chuyển: tạo nhãn, lấy tracking, nhận webhook cập nhật trạng thái |
| **Hệ thống (Load Planner)** | Tính phương án xếp kiện lên xe / container: số xe từng loại, vị trí & hướng đặt từng kiện, % lấp đầy; dựng mô hình 3D và sơ đồ xếp |
| **Đơn vị vận chuyển (Carrier / 3PL)** | Nhận hàng, vận chuyển, cập nhật trạng thái, giao cho khách, xử lý hoàn; với giao nguyên chuyến: cung cấp xe tải / container theo kế hoạch chuyến xe đã chốt |
| **Nhân viên Sale (Sales Staff)** | Theo dõi đơn trễ, xử lý khiếu nại giao hàng (CSKH), liên hệ khách |
| **Khách hàng** | Nhận hàng; theo dõi tracking; xác nhận nhận (hoặc từ chối) |

---

## 3. Input / Output

### Input

| Trường | Mô tả | Bắt buộc |
|--------|-------|----------|
| Kiện đã `Packed` | Package ID, khối lượng, kích thước, nội dung, giá trị khai báo | Có |
| Phương thức / dịch vụ vận chuyển | Khách chọn khi đặt, hoặc rule chọn hãng | Có |
| Địa chỉ giao & người nhận | Từ đơn hàng | Có |
| Địa chỉ gửi & địa chỉ hoàn hàng | Địa chỉ của kho xuất; địa chỉ hoàn hàng lấy `return_address` của kho, nếu không khai báo thì dùng địa chỉ kho (xem [06](../06-warehouse-map-slotting/README.md)) | Có |
| Điều khoản thu hộ (COD amount) nếu có | | Không |
| Tài khoản hãng vận chuyển & cấu hình dịch vụ | API key, hợp đồng giá | Có (cấu hình hệ thống) |
| Lịch lấy hàng (pickup schedule) / điểm gửi | | Không |

**Input bổ sung cho giao nguyên chuyến (Load Planning):**

| Trường | Mô tả | Bắt buộc |
|--------|-------|----------|
| **Danh sách kiện cần xếp** | Kiện `Packed` tại khu `DISPATCH` thuộc đơn giao nguyên chuyến hoặc lệnh chuyển kho: dài × rộng × cao, khối lượng, số lượng, **cờ cho phép xoay** (kế thừa từ SKU, Packer được ghi đè — xem [08](../08-packing/README.md)), đơn / dòng hàng mà kiện thuộc về. Có thể nhập danh sách kiện từ **file Excel theo mẫu** | Có |
| **Danh mục loại xe (Vehicle Type)** | Kích thước lòng thùng (dài × rộng × cao), tải trọng tối đa, số chiếc sẵn có, giá thuê. Cài sẵn container **20DC, 40DC, 40HC, 45HC**; xe tải được thêm bằng cách khai báo kích thước lòng thùng & tải trọng của từng loại | Có (cấu hình hệ thống) |
| **Chế độ chọn xe** | Một trong các chế độ ở [§4.2](#42-giao-nguyên-chuyến--lập-kế-hoạch-xếp-hàng-3d-load-planning) | Có (mặc định: *Một loại*) |
| Phân nhóm kiện chính / phụ | Chỉ dùng với chế độ *Ưu tiên + hàng chèn* | Không |

### Output

| Kết quả | Dùng bởi |
|---------|----------|
| **Shipping label / AWB** + **tracking number** | Dán lên kiện (08 Packing), gửi khách |
| **Shipment** — nhóm 1..n kiện của một đơn, có trạng thái & lịch sử tracking | 17 Order Management, khách hàng |
| **Manifest / bàn giao** — danh sách kiện giao cho hãng trong một lần | Đối soát với hãng vận chuyển |
| **Cập nhật trạng thái đơn**: `Shipped` → `In Transit` → `Delivered` / `Delivery Failed` / `Returned` | 17 Order Management, 15 Payment (đối soát COD) |
| **Chi phí vận chuyển thực tế** (từ hãng) | Đối soát cước, kế toán |
| **Sự kiện hoàn hàng (RTO – Return To Origin)** | 17 Order Management, quy trình nhập lại kho ([Receipt](../03-receipt/README.md)) |
| **Kế hoạch chuyến xe (Load Plan)** — với giao nguyên chuyến: số xe của từng loại; toạ độ (x, y, z) & hướng đặt của từng kiện; % lấp đầy thể tích & tải trọng của từng xe; danh sách kiện không xếp được | Dispatch Staff (đặt xe, xếp hàng lên xe), [10](../10-inter-warehouse-transfer/README.md) |
| **Mô hình 3D** của từng xe (xem & chỉnh tay được) và **sơ đồ xếp xuất PDF** | Dispatch Staff, tài xế / bốc xếp |

### Dữ liệu tối thiểu cần lưu (giao nguyên chuyến)

| Thực thể | Trường chính |
|----------|--------------|
| `VehicleType` | `id`, `name`, `inner_length`, `inner_width`, `inner_height`, `max_payload`, `available_count`, `rental_price?`, `is_preset`, `status` |
| `LoadPlan` | `id`, `code`, `warehouse_id`, `source_type` (đơn khách / TO), `selection_mode`, `status`, `confirmed_at?`, `dispatched_at?` |
| `LoadPlanVehicle` | `id`, `load_plan_id`, `vehicle_type_id`, `seq`, `volume_fill_pct`, `weight_fill_pct`, `manifest_ref?` |
| `LoadPlacement` | `load_plan_vehicle_id`, `package_id`, `x`, `y`, `z`, `orientation` |
| `LoadPlanUnplaced` | `load_plan_id`, `package_id`, `reason` |

---

## 4. Luồng nghiệp vụ từng bước (Step-by-step)

### 4.1. Gửi kiện qua hãng vận chuyển (Parcel)

1. **Tạo nhãn vận đơn (Create label)** — thường thực hiện ngay tại [Packing](../08-packing/README.md): hệ thống gọi API hãng vận chuyển với thông tin kiện & địa chỉ → nhận `tracking number` + file nhãn. Nếu tạo lỗi (địa chỉ sai, quá tải hãng) → đơn `On Hold`, chuyển Order Management.
2. **Gom nhóm shipment** — các kiện của cùng một đơn (và cùng hãng) gộp thành một **shipment**.
3. **Lên chuyến / chờ hãng lấy hàng** — xếp kiện vào khu chờ giao theo hãng & khung giờ pickup; hoặc chuẩn bị mang ra điểm gửi.
4. **Bàn giao (Handover / Manifest)** — khi tài xế hãng đến: quét từng kiện, đối chiếu danh sách, in **manifest**, hai bên ký xác nhận. Trạng thái shipment → `Handed Over` / `Picked up by carrier`.
5. **Cập nhật đơn** — Order chuyển `Shipped`; gửi khách email/SMS kèm tracking link.
6. **Theo dõi hành trình (Tracking)** — hệ thống nhận **webhook / poll** trạng thái từ hãng: `In Transit`, `Out for Delivery`, `Delivered`, `Delivery Attempt Failed`, `Held at hub`, `Returning`…
7. **Giao thành công (Delivered)** — nhận sự kiện `Delivered` (kèm thời gian, người ký nhận / ảnh POD nếu có). Order chuyển `Delivered`.
8. **Giao thất bại (Delivery failed)** — hãng báo không giao được (khách vắng, từ chối, sai địa chỉ):
   - Hệ thống ghi nhận số lần thử; thông báo CSKH liên hệ khách.
   - Theo chính sách: hãng giao lại (re-attempt) hoặc chuyển hoàn (RTO).
9. **Hoàn hàng về kho (RTO / Return)** — kiện quay về; kho tiếp nhận qua luồng [Receipt](../03-receipt/README.md) dạng "hàng hoàn", kiểm tra tình trạng, nhập lại tồn (`Available` nếu nguyên vẹn / `Quarantine` nếu nghi hư / huỷ nếu hỏng). Order chuyển `Returned`.
10. **Đối soát COD & cước** — với đơn COD: đối chiếu tiền hãng thu hộ chuyển về ([15 Payment](../../ecommerce/15-payment/README.md)). Đối soát bảng cước thực tế với hãng.
11. **Đóng đơn** — khi `Delivered` và (nếu COD) tiền đã về → [Order Management](../../ecommerce/17-order-management/README.md) chuyển đơn `Completed` / `Closed`.

### 4.2. Giao nguyên chuyến — Lập kế hoạch xếp hàng 3D (Load Planning)

Mục đích: từ danh sách kiện cần giao, hệ thống tính **cần loại xe nào, bao nhiêu chiếc**, và **mỗi kiện đặt ở vị trí nào trong thùng xe**; kết quả hiển thị 3D để kiểm tra trước khi thuê xe.

1. **Tạo kế hoạch chuyến xe (`Draft`)** — Dispatch Staff chọn các kiện `Packed` tại khu `DISPATCH` (của các đơn giao nguyên chuyến hoặc của một lệnh chuyển kho), hoặc nhập danh sách kiện từ file Excel theo mẫu; chọn các loại xe được dùng và **chế độ chọn xe**. Kiện giao nguyên chuyến không tạo nhãn hãng; shipment của đơn khách bắt đầu ở `Ready to Dispatch`.
2. **Tính phương án xếp** — Load Planner xếp kiện theo **quy tắc xếp** (BR-10) và chế độ đã chọn → kế hoạch `Planned`. Hệ thống trả về số xe từng loại, toạ độ & hướng đặt từng kiện, % lấp đầy thể tích & tải trọng từng xe, danh sách kiện không xếp được.
3. **Kiểm tra trên mô hình 3D & chỉnh tay** — Dispatch Staff xem từng xe; có thể **kéo thả kiện** trên mô hình rồi **tính lại riêng xe đó**. Nếu còn kiện không xếp được: thêm xe / đổi loại xe / đổi chế độ rồi tính lại (quay về `Draft` nếu đổi danh sách kiện, loại xe hoặc chế độ), hoặc xác nhận để kiện lại khu `DISPATCH` cho chuyến sau.
4. **Chốt kế hoạch (`Confirmed`)** — khoá danh sách kiện & sơ đồ xếp; xuất **PDF sơ đồ xếp** cho từng xe.
5. **Đặt xe & bàn giao** — Dispatch Staff đặt xe với đơn vị vận chuyển theo số xe / loại xe của kế hoạch. Khi xe đến: xếp kiện theo sơ đồ, quét từng kiện, in **manifest theo từng xe**, hai bên ký xác nhận. Shipment của các kiện → `Handed Over`; đơn khách → `Shipped`; lệnh chuyển kho → `In Transit` ([10](../10-inter-warehouse-transfer/README.md)). Khi mọi xe đã bàn giao & rời kho → kế hoạch `Dispatched`.
   > ⚠️ Giả định: việc đặt xe diễn ra ngoài hệ thống; hệ thống chỉ ghi nhận thông tin xe (biển số, tài xế) gắn với từng xe của kế hoạch. Xem [open-questions](../../open-questions/README.md).
6. **Theo dõi & giao** — không có tracking từ API hãng; Dispatch Staff cập nhật trạng thái shipment thủ công theo cùng vòng đời ở §5 (`In Transit` → `Out for Delivery` → `Delivered` / `Delivery Failed`, kèm POD nếu có). Với lệnh chuyển kho, kho đích nhận hàng theo [10](../10-inter-warehouse-transfer/README.md).

**Chế độ chọn xe:**

| Chế độ | Cách chọn xe & xếp kiện |
|--------|-------------------------|
| **Một loại** | Chọn loại xe xếp tốt nhất, rồi dùng nhiều chiếc cùng loại đó |
| **Ít xe nhất** | Trộn nhiều loại xe để tổng số xe ít nhất |
| **Giữ nguyên dòng** | Các kiện cùng một dòng hàng đi chung một xe |
| **Ưu tiên + hàng chèn** | Hàng chính quyết định cần bao nhiêu xe; hàng phụ chỉ chèn vào chỗ trống còn lại |
| **Vượt sức chứa** | Xếp tối đa những kiện vừa với số xe sẵn có và báo phần phải bỏ lại |
| **Chỉnh tay** | Kéo thả kiện trên mô hình 3D, rồi tính lại một xe |

**Cách tính phương án (tóm tắt):** đặt từng kiện vào **góc trống thấp nhất và sâu nhất** mà kiện vừa và được đỡ chắc. Hệ thống thử nhiều thứ tự xếp ngẫu nhiên trong một khoảng thời gian giới hạn và giữ phương án tốt nhất, đánh giá theo thứ tự:

1. Ít kiện bị bỏ lại nhất;
2. Ít xe nhất;
3. Ít khoảng trống thừa nhất.

**Giới hạn phạm vi hiện tại** (chưa hỗ trợ — xem [open-questions](../../open-questions/README.md)):

- Chưa tính **chi phí thuê xe**: xe được chọn theo số xe và độ lấp đầy, không theo giá; giá thuê trong danh mục loại xe chỉ lưu để tham khảo.
- Chưa tính **phân bố trọng tâm**, **hàng dễ vỡ** hay **giới hạn chồng tầng**.
- Chưa sắp xếp theo **thứ tự điểm giao**: hàng giao trước chưa được đặt ở phía cửa xe.
- Chỉ đặt kiện **vuông góc với các cạnh thùng** (không đặt nghiêng).

---

## 5. Trạng thái / Vòng đời (Lifecycle)

### Trạng thái Shipment

| Trạng thái | Ý nghĩa | Chuyển tiếp | Nguồn cập nhật |
|------------|---------|-------------|----------------|
| `Label Created` | Đã có nhãn & tracking, kiện chưa giao hãng (chỉ với gửi kiện qua hãng) | → `Ready to Dispatch`, → `Cancelled` (void label) | Hệ thống |
| `Ready to Dispatch` | Kiện ở khu chờ giao; với giao nguyên chuyến, shipment **bắt đầu từ trạng thái này** (không có nhãn hãng) | → `Handed Over` | Dispatch Staff |
| `Handed Over` | Đã bàn giao hãng (có manifest); với giao nguyên chuyến: đã xếp lên xe theo sơ đồ & ký manifest của xe | → `In Transit`, → `Exception` | Dispatch Staff / Carrier |
| `In Transit` | Đang vận chuyển | → `Out for Delivery`, → `Exception` | Carrier webhook / Dispatch Staff (giao nguyên chuyến) |
| `Out for Delivery` | Đang đi giao | → `Delivered`, → `Delivery Failed` | Carrier / Dispatch Staff (giao nguyên chuyến) |
| `Delivery Failed` | Giao không thành công (1..n lần) | → `Out for Delivery` (thử lại), → `Returning` | Carrier |
| `Returning` | Đang hoàn về kho (RTO) | → `Returned` | Carrier |
| `Delivered` | Giao thành công | → *(đơn Completed)* | Carrier |
| `Returned` | Đã về kho & xử lý nhập lại | *(cuối)* | Warehouse |
| `Exception` | Sự cố (mất, hỏng, kẹt hải quan, tra soát) | → trạng thái phù hợp sau xử lý | CSKH / Carrier |
| `Cancelled` | Void nhãn trước khi bàn giao | *(cuối)* | Hệ thống |

```mermaid
stateDiagram-v2
    [*] --> LabelCreated: Gửi kiện qua hãng
    [*] --> ReadyToDispatch: Giao nguyên chuyến
    LabelCreated --> ReadyToDispatch
    LabelCreated --> Cancelled: Void label
    ReadyToDispatch --> HandedOver: Bàn giao + manifest
    HandedOver --> InTransit
    InTransit --> OutForDelivery
    OutForDelivery --> Delivered
    OutForDelivery --> DeliveryFailed
    DeliveryFailed --> OutForDelivery: Giao lại
    DeliveryFailed --> Returning: Chuyển hoàn
    Returning --> Returned
    InTransit --> Exception
    HandedOver --> Exception
    Exception --> InTransit: Xử lý xong
    Delivered --> [*]
    Returned --> [*]
    Cancelled --> [*]
```

### Trạng thái Kế hoạch chuyến xe (Load Plan)

| Trạng thái | Ý nghĩa | Chuyển tiếp cho phép | Ai thực hiện |
|------------|---------|----------------------|--------------|
| `Draft` | Đang chọn kiện, loại xe, chế độ chọn xe | → `Planned`, → `Cancelled` | Dispatch Staff |
| `Planned` | Đã có phương án xếp; xem 3D, chỉnh tay & tính lại từng xe | → `Planned` (chỉnh tay, tính lại), → `Draft` (đổi kiện / loại xe / chế độ), → `Confirmed`, → `Cancelled` | Dispatch Staff / Load Planner |
| `Confirmed` | Đã chốt; khoá danh sách kiện & sơ đồ xếp; chờ đặt xe & xếp hàng | → `Dispatched`, → `Planned` (mở lại khi chưa bắt đầu xếp lên xe), → `Cancelled` | Dispatch Staff |
| `Dispatched` | Mọi xe đã xếp theo sơ đồ, ký manifest & rời kho | *(cuối)* | Dispatch Staff |
| `Cancelled` | Huỷ trước khi xe rời kho; kiện trở lại khu `DISPATCH` chờ kế hoạch khác | *(cuối)* | Dispatch Staff / Warehouse Manager |

```mermaid
stateDiagram-v2
    [*] --> Draft
    Draft --> Planned: Tính phương án xếp
    Planned --> Planned: Chỉnh tay + tính lại xe
    Planned --> Draft: Đổi kiện / loại xe / chế độ
    Planned --> Confirmed: Chốt kế hoạch
    Confirmed --> Planned: Mở lại (chưa xếp lên xe)
    Confirmed --> Dispatched: Mọi xe bàn giao & rời kho
    Draft --> Cancelled
    Planned --> Cancelled
    Confirmed --> Cancelled
    Dispatched --> [*]
    Cancelled --> [*]
```

---

## 6. Ràng buộc nghiệp vụ (Business Rules) & Edge case

### Business rules

- **BR-01:** Chỉ tạo nhãn cho kiện `Packed` hợp lệ (đúng khối lượng/kích thước, địa chỉ hợp lệ).
- **BR-02:** Nhãn đã tạo mà không giao (đơn huỷ, đóng lại kiện) **phải void** với hãng trong thời hạn cho phép để tránh bị tính cước.
- **BR-03:** Đơn chỉ chuyển `Shipped` khi **tất cả kiện** của đơn đã `Handed Over` — trừ khi chính sách cho phép giao lệch chuyến (split shipment).
- **BR-04:** Với đơn COD, số tiền thu hộ trên vận đơn phải khớp số phải thu của đơn (đã trừ phần đã thanh toán online nếu có).
- **BR-05:** Trạng thái đơn phía khách hàng phản ánh trạng thái shipment gần nhất; nhiều kiện → hiển thị trạng thái tổng hợp (ví dụ "đang giao" khi có ít nhất 1 kiện `Out for Delivery`).
- **BR-06:** RTO nhập lại kho phải qua [Receipt](../03-receipt/README.md) (không cộng thẳng tồn) để kiểm tra tình trạng hàng.
- **BR-07:** Chi phí vận chuyển thu của khách (nếu có) và cước trả hãng là hai khoản độc lập; đối soát riêng.

**Giao nguyên chuyến & Load Planning:**

- **BR-08:** Load Planning chỉ áp dụng cho **giao nguyên chuyến** (đơn khách được chỉ định giao nguyên chuyến và lệnh chuyển kho đi xe tải / container). Kiện gửi qua hãng vận chuyển không cần kế hoạch chuyến xe.
- **BR-09:** Chỉ đưa vào kế hoạch kiện `Packed` có **đủ kích thước & khối lượng**. Một kiện chỉ thuộc **tối đa một** kế hoạch chuyến xe chưa `Cancelled`.
- **BR-10:** Quy tắc xếp bắt buộc (ràng buộc cứng):
  - Kiện phải được đỡ **ít nhất 98% mặt đáy**; không lơ lửng, không chìa ra ngoài lòng thùng.
  - Tổng khối lượng các kiện trên một xe **không vượt tải trọng** của xe.
  - Kiện có cờ cho phép xoay được xoay theo **6 hướng**; kiện không cho phép xoay giữ nguyên hướng đặt.
  - **Chừa các góc gia cố** trên trần thùng.
  - Kiện chỉ đặt **vuông góc với các cạnh thùng**.
- **BR-11:** Chỉnh tay trên mô hình 3D vẫn phải thoả BR-10; vị trí vi phạm bị từ chối. Sau khi chỉnh, hệ thống tính lại xe bị chỉnh.
- **BR-12:** Số xe của một loại trong phương án **không vượt số chiếc sẵn có** của loại đó.
- **BR-13:** Kế hoạch chỉ được `Confirmed` khi mọi kiện đã có vị trí, hoặc các kiện không xếp được đã được Dispatch Staff xác nhận để lại.
- **BR-14:** Sau `Confirmed`, danh sách kiện và sơ đồ xếp bị khoá. Muốn sửa phải mở lại về `Planned` khi **chưa bắt đầu xếp lên xe**; đã bắt đầu xếp thì chỉ được huỷ kế hoạch có kiểm soát (dỡ kiện về khu `DISPATCH`).
- **BR-15:** Mỗi xe của kế hoạch có **manifest riêng**. Kế hoạch chỉ `Dispatched` khi **mọi xe** đã xếp theo sơ đồ, ký manifest & rời kho.

### Edge case

| Tình huống | Hướng xử lý |
|------------|-------------|
| API hãng vận chuyển lỗi/không phản hồi khi tạo nhãn | Retry theo hàng đợi; nếu vẫn lỗi → đơn `On Hold`, cảnh báo; cho chọn hãng thay thế |
| Địa chỉ giao sai/không giao tới (vùng hãng không phục vụ) | Chặn tạo nhãn; CSKH liên hệ khách đổi địa chỉ hoặc đổi hãng |
| Kiện thất lạc trong quá trình vận chuyển | `Exception`; mở tra soát với hãng; theo chính sách bồi thường / giao lại hàng mới |
| Hãng cập nhật `Delivered` nhưng khách báo chưa nhận | Giữ đơn ở trạng thái tranh chấp; đối chiếu POD; xử lý CSKH |
| Khách từ chối nhận (đổi ý) | `Delivery Failed` → `Returning` → `Returned`; áp phí/hoàn tiền theo chính sách |
| Một phần đơn giao được, một phần hỏng trên đường | Kiện hỏng vào `Exception`/RTO; kiện tốt vẫn `Delivered`; đơn ở trạng thái giao một phần |
| COD: tiền hãng chuyển về thiếu/chậm | Ghi nhận công nợ hãng; đối soát; không đóng đơn tài chính cho tới khi khớp |
| Nhiều hãng cho một đơn (kiện to đi hãng A, kiện nhỏ hãng B) | Mỗi kiện có tracking & hãng riêng; shipment gộp hiển thị nhiều tracking |
| Load Planning: còn kiện không xếp được | Hệ thống trả danh sách kiện bị bỏ lại; Dispatch Staff thêm xe / đổi loại xe / đổi chế độ rồi tính lại, hoặc xác nhận để kiện lại khu `DISPATCH` cho chuyến sau (đơn khách khi đó giao lệch chuyến theo chính sách split shipment) |
| Load Planning: kiện thiếu kích thước / khối lượng | Chặn tính phương án; yêu cầu cân đo lại kiện ở [08](../08-packing/README.md) |
| Chế độ *Giữ nguyên dòng*: các kiện của một dòng hàng không vừa một xe | Phần vượt được báo là không xếp được; Dispatch Staff chọn loại xe lớn hơn hoặc đổi chế độ |
| Xe thực tế đến khác loại / kích thước so với kế hoạch | Mở lại kế hoạch về `Planned` (chưa xếp lên xe), đổi loại xe & tính lại, chốt lại trước khi xếp |
| Kiện thực tế không vừa vị trí trên sơ đồ khi xếp lên xe | Dừng xếp xe đó; đo lại kiện; nếu kích thước sai thì cập nhật ở [08](../08-packing/README.md) và huỷ / lập lại kế hoạch theo BR-14 |
| Đơn bị huỷ khi kiện đã nằm trong kế hoạch chuyến xe | Chưa xếp lên xe → mở lại kế hoạch, bỏ kiện & tính lại; đã xếp lên xe → dỡ kiện, xử lý theo [08](../08-packing/README.md) (mở kiện, put-back) |

---

## 7. Liên kết với các module khác

| Module | Quan hệ |
|--------|---------|
| [08 · Packing](../08-packing/README.md) | Nhận kiện đã đóng; trả về nhãn & tracking để dán lên kiện; nhận kích thước, khối lượng & cờ cho phép xoay của kiện cho Load Planning |
| [10 · Inter-warehouse Transfer](../10-inter-warehouse-transfer/README.md) | Lệnh chuyển kho đi xe tải / container dùng Load Planning trước khi xuất kho nguồn (Dispatch) |
| [01 · Product Creation](../01-product-creation/README.md) | Cờ cho phép xoay của SKU là giá trị mặc định cho cờ xoay của kiện |
| [17 · Order Management](../../ecommerce/17-order-management/README.md) | Đẩy trạng thái `Shipped` / `In Transit` / `Delivered` / `Returned`; nhận yêu cầu huỷ, đổi địa chỉ; nhận chỉ định hình thức giao nguyên chuyến cho đơn |
| [15 · Payment](../../ecommerce/15-payment/README.md) | Đối soát tiền COD hãng thu hộ; kích hoạt hoàn tiền khi RTO |
| [03 · Receipt](../03-receipt/README.md) | Hàng hoàn (RTO) nhập lại kho qua luồng Receipt dạng "return" |
| [16 · Sales Chat](../../ecommerce/16-sales-chat/README.md) | Kênh CSKH xử lý đơn trễ / khiếu nại giao hàng, chia sẻ tracking |
| [00 · System Overview §5](../../00-system-overview/README.md) | Bước 3 (cuối) của luồng Fulfillment |
