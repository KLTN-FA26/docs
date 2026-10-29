# 11 — Chuyển hàng trong nội bộ một kho (Intra-warehouse Transfer)

## 1. Mục đích (Purpose)

Di chuyển tồn kho **giữa hai vị trí trong cùng một kho** (bin hoặc khu vực chứa hàng → bin hoặc khu vực chứa hàng) mà **không thay đổi tổng tồn của kho**. Dùng để:

- **gom hàng** của cùng SKU / lô về ít vị trí hơn, giải phóng bin;
- **sửa vị trí sai** sau khi cất hàng;
- **dọn vị trí** sắp ngưng hoặc bảo trì;
- đưa hàng từ bin dự trữ ra bin cho phép lấy hàng, khi người vận hành thấy cần.

Hiện tại đây là **một thao tác di chuyển thủ công**, ghi ngay khi xác nhận; không có nhiệm vụ di chuyển (move task) qua nhiều bước.

> ⚠️ Giả định: **bổ sung vùng picking tự động** (theo ngưỡng min / max của bin), **tối ưu lại vị trí** (re-slotting theo đề xuất, có duyệt), **nhiệm vụ di chuyển** có phân công và vòng đời, và **di chuyển nguyên thùng LPN** bằng tay thuộc phạm vi giai đoạn sau. Xem [open-questions C2, C24](../../open-questions/README.md).

Các luồng khác cũng đổi vị trí của hàng nhưng thuộc module riêng: cất hàng ([05](../05-putaway/README.md)), chuyển hàng nhận sang khu QC và kết luận QC ([03](../03-receipt/README.md)), xuất phôi cho xưởng ([19](../19-production/README.md)), cất lại hàng của đơn huỷ ([07](../07-picking/README.md)).

---

## 2. Actor liên quan

| Actor | Vai trò |
|-------|---------|
| **Nhân viên kho (Warehouse Staff)** | Di chuyển hàng giữa hai vị trí trong kho được gán |
| **Quản lý kho (Warehouse Manager)** | Như nhân viên kho; xử lý chênh lệch phát hiện khi di chuyển bằng kiểm kê / điều chỉnh tồn ([20](../20-inventory-control/README.md)) |
| **Inventory Planner, System Admin** | Có quyền di chuyển theo cấu hình phân quyền mặc định |

- Cả **vị trí nguồn và vị trí đích** phải thuộc kho người thao tác được gán (với role theo kho).

---

## 3. Input / Output

### Input

| Trường | Mô tả | Bắt buộc |
|--------|-------|----------|
| SKU | | Có |
| Lô | Khi SKU theo dõi lô | Theo SKU |
| Vị trí nguồn & vị trí đích | Mã vị trí (bin hoặc khu vực chứa hàng); khác nhau, **cùng kho** | Có |
| Số lượng | Số nguyên dương, không quá **phần tự do** (chưa bị giữ) của hàng lẻ ở vị trí nguồn | Có |
| Mã yêu cầu | Do màn hình sinh; gửi lại cùng mã không di chuyển lần hai | Có |

### Output

| Kết quả | Dùng bởi |
|---------|----------|
| **Tồn theo vị trí**: nguồn giảm, đích tăng; tổng tồn kho **không đổi**; trạng thái tồn **giữ nguyên** | 06 Map, 07 Picking, báo cáo tồn |
| **Dòng sổ biến động `MOVE`** (nguồn, đích, SKU, lô, số lượng, người, thời điểm) | [20 §4.7](../20-inventory-control/README.md) |

---

## 4. Luồng nghiệp vụ từng bước (Step-by-step)

### 4.1. Di chuyển thủ công

1. NV kho / Quản lý kho chọn **SKU, lô, vị trí nguồn, vị trí đích, số lượng**.
2. Hệ thống kiểm:
   - hai vị trí **khác nhau** và **cùng kho**; khác kho → dùng [lệnh chuyển kho](../10-inter-warehouse-transfer/README.md);
   - cả hai vị trí thuộc kho người thao tác được gán;
   - vị trí đích **dùng được** (trạng thái hiệu lực `Active` — [06](../06-warehouse-map-slotting/README.md)). Vị trí nguồn **không cần**: dọn hàng khỏi bin bị khoá / đang bảo trì chính là cách làm trống nó;
   - không vị trí nào đang **bị một phiếu kiểm kê giữ** cho SKU / lô đó ([20](../20-inventory-control/README.md) BR-10);
   - đủ **phần tự do** của hàng lẻ ở vị trí nguồn: hàng đang giữ cho đơn, cho sản xuất, cho lệnh chuyển kho, và hàng trong thùng LPN không di chuyển được bằng thao tác này;
   - nếu vị trí đích đã có cùng SKU, lô, ngày nhận thì phải **cùng trạng thái tồn** (không trộn `Available` với `Quarantine`…).
3. Hợp lệ → hàng chuyển **ngay**, giữ nguyên lô, hạn dùng, ngày nhận và trạng thái; ghi một dòng sổ `MOVE`.

> ⚠️ Giả định: thao tác này **không kiểm** lớp lưu trữ, sức chứa, tải trọng và cờ cho phép cất của vị trí đích (chỉ cất hàng ở [05](../05-putaway/README.md) kiểm). Người di chuyển tự chịu trách nhiệm chọn vị trí phù hợp. Xem [open-questions C24](../../open-questions/README.md).

### 4.2. Chênh lệch phát hiện khi di chuyển

4. Số thực tế ở vị trí nguồn ít hơn sổ sách → không di chuyển phần không có; báo Quản lý kho lập **phiếu kiểm kê** cho vị trí đó, chênh lệch đi qua **điều chỉnh tồn có duyệt** ([20 §4.2–4.3](../20-inventory-control/README.md)).

---

## 5. Trạng thái / Vòng đời (Lifecycle)

Di chuyển thủ công **không có vòng đời**: một lần xác nhận là một dòng sổ `MOVE`, không có trạng thái trung gian. Trạng thái của hàng (`Available`, `Quarantine`, `Blocked`…) đi theo hàng sang vị trí mới ([20 §5.1](../20-inventory-control/README.md)).

---

## 6. Ràng buộc nghiệp vụ (Business Rules) & Edge case

### Business rules

- **BR-01:** Vị trí nguồn và đích phải **khác nhau** và **cùng một kho**. Khác kho → [Inter-warehouse Transfer](../10-inter-warehouse-transfer/README.md).
- **BR-02:** Tổng tồn của kho **không đổi**; chỉ phân bố theo vị trí thay đổi. Trạng thái, lô, hạn dùng, ngày nhận giữ nguyên.
- **BR-03:** Vị trí đích phải **dùng được** (trạng thái hiệu lực `Active`). Vị trí nguồn thì không cần.
- **BR-04:** Chỉ di chuyển **phần tự do** của hàng lẻ. Hàng đang giữ / phân bổ (tránh picker tới vị trí trống) và hàng trong thùng LPN không di chuyển bằng thao tác này.
- **BR-05:** Không di chuyển ra / vào một vị trí đang bị **phiếu kiểm kê** giữ cho SKU / lô đó — kiểm kê đếm trên sổ sách lúc đếm, di chuyển giữa chừng sẽ làm sai chênh lệch.
- **BR-06:** Mọi di chuyển để lại dòng sổ `MOVE` (nguồn, đích, SKU, lô, số lượng, người, thời điểm). Gửi lại cùng mã yêu cầu không di chuyển lần hai.
- **BR-07:** Người thao tác chỉ di chuyển trong kho mình được gán (với role theo kho).

### Edge case

| Tình huống | Hướng xử lý |
|------------|-------------|
| Hàng ở vị trí nguồn đang giữ cho đơn | Bị từ chối; muốn dời thì nhả / phân bổ lại trước |
| Vị trí đích đang `Blocked` / `Maintenance` / `Inactive` | Bị từ chối |
| Vị trí nguồn đang `Blocked` / `Maintenance` | Được di chuyển ra (dọn bin) |
| Vị trí đang có phiếu kiểm kê mở cho SKU / lô đó | Bị từ chối tới khi phiếu xong và chênh lệch có quyết định |
| Đích đã có cùng lô nhưng khác trạng thái (ví dụ `Quarantine`) | Bị từ chối; chọn vị trí khác |
| Vị trí nguồn có hai tầng hàng cùng SKU và lô nhưng khác ngày nhận | Bị từ chối (chưa chọn được tầng nào); xử lý bằng kiểm kê / điều chỉnh, hoặc chờ một tầng hết |
| Di chuyển nhầm SKU / lô / vị trí | Di chuyển ngược lại; không sửa tồn trực tiếp |
| Hàng thực tế ít hơn sổ sách | Kiểm kê + điều chỉnh có duyệt ([20](../20-inventory-control/README.md)) |
| Muốn dời nguyên thùng LPN | Chưa hỗ trợ bằng tay (phạm vi sau — C24); thùng chỉ được dời nguyên thùng khi cất hàng / QC |

---

## 7. Liên kết với các module khác

| Module | Quan hệ |
|--------|---------|
| [06 · Warehouse Map & Slotting](../06-warehouse-map-slotting/README.md) | Vị trí, trạng thái hiệu lực; re-slotting là phạm vi sau |
| [07 · Picking](../07-picking/README.md) | Hàng đã phân bổ không bị dời; bổ sung bin picking tự động là phạm vi sau |
| [05 · Putaway](../05-putaway/README.md) | Cất hàng có kiểm ràng buộc cứng; di chuyển thủ công dùng để sửa vị trí sau khi cất |
| [10 · Inter-warehouse Transfer](../10-inter-warehouse-transfer/README.md) | Phân biệt phạm vi: trong kho (11) vs giữa kho (10) |
| [20 · Inventory Control](../20-inventory-control/README.md) | Dòng sổ `MOVE`; phiếu kiểm kê chặn di chuyển; chênh lệch xử lý bằng kiểm kê / điều chỉnh có duyệt |
| [00 · System Overview §5](../../00-system-overview/README.md) | Luồng Transfer / điều phối tồn kho |
