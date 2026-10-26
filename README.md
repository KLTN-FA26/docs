# StockFlowCommerce

Nền tảng hợp nhất **Quản lý kho vận (WMS)** và **Sàn thương mại điện tử** cho doanh nghiệp chuyên cung cấp **ly và bao bì dùng một lần cho ngành đồ uống** — bán hàng trơn hoặc in logo theo thiết kế, bán sỉ (B2B) cho khách đã liên hệ trước (chủ yếu là quán / chuỗi đồ uống), có xưởng in nội bộ và thuê gia công ngoài khi thiếu công suất.

## Tài liệu sản phẩm

Toàn bộ tài liệu nghiệp vụ (Business Requirement Document) nằm trong thư mục [`docs/`](docs/README.md):

- [Tổng quan hệ thống](docs/00-system-overview/README.md) — mục tiêu, đối tượng người dùng, sơ đồ luồng tổng thể
- [Mảng Kho vận](docs/warehouse/) — Product Creation → PO → Receipt → Invoice → Putaway → Slotting → Picking → Packing → Shipping → Transfer, và [Sản xuất in](docs/warehouse/19-production/README.md) (mẫu + đơn hàng, tại xưởng nội bộ hoặc gia công ngoài một phần), và [Kiểm soát tồn kho](docs/warehouse/20-inventory-control/README.md) (kiểm kê, điều chỉnh tồn, cảnh báo, truy xuất lô)
- [Mảng Bán hàng sỉ](docs/ecommerce/) — Product Design → Catalog → Cart & Order → Payment → Sales Chat → Order Management → Customer Information
- [Sơ đồ luồng & trạng thái (Mermaid)](docs/diagrams/README.md)
- [Bảng thuật ngữ](docs/glossary/README.md)
- [Câu hỏi mở cần xác nhận](docs/open-questions/README.md)

Bắt đầu đọc từ [`docs/README.md`](docs/README.md).
