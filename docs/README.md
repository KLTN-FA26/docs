# Tài liệu sản phẩm — StockFlowCommerce

> **Loại tài liệu:** Business Requirement Document (BRD) / Product Documentation
> **Phiên bản:** 0.1 (bản nháp)
> **Ngôn ngữ:** Tiếng Việt (kèm thuật ngữ tiếng Anh trong ngoặc)
> **Ngày cập nhật:** 2026-11-06

---

## 1. StockFlowCommerce là gì

**StockFlowCommerce** là nền tảng hợp nhất (unified platform) giữa **Quản lý kho thành phẩm (Warehouse Management System – WMS)** và **Bán hàng sỉ qua sàn bán hàng (B2B Customer Portal)**, phục vụ doanh nghiệp chuyên cung cấp **ly và bao bì dùng một lần cho ngành đồ uống**, **in logo theo mẫu của khách sỉ** (custom-printed cups & packaging) — không bán hàng trơn. Khách hàng chủ yếu là quán / chuỗi đồ uống. Doanh nghiệp **chỉ bán sỉ (B2B)** cho khách đã liên hệ từ trước — không bán lẻ cho khách vãng lai.

> **Đã chốt 2026-11-04 (review nghiệp vụ — thu hẹp phạm vi):** kho chỉ là **kho thành phẩm**. **Sản xuất nằm ngoài hệ thống**: ứng với đơn hàng thiếu tồn, hệ thống gửi **Đơn sản xuất** cho đơn vị sản xuất, rồi lập phiếu nhập kho khi sản xuất xong (không kiểm tra chất lượng tại kho). Sàn bán hàng chỉ để khách **xem hàng, xem mẫu thiết kế, chat và chốt đơn với Sale**. Mỗi đơn có một **hợp đồng mua bán**; khách **thanh toán theo đợt của hợp đồng** (chuyển khoản), không có thanh toán kiểu thương mại điện tử (QR / cổng thanh toán). Đã xoá các module 02 · Purchase Order, 04 · Invoice, 19 · Production. **Không còn hàng trơn**: kho chỉ chứa thành phẩm theo mẫu của khách, được đơn vị sản xuất đóng thùng sẵn; kho chỉ xuất nguyên thùng, không đóng lại hàng.
>
> **Đã chốt 2026-11-05 (khách tự thiết kế):** **chỉ khách tự thiết kế trên app** — trên **khuôn 2D** của quy cách, có **xem trước 3D**; bỏ luồng khách đưa mẫu cho Sale / nhân viên thiết kế hộ. Hệ thống **tự kiểm (preflight)** khi gửi duyệt; **Quản lý kinh doanh** duyệt kỹ thuật và kiểm duyệt nội dung; duyệt xong hệ thống **tự xuất file in PDF/X**, chốt design snapshot và sinh SKU riêng của khách. Đặt hàng từ bản thiết kế = **yêu cầu báo giá** gửi Sale; khách vẫn không tự đặt, sàn vẫn ẩn giá ([12](ecommerce/12-product-design/README.md)).
>
> **✅ Đã chốt 2026-11-06 (dự án thiết kế — góp ý hội đồng):** khách gom nhiều bản thiết kế của một bộ nhận diện vào **dự án thiết kế**: **thư viện logo / ảnh dùng chung**, **gửi duyệt cả bộ** (Quản lý kinh doanh xem cả bộ, quyết định từng bản), **yêu cầu báo giá cả dự án** → một báo giá nhiều dòng. Dự án không bắt buộc, không thêm trạng thái cho bản thiết kế ([12](ecommerce/12-product-design/README.md) §4.9–§4.11).
>
> **✅ Đã chốt 2026-11-06 (hoá đơn điện tử):** hệ thống **tự phát hành hoá đơn VAT điện tử** qua **API của nhà cung cấp hoá đơn điện tử** — mỗi lần giao thành công một hoá đơn, Kế toán bấm phát hành; khách tải XML / PDF trên trang đơn ([21](ecommerce/21-e-invoice/README.md)).

Dải mặt hàng gồm hai nhóm:

| Nhóm hàng | Ví dụ |
|-----------|-------|
| **Ly** | Ly nhựa PP / PET (ly trà sữa, cà phê đá), ly giấy nóng / lạnh |
| **Bao bì mang đi** | Túi đựng ly, khay giấy, túi giấy |

Hệ thống gồm hai mảng lớn, chia thành năm nhóm chức năng: **quản lý thiết kế**, **bán hàng** (đặt hàng theo số lượng, hợp đồng từng đơn), **quản lý thanh toán** (theo hợp đồng, công nợ, hoá đơn điện tử), **quản lý kho** (vận hành kho, xuất kho) và **quản lý giao hàng**.

| Mảng | Tên gọi | Vai trò chính |
|------|---------|---------------|
| **A. Kho thành phẩm** | Warehouse / Back-office | Dữ liệu sản phẩm, nhập kho thành phẩm theo Đơn sản xuất và nhập hàng hoàn, lưu kho, sức chứa kho, vận hành kho, khu chờ xuất, xuất kho, giao hàng, kiểm soát tồn |
| **B. Bán hàng sỉ** | Sales / Sàn bán hàng | Thiết kế của khách (khách tự thiết kế trên khuôn 2D, xem trước 3D, Quản lý kinh doanh duyệt, hệ thống xuất file in PDF/X), sàn bán hàng (xem hàng, không hiện giá), chat với Sale, báo giá (giá gợi ý từ bảng giá nội bộ), đơn hàng & hợp đồng mua bán, thanh toán theo đợt & công nợ, hoá đơn điện tử theo lần giao, Đơn sản xuất (kèm khách duyệt mẫu in), chăm sóc khách hàng |

Hai mảng gặp nhau tại **Tồn kho thành phẩm (Inventory)**: bán hàng sỉ phát sinh nhu cầu; kho kiểm tồn, nếu thiếu thì Đơn sản xuất được gửi ra ngoài và thành phẩm quay về kho qua phiếu nhập kho.

---

## 2. Cách đọc tài liệu

Tài liệu được tổ chức theo **thứ tự luồng nghiệp vụ** (business flow order). Đọc tuần tự từ `00` → `18` sẽ đi hết vòng đời một đơn hàng: từ lúc khai báo sản phẩm → nhập kho thành phẩm → khách đặt → xuất kho → giao hàng → thu tiền.

> Số thứ tự `02` (Purchase Order), `04` (Invoice), `19` (Production) — xoá 2026-11-04 — và `10` (Inter-warehouse Transfer) — xoá 2026-11-05 — **bỏ trống**; các module còn lại giữ nguyên số.
>
> Module `20-inventory-control` được bổ sung sau: **Kiểm soát tồn kho** (kiểm kê, điều chỉnh tồn, cảnh báo, truy xuất lô) không phải một bước trong luồng mà chạy song song, kiểm soát tồn mà các module 03, 05–11 ghi vào.
>
> Module `21-e-invoice` được bổ sung 2026-11-06: **Hoá đơn điện tử** phát hành qua nhà cung cấp hoá đơn điện tử sau mỗi lần giao thành công (09), chạy song song với thanh toán (15). Dùng số mới thay vì số `04` / `19` đã xoá để tránh nhầm với module cũ.

```
docs/
├── README.md                              ← bạn đang ở đây
├── 00-system-overview/                    Tổng quan hệ thống, actors, sơ đồ luồng tổng thể
│
├── warehouse/                             MẢNG A — KHO THÀNH PHẨM
│   ├── 01-product-creation/               Tạo sản phẩm (master data; SKU thành phẩm theo mẫu của khách)
│   ├── 03-receipt/                        Nhập kho thành phẩm (theo Đơn sản xuất)
│   ├── 05-putaway/                        Chuyển hàng vào kho
│   ├── 06-warehouse-map-slotting/         Kho Map 2D/3D & Gợi ý vị trí
│   ├── 07-picking/                        Lấy hàng nguyên thùng (về khu chờ xuất / khu xuất hàng)
│   ├── 08-packing/                        Kiểm thùng & Dán nhãn giao hàng
│   ├── 09-shipping/                       Giao hàng (bàn giao đơn vị vận chuyển)
│   ├── 11-intra-warehouse-transfer/       Chuyển hàng trong nội bộ một kho
│   └── 20-inventory-control/              Kiểm soát tồn kho (kiểm kê, điều chỉnh, cảnh báo, truy xuất lô)
│
├── ecommerce/                             MẢNG B — BÁN HÀNG SỈ (B2B)
│   ├── 12-product-design/                 Thiết kế của khách: khuôn 2D, xem 3D, duyệt, file in PDF/X, dự án thiết kế
│   ├── 13-catalog/                        Sàn bán hàng: xem hàng, mẫu thiết kế của tôi, yêu cầu báo giá, duyệt mẫu in (không hiện giá)
│   ├── 14-cart-and-order/                 Báo giá, Đơn hàng & Hợp đồng mua bán
│   ├── 15-payment/                        Thanh toán theo hợp đồng & Công nợ
│   ├── 16-sales-chat/                     Chat với Sale (gắn đơn / mẫu thiết kế / báo giá)
│   ├── 17-order-management/               Quản lý đơn hàng & Đơn sản xuất
│   ├── 18-customer-information/           Quản lý thông tin khách sỉ
│   └── 21-e-invoice/                      Hoá đơn điện tử theo lần giao (tích hợp nhà cung cấp hoá đơn điện tử)
│
├── diagrams/                              Sơ đồ luồng dữ liệu / trạng thái (Mermaid)
├── glossary/                              Bảng thuật ngữ tiếng Việt
└── open-questions/                        Câu hỏi mở cần Product Owner xác nhận
```

### Quy ước trình bày trong từng module

Mỗi module (`README.md` trong mỗi thư mục) được viết theo **template cố định**:

1. **Mục đích (Purpose)**
2. **Actor liên quan** — ai sử dụng
3. **Input / Output**
4. **Luồng nghiệp vụ từng bước (Step-by-step business flow)**
5. **Trạng thái / Vòng đời (Status / Lifecycle)** — dạng bảng nếu có
6. **Ràng buộc nghiệp vụ (Business rules) & Edge case**
7. **Liên kết với các module khác**

### Quy ước ký hiệu

| Ký hiệu | Ý nghĩa |
|---------|---------|
| `> ⚠️ Giả định` | Điểm nghiệp vụ do người viết **tự giả định** vì đề bài chưa nêu rõ. Tất cả giả định được gom lại trong [open-questions/](open-questions/README.md) để Product Owner xác nhận. |
| `(English Term)` | Thuật ngữ tiếng Anh tương ứng, đặt trong ngoặc sau thuật ngữ tiếng Việt |
| Bảng trạng thái | Cột: *Trạng thái · Ý nghĩa · Hành động chuyển tiếp cho phép · Ai thực hiện* |

---

## 3. Đối tượng người dùng (Personas)

Danh sách vai trò chuẩn (Role Registry) xem tại [00-system-overview §2](00-system-overview/README.md).

| Persona | Mô tả | Module thường dùng |
|---------|-------|--------------------|
| **Quản trị hệ thống (System Admin)** | Cấu hình hệ thống, phân quyền, khai báo kho, quản lý dữ liệu gốc sản phẩm, tạm ngưng / ngừng kinh doanh sản phẩm | 01, 06 |
| **Nhân viên kho (Warehouse Staff)** | Nhập kho thành phẩm theo thùng, nhập hàng hoàn, cất hàng, lấy hàng nguyên thùng, đưa hàng về khu chờ xuất, kiểm thùng & dán nhãn giao hàng, giao hàng, đếm kiểm kê | 03, 05, 07, 08, 09, 11, 20 |
| **Quản lý kho (Warehouse Manager)** | Quản lý các kho được gán: thiết lập kho & sức chứa; xử lý nhận thiếu / thừa / thùng lệch và hàng hoàn; release đơn, giao task lấy hàng, khu chờ xuất, ghi nhận kết quả giao; lập kiểm kê, duyệt điều chỉnh tồn, cách ly | 03, 05, 06, 07, 08, 09, 11, 20 |
| **Kế toán (Accountant)** | Lập phiếu thu theo đợt thanh toán của hợp đồng, ghi nhận hoàn tiền, công nợ phải thu của khách sỉ, duyệt tín dụng & lịch thanh toán, phát hành hoá đơn điện tử | 14, 15, 17, 18, 21 |
| **Quản lý kinh doanh (Sales Manager)** | Tạo & duyệt sản phẩm, danh mục, thuộc tính; **khai báo cấu hình in** cho quy cách (khuôn 2D, vùng in, lề, model 3D, ràng buộc file); nội dung sàn bán hàng, bảng giá nội bộ theo khách; **duyệt kỹ thuật & kiểm duyệt nội dung** bản thiết kế của khách; **duyệt & gửi Đơn sản xuất**; trạng thái & nhóm khách; quản trị tài khoản & phân quyền | 00, 01, 12, 13, 14, 17, 18 |
| **Nhân viên Sale (Sales Staff)** | Tạo & gửi duyệt sản phẩm, tạo khách sỉ, **nhận yêu cầu báo giá từ bản thiết kế**, báo giá, tạo đơn & hợp đồng, chọn kho xuất, lập & theo dõi Đơn sản xuất (kèm mẫu in), treo đơn, chat với khách | 01, 12, 14, 16, 17, 18 |
| **Khách hàng sỉ (B2B Customer)** | Doanh nghiệp đã liên hệ trước: **tự thiết kế** trên khuôn 2D / xem trước 3D, gửi duyệt, tạo bản sửa, **yêu cầu báo giá** từ bản thiết kế đã duyệt, duyệt mẫu in, xem hàng (không hiện giá), chat với Sale, chấp nhận báo giá, theo dõi đơn, hợp đồng, đợt thanh toán & công nợ, tải hoá đơn điện tử | 12, 13, 14, 15, 16, 21 |

---

## 4. Tài liệu liên quan

- [Tổng quan hệ thống & sơ đồ luồng tổng thể](00-system-overview/README.md)
- [Sơ đồ luồng & trạng thái (Mermaid)](diagrams/README.md)
- [Bảng thuật ngữ (Glossary)](glossary/README.md)
- [Câu hỏi mở (Open Questions)](open-questions/README.md)
