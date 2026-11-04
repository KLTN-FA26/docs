# StockFlowCommerce

Nền tảng hợp nhất **Quản lý kho thành phẩm (WMS)** và **Sàn bán hàng sỉ (B2B)** cho doanh nghiệp chuyên cung cấp **ly và bao bì dùng một lần cho ngành đồ uống** — in logo theo mẫu của khách (không bán hàng trơn), bán sỉ cho khách đã liên hệ trước (chủ yếu là quán / chuỗi đồ uống), theo hợp đồng mua bán từng đơn. Sản xuất nằm ngoài hệ thống: hệ thống gửi Yêu cầu sản xuất và nhập kho thành phẩm khi sản xuất xong.

## Tài liệu sản phẩm

Toàn bộ tài liệu nghiệp vụ (Business Requirement Document) nằm trong thư mục [`docs/`](docs/README.md):

- [Tổng quan hệ thống](docs/00-system-overview/README.md) — mục tiêu, đối tượng người dùng, sơ đồ luồng tổng thể
- [Mảng Kho thành phẩm](docs/warehouse/) — Product Creation → Nhập kho thành phẩm → Putaway → Slotting → Picking → Kiểm thùng & Dán nhãn → Shipping → Transfer, và [Kiểm soát tồn kho](docs/warehouse/20-inventory-control/README.md) (kiểm kê, điều chỉnh tồn, cảnh báo, truy xuất lô)
- [Mảng Bán hàng sỉ](docs/ecommerce/) — Mẫu & Thiết kế → Sàn bán hàng (Catalog) → Báo giá, Đơn hàng & Hợp đồng → Thanh toán theo hợp đồng → Sales Chat → Order Management (kèm Yêu cầu sản xuất) → Customer Information
- [Sơ đồ luồng & trạng thái (Mermaid)](docs/diagrams/README.md)
- [Bảng thuật ngữ](docs/glossary/README.md)
- [Câu hỏi mở cần xác nhận](docs/open-questions/README.md)

Bắt đầu đọc từ [`docs/README.md`](docs/README.md).
