# 05 — Chuyển hàng vào kho (Putaway)

## 1. Mục đích (Purpose)

Di chuyển các **thùng thành phẩm (LPN)** từ **khu nhận hàng** (Area `RECEIVING`) sau khi [nhập kho thành phẩm](../03-receipt/README.md) tới **bin lưu trữ chính thức**, và **cập nhật tồn kho theo vị trí** để hàng trở nên **khả dụng cho lấy hàng / bán hàng**.

Putaway quyết định *hàng nằm ở đâu* — ảnh hưởng trực tiếp đến hiệu suất [Picking](../07-picking/README.md), mật độ lưu trữ và an toàn kho. Hệ thống **gợi ý bin** theo ràng buộc cứng và thứ tự ưu tiên (§4.2). Quyết định cuối cùng là **bin nhân viên quét thực tế**.

Putaway là **chặng cuối** của luồng nhập kho **2 bước**: WH/Input (`RECEIVING`) → WH/Stock (bin). Không có chặng kiểm tra chất lượng tại kho *(Đã chốt 2026-11-04 — [03](../03-receipt/README.md))*.

> Nguyên tắc dữ liệu *(Đã chốt 2026-11-04)*: mọi thành phẩm nằm trong **thùng LPN** do bộ phận sản xuất đóng sẵn; thao tác vật lý luôn theo **nguyên thùng**, không mở thùng. Mọi thùng đúng **số cái chuẩn / thùng** của SKU *(Đã chốt 2026-11-05 — [03](../03-receipt/README.md) BR-13)*. Tồn nằm trong từng thùng; nội bộ có thể ghi theo **đơn vị cơ sở** để truy vết, còn màn hình và tồn khả dụng hiển thị theo **số thùng** kèm số cái ([01](../01-product-creation/README.md) BR-09).

> ⚠️ Giả định: các phần sau thuộc **phạm vi giai đoạn sau** (xem [open-questions C18](../../open-questions/README.md)):
> - **phân công** nhiệm vụ cất cho từng người;
> - **khoá mềm** sức chứa bin khi nhận nhiệm vụ;
> - **bắt buộc mã lý do** khi cất khác bin gợi ý;
> - **cất nhiều chặng** qua khu trung gian;
> - **tiêu chí mềm nâng cao** của slotting (velocity / ABC, quãng đường, FEFO, cân bằng tải — [06](../06-warehouse-map-slotting/README.md), [C1](../../open-questions/README.md)).
>
> Hiện mọi nhân viên kho của kho đó đều thấy và cất được mọi nhiệm vụ.

---

## 2. Actor liên quan

| Actor | Vai trò |
|-------|---------|
| **Nhân viên kho (Warehouse Staff)** | Xem nhiệm vụ cất của kho, xem bin gợi ý, quét thùng, quét bin đích, xác nhận cất |
| **Quản lý kho (Warehouse Manager)** | Như nhân viên kho; xử lý khi không còn bin hợp lệ, lệch tồn |
| **Admin / System Config** | Khai báo bản đồ kho: kệ, tầng, bin, cờ **cho phép cất** (`is_putaway_target`), **lớp lưu trữ**, **sức chứa**, **tải trọng tối đa** — chi tiết ở [06](../06-warehouse-map-slotting/README.md) |
| **Hệ thống** | Mở nhiệm vụ cất khi thùng sẵn sàng; gợi ý tối đa 10 bin; chặn bin vi phạm ràng buộc cứng; ghi sổ biến động; đóng phiếu nhập khi cất xong |
| **Quản lý kho — kiểm soát tồn** | Lập kiểm kê, duyệt điều chỉnh tồn khi có sai lệch phát hiện trong / sau putaway — xem [20 · Inventory Control](../20-inventory-control/README.md) |

- Mọi thao tác chỉ trong phạm vi **kho được gán**, và theo vị trí hàng đang đứng.
- Kiểm kê và điều chỉnh tồn không có role riêng: do Quản lý kho / Nhân viên kế hoạch tồn kho thực hiện ở [20](../20-inventory-control/README.md).

---

## 3. Input / Output

### Input

| Trường | Mô tả | Bắt buộc |
|--------|-------|----------|
| **Nhiệm vụ cất (putaway task)** | Mở tự động khi phiếu nhập được xác nhận ([03](../03-receipt/README.md)): **mỗi thùng LPN một nhiệm vụ** | Có |
| SKU, lô, số lượng | Từ thùng trên phiếu nhập; một nhiệm vụ = một thùng (số cái = số cái chuẩn / thùng của SKU) | Có |
| Vị trí nguồn | Khu nhận hàng (Area `RECEIVING`) | Có |
| **Bin gợi ý** | Tối đa 10 bin, tính mỗi lần xem nhiệm vụ (không lưu) | Không |
| **Mã LPN** | NV kho quét đúng thùng của nhiệm vụ | Có |
| **Bin đích thực tế** | NV kho quét khi đặt thùng; bất kỳ bin hợp lệ nào, không bắt buộc là bin gợi ý | Có |

### Output

| Kết quả | Dùng bởi |
|---------|----------|
| **Cập nhật tồn kho theo vị trí** (thùng nằm ở bin); tồn `Inbound` → **`Available`** | 06 Map, 07 Picking, báo cáo tồn |
| **Tồn khả dụng để bán** tăng thêm một thùng mỗi nhiệm vụ | 13 Catalog / 14 Order (ATP) |
| **Dòng sổ biến động** `MOVE` cho mỗi thùng được cất (hoặc `STATUS_CHANGE` khi thùng đã nằm sẵn ở bin) | [20](../20-inventory-control/README.md), truy vết |
| **Nhiệm vụ `Done`** → phiếu nhập `Closed` khi mọi thùng đã cất | 03 Receipt |
| **Tồn `Available` của SKU** sau khi cất | [17](../../ecommerce/17-order-management/README.md) (đưa đơn `Awaiting Stock` sang `Ready to Fulfill`) |

---

## 4. Luồng nghiệp vụ từng bước (Step-by-step)

### 4.1. Mở nhiệm vụ

1. Khi phiếu nhập được xác nhận ([03](../03-receipt/README.md)), hệ thống mở **một nhiệm vụ cất cho mỗi thùng LPN `In Stock`** của phiếu (cất nguyên thùng). Thùng đã có nhiệm vụ thì không mở thêm (mở lại không sinh trùng). Nhiệm vụ mới ở `Pending`.
2. NV kho xem danh sách nhiệm vụ đang mở theo kho, hoặc theo phiếu nhập.

### 4.2. Gợi ý bin

3. Khi mở một nhiệm vụ, hệ thống gợi ý **tối đa 10 bin**. Bin phải thoả **ràng buộc cứng**:
   - là **bin** (không phải khu vực) của **cùng kho**, cho phép cất (`is_putaway_target`);
   - cùng **lớp lưu trữ** với SKU;
   - bin, kệ và kho đều đang **`Active`**;
   - **còn chỗ**: số đơn vị trống ≥ số trong thùng. Bin không khai báo sức chứa coi như không giới hạn;
   - **không vượt tải trọng**: chỉ kiểm khi biết cả tải trọng tối đa của bin và khối lượng đơn vị của SKU.

   Độ lấp đầy tính trên **mọi SKU, mọi trạng thái** đang nằm trong bin.
4. Bin hợp lệ được **xếp hạng** theo thứ tự:
   1. **gom hàng trước khi trải rộng** — bin đã có **cùng lô**, rồi bin có **cùng SKU**, rồi **bin trống**, rồi bin **còn chỗ**;
   2. bin **còn nhiều chỗ hơn** đứng trước (bin không giới hạn đứng đầu);
   3. theo **mã vị trí**.

### 4.3. Cất hàng

5. NV kho lấy thùng ở khu nhận hàng, **quét đúng mã LPN** của nhiệm vụ, đi tới bin, **quét bin đích**. Cả thùng được chuyển; không nhập số lượng, không chia thùng.
6. Hệ thống kiểm lại ràng buộc cứng (§4.2) **tại bin được quét**, kể cả khi bin đó không nằm trong gợi ý. Vi phạm → từ chối:
   - không tìm thấy vị trí;
   - không phải bin;
   - khác kho;
   - bin không dùng được hoặc không cho phép cất;
   - sai lớp lưu trữ;
   - vượt sức chứa.

   Hai người không cùng lúc cất vào một bin: hệ thống xếp hàng các lần cất vào cùng bin.
7. **Ghi tồn** — thùng chuyển từ khu nhận hàng sang bin, tồn **`Inbound` → `Available`**, ghi dòng sổ `MOVE`. Thùng **đã được người chuyển tay** vào đúng bin đó ([11](../11-intra-warehouse-transfer/README.md)) thì được cất tại chỗ (dòng sổ `STATUS_CHANGE`), và không bị tính hai lần vào sức chứa.
8. **Hoàn tất** — nhiệm vụ → **`Done`**.
9. **Đóng phiếu nhập** — khi mọi thùng của phiếu đã được cất, phiếu nhập tự `Closed` ([03](../03-receipt/README.md)).

### Các luồng biến thể

| Luồng | Khi dùng | Cách xử lý |
|-------|----------|------------|
| **Cất khác bin gợi ý** | Gợi ý không đúng thực tế | Quét bin khác; được nếu bin thoả ràng buộc cứng |
| **Không có bin hợp lệ** | Kho đầy, sai cấu hình, thùng quá khổ | Danh sách gợi ý rỗng; Quản lý kho khai báo / mở bin cho phép cất, hoặc chuyển kho ([10](../10-inter-warehouse-transfer/README.md)) |

---

## 5. Trạng thái / Vòng đời (Lifecycle)

### Trạng thái nhiệm vụ cất (Putaway Task status)

| Trạng thái | Ý nghĩa | Chuyển tiếp cho phép | Ai thực hiện |
|------------|---------|----------------------|--------------|
| `Pending` | Đã mở, thùng chưa cất | → `Done` (cất cả thùng) | Hệ thống mở; NV kho cất |
| `Done` | Thùng đã cất vào bin; tồn đã cập nhật | *(cuối)* | NV kho |
| `Cancelled` | Dành sẵn; hiện không có thao tác huỷ | *(cuối)* | — |

```mermaid
stateDiagram-v2
    [*] --> Pending: Thùng sẵn sàng cất
    Pending --> Done: Quét LPN + quét bin, cất cả thùng
    Done --> [*]
```

### Trạng thái hàng trong quá trình

| Trạng thái tồn | Ý nghĩa |
|----------------|---------|
| `Inbound` | Đã nhận, thùng ở khu nhận hàng, **chưa khả dụng** để bán |
| `Available` | Thùng đã ở bin lưu trữ, **khả dụng** để lấy hàng / bán |

Hàng không có trạng thái "đang di chuyển": mỗi lần cất chuyển thẳng thùng từ khu nhận hàng sang bin. Trạng thái tồn và vòng đời thùng đầy đủ xem [20 §5](../20-inventory-control/README.md).

### Dữ liệu tối thiểu cần lưu

| Thực thể | Trường chính |
|----------|--------------|
| `PutawayTask` | `id`, `receiptLineId`, `stockPackageId` (thùng), `sku`, `lot`, `quantity` (số cái trong thùng = số cái chuẩn / thùng), `status` |
| `StockMovement` | `type` (`MOVE` / `STATUS_CHANGE`), `reference = PUTAWAY_TASK`, `fromLocation`, `toLocation`, `sku`, `lot`, `quantity`, `stockPackageId`, `performedBy`, `performedAt` |

---

## 6. Ràng buộc nghiệp vụ (Business Rules) & Edge case

### Business rules

- **BR-01:** Chỉ thùng của **phiếu nhập đã xác nhận** (tồn `Inbound` ở khu `RECEIVING`) mới có nhiệm vụ cất. Hàng ở khu cách ly (`QUARANTINE`) không cất vào vùng lưu trữ qua nhiệm vụ cất.
- **BR-02:** Tồn kho **luôn gắn với một vị trí** — một **bin** hoặc một **khu vực chứa hàng**. Không tồn tại tồn "không vị trí" ngoài trạng thái quá cảnh của lệnh chuyển kho.
- **BR-03:** Không đặt hàng vượt **sức chứa** khai báo của bin (`capacity_units`); không vượt **tải trọng tối đa** khi biết cả tải trọng bin và khối lượng SKU. Trường nào không khai báo thì không kiểm tra.
- **BR-04:** Chỉ cất vào **bin** cho phép cất (`is_putaway_target`) của cùng kho, có **lớp lưu trữ trùng lớp lưu trữ của SKU**, với bin, kệ và kho đều `Active`. Một bin có thể chứa nhiều thùng, nhiều SKU / lô nếu còn sức chứa.
- **BR-05:** Gợi ý ưu tiên **gom cùng lô, cùng SKU** trước khi trải sang bin trống (BR-SLT-004). FEFO được bảo đảm ở bước **phân bổ khi lấy hàng** ([20](../20-inventory-control/README.md) BR-01, [07](../07-picking/README.md)), không ở bước gợi ý bin.
- **BR-06:** Nhiệm vụ `Done` không sửa; điều chỉnh vị trí sau đó dùng [Intra-warehouse Transfer](../11-intra-warehouse-transfer/README.md) (di chuyển nguyên thùng).
- **BR-07:** Cất vào bin khác gợi ý được phép khi bin thoả ràng buộc cứng; không bắt buộc mã lý do (⚠️ giả định — C18).
- **BR-08:** **Cất nguyên thùng** *(Đã chốt 2026-11-04)*: mỗi nhiệm vụ là **một thùng LPN**, phải quét đúng mã thùng, cất cả thùng vào **một bin** trong một lần; không mở thùng, không chia thùng ra nhiều bin.
- **BR-09:** Ràng buộc cứng được kiểm **tại lúc xác nhận**, trên bin được quét. Các lần cất vào cùng một bin được xếp hàng tuần tự, để sức chứa không bị vượt do hai người cất cùng lúc.
- **BR-10:** Mỗi lần cất ghi **dòng sổ biến động** có chứng từ nguồn là nhiệm vụ cất.
- **BR-11:** Nếu phát hiện một thùng lệch số cái chuẩn tại khu nhận hàng hoặc tại bin đích, không tự sửa nhiệm vụ đã chốt; thùng lệch không được phân bổ, xử lý bằng khu cách ly / kiểm kê / điều chỉnh tồn có phê duyệt (⚠️ giả định — [03](../03-receipt/README.md)) — xem [20 §4.2–4.3](../20-inventory-control/README.md).
- **BR-12:** Available-to-promise chỉ tăng khi thùng đã cất vào bin; hàng ở khu nhận hàng, khu cách ly không được bán.

### Đối chiếu chuẩn tham chiếu

| Chủ đề | Cách thiết kế trong hệ thống | Ghi chú so với Odoo/MISA |
|--------|------------------------------|---------------------------|
| Nhập kho nhiều bước | 2 bước: khu nhận hàng → bin; không có chặng kiểm tra chất lượng tại kho | Khớp mô hình nhận hàng 2 bước (Input → Stock) của Odoo; MISA cũng hỗ trợ quy trình nhập/xuất nhiều bước |
| Gợi ý vị trí | Lọc ràng buộc cứng, rồi xếp hạng: cùng lô → cùng SKU → bin trống → còn chỗ; nhiều chỗ trống trước | Tương tự tinh thần Odoo Putaway Rules/Storage Categories; thứ tự xếp hạng là thiết kế riêng |
| Capacity | Số đơn vị tối đa của bin, tải trọng tối đa (khi biết khối lượng SKU) | Odoo có storage category/capacity |
| Cất khác gợi ý | Cho phép nếu không vi phạm ràng buộc cứng | Phù hợp thực tế vận hành barcode/WMS: người làm kho xác nhận theo thực địa |
| Theo thùng | Mỗi thùng LPN là một nhiệm vụ, cất nguyên thùng; không có hàng ngoài thùng | Tương tự package / LPN putaway của các WMS phổ biến |
| FIFO/FEFO | Bảo đảm ở bước phân bổ khi lấy hàng | Khớp chiến lược xuất kho phổ biến trong Odoo/MISA |

### Edge case

| Tình huống | Hướng xử lý |
|------------|-------------|
| Bin gợi ý đã đầy (do lần cất khác) | Lúc xác nhận hệ thống kiểm lại và từ chối; xem lại gợi ý (tính lại theo tồn hiện tại) và chọn bin khác |
| Không còn chỗ cho SKU (kho đầy) | Gợi ý rỗng; Quản lý kho mở thêm bin cho phép cất, hoặc xem xét [chuyển kho](../10-inter-warehouse-transfer/README.md) |
| Thùng không vừa chỗ trống của bin nào | Gợi ý rỗng; không chia thùng. Quản lý kho mở bin lớn hơn hoặc dọn chỗ bằng [di chuyển nguyên thùng](../11-intra-warehouse-transfer/README.md) |
| Thùng cồng kềnh không vừa bin tiêu chuẩn | Cất vào bin có lớp lưu trữ phù hợp (ví dụ `OVERSIZE`); nếu không có, Admin khai báo thêm ([06](../06-warehouse-map-slotting/README.md)) |
| Quét sai thùng | Từ chối (thùng không khớp nhiệm vụ) |
| Thùng đã được chuyển tay vào bin trước khi xác nhận | Xác nhận đúng bin đó: hệ thống cất tại chỗ, không tính trùng sức chứa |
| Nhãn LPN bị rách / không quét được | In lại nhãn theo mã LPN ([03](../03-receipt/README.md)) rồi quét |
| Mất mát / hư hỏng phát hiện khi cất | Không sửa nhiệm vụ; điều chỉnh tồn (lý do `DAMAGED` / `LOST`, có duyệt — [20 §4.3](../20-inventory-control/README.md)) hoặc chuyển thùng sang khu cách ly `QUARANTINE`; đổi trả với bộ phận sản xuất ngoài hệ thống (⚠️ giả định — [03](../03-receipt/README.md)) |
| Đặt nhầm vị trí và đã xác nhận | Sửa bằng di chuyển nguyên thùng ([11](../11-intra-warehouse-transfer/README.md)), không sửa trực tiếp nhiệm vụ |

---

## 7. Liên kết với các module khác

| Module | Quan hệ |
|--------|---------|
| [03 · Receipt](../03-receipt/README.md) | Xác nhận phiếu nhập thành phẩm mở nhiệm vụ cất (mỗi thùng một nhiệm vụ); cất xong mọi thùng → phiếu nhập `Closed` |
| [06 · Warehouse Map & Slotting](../06-warehouse-map-slotting/README.md) | Bin, cờ cho phép cất, lớp lưu trữ, sức chứa, tải trọng; khu `RECEIVING` là vị trí nguồn |
| [01 · Product Creation](../01-product-creation/README.md) | Lớp lưu trữ, khối lượng đơn vị và quy cách thùng của SKU |
| [07 · Picking](../07-picking/README.md) | Vị trí cất hợp lý và gom lô giúp lấy nguyên thùng nhanh; FEFO ở bước phân bổ |
| [11 · Intra-warehouse Transfer](../11-intra-warehouse-transfer/README.md) | Di chuyển nguyên thùng bằng tay để sửa vị trí sau khi đã cất |
| [13 · Catalog](../../ecommerce/13-catalog/README.md) / [14 · Order](../../ecommerce/14-cart-and-order/README.md) | Cất xong làm tăng tồn khả dụng (ATP) |
| [17 · Order Management](../../ecommerce/17-order-management/README.md) | Hàng của Yêu cầu sản xuất cất xong là căn cứ đưa đơn `Awaiting Stock` sang `Ready to Fulfill` |
| [20 · Inventory Control](../20-inventory-control/README.md) | Thùng LPN và vòng đời thùng; dòng sổ biến động; lệch số khi cất xử lý bằng kiểm kê / điều chỉnh tồn có duyệt; `Inbound` → `Available` |
