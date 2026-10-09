# 06 — Kho Map 2D/3D & Gợi ý vị trí (Warehouse Map & Slotting Suggestion)

## 1. Mục đích (Purpose)

Cung cấp **mô hình không gian của kho** (sơ đồ 2D và trực quan 3D) và **động cơ gợi ý vị trí lưu trữ (slotting / putaway recommendation engine)**:

- **Warehouse Map 2D/3D:** biểu diễn mặt bằng kho — **kệ (Shelf) → tầng kệ (Shelf Level) → ô (Bin)**, các **khu vực trên sàn (Area)**, **tường & cửa (Boundary)**, nhóm theo **Zone** (tùy chọn) — kèm trạng thái lấp đầy, giúp NV kho định vị nhanh và quản lý trực quan.
- **Slotting Suggestion:** với mỗi lần [Putaway](../05-putaway/README.md) hoặc [Intra-warehouse Transfer](../11-intra-warehouse-transfer/README.md), đề xuất vị trí **tối ưu** dựa trên bộ quy tắc (rules) và dữ liệu (lớp lưu trữ, kích thước SKU, tần suất xuất, chỗ trống, FIFO/FEFO, khoảng cách đi thực tế trên bản đồ).

Mục tiêu: giảm quãng đường di chuyển, tăng mật độ lưu trữ hợp lý, đảm bảo an toàn và tuân thủ FEFO, giảm phụ thuộc kinh nghiệm cá nhân.

> ⚠️ Giả định lớn: Đề bài **không nêu tiêu chí cụ thể** của thuật toán gợi ý. Tài liệu này đề xuất một bộ tiêu chí mặc định (bên dưới) — **cần Product Owner xác nhận trọng số & thứ tự ưu tiên**. Xem [open-questions](../../open-questions/README.md).

### Mô hình không gian kho

| Thành phần | Thuộc về | Vai trò |
|------------|----------|---------|
| **Kho (Warehouse)** | — | Khung bản đồ (đơn vị đo, chiều rộng × chiều dài), địa chỉ kho & địa chỉ hoàn hàng, tiền tố mã (prefix) |
| **Zone** | Kho | **Nhóm logic tùy chọn** (có màu hiển thị) để gom kệ/khu vực, phục vụ hiển thị và zone picking. Zone không mang quy tắc lưu trữ |
| **Kệ (Shelf)** | Kho, Zone (tùy chọn) | Khối kệ đặt trên mặt bằng (toạ độ, kích thước, góc xoay); khai báo **mặt lấy hàng** (bắc/đông/nam/tây) và **lớp lưu trữ mặc định** |
| **Tầng kệ (Shelf Level)** | Kệ | Tầng thứ n của kệ; cao độ, chiều cao lọt, tải trọng tối đa (phục vụ 3D & kiểm tra tải) |
| **Ô (Bin)** | Tầng kệ | Điểm chứa hàng nhỏ nhất trên kệ; toạ độ tương đối trong kệ, sức chứa theo số đơn vị, cờ cho phép lấy hàng / cho phép cất hàng, loại ô, lớp lưu trữ ghi đè |
| **Khu vực (Area)** | Kho, Zone (tùy chọn) | Vùng trên sàn **không phải kệ**, có mã khu vực riêng: khu nhận, kiểm tra chất lượng, cách ly, xưởng in, đóng gói, chờ giao, khu tràn, hoặc khu không chứa hàng |
| **Ranh giới (Boundary)** | Kho | Đoạn **tường (WALL)** hoặc **cửa (DOOR)**; cho biết có đi qua được không — dùng tính đường đi |

**Vị trí tồn kho (location)** = một **Bin** hoặc một **Area chứa hàng**; mỗi vị trí có một **mã vị trí đầy đủ** duy nhất toàn hệ thống (BR-10). Lối đi không khai báo thành thực thể: là phần sàn trống giữa kệ, khu vực vật cản và tường.

---

## 2. Actor liên quan

| Actor | Vai trò |
|-------|---------|
| **Quản trị hệ thống (System Admin) / Quản lý kho** | Khai báo kho, vẽ bản đồ (tường/cửa, khu vực, kệ, tầng, bin), thuộc tính vị trí, cấu hình quy tắc slotting & trọng số |
| **Hệ thống (Slotting Engine)** | Tính toán và trả về gợi ý theo thời điểm putaway/transfer; tính khoảng cách đi thực tế trên bản đồ |
| **Nhân viên kho** | Xem map để định vị; nhận & xác nhận (hoặc ghi đè) gợi ý |
| **Quản lý kho** | Chạy phân tích tối ưu định kỳ (re-slotting), duyệt di chuyển tối ưu hoá; đổi trạng thái kệ/bin/khu vực/cửa |

---

## 3. Input / Output

### Input

| Nhóm | Dữ liệu | Nguồn |
|------|---------|-------|
| **Khung kho** | Tên, tiền tố mã (prefix), địa chỉ, địa chỉ hoàn hàng, đơn vị bản đồ, chiều rộng × chiều dài bản đồ | System Admin khai báo |
| **Bố cục mặt bằng** | Zone (tùy chọn); tường & cửa; khu vực (mã, loại, toạ độ, kích thước, góc xoay, vật cản); kệ (mã, toạ độ, kích thước, góc xoay, vật cản, mặt lấy hàng) | System Admin / Quản lý kho |
| **Cấu trúc kệ** | Tầng (thứ tự, cao độ, chiều cao lọt, tải trọng tối đa); bin (mã, toạ độ tương đối, kích thước, sức chứa theo đơn vị, cho phép lấy/cất, loại ô) — nhập từng bin, hoặc **tự sinh** theo số hàng × số cột và kiểu đặt mã (BR-14) | System Admin / Quản lý kho |
| **Lớp lưu trữ (storage class)** | Lớp mặc định của kệ; lớp ghi đè của bin | System Admin |
| **Thuộc tính SKU** | Lớp lưu trữ yêu cầu, kích thước, khối lượng, quy cách đóng gói | [01 Product Creation](../01-product-creation/README.md) (Inventory Item) |
| **Dữ liệu vận hành** | Tần suất xuất (order frequency / velocity), phân loại ABC/XYZ, tồn hiện tại theo vị trí, lô & hạn dùng | 03, 05, 07, tồn kho |

### Output

| Kết quả | Dùng bởi |
|---------|----------|
| **Bản đồ kho 2D/3D tương tác** (heatmap độ lấp đầy, tìm vị trí, click xem chi tiết bin/khu vực) | Warehouse Staff, Manager |
| **Mã vị trí đầy đủ** (location barcode) cho từng bin và khu vực | 05, 07, 11 (quét xác nhận) |
| **Khoảng cách / đường đi thực tế** giữa các điểm trên bản đồ | Slotting scoring, 07 Picking (lộ trình) |
| **Gợi ý vị trí (ranked suggestions)** cho một putaway/transfer: danh sách vị trí + điểm số + lý do | 05 Putaway, 11 Intra-transfer |
| **Báo cáo tối ưu / đề xuất re-slotting** (di chuyển SKU bán chạy ra gần khu đóng gói…) | Manager |
| **Chỉ số sức khoẻ kho** (utilization %, honeycombing, số vị trí quá tải) | Manager |

---

## 4. Luồng nghiệp vụ từng bước (Step-by-step)

### 4.1. Thiết lập & duy trì bản đồ kho

1. **Khai báo kho** — tên, prefix, địa chỉ, địa chỉ hoàn hàng, đơn vị bản đồ và kích thước khung bản đồ.
2. **(Tùy chọn) Khai báo Zone** — tên, màu hiển thị.
3. **Vẽ ranh giới** — các đoạn tường và cửa; với cửa, đánh dấu có đi qua được không và trạng thái vận hành (mở/đóng).
4. **Đặt khu vực (Area)** — nhập mã khu vực, chọn loại, vẽ vị trí & kích thước, gán zone nếu có; đánh dấu vật cản nếu không đi xuyên được. Hệ thống sinh **mã vị trí đầy đủ** của khu vực (BR-10).
5. **Đặt kệ (Shelf)** — mã, vị trí, kích thước, góc xoay, gán zone nếu có; chọn **mặt lấy hàng** và **lớp lưu trữ mặc định**.
6. **Khai báo tầng kệ** — thứ tự tầng, cao độ, chiều cao lọt, tải trọng tối đa.
7. **Khai báo bin trên từng tầng** — theo một trong hai cách:
   - **Nhập từng bin:** mã, vị trí tương đối trong kệ, kích thước, sức chứa theo đơn vị, cho phép lấy hàng / cất hàng, loại ô, lớp lưu trữ ghi đè (nếu khác kệ).
   - **Tự sinh bin (autogenerate):** chọn kệ, chọn các tầng chưa có bin, nhập số hàng × số cột và kiểu đặt mã. Hệ thống chia đều mặt kệ, tính toạ độ & kích thước từng bin, đặt mã theo kiểu đã chọn và lưu các bin (xem BR-14); thuộc tính khác của bin chỉnh sửa sau.

   Với mỗi bin, hệ thống sinh **mã vị trí đầy đủ** theo prefix kho (BR-10).
8. **Kiểm tra bố cục** — hệ thống kiểm tra các ràng buộc hình học (xem BR-06 → BR-09) trước khi lưu; sinh mô hình 2D (mặt bằng) và 3D (kệ theo cao độ tầng).
9. **Duy trì** — thêm/khoá kệ, bin, khu vực khi thay đổi layout; đánh dấu `Maintenance` / `Blocked`; mở/đóng cửa. Các mã tạo nên mã vị trí không sửa được; tạo nhầm thì chuyển `Inactive` và tạo mới (BR-13). Mọi thay đổi được version hoá.

### 4.2. Gợi ý vị trí khi Putaway / Transfer

1. **Nhận yêu cầu** — module 05/11 gọi Slotting Engine kèm: SKU, số lượng, lô/hạn dùng, kho, vị trí nguồn.
2. **Lọc vị trí hợp lệ (hard constraints)** — chỉ giữ bin thoả mãn:
   - Bin **cho phép cất hàng** (`is_putaway_bin`), bin/kệ ở trạng thái `Active`.
   - **Lớp lưu trữ hiệu lực** của bin (lớp ghi đè của bin, nếu không có thì lớp mặc định của kệ) **trùng** lớp lưu trữ của SKU.
   - Còn đủ **sức chứa theo đơn vị** (`capacity_units`, nếu khai báo).
   - Tổng khối lượng trên tầng không vượt **tải trọng tối đa** của tầng; chiều cao hàng không vượt **chiều cao lọt** của tầng (nếu khai báo).
   - Kích thước hàng vừa kích thước bin.
3. **Chấm điểm vị trí còn lại (soft scoring)** theo tiêu chí (trọng số cấu hình):
   | Tiêu chí | Ý nghĩa |
   |----------|---------|
   | Gần khu đóng gói / vị trí xuất | SKU bán chạy (A) ưu tiên bin có **quãng đường đi thực tế** ngắn tới khu `PACKING`/`DISPATCH`, tầng thấp (golden zone) |
   | Gom cùng SKU | Ưu tiên bin đã chứa cùng SKU để giảm phân mảnh |
   | FEFO/FIFO | Chọn vị trí giúp lô hạn dùng gần được lấy trước |
   | Cân bằng tải | Tránh dồn quá nhiều vào một kệ / một khu gây tắc nghẽn lối đi |
   | Khối lượng/công thái học | Hàng nặng → tầng có cao độ thấp |
   | Mật độ lưu trữ | Ưu tiên lấp đầy bin đang dùng dở trước khi mở bin mới |
4. **Trả về danh sách xếp hạng** (top N) kèm điểm và lý do ("gần khu đóng gói, cùng SKU, còn 60% chỗ").
5. **NV kho chọn** — chấp nhận gợi ý #1, chọn phương án khác trong danh sách, hoặc **ghi đè** bằng vị trí tự nhập kèm **mã lý do**.
6. **Ghi nhận kết quả thực tế** — engine lưu (gợi ý vs thực tế, lý do lệch) để phân tích và tinh chỉnh quy tắc.

### 4.3. Tối ưu định kỳ (Re-slotting)

1. Manager chạy phân tích: SKU nào đổi hạng ABC, SKU bán chạy đang nằm xa, vị trí honeycombing.
2. Engine đề xuất danh sách di chuyển tối ưu (move list).
3. Manager duyệt → sinh các [Intra-warehouse Transfer](../11-intra-warehouse-transfer/README.md) tương ứng.

---

## 5. Trạng thái / Vòng đời

Module này **không có chứng từ vòng đời riêng**. Trạng thái của các thực thể liên quan:

| Thực thể | Trạng thái |
|----------|-----------|
| **Kho (Warehouse)** | `Active` · `Inactive` |
| **Kệ (Shelf) / Ô (Bin) / Khu vực (Area)** | `Active` (dùng bình thường) · `Blocked` (khoá kiểm kê/QC) · `Maintenance` (hỏng/bảo trì) · `Inactive` (bỏ khỏi layout). **`Full` (đầy) không lưu** — tính ra từ tồn so với sức chứa |
| **Cửa (Boundary loại DOOR)** | `Open` (đi qua được) · `Closed` (không đi qua được) |
| **Gợi ý (Suggestion)** | `Generated` → `Accepted` / `Overridden` / `Expired` (không dùng trong thời hạn) |
| **Đề xuất re-slotting** | `Draft` → `Approved` → `Executed` (đã sinh transfer) / `Rejected` |

Kệ ở trạng thái khác `Active` kéo theo mọi bin của kệ đó không được gợi ý cất/lấy hàng.

### Giá trị phân loại

| Trường | Giá trị | Ý nghĩa |
|--------|---------|---------|
| **Loại khu vực (`Area.type`)** | `RECEIVING` | Khu nhận hàng (WH/Input, inbound staging): hàng vừa nhận, chờ chuyển sang khu QC hoặc chờ cất |
| | `QUALITY_CONTROL` | Khu kiểm tra chất lượng (WH/Quality Control): hàng đã nhận, đang chờ QC kiểm — chặng giữa của luồng nhập kho 3 bước |
| | `QUARANTINE` | Khu cách ly hàng **sau khi QC**: hàng `Quarantine` chờ kết luận lại và hàng `Rejected` chờ trả NCC |
| | `PRODUCTION` | Khu xưởng in (xem [19](../19-production/README.md)): phôi đã cấp cho lệnh sản xuất, hàng đang in và thành phẩm chờ QC |
| | `PACKING` | Khu đóng gói, gồm chỗ gom hàng đã pick và thành phẩm in đạt QC chờ đóng gói |
| | `DISPATCH` | Khu chờ giao / bàn giao hãng vận chuyển (outbound dock) |
| | `OVERFLOW` | Khu tràn cho hàng quá khổ hoặc khi hết chỗ trên kệ |
| | `NON_STORAGE` | Khu không chứa hàng (văn phòng, khu sạc xe…) |
| **Lớp lưu trữ (storage class)** | `NORMAL` · `COLD` · `HAZMAT` · `FRAGILE` · `OVERSIZE` | Hàng thường · lạnh · nguy hiểm · dễ vỡ · quá khổ |
| **Loại ô (`Bin.type`)** | `STANDARD` · `PALLET` · `SMALL_PARTS` | Ô tiêu chuẩn · ô pallet · ô hàng nhỏ |
| **Đơn vị bản đồ (`Warehouse.map_unit`)** | `M` | Mét; toạ độ và kích thước trên bản đồ kho tính theo đơn vị này |
| **Kiểu đặt mã bin khi tự sinh** | `SEQUENTIAL` | Số liên tục theo hàng rồi đến cột: `01`, `02`, `03`… |
| | `ROW_LETTER_COLUMN` | Chữ cái là hàng, số là cột: `A01`, `A02`, `B01`… (tối đa 26 hàng) |

> ⚠️ Giả định: `Bin.type` chỉ dùng để mô tả & ưu tiên (soft scoring — ví dụ hàng xử lý theo pallet ưu tiên ô `PALLET`), không phải ràng buộc cứng. Xem [open-questions](../../open-questions/README.md).

### Dữ liệu tối thiểu cần lưu

| Thực thể | Trường chính |
|----------|--------------|
| `Warehouse` | `id`, `name`, `prefix`, `address`, `return_address?`, `map_unit`, `map_width`, `map_height`, `status` |
| `Zone` | `id`, `warehouse_id`, `name`, `color?` |
| `Shelf` | `id`, `warehouse_id`, `zone_id?`, `code`, `name`, `description?`, `x`, `y`, `width`, `length`, `rotation`, `is_obstacle`, `pick_north`, `pick_east`, `pick_south`, `pick_west`, `default_storage_class`, `status` |
| `ShelfLevel` | `id`, `shelf_id`, `level_index`, `elevation?`, `usable_height?`, `max_weight?` |
| `Bin` | `id`, `level_id`, `code`, `location_code`, `description?`, `x`, `y` (tương đối trong kệ), `width`, `length`, `rotation`, `capacity_units?`, `is_pickable`, `is_putaway_bin`, `type`, `storage_class_override?`, `status` |
| `Area` | `id`, `warehouse_id`, `zone_id?`, `code`, `location_code`, `type`, `name`, `x`, `y`, `width`, `length`, `rotation`, `is_obstacle`, `status` |
| `Boundary` | `id`, `warehouse_id`, `type` (`WALL` / `DOOR`), `start_x`, `start_y`, `end_x`, `end_y`, `is_passable`, `operational_status?` |

Trường có dấu `?` là tùy chọn; nếu để trống thì hệ thống **không kiểm tra** ràng buộc tương ứng (ví dụ không có `max_weight` → không kiểm tra tải trọng tầng).

`code` là **mã cục bộ** do người dùng nhập hoặc hệ thống tự sinh, duy nhất trong phạm vi cha trực tiếp. `location_code` là **mã vị trí đầy đủ** do hệ thống ghép khi tạo (BR-10), chỉ có ở `Bin` và `Area` vì đây là hai loại vị trí chứa tồn; đây là chuỗi được in trên nhãn, quét khi cất/lấy hàng và ghi trong tồn kho. Số hàng × số cột khi tự sinh bin **không lưu** — chỉ là tham số của thao tác sinh.

---

## 6. Ràng buộc nghiệp vụ (Business Rules) & Edge case

### Business rules

- **BR-01:** Hard constraints (trạng thái vị trí, cho phép cất, lớp lưu trữ, sức chứa, tải trọng & chiều cao lọt của tầng, kích thước bin) **không bao giờ bị vi phạm** kể cả khi ghi đè — nếu NV kho nhập vị trí vi phạm hard constraint, hệ thống chặn.
- **BR-02:** Soft scoring có thể bị ghi đè bởi NV kho **kèm mã lý do**.
- **BR-03:** Gợi ý phải phản ánh tồn kho & sức chứa **tại thời điểm gọi**; gợi ý cũ quá thời hạn (ví dụ > X phút) phải tính lại.
- **BR-04:** Mọi thay đổi cấu trúc/thuộc tính kho được version hoá; không xoá kệ, tầng, bin hay khu vực còn tồn > 0.
- **BR-05:** Bản đồ 3D là công cụ trực quan, **không phải nguồn sự thật của tồn** — nguồn sự thật là dữ liệu tồn kho theo vị trí.
- **BR-06:** Toàn bộ kệ, khu vực và ranh giới phải nằm trong khung bản đồ của kho; toạ độ và kích thước tính theo đơn vị bản đồ của kho.
- **BR-07:** Kệ và khu vực **không được chồng lấn** nhau trên mặt bằng; bin phải nằm trong phạm vi kệ chứa nó và không chồng lấn bin khác cùng tầng.
- **BR-08:** Kệ có bin cho phép lấy hàng (`is_pickable`) phải có **ít nhất một mặt lấy hàng**, và mặt đó không bị tường/vật cản che kín.
- **BR-09:** Khoảng cách dùng cho slotting và lộ trình picking là **quãng đường đi thực tế**: không đi xuyên tường, kệ hay khu vực vật cản; chỉ đi qua cửa `is_passable` đang `Open`; tiếp cận kệ từ mặt lấy hàng.
- **BR-10:** Mã vị trí đầy đủ (`location_code`) do hệ thống ghép khi tạo và **duy nhất toàn hệ thống**:
  - Bin = **prefix kho + mã kệ + thứ tự tầng + mã bin**, ví dụ `HN-A01-2-03`.
  - Khu vực = **prefix kho + mã khu vực**, ví dụ `HN-RCV01`.

  Prefix kho duy nhất toàn hệ thống; mã kệ và mã khu vực duy nhất trong kho; thứ tự tầng duy nhất trong kệ; mã bin duy nhất trong tầng. Mỗi thành phần mã chỉ gồm chữ in hoa và chữ số (`A–Z`, `0–9`), **không chứa dấu `-`** vì dấu `-` dùng để nối các thành phần — nhờ đó hai vị trí khác nhau không thể ghép ra cùng một mã. Chữ thường được tự chuyển thành chữ hoa.
- **BR-11:** Tồn chỉ được gắn vào **bin** hoặc **khu vực chứa hàng** (`RECEIVING`, `QUALITY_CONTROL`, `QUARANTINE`, `PRODUCTION`, `PACKING`, `DISPATCH`, `OVERFLOW`); khu `NON_STORAGE` không chứa tồn. Khu vực không có sức chứa khai báo và không phải đích gợi ý putaway mặc định; `OVERFLOW` chỉ dùng khi Manager chỉ định.
- **BR-12:** Picking chỉ lấy từ bin `is_pickable`; bin không cho phép lấy hàng là **bin dự trữ**, hàng phải được bổ sung (replenishment) sang bin picking trước.
- **BR-13:** Các thành phần tạo nên mã vị trí — **prefix kho, mã kệ, thứ tự tầng, mã bin, mã khu vực** — **không sửa được** sau khi tạo, vì mã vị trí đã được in trên nhãn, ghi trong tồn kho và lịch sử di chuyển. Tạo nhầm thì chuyển vị trí sang `Inactive` rồi tạo vị trí mới với mã đúng; mã của vị trí `Inactive` vẫn được giữ và không cấp lại cho vị trí khác. Các thuộc tính còn lại (tên, mô tả, toạ độ, kích thước, góc xoay, zone, lớp lưu trữ, sức chứa, mặt lấy hàng, trạng thái) sửa được.
- **BR-14:** Tự sinh bin (autogenerate):
  - Chỉ áp dụng cho tầng **chưa có bin nào**, kể cả bin `Inactive`. Tầng đã có bin thì từ chối; muốn bổ sung bin thì nhập từng bin.
  - Mọi tầng được chọn phải thuộc kệ đã chọn; có một tầng không hợp lệ thì không sinh bin cho tầng nào.
  - Số hàng và số cột ≥ 1. Mặt kệ được chia đều: chiều rộng bin = chiều rộng kệ ÷ số cột; chiều dài bin = chiều dài kệ ÷ số hàng. Kích thước **làm tròn xuống** để bin không vượt ra ngoài kệ (BR-07); góc xoay của bin = 0 (tương đối theo kệ).
  - Mã bin đặt theo hàng rồi đến cột, bắt đầu từ góc gốc toạ độ của kệ, theo kiểu đặt mã đã chọn (`SEQUENTIAL` hoặc `ROW_LETTER_COLUMN`, xem §5). Phần số có ít nhất 2 chữ số và được thêm `0` ở đầu cho cùng độ dài trong một tầng.

### Edge case

| Tình huống | Hướng xử lý |
|------------|-------------|
| Không có vị trí nào thoả hard constraints | Trả kết quả rỗng + lý do; đề xuất khu tràn (`OVERFLOW`) hoặc Inter-warehouse Transfer |
| Hai putaway song song cùng nhắm một bin | Khoá mềm (soft-reserve) bin khi gợi ý được `Accepted`; task sau tính lại |
| SKU mới chưa có dữ liệu velocity | Dùng mặc định theo nhóm hàng / phân hạng tạm; cập nhật sau khi có lịch sử |
| Kích thước SKU khai báo sai → gợi ý vị trí không vừa | NV kho ghi đè + lý do "kích thước sai"; hệ thống cảnh báo cập nhật lại module 01 |
| Layout kho thay đổi giữa ca | Khoá kệ/bin/khu vực ảnh hưởng thành `Maintenance`; task đang chờ được gợi ý lại |
| Cửa chuyển `Closed` hoặc thêm vật cản làm một khu không còn đường tới | Tính lại khoảng cách & lộ trình; cảnh báo Manager các kệ/khu không tiếp cận được |
| Đổi lớp lưu trữ của kệ/bin khi đang chứa hàng khác lớp | Cho lưu nhưng cảnh báo; hàng hiện có không bị chặn lấy ra, chỉ chặn cất thêm hàng khác lớp; đề xuất move task |
| Di chuyển / xoay kệ đang có tồn | Chỉ đổi toạ độ bản đồ, không đổi tồn; kiểm tra lại BR-06 → BR-08 |
| Nhập nhầm mã kệ / bin / khu vực | Không sửa mã; chuyển vị trí nhầm sang `Inactive` và tạo vị trí mới với mã đúng (BR-13) |
| Tự sinh bin cho tầng đã có bin | Từ chối; bổ sung bin bằng cách nhập từng bin (BR-14) |
| Kích thước kệ không chia hết cho số hàng / số cột | Làm tròn xuống kích thước bin; phần dư nhỏ ở cuối kệ để trống (BR-14) |
| Bản đồ 3D lệch thực địa | Công cụ đối soát định kỳ (cycle count theo vị trí); ưu tiên số liệu kiểm kê |

---

## 7. Liên kết với các module khác

| Module | Quan hệ |
|--------|---------|
| [05 · Putaway](../05-putaway/README.md) | Nhận gợi ý vị trí khi cất hàng; khu `QUALITY_CONTROL` / `RECEIVING` là vị trí nguồn; gửi lại kết quả thực tế |
| [11 · Intra-warehouse Transfer](../11-intra-warehouse-transfer/README.md) | Nhận gợi ý vị trí đích khi di chuyển nội bộ / re-slotting; replenishment từ bin dự trữ sang bin picking |
| [07 · Picking](../07-picking/README.md) | Cung cấp quãng đường đi thực tế cho lộ trình picking; chỉ lấy từ bin `is_pickable`; zone dùng cho zone picking; velocity từ picking là đầu vào re-slotting |
| [03 · Receipt](../03-receipt/README.md) / [08 · Packing](../08-packing/README.md) | Hàng nhận nằm ở khu `RECEIVING`, hàng chờ QC ở khu `QUALITY_CONTROL`, hàng cách ly/không đạt ở khu `QUARANTINE`; hàng chờ đóng gói ở khu `PACKING`, chờ giao ở khu `DISPATCH` |
| [19 · Production](../19-production/README.md) | Phôi cấp cho lệnh sản xuất và hàng đang in nằm ở khu `PRODUCTION`; thành phẩm đạt QC chuyển sang khu `PACKING` |
| [09 · Shipping](../09-shipping/README.md) | Dùng địa chỉ kho & địa chỉ hoàn hàng khi tạo vận đơn |
| [01 · Product Creation](../01-product-creation/README.md) | Cung cấp lớp lưu trữ yêu cầu, kích thước/khối lượng của SKU (Inventory Item) |
| [10 · Inter-warehouse Transfer](../10-inter-warehouse-transfer/README.md) | Khi kho hết chỗ, đề xuất chuyển bớt sang kho khác |
| [00 · System Overview](../../00-system-overview/README.md) | Là lớp hỗ trợ xuyên suốt luồng lưu kho |
