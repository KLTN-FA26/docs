# Tài liệu sản phẩm — StockFlowCommerce

> **Loại tài liệu:** Business Requirement Document (BRD) / Product Documentation
> **Phiên bản:** 0.1 (bản nháp)
> **Ngôn ngữ:** Tiếng Việt (kèm thuật ngữ tiếng Anh trong ngoặc)
> **Ngày cập nhật:** 2026-08-28

---

## 1. StockFlowCommerce là gì

**StockFlowCommerce** là nền tảng hợp nhất (unified platform) giữa **Quản lý quy trình kho vận (Warehouse Management System – WMS)** và **Sàn thương mại điện tử (E-commerce Storefront)**, phục vụ doanh nghiệp chuyên cung cấp **ly và bao bì dùng một lần cho ngành đồ uống**, bán dưới dạng hàng trơn hoặc **in logo theo yêu cầu** (customizable / print-on-demand products). Khách hàng chủ yếu là quán / chuỗi đồ uống (B2B).

Dải mặt hàng gồm hai nhóm:

| Nhóm hàng | Ví dụ |
|-----------|-------|
| **Ly** | Ly nhựa PP / PET (ly trà sữa, cà phê đá), ly giấy nóng / lạnh |
| **Bao bì mang đi** | Túi đựng ly, khay giấy, túi giấy |

Hệ thống gồm hai mảng lớn:

| Mảng | Tên gọi | Vai trò chính |
|------|---------|---------------|
| **A. Kho vận** | Warehouse / Back-office | Tạo sản phẩm, nhập hàng, lưu kho, vận hành kho, sản xuất / in theo đơn, xuất kho, chuyển kho |
| **B. Sàn TMĐT** | E-commerce / Storefront | Thiết kế sản phẩm in ấn, catalog, giỏ hàng, đặt hàng, thanh toán, chăm sóc khách hàng |

Hai mảng gặp nhau tại **Tồn kho (Inventory)**: kho vận là nơi *cung* hàng hoá, sàn TMĐT là nơi *phát sinh nhu cầu* xuất hàng.

---

## 2. Cách đọc tài liệu

Tài liệu được tổ chức theo **thứ tự luồng nghiệp vụ** (business flow order). Đọc tuần tự từ `00` → `18` sẽ đi hết vòng đời một sản phẩm: từ lúc khai báo → nhập kho → khách đặt → xuất kho → giao hàng.

> Module `19-production` được bổ sung sau, nên số thứ tự không theo luồng: về nghiệp vụ, **Sản xuất / In theo đơn** nằm giữa `17` (Order Management) và `07`/`08` (Picking / Packing) cho các dòng đơn tùy chỉnh.

```
docs/
├── README.md                              ← bạn đang ở đây
├── 00-system-overview/                    Tổng quan hệ thống, actors, sơ đồ luồng tổng thể
│
├── warehouse/                             MẢNG A — KHO VẬN
│   ├── 01-product-creation/               Tạo sản phẩm (master data)
│   ├── 02-purchase-order/                 Đơn đặt hàng NCC (PO)
│   ├── 03-receipt/                        Nhận hàng
│   ├── 04-invoice/                        Hoá đơn
│   ├── 05-putaway/                        Chuyển hàng vào kho
│   ├── 06-warehouse-map-slotting/         Kho Map 2D/3D & Gợi ý vị trí
│   ├── 07-picking/                        Lấy hàng
│   ├── 08-packing/                        Đóng gói
│   ├── 09-shipping/                       Giao cho đơn vị vận chuyển
│   ├── 10-inter-warehouse-transfer/       Chuyển hàng giữa các kho
│   ├── 11-intra-warehouse-transfer/       Chuyển hàng trong nội bộ một kho
│   └── 19-production/                     Sản xuất / In theo đơn (xưởng in nội bộ)
│
├── ecommerce/                             MẢNG B — SÀN TMĐT
│   ├── 12-product-design/                 Thiết kế 2D + Xem trước 3D trực tiếp
│   ├── 13-catalog/                        Catalog sản phẩm
│   ├── 14-cart-and-order/                 Giỏ hàng & Đặt hàng
│   ├── 15-payment/                        Thanh toán & các luồng thanh toán
│   ├── 16-sales-chat/                     Nhắn tin trực tiếp với Sale
│   ├── 17-order-management/               Quản lý đơn hàng
│   └── 18-customer-information/           Quản lý thông tin khách hàng
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
| **Quản trị hệ thống (System Admin)** | Cấu hình hệ thống, phân quyền, khai báo kho, quản lý dữ liệu gốc sản phẩm | 01, 06 |
| **Nhân viên mua hàng (Procurement Staff)** | Lập & chốt PO, đối chiếu hoá đơn | 02, 03, 04 |
| **Nhân viên kho (Warehouse Staff)** | Nhận hàng, cất hàng, lấy hàng (kể cả cấp phôi cho xưởng), đóng gói, chuyển kho | 03, 05, 07, 08, 09, 10, 11 |
| **Quản lý kho (Warehouse Manager)** | Giám sát vận hành, duyệt điều chỉnh tồn, duyệt chuyển kho, điều phối xưởng in | 05, 06, 10, 11, 19 |
| **Nhân viên sản xuất (Production Staff)** | Vận hành trạm in: in theo lệnh sản xuất, báo số lượng đạt / hỏng | 19 |
| **Nhân viên kế hoạch tồn kho (Inventory Planner)** | Hoạch định nhu cầu nhập hàng, cân bằng tồn giữa các kho, tạo lệnh chuyển kho | 02, 10 |
| **Nhân viên QC (QC Staff)** | Kiểm tra chất lượng hàng nhập & thành phẩm in | 03, 19 |
| **Kế toán (Accountant)** | Đối chiếu ba chiều, công nợ NCC, đối soát thanh toán/COD | 04, 15, 17 |
| **Quản trị sàn (E-commerce Admin)** | Quản lý catalog, giá, khuyến mãi, nội dung, template thiết kế; kiểm file in trước sản xuất (prepress) | 12, 13, 14, 18, 19 |
| **Nhân viên Sale (Sales Staff)** | Tư vấn khách qua chat, hỗ trợ tạo/sửa đơn, xử lý đơn hàng | 16, 17, 18 |
| **Điều phối đơn hàng (Order Coordinator)** | Release đơn cho kho, xử lý đơn treo, chọn kho xuất | 17 |
| **Khách hàng (Customer / Buyer)** | Thiết kế sản phẩm, xem catalog, đặt hàng, thanh toán, theo dõi đơn | 12, 13, 14, 15, 16 |

---

## 4. Tài liệu liên quan

- [Tổng quan hệ thống & sơ đồ luồng tổng thể](00-system-overview/README.md)
- [Sơ đồ luồng & trạng thái (Mermaid)](diagrams/README.md)
- [Bảng thuật ngữ (Glossary)](glossary/README.md)
- [Câu hỏi mở (Open Questions)](open-questions/README.md)
