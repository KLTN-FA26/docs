# 00 — Tổng quan hệ thống (System Overview)

> **Đã chốt 2026-11-04 (review nghiệp vụ — thu hẹp phạm vi):** kho chỉ là **kho thành phẩm**; **sản xuất nằm ngoài hệ thống** (hệ thống chỉ gửi **Đơn sản xuất** và nhập kho thành phẩm khi sản xuất xong); kho **không kiểm tra chất lượng (QC)** — chỉ đếm nhận đủ / thiếu / thừa; không còn mua hàng từ nhà cung cấp (bỏ PO, hoá đơn NCC); **sàn bán hàng** chỉ để khách xem hàng, xem mẫu thiết kế, chat và chốt đơn với Sale; mỗi đơn có một **hợp đồng mua bán**, khách **thanh toán theo đợt của hợp đồng** (chuyển khoản, không có QR / cổng thanh toán online). Đã xoá các module 02 · Purchase Order, 04 · Invoice, 19 · Production.
>
> **Đã chốt 2026-11-04 (bổ sung):** **không còn hàng trơn** — kho chỉ chứa thành phẩm in theo mẫu của khách, do đơn vị sản xuất **đóng thùng sẵn** rồi nhập vào; Đơn sản xuất chỉ phát sinh từ đơn hàng; kho **chỉ xuất nguyên thùng** và không đóng lại hàng (08 thu gọn thành kiểm thùng & dán nhãn giao hàng).
>
> **Đã chốt 2026-11-05:** khách sỉ **đặt theo số thùng**, không đặt lẻ; giá vẫn theo cái; mọi thùng đúng số cái chuẩn của SKU (thùng lệch không nhận).
>
> **Đã chốt 2026-11-05 (bổ sung):** bỏ role **Inventory Planner** và **Điều phối đơn hàng** (việc chuyển cho Sale và Quản lý kho); đổi **Quản trị bán hàng (E-commerce Admin)** thành **Quản lý kinh doanh (Sales Manager)**; "Yêu cầu sản xuất" đổi thành **Đơn sản xuất (Production Order)** — một chứng từ chung cho xưởng của công ty và đối tác, Sale lập, Sales Manager duyệt và gửi, có bước **khách duyệt mẫu in**; **bỏ module 10 · chuyển kho** — mỗi đơn xuất từ một kho, Đơn sản xuất giao thẳng về kho xuất, kho theo dõi **sức chứa (m³)** gồm hàng đang có và hàng sắp về; **bỏ hạn dùng** (ly và bao bì không có hạn dùng); sàn **ẩn giá**, bảng giá chỉ dùng nội bộ; thêm luồng **nhập hàng hoàn** vào khu cách ly.

## 1. Mục tiêu hệ thống (Goals)

StockFlowCommerce hướng đến việc **loại bỏ khoảng cách giữa vận hành kho thành phẩm và bán hàng sỉ** cho doanh nghiệp chuyên cung cấp **ly và bao bì dùng một lần cho ngành đồ uống** (ly nhựa, ly giấy, bao bì mang đi), **in logo theo mẫu của khách** (không bán hàng trơn), và **chỉ bán sỉ (B2B)** cho khách đã liên hệ từ trước — chủ yếu là quán / chuỗi đồ uống:

| # | Mục tiêu | Diễn giải |
|---|----------|-----------|
| G1 | **Một nguồn dữ liệu sản phẩm duy nhất (single source of truth)** | Sản phẩm khai báo một lần ở back-office, dùng chung cho kho thành phẩm và sàn bán hàng. Mỗi mẫu đã chốt của khách là một **SKU thành phẩm riêng** của khách đó — loại hàng duy nhất có tồn trong kho. |
| G2 | **Tồn kho thành phẩm thời gian thực (real-time inventory)** | Mọi nghiệp vụ nhập – xuất – điều chỉnh cập nhật tồn ngay; khi nhận đơn, hệ thống cho biết đủ hàng hay phải gửi Đơn sản xuất, và kho nhận còn đủ chỗ (m³) hay không. |
| G3 | **Truy vết đầu-cuối (end-to-end traceability)** | Từ Đơn sản xuất → lô hàng nhập kho → vị trí lưu kho → đơn hàng & hợp đồng → kiện hàng giao đi → đợt thanh toán, tất cả liên kết bằng chứng từ. |
| G4 | **Tối ưu thao tác kho** | Kho Map 2D/3D + gợi ý vị trí (slotting) + khu chờ xuất (Staging) giúp giảm quãng đường di chuyển và sai sót. |
| G5 | **Quản lý mẫu & thiết kế của khách** | Doanh nghiệp đưa mẫu → mẫu lên hệ thống → khách xác nhận trên sàn; mẫu đã chốt sinh SKU thành phẩm và đi kèm Đơn sản xuất gửi đơn vị sản xuất. |
| G6 | **Kết nối Sale – Khách hàng** | Kênh chat trên sàn, gắn ngữ cảnh đơn hàng / mẫu thiết kế / báo giá, để tư vấn và chốt đơn ([16](../ecommerce/16-sales-chat/README.md)). |
| G7 | **Bán sỉ theo hợp đồng, kiểm soát công nợ** | Giá do Sale thoả thuận trên báo giá (sàn không hiện giá; bảng giá riêng chỉ dùng nội bộ làm giá gợi ý); mỗi đơn có hợp đồng mua bán với lịch thanh toán theo đợt; theo dõi công nợ phải thu có hạn mức & kỳ hạn. |

---

## 2. Vai trò & đối tượng người dùng (Role Registry)

Đây là **sổ đăng ký vai trò chuẩn** — nguồn tham chiếu duy nhất về tên vai trò trong toàn bộ tài liệu. Mọi module chỉ dùng tên trong danh sách này.

> ✅ Đã chốt 2026-11-04: bỏ ba role **Nhân viên mua hàng (Procurement Staff)**, **Nhân viên sản xuất (Production Staff)** và **Nhân viên QC (QC Staff)**. Việc duyệt kỹ thuật bộ file thiết kế chuyển cho **Quản lý kinh doanh** ([12](../ecommerce/12-product-design/README.md)).
>
> ✅ Đã chốt 2026-11-05: bỏ thêm hai role **Nhân viên kế hoạch tồn kho (Inventory Planner)** và **Điều phối đơn hàng (Order Coordinator)**. Việc phía khách và sản xuất (kho xuất, Đơn sản xuất, treo đơn, cách giao, yêu cầu huỷ) chuyển cho **Sale**; việc trong kho (release, task lấy hàng, khu chờ xuất, kết quả giao, kiểm kê) chuyển cho **Quản lý kho**. Role "Quản trị bán hàng (E-commerce Admin)" đổi tên thành **Quản lý kinh doanh (Sales Manager)**.

### 2.1. Vai trò nội bộ (role đăng nhập)

#### Nhóm kho vận (back-office)

| Role | Trách nhiệm chính | Quyền hạn tiêu biểu |
|------|-------------------|---------------------|
| **Quản trị hệ thống (System Admin)** | Cấu hình toàn hệ thống, khai báo kho & khu vực, phân quyền, quản lý dữ liệu gốc sản phẩm; **tạm ngưng / tiếp tục / ngừng kinh doanh sản phẩm** ([01](../warehouse/01-product-creation/README.md)) | Toàn quyền; cấu hình quy tắc slotting, quy tắc gợi ý |
| **Nhân viên kho (Warehouse Staff)** | Nhập kho thành phẩm theo thùng, kiểm đếm, cất hàng (putaway), lấy hàng nguyên thùng (picking), đưa hàng về khu chờ xuất, kiểm thùng & dán nhãn giao hàng, bàn giao vận chuyển (shipping), nhập hàng hoàn, đếm kiểm kê | Thao tác trên phiếu được phân công; ghi nhận số lượng thực tế |
| **Quản lý kho (Warehouse Manager)** | Quản lý **các kho được gán** (một hoặc nhiều kho). **Thiết lập:** kho, bản đồ, vị trí (thể tích m³, tải trọng), danh mục hãng vận chuyển & loại xe, quy cách thùng; theo dõi **bảng sức chứa kho** ([06](../warehouse/06-warehouse-map-slotting/README.md)). **Nhập kho:** xử lý nhận thiếu / thừa / thùng lệch số chuẩn; quyết định hàng hoàn trong khu cách ly ([03](../warehouse/03-receipt/README.md)). **Xuất kho:** release đơn, tạo task lấy hàng & giao người, khu chờ xuất, chuyển task sang khu kiểm & dán nhãn, huỷ chuyến bàn giao, ghi nhận kết quả giao ([07](../warehouse/07-picking/README.md), [09](../warehouse/09-shipping/README.md)). **Kiểm soát tồn:** lập kiểm kê, **duyệt điều chỉnh tồn**, cách ly / gỡ cách ly ([20](../warehouse/20-inventory-control/README.md)) | Duyệt / từ chối điều chỉnh tồn (người duyệt khác người gửi; kho chỉ có một quản lý thì System Admin duyệt); ghi đè gợi ý slotting; có đủ quyền thao tác như nhân viên kho — đó là quyền, không phải việc chính |
| **Kế toán (Accountant)** | **Ghi nhận & phân bổ tiền khách chuyển khoản** cho các **đợt thanh toán của hợp đồng**, **công nợ phải thu của khách sỉ**, hoàn tiền; **duyệt / từ chối đơn bị treo do công nợ** và **duyệt hạn mức, kỳ hạn công nợ, lịch thanh toán mặc định của khách** ([15](../ecommerce/15-payment/README.md)) | Ghi nhận, phân bổ, đảo phân bổ tiền về; duyệt tín dụng & hạn mức; chạy đối soát; **xem chi tiết & lịch sử đơn, hợp đồng (chỉ đọc)** *(Quyết định 21/10/2026)* |

#### Nhóm bán hàng sỉ (sales)

| Role | Trách nhiệm chính | Quyền hạn tiêu biểu |
|------|-------------------|---------------------|
| **Quản lý kinh doanh (Sales Manager)** | Quản lý **dữ liệu sản phẩm** — tạo, **duyệt (bốn mắt)**, đăng ảnh, danh mục, thương hiệu, thuộc tính ([01](../warehouse/01-product-creation/README.md)); quản lý nội dung trên sàn bán hàng, **bảng giá nội bộ** theo khách (giá gợi ý cho Sale, không hiện trên sàn); lập bản thiết kế cho khách, phân công người thiết kế và **duyệt kỹ thuật bộ file thiết kế** ([12](../ecommerce/12-product-design/README.md)); **đổi trạng thái khách** và quản lý **nhóm khách** ([18](../ecommerce/18-customer-information/README.md)); **duyệt và gửi Đơn sản xuất** ([17](../ecommerce/17-order-management/README.md)); **quản trị tài khoản nhân viên & phân quyền** trong giới hạn quyền mình có (§2.5) | Duyệt / từ chối sản phẩm; duyệt / trả lại Đơn sản xuất (không phải người lập); xuất bản / ẩn sản phẩm; lập & đổi phân công bản thiết kế; duyệt Đạt / Không đạt bộ file thiết kế (không phải người thiết kế); chặn / gỡ chặn / đóng / mở lại khách |
| **Nhân viên Sale (Sales Staff)** | Tạo & gửi duyệt sản phẩm ([01](../warehouse/01-product-creation/README.md)), tạo hồ sơ khách sỉ, **ghi nhận mẫu khách đưa**, thiết kế (tải bộ file) & gửi khách xác nhận ([12](../ecommerce/12-product-design/README.md)), lập **báo giá**, tạo **đơn hàng** từ báo giá đã chấp nhận, lập & gửi **hợp đồng mua bán**, tải bản hợp đồng đã ký ([14](../ecommerce/14-cart-and-order/README.md)), **chat với khách** ([16](../ecommerce/16-sales-chat/README.md)); với đơn đã `Confirmed`: **chọn kho xuất**, **lập & theo dõi Đơn sản xuất** cho phần thiếu (ghi nhận bên sản xuất nhận đơn, tiến độ, **mẫu in** cho khách duyệt), **treo / gỡ treo đơn**, **chọn cách giao** (gửi kiện / nguyên chuyến), **duyệt / từ chối yêu cầu huỷ**, **quyết định cho phép giao một phần** ([17](../ecommerce/17-order-management/README.md)), xử lý khiếu nại | Tạo / sửa sản phẩm nháp; xem & thao tác khách và đơn được phân công; tạo báo giá, đơn, hợp đồng, Đơn sản xuất; huỷ đơn / đóng phần còn lại; giảm giá trong hạn mức; **bật / tắt cờ cho phép giao một phần** của đơn *(Quyết định 21/10/2026)* |

### 2.2. Vai thao tác (operational hat — không phải role riêng)

Warehouse Staff được gọi theo công đoạn được phân công. Đây chỉ là **cách gọi theo ngữ cảnh**, không phải role đăng nhập riêng:

| Vai thao tác | Công đoạn | Module |
|--------------|-----------|--------|
| Receiver | Nhập kho thành phẩm theo Đơn sản xuất, nhập hàng hoàn vào khu cách ly, kiểm đếm số thùng | [03](../warehouse/03-receipt/README.md) |
| Putaway Operator | Cất hàng vào vị trí | [05](../warehouse/05-putaway/README.md) |
| Picker | Lấy nguyên thùng theo task được giao (mỗi đơn một task), đưa về khu chờ xuất hoặc khu kiểm thùng, báo thiếu, cất lại hàng của đơn huỷ | [07](../warehouse/07-picking/README.md) |
| Packer | Quét kiểm đủ thùng cho đơn, dán nhãn giao hàng, gom thùng thành shipment (không đóng lại hàng) | [08](../warehouse/08-packing/README.md) |
| Dispatch Staff | Mở chuyến bàn giao, quét kiện, ký bàn giao; lập kế hoạch xếp hàng lên xe (Load Planning) khi giao nguyên chuyến, ghi nhận đặt xe, xếp kiện lên xe | [09](../warehouse/09-shipping/README.md) |

### 2.3. Quyền theo ngữ cảnh (permission — không phải role)

Một số chức năng là **quyền** gán cho role qua cấu hình RBAC, không phải chức danh riêng. Trong tài liệu, "Người duyệt (Approver)" là cách gọi tắt của *người giữ quyền phê duyệt tương ứng*.

> **Quyết định 22/10/2026:** mọi kiểm tra truy cập trong hệ thống đi qua **mã quyền**, không chức năng nào bị khoá cứng theo tên vai trò. Cột "Gán mặc định cho" dưới đây (và "Quyền hạn tiêu biểu" ở §2.1) chỉ là cấu hình ban đầu: System Admin cấp một quyền cho vai trò khác trên màn phân quyền thì chức năng đó mở cho vai trò ấy, không cần sửa mã. Phạm vi dữ liệu (theo kho, đơn của chính khách) vẫn là điều kiện riêng, không phải kiểm tra vai trò.

| Quyền | Nội dung | Gán mặc định cho |
|-------|----------|------------------|
| Quyết định giao một phần | Bật / tắt cờ cho phép giao một phần của đơn ([17](../ecommerce/17-order-management/README.md) §4.2) *(Quyết định 21/10/2026)* | Sales Staff, System Admin |
| Duyệt kỹ thuật thiết kế | Duyệt Đạt / Không đạt bộ file thiết kế trước khi gửi khách; người duyệt khác người thiết kế ([12](../ecommerce/12-product-design/README.md)) *(Đã chốt 2026-11-04)* | Quản lý kinh doanh |
| Duyệt Đơn sản xuất | Duyệt / trả lại Đơn sản xuất trước khi gửi đơn vị sản xuất; người duyệt khác người lập ([17](../ecommerce/17-order-management/README.md)) *(Đã chốt 2026-11-05)* | Quản lý kinh doanh |
| Huỷ có kiểm soát kế hoạch xe | Huỷ có kiểm soát cả kế hoạch chuyến xe hoặc một xe đã bắt đầu xếp; bỏ xe `Planned` khỏi kế hoạch khi mọi xe khác đã rời kho; bắt buộc mã lý do + ghi chú ([09](../warehouse/09-shipping/README.md) BR-14) *(Quyết định 02/11/2026)* | Quản lý kho, System Admin |
| Duyệt điều chỉnh tồn | Duyệt / từ chối điều chỉnh tồn; người duyệt khác người gửi ([20](../warehouse/20-inventory-control/README.md)) | Quản lý kho (System Admin khi kho chỉ có một quản lý) |
| Duyệt tín dụng & lịch thanh toán | Duyệt / từ chối đơn bị treo do công nợ (vượt hạn mức, khách quá hạn); duyệt hạn mức, kỳ hạn công nợ, lịch thanh toán mặc định của khách | Kế toán |
| Hoàn tiền | Tạo lệnh hoàn tiền dư cho khách; xử lý lệnh hoàn khi huỷ / đóng phần còn lại | Kế toán (System Admin có toàn quyền) |
| Giám sát chat (Chat Supervision) | Phân công hội thoại, giám sát SLA, nhận escalate | Quản lý kinh doanh hoặc Sales Staff được chỉ định |

Phân bổ trên là cấu hình mặc định khi cài đặt; người có quyền duyệt phân quyền sửa được ma trận quyền của từng role (§2.5.3).

### 2.4. Đối tác ngoài & actor hệ thống

| Actor | Vai trò |
|-------|---------|
| **Khách hàng sỉ (B2B Customer)** | Doanh nghiệp đã liên hệ trước, được Sale tạo tài khoản; **đưa mẫu** (logo, file, mẫu vật) cho Sale. Trên sàn bán hàng: xem hàng (**sàn không hiện giá**), xem và **xác nhận / yêu cầu sửa thiết kế**, **duyệt / yêu cầu in lại mẫu in**, **chat với Sale**, chấp nhận báo giá, xem đơn, hợp đồng, đợt thanh toán & công nợ. **Không tự đặt hàng** trên sàn — đơn do Sale chốt. **Không có khách vãng lai (guest).** |
| **Đơn vị sản xuất (ngoài hệ thống)** | Xưởng của công ty **hoặc** đối tác bên ngoài — hệ thống xử lý như nhau, chỉ khác **kênh kết nối** khai báo trong danh mục Đơn vị sản xuất. Nhận **Đơn sản xuất** (kèm bộ file thiết kế đã chốt), xác nhận nhận đơn, in **mẫu thử** khi đơn yêu cầu duyệt mẫu in, sản xuất và tự kiểm tra chất lượng, đóng thùng đúng số chuẩn, giao về **đúng kho nhận**. Hệ thống không quản lý quá trình sản xuất; Sale ghi nhận phản hồi và tiến độ ([17](../ecommerce/17-order-management/README.md)) |
| **Đơn vị vận chuyển (Carrier / 3PL)** | Bên thứ ba **ngoài hệ thống**: nhận kiện khi bàn giao, vận chuyển, giao cho khách, hoàn hàng về kho (RTO); cung cấp xe tải / container cho giao nguyên chuyến. Hệ thống không tích hợp API và không theo dõi hành trình; nhân viên cập nhật kết quả giao khi hãng báo lại |

Ngoài ra, nhiều module liệt kê **actor hệ thống** — các engine tự động (gợi ý bin khi cất hàng, Wave/Route Planner và Cartonization (phạm vi giai đoạn sau — [open-questions C17](../open-questions/README.md)), Load Planner, Payment Service, Notification (hộp thông báo — §6)…). Đây không phải người dùng; chúng thực thi quy tắc do các role trên cấu hình.

Một tài khoản có thể mang nhiều role (ví dụ Warehouse Staff kiêm Warehouse Manager ở kho nhỏ): quyền là hợp các role, phạm vi dữ liệu là phạm vi rộng nhất. Chi tiết tài khoản và phân quyền ở §2.5.

### 2.5. Tài khoản & phân quyền (Accounts & RBAC)

#### 2.5.1. Tài khoản nhân viên

| Trường | Ghi chú |
|--------|---------|
| Tên đăng nhập | Không đổi được |
| Email, họ tên | Sửa được |
| Role | Một hoặc nhiều role (§2.1) |
| Kho được gán | Một hoặc nhiều kho; quyết định phạm vi dữ liệu với role theo kho (§2.5.3) |
| Trạng thái | `Active` / `Locked` / `Disabled` |

1. **Tạo tài khoản** — người quản trị nhập thông tin, role, kho. Không nhập mật khẩu thì hệ thống sinh **mật khẩu tạm 16 ký tự**, hiện **một lần**; tài khoản bị đánh dấu **phải đổi mật khẩu**.
2. **Mật khẩu** — 10–72 ký tự, có ít nhất một chữ hoa, một chữ thường, một chữ số.
   - Người dùng tự đổi mật khẩu (phải nhập đúng mật khẩu cũ, mật khẩu mới khác mật khẩu cũ); tuỳ chọn đăng xuất các thiết bị khác.
   - Người quản trị đặt lại mật khẩu (nhập hoặc sinh mật khẩu tạm) → mọi phiên của tài khoản bị huỷ, phải đổi mật khẩu.
3. **Đăng nhập** — sai mật khẩu **5 lần liên tiếp** → **tự khoá 15 phút**, hết hạn tự mở; trong lúc khoá, hệ thống trả lời giống hệt sai mật khẩu. Mỗi địa chỉ IP tối đa 5 lần đăng nhập / phút.
4. **Phiên đăng nhập** — mỗi lần đăng nhập là một phiên, hiệu lực **8 giờ** (không có làm mới phiên); hệ thống kiểm phiên ở mọi thao tác. Người dùng xem được các phiên của mình, đăng xuất, đăng xuất các thiết bị khác.
5. **Phiên bị huỷ ngay** khi: đổi role của tài khoản, khoá, vô hiệu hoá, người quản trị đặt lại mật khẩu, hoặc người dùng đổi mật khẩu và chọn đăng xuất thiết bị khác.
6. **Đổi kho được gán** — có hiệu lực từ thao tác kế tiếp, không cần đăng nhập lại.

| Trạng thái | Ý nghĩa | Chuyển tiếp cho phép | Ai thực hiện |
|------------|---------|----------------------|--------------|
| `Active` | Đăng nhập được (trừ khi đang tự khoá 15 phút do sai mật khẩu) | → `Locked` (khoá), → `Disabled` (vô hiệu hoá) | Người quản trị tài khoản |
| `Locked` | Bị người quản trị khoá; chỉ **mở khoá** mới hết | → `Active` (mở khoá — cũng xoá khoá tự động), → `Disabled` | Người quản trị tài khoản |
| `Disabled` | Ngừng dùng; tài khoản được giữ để tra lịch sử | → `Active` (kích hoạt lại) | Người quản trị tài khoản |

```mermaid
stateDiagram-v2
    [*] --> Active: Tạo tài khoản
    Active --> Locked: Khoá
    Locked --> Active: Mở khoá
    Active --> Disabled: Vô hiệu hoá
    Locked --> Disabled: Vô hiệu hoá
    Disabled --> Active: Kích hoạt lại
```

**Ràng buộc với người quản trị tài khoản:**

- Chỉ **cấp được** quyền, role (hoặc bản sao role) mà **chính mình đang có đủ**.
- Chỉ **quản lý được** tài khoản không có quyền nào vượt mình — vì vậy chỉ System Admin cấp được role System Admin.
- **Không tự quản lý tài khoản của mình** (khoá, đổi role… cho chính mình).
- Luôn giữ **ít nhất một System Admin đang hoạt động**: không khoá, vô hiệu hoá hay thu role của System Admin cuối cùng.
- Luôn có **ít nhất một role giữ quyền sửa phân quyền**.
- Role khách hàng (`CUSTOMER`) không gán cho tài khoản nhân viên; tài khoản khách không mang role khác.

> ⚠️ Giả định: **quên mật khẩu tự phục vụ qua email**, **xác thực hai lớp (2FA)** và **làm mới phiên** thuộc phạm vi giai đoạn sau. Hiện người quên mật khẩu nhờ người quản trị đặt lại. Xem [open-questions A10](../open-questions/README.md).

#### 2.5.2. Tài khoản khách hàng sỉ

1. Khách **không tự đăng ký**. Sale tạo hồ sơ khách ([18](../ecommerce/18-customer-information/README.md)) → hệ thống tạo tài khoản cho người liên hệ (chưa có mật khẩu) và gửi **email mời**.
2. Link mời hiệu lực **72 giờ**; Sale gửi lại được sau tối thiểu 1 phút.
3. Người liên hệ mở link, **đặt mật khẩu** → kích hoạt. Tên đăng nhập là **email**.
4. Trạng thái hồ sơ khách quyết định đăng nhập: `Active` / `Pending Activation` đăng nhập được; `Blocked` / `Inactive` → tài khoản bị vô hiệu hoá, mọi phiên bị huỷ ([18](../ecommerce/18-customer-information/README.md)).

#### 2.5.3. Role, quyền và phạm vi dữ liệu

1. **Quyền** = **tài nguyên × hành động**, ví dụ `sales-contracts:APPROVE`. Hành động gồm: xem trang (`VIEW_PAGE`), đọc (`READ`), tạo (`CREATE`), sửa (`UPDATE`), xoá (`DELETE`), duyệt (`APPROVE`), xuất dữ liệu (`EXPORT`). Ba hành động cuối là **nhạy cảm**.
2. **Role** là tập quyền kèm **phạm vi dữ liệu**:

   | Phạm vi | Ý nghĩa | Role mặc định |
   |---------|---------|---------------|
   | `WAREHOUSE` | Chỉ dữ liệu của **các kho được gán** cho tài khoản; không được gán kho thì không thấy gì | Warehouse Staff, Warehouse Manager |
   | `ALL` | Toàn hệ thống | Mọi role nhân viên khác |
   | `OWN` | Chỉ dữ liệu của chính mình | Dùng cho các chức năng của khách trên cổng (đơn, công nợ, báo giá của chính khách) |

   Phạm vi theo nhóm (`TEAM`) chưa dùng.
3. **7 role hệ thống**: 6 role nhân viên ở §2.1 và role **Khách hàng** (`CUSTOMER`). Role hệ thống không xoá, không đổi tên.
4. **Role tự tạo** — người có quyền tạo role tạo được role mới (tuỳ chọn sao chép từ role khác), đổi tên, đổi phạm vi; xoá được khi không còn ai mang role đó.
5. **Sửa ma trận quyền** của role (thay cả ma trận, hoặc cấp / thu từng quyền) cần quyền **duyệt phân quyền**. Ma trận của `CUSTOMER` và `SYSTEM_ADMIN` cố định. Thay đổi có hiệu lực từ thao tác kế tiếp của người mang role, không cần đăng nhập lại.
6. **Nhiều role** — một tài khoản mang nhiều role thì quyền là **hợp** các role, phạm vi là phạm vi **rộng nhất**.
7. **Ai quản trị tài khoản & phân quyền** — System Admin (toàn quyền) và **Quản lý kinh doanh** (tạo / sửa tài khoản, role, ma trận quyền — trong giới hạn quyền mình có). Chỉ System Admin xoá được role.

---

## 3. Bản đồ module & mối liên kết (Module Map)

Theo năm nhóm chức năng đã chốt 2026-11-04:

| Nhóm chức năng | Module |
|----------------|--------|
| **Quản lý thiết kế** (mẫu của khách) | [12](../ecommerce/12-product-design/README.md) |
| **Bán hàng** — sàn bán hàng, bảng giá nội bộ, báo giá, đơn hàng theo số lượng, hợp đồng từng đơn, chat, khách hàng | [13](../ecommerce/13-catalog/README.md), [14](../ecommerce/14-cart-and-order/README.md), [16](../ecommerce/16-sales-chat/README.md), [17](../ecommerce/17-order-management/README.md), [18](../ecommerce/18-customer-information/README.md) |
| **Quản lý thanh toán** — theo hợp đồng, công nợ | [15](../ecommerce/15-payment/README.md) |
| **Quản lý kho** — dữ liệu sản phẩm, nhập kho thành phẩm, vận hành kho, xuất kho, kiểm soát tồn | [01](../warehouse/01-product-creation/README.md), [03](../warehouse/03-receipt/README.md), [05](../warehouse/05-putaway/README.md)–[08](../warehouse/08-packing/README.md), [11](../warehouse/11-intra-warehouse-transfer/README.md), [20](../warehouse/20-inventory-control/README.md) |
| **Quản lý giao hàng** | [09](../warehouse/09-shipping/README.md) |

```mermaid
flowchart TB
    subgraph MASTER["Master Data"]
        PC[01 · Product Creation]
    end

    subgraph WH["Mảng A — Kho thành phẩm (Warehouse / Back-office)"]
        direction TB
        RC[03 · Nhập kho thành phẩm]
        PA[05 · Putaway]
        MAP[06 · Warehouse Map & Slotting]
        PICK[07 · Picking]
        PACK[08 · Kiểm thùng & Dán nhãn]
        SHIP[09 · Shipping]
        INTRA[11 · Intra-warehouse Transfer]
        CTRL[20 · Inventory Control]
    end

    subgraph EC["Mảng B — Bán hàng sỉ (B2B Sales / Sàn bán hàng)"]
        direction TB
        DSN[12 · Mẫu & Thiết kế]
        CAT[13 · Catalog]
        CO[14 · Báo giá, Đơn hàng & Hợp đồng]
        PAY[15 · Thanh toán theo hợp đồng]
        CHAT[16 · Sales Chat]
        OM[17 · Order Management]
        CIM[18 · Customer Information]
    end

    PRODEXT[[Đơn vị sản xuất\n— ngoài hệ thống]]
    INVENTORY[(Tồn kho thành phẩm\n— theo Inventory Item & vị trí,\ntổng hợp theo kho)]

    DSN -- mẫu đã chốt sinh SKU --> PC
    PC --> CAT
    CAT --> CO
    DSN --> CO
    CHAT -. hỗ trợ .-> CO
    CHAT -. hỗ trợ .-> OM
    CIM --> CO
    CIM --> CHAT
    CO -- hợp đồng & đợt thanh toán --> PAY
    CO --> OM
    PAY -. đủ điều kiện theo hợp đồng .-> OM

    OM -- Đơn sản xuất --> PRODEXT
    PRODEXT -- thành phẩm --> RC
    RC --> PA
    PA --> MAP
    MAP -. gợi ý vị trí .-> PA
    PA --> INVENTORY
    MAP -. sức chứa kho m³ .-> OM

    OM --> PICK
    INVENTORY --> PICK
    PICK -- khu chờ xuất / khu kiểm thùng --> PACK
    PACK --> SHIP
    PAY -. chặn bàn giao nếu đợt trước giao chưa thu .-> SHIP
    SHIP --> OM

    INTRA <--> INVENTORY
    CTRL <-->|kiểm kê, điều chỉnh, cách ly| INVENTORY
    MAP -. gợi ý vị trí .-> INTRA
```

---

## 4. Sơ đồ luồng tổng thể end-to-end (Mẫu → Giao hàng → Công nợ)

Luồng "hạnh phúc" (happy path) từ lúc doanh nghiệp đưa mẫu đến lúc giao hàng và thu đủ tiền:

```mermaid
flowchart LR
    A[Khách đưa mẫu\nquản lý mẫu & thiết kế] --> A1[Khách xác nhận\nthiết kế trên sàn]
    A1 --> A2[Sinh SKU thành phẩm\ncủa khách]
    A2 --> B[Bảng giá / Báo giá]
    I[Sàn bán hàng\nxem hàng · chat với Sale] --> B
    B --> C[Đơn hàng\ntheo số lượng]
    C --> D[Hợp đồng mua bán\nký & tải lên]
    D --> D1{Đợt thu khi ký\nđã thu?}
    D1 -- Có / không có đợt này --> E[Đơn xác nhận\nConfirmed]
    E --> F{Đủ tồn\nthành phẩm?}
    F -- Thiếu --> G[Đơn sản xuất\nSale lập · Sales Manager duyệt & gửi]
    G --> G1{Cần duyệt\nmẫu in?}
    G1 -- Có --> G2[Khách duyệt\nmẫu in]
    G2 --> G3[Sản xuất\nhàng loạt]
    G1 -- Không --> G3
    G3 --> H[Nhập kho thành phẩm\nđã đóng thùng · đếm đủ / thiếu]
    H --> H1[Cất hàng\nPutaway]
    H1 --> F
    F -- Đủ --> K[Lấy nguyên thùng\nPicking]
    K --> K1[Khu chờ xuất\nnếu cần giữ hàng]
    K1 --> L[Kiểm thùng & dán nhãn\ngiao hàng]
    K --> L
    L --> M[Bàn giao vận chuyển\nShipping]
    M --> N[Giao thành công\nDelivered]
    N --> O[Công nợ phải thu\nthu các đợt còn lại]
    O --> P[Đóng đơn\nOrder Closed]
```

**Diễn giải theo từng chặng:**

| Chặng | Bắt đầu bằng | Kết thúc bằng | Module chịu trách nhiệm |
|-------|--------------|---------------|-------------------------|
| **1. Mẫu & thiết kế** | Doanh nghiệp đưa mẫu cho Sale | Khách xác nhận thiết kế; SKU thành phẩm của khách được tạo | 18 → 12 → 01 (hỗ trợ: 16) |
| **2. Bán hàng & hợp đồng** | Khách xem hàng / chat với Sale | Hợp đồng đã ký, đơn `Confirmed` | 13 → 14 → 15 (hỗ trợ: 16, 18) |
| **3. Đáp ứng tồn (Đơn sản xuất & nhập kho)** | Đơn thiếu tồn thành phẩm tại kho xuất (SKU riêng của khách) | Thành phẩm nằm ở vị trí kho xuất, sẵn cho đơn | 17 (Đơn sản xuất, duyệt mẫu in) → (đơn vị sản xuất, ngoài hệ thống) → 03 → 05 → 06 |
| **4. Xuất kho & giao hàng (Fulfillment)** | Đơn release cho kho | Kiện hàng đã giao cho khách | 17 → 07 → 08 → 09 |
| **5. Thanh toán & công nợ** | Hợp đồng đã ký | Mọi đợt thanh toán đã thu, đơn đóng | 15 (liên quan 14, 17, 18) |
| **6. Kiểm soát tồn kho (Inventory ops)** | Chênh lệch / tối ưu vị trí; theo dõi sức chứa kho; hàng hoàn | Số tồn khớp thực tế, kho đủ chỗ cho hàng sắp về | 11, 06, 20, 03 |

---

## 5. Các luồng nghiệp vụ chính (Core Business Flows)

| Luồng | Chuỗi module | Chứng từ xuyên suốt | Sơ đồ chi tiết |
|-------|--------------|---------------------|----------------|
| **Nhập kho thành phẩm** | Đơn sản xuất → (đơn vị sản xuất) → Nhập kho (WH/Input) → Putaway (WH/Stock) | Số Đơn sản xuất liên kết phiếu nhập kho, lô hàng & nhiệm vụ cất hàng | [diagrams §1](../diagrams/README.md#1-luồng-nhập-kho-thành-phẩm) |
| **Fulfillment** | Order → kiểm tồn (thiếu → Đơn sản xuất) → Picking nguyên thùng → (khu chờ xuất) → Kiểm thùng & dán nhãn giao hàng → Shipping (gửi kiện qua hãng, hoặc giao nguyên chuyến có lập kế hoạch xếp hàng 3D) | Order number liên kết Đơn sản xuất, pick task, thùng LPN (kiện giao), Shipment, Load Plan (nếu giao nguyên chuyến) | [diagrams §2](../diagrams/README.md#2-luồng-fulfillment) |
| **Nhập hàng hoàn** | Hãng hoàn hàng (RTO) / khách trả → Phiếu nhập loại Hàng hoàn → khu `QUARANTINE` → gỡ cách ly về bin hoặc ghi giảm có duyệt | Shipment / yêu cầu trả liên kết phiếu nhập hàng hoàn | [diagrams §1.3](../diagrams/README.md#1-luồng-nhập-kho-thành-phẩm), [03](../warehouse/03-receipt/README.md) |
| **Bán sỉ theo hợp đồng** | Mẫu & thiết kế → Báo giá → Order → Hợp đồng mua bán → Đợt thanh toán → Công nợ | Design snapshot → Quote number → Order number ↔ Contract number → Payment installment → Payment transaction | [diagrams §3.1](../diagrams/README.md#3-phụ-lục) |

---

## 6. Nguyên tắc dữ liệu xuyên suốt (Cross-cutting Data Principles)

1. **SKU là đơn vị định danh hàng hoá nhỏ nhất** để quản lý tồn (mỗi biến thể sản phẩm = 1 SKU). Mỗi SKU có **đúng một Inventory Item** (hồ sơ hàng vật lý, quan hệ 1:1) — đối tượng được theo dõi tồn; nội dung bán hàng thuộc Product/Variant, không thuộc Inventory Item. Xem [01 · Product Creation](../warehouse/01-product-creation/README.md) và [Glossary](../glossary/README.md).
2. **Tồn kho luôn gắn với vị trí (location-based inventory)**: một số lượng SKU luôn thuộc về một vị trí cụ thể — **bin** trên tầng kệ, hoặc **khu vực chứa hàng (Area)** như khu nhận, cách ly, chờ xuất (Staging), đóng gói, chờ giao (xem [06](../warehouse/06-warehouse-map-slotting/README.md)). Tồn **theo kho** (Inventory Level) và tồn toàn hệ thống là số **tổng hợp** từ tồn theo vị trí, không phải nguồn dữ liệu riêng.
3. **Mọi thay đổi tồn kho phải có chứng từ nguồn (source document)**: phiếu nhập kho thành phẩm hoặc hàng hoàn (Receipt), Pick, Adjustment… Không có nghiệp vụ nào sửa tồn "trần" mà không để lại vết.
4. **Trạng thái chứng từ chỉ tiến, không lùi tự do**: chuyển trạng thái theo sơ đồ vòng đời của từng module; huỷ/hoàn phải là hành động có kiểm soát và ghi log.
5. **Hàng in theo mẫu của khách là SKU thành phẩm riêng**: khi khách xác nhận thiết kế, hệ thống sinh một SKU gắn với khách và bản thiết kế đã chốt (design snapshot, bất biến). Tồn, nhập, xuất đều theo SKU này; snapshot được đính kèm Đơn sản xuất gửi đơn vị sản xuất ([12](../ecommerce/12-product-design/README.md), [17](../ecommerce/17-order-management/README.md)).
6. **Phạm vi tổ chức**: hệ thống phục vụ **một pháp nhân duy nhất** (một công ty, một mã số thuế, một sổ cái) sở hữu **nhiều kho vật lý (multi-warehouse)**. *(Đã chốt 2026-11-05)* **Mỗi đơn bán xuất từ đúng một kho** (Sale chọn khi đơn `Confirmed`); mọi Đơn sản xuất của đơn ghi **kho nhận = kho xuất**, nên **không có chuyển hàng giữa các kho** (đã bỏ module 10). Đơn vẫn giao nhiều đợt từ cùng kho xuất. Dời / đóng kho (hiếm) xử lý bằng điều chỉnh tồn có duyệt. Hệ thống **không** hỗ trợ đa công ty / đa pháp nhân (multi-entity).
7. **Công thức số lượng tồn thống nhất**: **ATP = Tồn khả dụng (`Available`) − Phần đang giữ hàng (reservation)**. Tồn khả dụng chỉ gồm hàng `Available` **không bị giữ** (không có khoản phân bổ cho đơn — kể cả hàng đang giữ ở khu chờ xuất). Hàng `Inbound`, `Allocated`, `Quarantine` không thuộc tồn khả dụng. Chi tiết: [20 · Inventory Control BR-01](../warehouse/20-inventory-control/README.md). Định nghĩa On-hand, Inventory Level, Available, ATP xem [Glossary](../glossary/README.md).
8. **Bán, nhập và xuất theo thùng; giá theo cái** *(Đã chốt 2026-11-05)*: khách sỉ đặt theo **số thùng** (không đặt lẻ); mỗi SKU riêng có **số cái chuẩn / thùng** cố định và mọi thùng nhập về phải đúng số chuẩn. Báo giá, đơn, hợp đồng, Đơn sản xuất, phiếu nhập, phân bổ, kiểm kê đều tính theo số thùng; **đơn giá ghi theo cái**, thành tiền = số thùng × số cái / thùng × đơn giá cái. Nội bộ tồn vẫn quy ra đơn vị cơ sở (cái) để truy vết. Xem [01 · Product Creation](../warehouse/01-product-creation/README.md).
9. **Tiền tố kho (warehouse prefix)**: mỗi kho có một prefix duy nhất, không sửa được sau khi tạo, dùng cho **mã vị trí** (bin: prefix + mã kệ + tầng + mã bin, ví dụ `HN-A01-2-03`; khu vực: prefix + mã khu vực, ví dụ `HN-RCV01`) và **mã chứng từ phát sinh tại kho** (ví dụ `HN-RC-000123`). Xem [06](../warehouse/06-warehouse-map-slotting/README.md).


### Thông báo trong ứng dụng (In-app notifications)

1. Mỗi tài khoản (nhân viên và khách) có **hộp thông báo** riêng. Mỗi thông báo có nhóm, **mức độ** (`INFO` / `WARN` / `CRITICAL`), tiêu đề, nội dung, đường dẫn tới màn hình liên quan, trạng thái đã đọc.
2. Người nhận được xác định **theo role** (và theo kho, nếu có) **tại lúc gửi**; mỗi người một bản. Cùng một sự việc không tạo thông báo trùng cho cùng người.
3. Người dùng xem danh sách (lọc chưa đọc), số chưa đọc, đánh dấu đã đọc từng cái hoặc tất cả. Màn hình **tự hỏi lại số chưa đọc** theo chu kỳ (chưa đẩy thời gian thực).
4. **Thông báo đã đọc tự xoá sau 90 ngày**; thông báo chưa đọc được giữ.

| Sự việc | Mức độ | Người nhận | Kèm email |
|---------|--------|------------|-----------|
| Khách **chấp nhận / yêu cầu sửa** báo giá ([14](../ecommerce/14-cart-and-order/README.md)) | `INFO` | Sale phụ trách báo giá; tài khoản đó ngừng hoạt động thì mọi Sale (đánh dấu bàn giao) | Có |
| **Đơn sản xuất được gửi** về kho (số thùng, thể tích, ngày cần hàng) ([17](../ecommerce/17-order-management/README.md) §4.3) *(Đã chốt 2026-11-05)* | `INFO` | Quản lý kho của kho nhận | Không |
| **Còn trống dự kiến** của kho chuyển sang **âm** do một Đơn sản xuất được gửi ([06](../warehouse/06-warehouse-map-slotting/README.md) §4.6) *(Đã chốt 2026-11-05)* | `WARN` | Quản lý kho của kho nhận | Không |

Email ngoài hộp thông báo: **email mời** người liên hệ của khách (§2.5.2).

> ⚠️ Giả định: **thông báo cho khách** (đơn đã tạo, hợp đồng chờ ký, đợt thanh toán đến hạn, đã bàn giao / đã giao) và **thông báo yêu cầu huỷ mới cho nhân viên** thuộc phạm vi giai đoạn sau — hiện hệ thống chỉ ghi nhật ký, không gửi. Xem [open-questions A10](../open-questions/README.md).

---

## 7. Liên kết

- Chi tiết từng module: xem thư mục [warehouse/](../warehouse/) và [ecommerce/](../ecommerce/)
- Sơ đồ Mermaid đầy đủ: [diagrams/](../diagrams/README.md)
- Thuật ngữ: [glossary/](../glossary/README.md)
