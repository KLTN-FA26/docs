# 11 — Chuyển hàng trong nội bộ một kho (Intra-warehouse Transfer)

## 1. Mục đích (Purpose)

Di chuyển tồn kho **giữa các vị trí trong cùng một kho** (bin hoặc khu vực chứa hàng → bin hoặc khu vực chứa hàng) mà **không thay đổi tổng tồn của kho**. Các mục đích chính:

- **Bổ sung khu vực lấy hàng (Replenishment):** chuyển hàng từ bin dự trữ (bin không cho phép lấy hàng, `is_pickable` = false) sang bin picking (`is_pickable` = true) khi bin picking sắp cạn.
- **Tối ưu vị trí (Re-slotting):** đưa SKU bán chạy ra gần khu đóng gói theo đề xuất của [Slotting](../06-warehouse-map-slotting/README.md).
- **Gom hàng (Consolidation):** gộp nhiều vị trí lẻ của cùng SKU/lô để giải phóng bin.
- **Sửa vị trí sai sau Putaway** hoặc dọn vị trí đang bảo trì.
- **Đổi trạng thái vị trí:** chuyển hàng ra khỏi vùng `Blocked` / `Quarantine` sau khi có kết luận.

Là nghiệp vụ **thuần vận hành nội bộ**, không có giai đoạn in-transit liên kho, chu kỳ ngắn.

---

## 2. Actor liên quan

| Actor | Vai trò |
|-------|---------|
| **Nhân viên kho (Warehouse Staff)** | Thực hiện di chuyển, quét vị trí nguồn/đích & số lượng |
| **Hệ thống (Replenishment / Slotting Engine)** | Sinh tự động nhiệm vụ bổ sung khi vị trí picking dưới ngưỡng; gợi ý vị trí đích |
| **Quản lý kho (Warehouse Manager)** | Duyệt đề xuất re-slotting, xử lý chênh lệch khi di chuyển, cấu hình ngưỡng min/max theo vị trí |

---

## 3. Input / Output

### Input

| Trường | Mô tả | Bắt buộc |
|--------|-------|----------|
| Kho (một kho duy nhất) | | Có |
| Vị trí nguồn & vị trí đích | Bin hoặc khu vực chứa hàng (Area), trong cùng kho | Có |
| SKU, số lượng, (lô/serial nếu áp dụng) | | Có |
| Lý do di chuyển | Replenishment / re-slotting / consolidation / fix / unblock | Không (khuyến nghị) |
| Nguồn phát sinh | Auto (ngưỡng min) / Manual / Đề xuất Slotting | Có |

### Output

| Kết quả | Dùng bởi |
|---------|----------|
| **Movement record** (bút toán di chuyển bin→bin) | Truy vết, lịch sử vị trí |
| **Cập nhật tồn theo vị trí** — nguồn giảm, đích tăng; tổng tồn kho **không đổi** | Tồn kho, 06 Map, 07 Picking |
| **Nhiệm vụ di chuyển (move task)** hoàn tất | Hàng đợi công việc NV kho |
| **Chênh lệch** (nếu số đặt vào ≠ số lấy ra) | Manager, điều tra |

---

## 4. Luồng nghiệp vụ từng bước (Step-by-step)

### 4.1. Bổ sung vùng picking (Replenishment) — thường tự động

1. Hệ thống giám sát tồn tại các vị trí picking so với ngưỡng min/max cấu hình.
2. Khi một vị trí picking ≤ ngưỡng min (hoặc dự báo sẽ cạn theo wave picking sắp tới) → sinh **replenishment task**: chuyển từ bin dự trữ của cùng SKU (ưu tiên FEFO) đến bin picking đó, số lượng đưa về mức max.
3. Task vào hàng đợi NV kho; ưu tiên cao nếu đang chặn picking.
4. NV kho: quét vị trí nguồn → quét SKU/lô → nhập số lượng lấy → di chuyển → quét vị trí đích → xác nhận số lượng đặt vào.
5. Tồn cập nhật; task `Completed`.

### 4.2. Tối ưu vị trí (Re-slotting) — theo đề xuất

1. [Slotting Engine](../06-warehouse-map-slotting/README.md) sinh **move list** (SKU nào nên chuyển đi đâu) dựa trên thay đổi hạng ABC, khoảng cách, honeycombing.
2. Manager xem xét & duyệt (toàn bộ hoặc từng dòng).
3. Mỗi dòng duyệt → sinh một intra-transfer task.
4. NV kho thực hiện như 4.1 bước 4.

### 4.3. Di chuyển thủ công (Manual)

1. NV kho / Manager tạo move task: chọn vị trí nguồn, SKU, số lượng, vị trí đích (có thể lấy gợi ý từ Slotting).
2. Hệ thống kiểm tra hard constraints của vị trí đích (trạng thái, cho phép cất, lớp lưu trữ, sức chứa, tải trọng & chiều cao lọt của tầng) — xem [06](../06-warehouse-map-slotting/README.md).
3. Thực hiện & xác nhận bằng quét mã.

### 4.4. Xử lý chênh lệch

- Nếu số lượng đặt vào vị trí đích khác số lấy ra khỏi nguồn (rơi vãi, đếm sai) → ghi **discrepancy**; tạo yêu cầu kiểm kê (cycle count) cho cả hai vị trí; điều chỉnh tồn có phê duyệt — xem [20 §4.2–4.3](../20-inventory-control/README.md).

---

## 5. Trạng thái / Vòng đời (Lifecycle)

### Trạng thái move task

| Trạng thái | Ý nghĩa | Chuyển tiếp | Ai |
|------------|---------|-------------|----|
| `Suggested` | Đề xuất từ Slotting, chờ duyệt (chỉ với re-slotting) | → `Pending`, → `Rejected` | Manager |
| `Pending` | Đã sinh (auto/manual/approved), chờ phân công | → `Assigned`, → `Cancelled` | System |
| `Assigned` | Đã giao NV kho | → `In Progress`, → `Pending` | Staff / Manager |
| `In Progress` | Đang di chuyển | → `Completed`, → `On Hold`, → `Discrepancy` | Staff |
| `On Hold` | Tạm dừng (đích hết chỗ, chờ Manager) | → `In Progress`, → `Cancelled` | Staff / Manager |
| `Discrepancy` | Lệch số lượng, chờ kiểm kê | → `Completed` (đã chốt) | Manager |
| `Completed` | Hàng đã ở vị trí đích, tồn cập nhật | *(cuối)* | Staff |
| `Cancelled` / `Rejected` | Không thực hiện | *(cuối)* | Manager |

---

## 6. Ràng buộc nghiệp vụ (Business Rules) & Edge case

### Business rules

- **BR-01:** Vị trí nguồn và đích phải **cùng một kho**. Khác kho → dùng [Inter-warehouse Transfer](../10-inter-warehouse-transfer/README.md).
- **BR-02:** Tổng tồn kho của kho **không thay đổi** sau intra-transfer; chỉ phân bố theo vị trí thay đổi.
- **BR-03:** Không vi phạm hard constraints của vị trí đích (trạng thái, cho phép cất, lớp lưu trữ, sức chứa, tải trọng & chiều cao lọt của tầng).
- **BR-04:** Không di chuyển hàng đang `Allocated` cho một pick task đang chạy trừ khi hủy phân bổ trước (tránh picker tới vị trí trống).
- **BR-05:** Di chuyển hàng `Quarantine`/`Blocked` chỉ khi đã có kết luận & quyền phù hợp; ghi rõ lý do unblock.
- **BR-06:** Mọi di chuyển để lại movement record (nguồn, đích, SKU, lô, số lượng, người, thời gian, lý do).
- **BR-07:** FEFO: khi bổ sung bin picking, ưu tiên lấy lô hạn dùng gần nhất từ bin dự trữ.

### Edge case

| Tình huống | Hướng xử lý |
|------------|-------------|
| Vị trí picking cạn giữa wave, chưa kịp replenish | Sinh replenishment ưu tiên khẩn; picker tạm bỏ dòng → short pick → re-allocate |
| Vị trí đích đầy hơn dự kiến | Task `On Hold`; Slotting gợi ý vị trí thay thế; hoặc chia nhỏ số lượng |
| Rơi vãi/hư khi di chuyển | Ghi `Discrepancy` + adjustment; cycle count hai vị trí ([20](../20-inventory-control/README.md)) |
| Di chuyển nhầm SKU/lô | Sửa bằng move task ngược + ghi log; không sửa trực tiếp tồn |
| Re-slotting khối lượng lớn làm gián đoạn vận hành | Manager lên lịch ngoài giờ cao điểm; chia lô nhỏ |
| Hàng ở vị trí `Maintenance` cần giải toả gấp | Move task ưu tiên đưa hàng sang vị trí hợp lệ trước khi khoá vị trí |
| Hai task cùng nhắm vị trí đích | Soft-reserve sức chứa vị trí đích khi task `Assigned`; task sau tính lại |

---

## 7. Liên kết với các module khác

| Module | Quan hệ |
|--------|---------|
| [06 · Warehouse Map & Slotting](../06-warehouse-map-slotting/README.md) | Nguồn đề xuất re-slotting & gợi ý vị trí đích |
| [07 · Picking](../07-picking/README.md) | Replenishment giữ cho vùng picking không cạn; giảm short pick |
| [05 · Putaway](../05-putaway/README.md) | Công cụ sửa vị trí sai sau khi putaway đã `Completed` |
| [03 · Receipt](../03-receipt/README.md) | Di chuyển hàng ra khỏi vùng `Quarantine` sau khi QC kết luận `Accepted` |
| [10 · Inter-warehouse Transfer](../10-inter-warehouse-transfer/README.md) | Phân biệt phạm vi: trong kho (11) vs giữa kho (10) |
| [00 · System Overview §5](../../00-system-overview/README.md) | Luồng Transfer / điều phối tồn kho |
| [20 · Inventory Control](../20-inventory-control/README.md) | Chênh lệch khi di chuyển xử lý bằng kiểm kê và điều chỉnh tồn có duyệt |
