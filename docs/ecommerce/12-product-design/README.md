# 12 — Thiết kế sản phẩm in ấn (Product Design: 2D Design + 3D Live Preview)

## 1. Mục đích (Purpose)

Cho phép khách hàng **tự thiết kế nội dung in ấn** trên sản phẩm tùy chỉnh (áo, cốc, túi, hộp, name card…) bằng trình thiết kế 2D, và **xem trước kết quả trên mô hình 3D theo thời gian thực (Live 3D Preview)** trước khi thêm vào giỏ và đặt hàng.

Đầu ra là một **"artifact thiết kế" (design artifact / design snapshot)** — bản mô tả đầy đủ và bất biến của thiết kế khách chốt — đi kèm đơn hàng để **xưởng in / sản xuất** thực hiện chính xác.

---

## 2. Actor liên quan

| Actor | Vai trò |
|-------|---------|
| **Khách hàng (Customer)** | Tạo/sửa thiết kế, tải ảnh, nhập text, chọn phông/màu, xem preview 3D, lưu & chốt thiết kế |
| **Nhân viên Sale** | Hỗ trợ khách qua [chat](../16-sales-chat/README.md): tinh chỉnh thiết kế hộ, kiểm tra tính khả thi in ấn, chốt cùng khách |
| **Admin / Nhân viên chuẩn bị file in (Prepress)** | Cấu hình template, print area, ràng buộc kỹ thuật; kiểm tra file trước sản xuất |
| **Hệ thống (Design Engine + 3D Renderer)** | Kiểm tra ràng buộc (DPI, tràn lề, vùng an toàn), sinh preview 3D, xuất file in (print-ready) |

---

## 3. Input / Output

### Input

| Nhóm | Dữ liệu | Nguồn |
|------|---------|-------|
| **Template & khung sản phẩm** | Model 3D, template 2D, danh sách **print area** (vị trí, kích thước, hình dạng), vùng an toàn (safe area), bleed | [01 Product Creation](../01-product-creation/README.md) |
| **Ràng buộc kỹ thuật** | DPI tối thiểu, hệ màu (RGB/CMYK), định dạng & dung lượng file cho phép, số màu in tối đa (nếu in lụa), font được phép nhúng | [01 Product Creation](../01-product-creation/README.md) |
| **Nội dung khách đưa vào** | Ảnh tải lên, chữ (text) + phông + màu + hiệu ứng, hình khối, clipart có sẵn, layout | Khách hàng |
| **Lựa chọn biến thể** | Màu sản phẩm, kích cỡ, chất liệu (ảnh hưởng preview & vùng in) | Khách hàng / [13 Catalog](../13-catalog/README.md) |

### Output

| Kết quả | Mô tả | Dùng bởi |
|---------|-------|----------|
| **Design draft** | Thiết kế đang chỉnh, lưu tạm theo tài khoản khách | Khách, Sale |
| **3D preview** | Ảnh/scene 3D render nội dung lên mô hình sản phẩm, cập nhật realtime | Khách (ra quyết định mua) |
| **Design snapshot / artifact** | Bản chốt bất biến: layout, tài nguyên (ảnh, font), thông số print area, phiên bản, hash | [14 Cart & Order](../14-cart-and-order/README.md), [08 Packing](../../warehouse/08-packing/README.md), xưởng in |
| **Print-ready file** | File xuất bản in (PDF/CMYK, đúng bleed & DPI) | Xưởng in / Prepress |
| **Preflight report** | Kết quả kiểm tra kỹ thuật (đạt / cảnh báo / lỗi) | Khách, Prepress |

---

## 4. Luồng nghiệp vụ từng bước (Step-by-step)

1. **Chọn sản phẩm để thiết kế** — từ [Catalog](../13-catalog/README.md), khách chọn sản phẩm tùy chỉnh & biến thể (màu, size). Hệ thống nạp template 2D + model 3D + print area + ràng buộc.
2. **Mở trình thiết kế 2D** — khung làm việc hiển thị print area, vùng an toàn, đường bleed.
3. **Thêm nội dung:**
   - Tải ảnh: hệ thống kiểm tra định dạng, dung lượng, **DPI theo kích thước đặt** → cảnh báo nếu ảnh mờ.
   - Thêm text: chọn phông (trong danh sách cho phép), cỡ, màu, canh lề, hiệu ứng.
   - Thêm hình khối / clipart / template có sẵn.
4. **Sắp xếp layout** — kéo–thả, xoay, scale, canh; hệ thống chặn/cảnh báo khi phần tử **tràn ra ngoài print area** hoặc **đè vùng cấm**.
5. **Xem trước 3D trực tiếp (Live 3D Preview)** — mọi thay đổi ở canvas 2D được **map texture** lên model 3D và render lại gần như tức thời; khách xoay/zoom mô hình để xem mọi góc.
6. **Preflight (kiểm tra kỹ thuật)** — hệ thống chạy kiểm tra: độ phân giải, hệ màu, phông đã nhúng, khoảng cách an toàn, số màu. Trả **preflight report**: `Pass` / `Warning` / `Fail`.
7. **Lưu nháp (Save draft)** — thiết kế lưu theo tài khoản khách, có thể quay lại chỉnh sau; hỗ trợ nhiều phiên bản.
8. **(Tuỳ chọn) Nhờ Sale hỗ trợ** — khách mở [chat](../16-sales-chat/README.md), chia sẻ link thiết kế; Sale xem, góp ý hoặc chỉnh hộ, gửi lại.
9. **Chốt thiết kế (Confirm design)** — khách xác nhận. Hệ thống:
   - Yêu cầu preflight không còn lỗi `Fail` (Warning có thể cho qua kèm khách xác nhận rủi ro).
   - Sinh **design snapshot** bất biến (đóng băng tài nguyên & thông số) + phiên bản.
10. **Đưa vào giỏ** — thiết kế đã chốt gắn với dòng sản phẩm trong [Cart](../14-cart-and-order/README.md). Giá có thể phụ thuộc thông số in (số màu, diện tích in, kỹ thuật).
11. **Khoá sau khi đặt hàng** — khi [Order](../14-cart-and-order/README.md) `Confirmed`, design snapshot **không thể sửa**; muốn đổi phải huỷ/tạo dòng mới hoặc quy trình yêu cầu thay đổi có kiểm soát.
12. **Xuất file cho sản xuất** — hệ thống sinh print-ready file từ snapshot cho xưởng in; [Packing](../../warehouse/08-packing/README.md) dùng preview để đối chiếu thành phẩm.

---

## 5. Trạng thái / Vòng đời (Lifecycle)

### Trạng thái Design

| Trạng thái | Ý nghĩa | Chuyển tiếp | Ai |
|------------|---------|-------------|----|
| `Draft` | Đang thiết kế, lưu tạm | → `Draft` (lưu bản mới), → `Ready` | Khách / Sale |
| `Ready` | Preflight đạt, khách hài lòng, chưa gắn đơn | → `Confirmed`, → `Draft` (sửa tiếp) | Khách |
| `Confirmed` | Đã chốt, snapshot sinh ra, đã/đang gắn vào giỏ | → `Locked`, → `Superseded` (khách sửa tạo bản mới trước khi đặt) | Khách |
| `Locked` | Đơn đã đặt & confirmed — bất biến | → `Cancelled` (đơn huỷ) | System |
| `Superseded` | Bị thay bằng phiên bản mới hơn | *(cuối)* | System |
| `Cancelled` | Bỏ / đơn huỷ | *(cuối)* | Khách / System |

### Trạng thái Preflight

`Pass` (in được) · `Warning` (in được nhưng có rủi ro chất lượng — cần khách xác nhận) · `Fail` (chặn chốt: thiếu DPI, sai hệ màu, phông không nhúng…)

---

## 6. Ràng buộc nghiệp vụ (Business Rules) & Edge case

### Business rules

- **BR-01:** Không cho `Confirmed` khi preflight còn lỗi `Fail`.
- **BR-02:** Nội dung thiết kế **không được vượt ra ngoài print area** đã khai báo cho biến thể đang chọn.
- **BR-03:** Design snapshot là **bất biến** sau khi đơn `Confirmed`; mọi tài nguyên (ảnh upload, font) được lưu kèm phiên bản, không phụ thuộc file gốc của khách sau này.
- **BR-04:** Đổi biến thể sản phẩm (màu/size/chất liệu) có thể làm print area thay đổi → hệ thống phải kiểm tra lại layout và cảnh báo phần tử không hợp lệ.
- **BR-05:** Giá dòng hàng có thể phụ thuộc thông số in (diện tích phủ mực, số màu, kỹ thuật in) — công thức tính giá lấy từ cấu hình sản phẩm.  ⚠️ Giả định — xem [open-questions](../../open-questions/README.md).
- **BR-06:** Ảnh/nội dung khách tải lên phải qua kiểm tra cơ bản (định dạng, dung lượng, có thể cả rà nội dung vi phạm bản quyền/không phù hợp theo chính sách).
- **BR-07:** 3D preview là hình ảnh **minh hoạ** — có disclaimer màu sắc hiển thị có thể sai lệch so với thành phẩm in thật.

### Edge case

| Tình huống | Hướng xử lý |
|------------|-------------|
| Ảnh khách tải lên độ phân giải thấp | Cảnh báo `Warning`; cho phép tiếp tục nếu khách chấp nhận, ghi nhận đã cảnh báo |
| Khách chốt thiết kế rồi muốn sửa trước khi thanh toán | Tạo phiên bản mới; bản cũ `Superseded`; cập nhật dòng giỏ |
| Khách muốn sửa sau khi đơn đã `Confirmed` | Không sửa snapshot; qua quy trình "yêu cầu thay đổi": nếu chưa sản xuất → huỷ dòng + tạo lại; nếu đã sản xuất → tính phí/từ chối theo chính sách |
| Font khách dùng không có quyền nhúng | Chặn hoặc thay bằng font tương đương; ghi rõ trong preflight |
| Model 3D của sản phẩm lỗi/thiếu | Cho thiết kế 2D + preview 2D phẳng; đánh dấu preview 3D không khả dụng |
| Khách bỏ giữa chừng | Lưu `Draft` tự động; khôi phục khi quay lại |
| Sale chỉnh hộ nhưng khách không đồng ý | Giữ nhiều phiên bản; khách chọn bản để `Confirmed` |
| Trình duyệt/thiết bị yếu không render 3D | Hạ cấp xuống preview ảnh tĩnh render phía server |

---

## 7. Liên kết với các module khác

| Module | Quan hệ |
|--------|---------|
| [01 · Product Creation](../../warehouse/01-product-creation/README.md) | Cung cấp template 2D, model 3D, print area, ràng buộc kỹ thuật, công thức giá in |
| [13 · Catalog](../13-catalog/README.md) | Điểm vào: khách chọn sản phẩm tùy chỉnh từ catalog để thiết kế |
| [14 · Cart & Order](../14-cart-and-order/README.md) | Design snapshot gắn vào dòng giỏ/đơn; ảnh hưởng giá dòng hàng |
| [16 · Sales Chat](../16-sales-chat/README.md) | Kênh phối hợp khách – Sale để tinh chỉnh & chốt thiết kế |
| [08 · Packing](../../warehouse/08-packing/README.md) | Dùng preview/snapshot để đối chiếu thành phẩm in ấn khi đóng gói |
| [17 · Order Management](../17-order-management/README.md) | Quản lý yêu cầu thay đổi thiết kế sau khi đặt; liên kết file sản xuất |
| [18 · Customer Information](../18-customer-information/README.md) | Thiết kế nháp lưu theo hồ sơ khách hàng |
