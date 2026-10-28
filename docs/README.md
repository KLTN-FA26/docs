# Tài liệu sản phẩm — StockFlowCommerce

> **Loại tài liệu:** Business Requirement Document (BRD) / Product Documentation
> **Phiên bản:** 0.1 (bản nháp)
> **Ngôn ngữ:** Tiếng Việt (kèm thuật ngữ tiếng Anh trong ngoặc)
> **Ngày cập nhật:** 2026-10-06

---

## 1. StockFlowCommerce là gì

**StockFlowCommerce** là nền tảng hợp nhất (unified platform) giữa **Quản lý quy trình kho vận (Warehouse Management System – WMS)** và **Bán hàng sỉ qua cổng khách hàng (B2B Customer Portal)**, phục vụ doanh nghiệp chuyên cung cấp **ly và bao bì dùng một lần cho ngành đồ uống**, bán dưới dạng hàng trơn hoặc **in logo theo thiết kế của khách sỉ** (custom-printed cups & packaging), có **xưởng in nội bộ**. Khách hàng chủ yếu là quán / chuỗi đồ uống. Doanh nghiệp **chỉ bán sỉ (B2B)** cho khách đã liên hệ từ trước — không bán lẻ cho khách vãng lai.

Dải mặt hàng gồm hai nhóm:

| Nhóm hàng | Ví dụ |
|-----------|-------|
| **Ly** | Ly nhựa PP / PET (ly trà sữa, cà phê đá), ly giấy nóng / lạnh |
| **Bao bì mang đi** | Túi đựng ly, khay giấy, túi giấy |

Hệ thống gồm hai mảng lớn:

| Mảng | Tên gọi | Vai trò chính |
|------|---------|---------------|
| **A. Kho vận & sản xuất** | Warehouse / Back-office | Tạo sản phẩm, nhập hàng, lưu kho, vận hành kho, **sản xuất in (mẫu & đơn hàng)**, xuất kho, chuyển kho |
| **B. Bán hàng sỉ** | Sales / Customer Portal | Thiết kế sản phẩm in, báo giá, làm mẫu, cổng khách hàng (catalog, đặt hàng), thanh toán (cọc / trả trước / công nợ), chăm sóc khách hàng |

Hai mảng gặp nhau tại **Tồn kho (Inventory)**: kho vận & xưởng in là nơi *cung* hàng hoá, bán hàng sỉ là nơi *phát sinh nhu cầu* sản xuất và xuất hàng.

---

## 2. Cách đọc tài liệu

Tài liệu được tổ chức theo **thứ tự luồng nghiệp vụ** (business flow order). Đọc tuần tự từ `00` → `18` sẽ đi hết vòng đời một sản phẩm: từ lúc khai báo → nhập kho → khách đặt → xuất kho → giao hàng.

> Module `19-production` được bổ sung sau, nên số thứ tự không theo luồng: về nghiệp vụ, **Sản xuất / In theo đơn** (mẫu và đơn hàng) nằm giữa `17` (Order Management) và `07`/`08` (Picking / Packing) cho các dòng đơn tùy chỉnh.
>
> Module `20-inventory-control` cũng được bổ sung sau: **Kiểm soát tồn kho** (kiểm kê, điều chỉnh tồn, cảnh báo, truy xuất lô) không phải một bước trong luồng mà chạy song song, kiểm soát tồn mà các module 03–11 và 19 ghi vào.

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
│   ├── 19-production/                     Sản xuất / In ly & bao bì (mẫu + đơn hàng, xưởng in nội bộ)
│   └── 20-inventory-control/              Kiểm soát tồn kho (kiểm kê, điều chỉnh, cảnh báo, truy xuất lô)
│
├── ecommerce/                             MẢNG B — BÁN HÀNG SỈ (B2B)
│   ├── 12-product-design/                 Thiết kế in: nhân viên thiết kế, QC duyệt, khách xác nhận
│   ├── 13-catalog/                        Catalog trên cổng khách hàng (giá riêng)
│   ├── 14-cart-and-order/                 Báo giá & Đặt hàng sỉ
│   ├── 15-payment/                        Thanh toán & công nợ khách sỉ
│   ├── 16-sales-chat/                     Nhắn tin trực tiếp với Sale
│   ├── 17-order-management/               Quản lý đơn hàng
│   └── 18-customer-information/           Quản lý thông tin khách sỉ
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
| **Nhân viên mua hàng (Procurement Staff)** | Lập & chốt PO (kể cả PO gia công), đối chiếu hoá đơn | 02, 03, 04, 19 |
| **Nhân viên kho (Warehouse Staff)** | Nhận hàng, cất hàng, lấy hàng (kể cả cấp phôi cho xưởng), đóng gói, chuyển kho, đếm kiểm kê | 03, 05, 07, 08, 09, 10, 11, 20 |
| **Quản lý kho (Warehouse Manager)** | Giám sát vận hành, duyệt điều chỉnh tồn, duyệt chuyển kho, điều phối xưởng in, quyết định đưa gia công ngoài | 05, 06, 10, 11, 19, 20 |
| **Nhân viên sản xuất (Production Staff)** | Tại xưởng nội bộ: kiểm file in (prepress), in theo lệnh sản xuất (mẫu và đơn hàng), báo số lượng in được / phế phẩm kèm lý do, treo lệnh | 19 |
| **Nhân viên kế hoạch tồn kho (Inventory Planner)** | Hoạch định nhu cầu nhập hàng, cân bằng tồn giữa các kho, tạo lệnh chuyển kho, lập kiểm kê, theo dõi cảnh báo tồn | 02, 10, 20 |
| **Nhân viên QC (QC Staff)** | Kiểm tra chất lượng hàng nhập, mẫu và thành phẩm in; duyệt kỹ thuật bản thiết kế trước khi gửi khách | 03, 12, 19 |
| **Kế toán (Accountant)** | Đối chiếu ba chiều, công nợ NCC, ghi nhận & phân bổ tiền về, công nợ phải thu của khách sỉ, duyệt tín dụng & điều khoản thanh toán | 04, 14, 15, 17, 18 |
| **Quản trị bán hàng (E-commerce Admin)** | Quản lý catalog, bảng giá theo khách, nội dung; lập & phân công bản thiết kế cho khách; trạng thái & nhóm khách | 12, 13, 14, 18 |
| **Nhân viên Sale (Sales Staff)** | Tạo khách sỉ, thiết kế (tải bộ file) & gửi khách xác nhận, báo giá, yêu cầu làm mẫu, tạo đơn từ báo giá, tư vấn qua chat | 12, 14, 16, 17, 18, 19 |
| **Điều phối đơn hàng (Order Coordinator)** | Release đơn cho kho, xử lý đơn treo, chọn kho xuất | 17 |
| **Khách hàng sỉ (B2B Customer)** | Doanh nghiệp đã liên hệ trước: xác nhận / yêu cầu sửa thiết kế, xem catalog & giá riêng, chấp nhận báo giá, duyệt mẫu, đặt hàng, theo dõi đơn & công nợ | 12, 13, 14, 15, 16, 19 |

---

## 4. Tài liệu liên quan

- [Tổng quan hệ thống & sơ đồ luồng tổng thể](00-system-overview/README.md)
- [Sơ đồ luồng & trạng thái (Mermaid)](diagrams/README.md)
- [Bảng thuật ngữ (Glossary)](glossary/README.md)
- [Câu hỏi mở (Open Questions)](open-questions/README.md)
