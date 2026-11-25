# 12 — Quản lý mẫu & thiết kế (Design Management)

> ✅ Đã chốt 2026-11-05 (khách tự thiết kế — thay quyết định 2026-11-04 "khách đưa mẫu, nhân viên thiết kế"): **chỉ khách tự thiết kế** trên app, trên **khuôn 2D** của quy cách, có **xem trước 3D**. Bỏ luồng khách đưa mẫu cho Sale, nhân viên thiết kế hộ, phân công người thiết kế. Khi khách gửi duyệt, hệ thống **tự kiểm (preflight)**; **Quản lý kinh doanh (Sales Manager)** duyệt kỹ thuật và kiểm duyệt nội dung *(✅ Chốt lại 2026-11-20: nay Điều phối sản xuất — §1 bước 3)*. Duyệt xong, hệ thống **tự xuất file in PDF/X**, chốt **design snapshot** và sinh **SKU thành phẩm riêng của khách**. **Đặt hàng từ bản thiết kế = yêu cầu báo giá** gửi Sale: khách vẫn không tự đặt, sàn vẫn ẩn giá ([13](../13-catalog/README.md), [14](../14-cart-and-order/README.md), [16](../16-sales-chat/README.md)). Xem [open-questions B18, B7, B8](../../open-questions/README.md).
>
> Đã chốt 2026-11-05: chứng từ gửi sản xuất là **Đơn sản xuất (Production Order)**, gửi cho **đơn vị sản xuất** (xưởng của công ty hoặc đối tác ngoài, ngoài hệ thống). Đơn sản xuất có bước **duyệt mẫu in** ([17](../17-order-management/README.md)): khách duyệt mẫu in thử theo file đã chốt; nếu khách cần **sửa file thiết kế** (không phải lỗi in) thì quay về module này tạo **bản sửa** và sinh **SKU riêng mới** (§4.7).
>
> ✅ Đã chốt 2026-11-06 (góp ý hội đồng): thêm **Dự án thiết kế (Design Project)**. Khách gom nhiều bản thiết kế của **một bộ nhận diện** (ví dụ ly 500ml, ly 700ml, túi giấy cho một chi nhánh mới) vào một dự án. Dự án có **thư viện tài nguyên dùng chung**, cho **gửi duyệt cả bộ** (Quản lý kinh doanh xem cả bộ, quyết định từng bản — *✅ Chốt lại 2026-11-20: nay Điều phối sản xuất*) và **yêu cầu báo giá cả dự án** (một báo giá nhiều dòng). Mỗi bản thiết kế vẫn thuộc một quy cách và đi theo vòng đời hiện có; dự án **không bắt buộc** *(✅ Chốt lại 2026-11-25: nay bắt buộc — BR-23)* (§4.9–§4.11, §5.4). Xem [open-questions B41](../../open-questions/README.md).
>
> ✅ Đã chốt 2026-11-25 *(thay "dự án không bắt buộc" của 2026-11-06)*: **mọi bản thiết kế thuộc đúng một dự án thiết kế** (BR-23). Tạo bản mới, bản sửa, bản nhân bản đều vào một dự án `Active`; không còn "gỡ khỏi dự án", chỉ còn **chuyển** sang một dự án `Active` khác. **Gửi duyệt cả bộ được kèm yêu cầu báo giá dự án** (BR-25): yêu cầu chứa được bản đang chờ duyệt hoặc bị trả lại; Sale chỉ lập báo giá khi **mọi dòng đã `Approved`** và được báo khi yêu cầu **đủ điều kiện báo giá**. Xem [open-questions B41, B42](../../open-questions/README.md).

## 1. Mục đích (Purpose)

Cho **khách sỉ tự thiết kế** bản in (logo, nhận diện thương hiệu của quán đồ uống) lên ly và bao bì, rồi chốt bản thiết kế đó thành **bộ file in** và **SKU thành phẩm riêng** của khách. Luồng gồm các bước:

1. **Thiết kế** — Trên sàn ([13](../13-catalog/README.md)), khách chọn một **quy cách** đã có khuôn in ([01 §4.10](../../warehouse/01-product-creation/README.md)), tải ảnh / logo lên, đặt lên **khuôn 2D**, **xem trước 3D**, lưu và xem lại trong **mẫu thiết kế của tôi**.
2. **Gửi duyệt & kiểm tự động** — Hệ thống **preflight**: độ phân giải ở kích thước in, nội dung trong vùng in / lề an toàn, định dạng và dung lượng file.
3. **Duyệt** — **Điều phối sản xuất** duyệt kỹ thuật và **kiểm duyệt nội dung** (bản quyền, nội dung không phù hợp): **Duyệt** hoặc **Trả lại** kèm lý do.

> ✅ Chốt lại 2026-11-20 *(thay "Quản lý kinh doanh duyệt")*: **Điều phối sản xuất duyệt bản thiết kế** của khách (cả bản lẻ lẫn lượt duyệt dự án, gồm kiểm duyệt nội dung). Điều phối sản xuất là người đứng giữa doanh nghiệp và đơn vị sản xuất — người chịu trách nhiệm file in có in được không — nên duyệt luôn file khách gửi; Quản lý kinh doanh chỉ còn khai báo **cấu hình in** của quy cách ([01 §4.10](../../warehouse/01-product-creation/README.md)). Xem [open-questions B46](../../open-questions/README.md).
4. **Chốt thiết kế** — Hệ thống **xuất bộ file** (file in **PDF/X** theo khuôn, ảnh xem trước, đặc tả kỹ thuật, hướng dẫn sản xuất), chốt **design snapshot** và sinh **SKU thành phẩm riêng** của khách ([01](../../warehouse/01-product-creation/README.md)).
5. **Yêu cầu báo giá** — Từ bản đã duyệt, khách gửi **yêu cầu báo giá** cho Sale qua [chat](../16-sales-chat/README.md); Sale lập báo giá, đơn, hợp đồng ([14](../14-cart-and-order/README.md)).

Mọi bản thiết kế nằm trong một **dự án thiết kế (Design Project)** — bộ nhận diện của khách (BR-23); dự án chỉ có một bản vẫn đi đủ luồng trên. Khi cần **nhiều sản phẩm in cùng một bộ nhận diện**, khách tải logo / ảnh **một lần** vào thư viện của dự án và dùng lại cho mọi bản, **gửi duyệt cả bộ** (có thể kèm **yêu cầu báo giá**) để Điều phối sản xuất kiểm sự đồng bộ, và **yêu cầu báo giá cả dự án** để Sale lập một báo giá nhiều dòng khi mọi bản đã được duyệt (§4.9–§4.11).

Đầu ra:

- **Design snapshot** — bộ file xuất và nội dung thiết kế đã được duyệt, **bất biến**, mỗi file có **mã kiểm tra (checksum SHA-256)**.
- **SKU thành phẩm riêng của khách (customer-exclusive SKU)** — gắn với khách và snapshot; là **loại hàng duy nhất có tồn và được bán** (không có hàng trơn): tồn, nhập, xuất, báo giá, đơn hàng đều theo SKU này.

Snapshot (qua SKU riêng) được dùng ở các bước sau:

- **Báo giá, đơn hàng & hợp đồng:** [14](../14-cart-and-order/README.md).
- **Đơn sản xuất:** [17](../17-order-management/README.md) — bộ file chốt (file in PDF/X) được đính kèm Đơn sản xuất gửi đơn vị sản xuất; mẫu in thử (khi Đơn sản xuất bật cờ duyệt mẫu in) được in theo đúng bộ file này.

Ở mỗi lần gắn snapshot hoặc cấp link tải file, hệ thống kiểm lại checksum để chắc chắn đúng bộ file đã duyệt.

**Ngoài phạm vi hiện tại (phạm vi sau):** thư viện mẫu có sẵn cho khách chọn; tự tách màu theo **kỹ thuật in**; nhiều người cùng sửa một bản thiết kế. Kỹ thuật in của quy cách cũng là phạm vi sau ([01 §4.10](../../warehouse/01-product-creation/README.md)).

---

## 2. Actor liên quan

| Actor | Vai trò |
|-------|---------|
| **Khách hàng sỉ (B2B Customer)** — chủ sở hữu | Trên sàn: bấm **"Thiết kế"** ở một quy cách để tạo bản thiết kế; tải ảnh / logo, đặt lên khuôn 2D, thêm chữ, xem trước 3D; **lưu**, xem lại trong **mẫu thiết kế của tôi** ([13](../13-catalog/README.md)); xoá bản nháp; **gửi duyệt** (có thể kèm yêu cầu báo giá); **tạo bản sửa** từ bản đã duyệt; **yêu cầu báo giá** từ bản đã duyệt. Tạo và đóng **dự án thiết kế** (mọi bản thiết kế thuộc một dự án — BR-23), thêm bản, **chuyển bản** sang dự án khác, quản lý **thư viện tài nguyên** của dự án; **gửi duyệt cả bộ** (có thể kèm yêu cầu báo giá); **yêu cầu báo giá cả dự án**, bỏ dòng khỏi yêu cầu đang mở. Duyệt **mẫu in** của Đơn sản xuất ([17](../17-order-management/README.md)). |
| **Quản lý kinh doanh (Sales Manager)** | Khai báo **cấu hình in** (khuôn 2D, vùng in, lề, model 3D, ràng buộc file) cho quy cách ([01 §4.10](../../warehouse/01-product-creation/README.md)). Không duyệt bản thiết kế, không thấy hàng chờ duyệt; bản đang chờ duyệt chỉ xem được qua yêu cầu báo giá đang mở (BR-03) *(✅ Chốt lại 2026-11-20: chuyển việc duyệt thiết kế sang Điều phối sản xuất; bỏ "duyệt và gửi Đơn sản xuất" — [17](../17-order-management/README.md) BR-23)*. |
| **Nhân viên Sale (Sales Staff)** | **Nhận yêu cầu báo giá** từ bản thiết kế hoặc **cả dự án** qua [chat](../16-sales-chat/README.md); tra thiết kế đã duyệt và SKU riêng của khách để báo giá ([14](../14-cart-and-order/README.md)). Với yêu cầu cả dự án: được báo khi yêu cầu **đủ điều kiện báo giá** (mọi dòng đã `Approved`) rồi mới lập báo giá; **đóng yêu cầu** kèm lý do khi yêu cầu treo (BR-25). Không thiết kế, không sửa bản thiết kế của khách. |
| **Điều phối sản xuất (Production Coordinator)** *(thêm 2026-11-07)* | **Duyệt kỹ thuật + kiểm duyệt nội dung** bản thiết kế khách gửi: **Duyệt** hoặc **Trả lại** kèm lý do (quyền **Duyệt kỹ thuật thiết kế**); với **lượt duyệt dự án**, xem cả bộ cạnh nhau và kiểm thêm sự đồng bộ giữa các bản (§4.10) *(✅ Chốt lại 2026-11-20; trước đây Quản lý kinh doanh)*. Lập và gửi **Đơn sản xuất** cho SKU riêng ([17](../17-order-management/README.md)) — snapshot của SKU được đính kèm; tải file theo quyền. |
| **Đơn vị sản xuất** *(ngoài hệ thống — xưởng của công ty hoặc đối tác ngoài)* | Nhận bộ file chốt kèm Đơn sản xuất; in **mẫu in thử** theo bộ file khi Đơn sản xuất cần duyệt mẫu in ([17](../17-order-management/README.md)); không thao tác trong module này. |
| **Hệ thống** | Quét mã độc file tải lên; dựng xem trước 2D / 3D; **preflight** khi gửi duyệt; khi duyệt: **xuất bộ file** (file in PDF/X), tính checksum, chốt snapshot, **sinh SKU thành phẩm riêng**. Kiểm lại checksum khi gắn báo giá / đơn / Đơn sản xuất và khi cấp link tải. Mở chat kèm thẻ yêu cầu báo giá (nhiều dòng khi yêu cầu cả dự án — [16](../16-sales-chat/README.md)). Chạy preflight từng bản khi gửi duyệt cả bộ, mở và tự đóng **lượt duyệt dự án**; mở **yêu cầu báo giá dự án** khi gửi cả bộ kèm yêu cầu, báo Sale khi yêu cầu đủ điều kiện báo giá, bỏ dòng của bản bị xoá (BR-25). Ghi lịch sử quyết định. |

- Khách là **người tạo duy nhất** của bản thiết kế; người duyệt luôn là nhân viên Điều phối sản xuất *(✅ Chốt lại 2026-11-20; trước đây Quản lý kinh doanh)*, nên người duyệt không bao giờ là người tạo.
- Khả năng nhìn thấy theo trạng thái: xem BR-03.

---

## 3. Input / Output

### Input

| Trường | Mô tả | Bắt buộc |
|--------|-------|----------|
| Khách hàng & chủ sở hữu | Tài khoản khách hàng đang đăng nhập, **thuộc đúng một khách sỉ** ([18](../18-customer-information/README.md)); hệ thống gán | Có |
| Sản phẩm & quy cách | Một sản phẩm in theo mẫu (`CUSTOMIZABLE`) và một **quy cách** (dung tích, chất liệu…) `Active`, **đã có khuôn in** ([01 §4.10](../../warehouse/01-product-creation/README.md)) | Có |
| **Cấu hình in của quy cách** | Do Quản lý kinh doanh khai báo ở [01 §4.10](../../warehouse/01-product-creation/README.md): **khuôn 2D** (bản trải phẳng, kích thước thật), **vùng in**, **lề an toàn**, **lề tràn**, **model 3D** (xem trước), **ràng buộc file** (độ phân giải tối thiểu ở kích thước in thật, định dạng, dung lượng tối đa) | Có (khuôn 2D + vùng in); model 3D không bắt buộc |
| Tên thiết kế | ≤ 200 ký tự, khách đặt | Có |
| **Ảnh / logo tải lên** | File khách tải lên, đúng **định dạng** và **dung lượng** theo ràng buộc file của quy cách | Không khi lưu nháp; bản gửi duyệt phải có ít nhất một phần tử (ảnh hoặc chữ) |
| **Bố cục (layout)** | Các phần tử đặt trên khuôn 2D: ảnh (vị trí, góc xoay, kích thước), chữ (nội dung, phông, cỡ, màu, vị trí, góc xoay) | Có khi gửi duyệt |
| Yêu cầu báo giá *(tuỳ chọn)* | **Số thùng** mong muốn (số nguyên ≥ 1), **ngày giao mong muốn** (nếu có) — §4.8 | Không |
| **Dự án thiết kế** | Tên (≤ 200 ký tự), mô tả (≤ 2.000 ký tự, không bắt buộc); các bản thiết kế thuộc dự án — §4.9. Tạo bản thiết kế phải chỉ định một dự án `Active` của khách, còn chỗ (BR-23) | Có |
| **Tài nguyên dự án** | Logo / ảnh khách tải vào **thư viện** của dự án, dùng lại cho mọi bản trong dự án (BR-18) | Không |
| Yêu cầu báo giá cả dự án *(tuỳ chọn)* | Các bản được chọn trong dự án — `Pending Review`, bị trả lại (`Draft`) hoặc `Approved` (BR-25.2); khi kèm gửi duyệt cả bộ thì là các bản trong lô gửi (§4.10). **Số thùng** mong muốn từng bản (số nguyên ≥ 1), **một ngày giao mong muốn chung** (nếu có) — §4.11 | Không |
| **Quyết định duyệt** | **Duyệt** / **Trả lại**; nhận xét (≤ 2.000 ký tự) — bắt buộc khi Trả lại | Có (người duyệt) |

> ⚠️ Giả định: ràng buộc file mặc định khi Quản lý kinh doanh chưa chỉnh — độ phân giải tối thiểu **300 DPI** ở kích thước in thật; định dạng ảnh tải lên **PNG / JPG / SVG / PDF**; dung lượng tối đa **50 MB mỗi file**. Xem [open-questions B34](../../open-questions/README.md).

### Output

| Kết quả | Mô tả | Dùng bởi |
|---------|-------|----------|
| **Bản thiết kế (design)** | Trạng thái, quy cách, bố cục, ảnh tải lên, ảnh xem trước, kết quả preflight gần nhất, quyết định duyệt, phiên bản | Khách (chủ sở hữu), Điều phối sản xuất (người duyệt); Sale, Quản lý kinh doanh (theo BR-03 — bản đang chờ duyệt chỉ qua yêu cầu báo giá đang mở) *(✅ Chốt lại 2026-11-20: Quản lý kinh doanh không còn đọc bản đang chờ duyệt)* |
| **Kết quả preflight** | Đạt / Không đạt, kèm **danh sách lỗi cụ thể** (phần tử nào, lỗi gì) | Khách |
| **Bộ file xuất** | Sinh tự động khi duyệt; mỗi file một **vai trò** (bảng dưới) | [17](../17-order-management/README.md), khách, Sale |
| **Design snapshot** | Bộ file xuất + nội dung thiết kế (bố cục, ảnh gốc khách tải lên) lúc duyệt, checksum từng file và checksum cả bộ, người và thời điểm duyệt. Bất biến. | [14](../14-cart-and-order/README.md), [17](../17-order-management/README.md) |
| **SKU thành phẩm riêng của khách** | SKU mới thuộc sản phẩm và quy cách của bản thiết kế, gắn khách sở hữu và snapshot; chỉ khách đó thấy và được báo giá / đặt | [01](../../warehouse/01-product-creation/README.md), [13](../13-catalog/README.md), [14](../14-cart-and-order/README.md), [17](../17-order-management/README.md), [03](../../warehouse/03-receipt/README.md), [20](../../warehouse/20-inventory-control/README.md) |
| **Yêu cầu báo giá** | Số thùng, ngày giao mong muốn, gắn bản thiết kế / SKU riêng; hiện thành **thẻ yêu cầu báo giá** trong chat | [16](../16-sales-chat/README.md), [14](../14-cart-and-order/README.md) |
| **Dự án thiết kế** | Tên, mô tả, trạng thái (`Active` / `Archived`), danh sách bản thiết kế kèm trạng thái và ảnh xem trước, thư viện tài nguyên | Khách (chủ sở hữu); Điều phối sản xuất (người duyệt), Sale thấy **tên dự án** làm ngữ cảnh (BR-17) |
| **Lượt duyệt dự án** | Các bản gửi duyệt cùng lúc từ một dự án; hiện thành **một mục** trong hàng chờ duyệt; tự đóng khi mọi bản đã có quyết định (§4.10) | Điều phối sản xuất |
| **Yêu cầu báo giá cả dự án** | Dự án, các dòng (bản thiết kế, số thùng, **trạng thái hiện tại** của bản và lý do trả lại nếu có), ngày giao mong muốn chung, số dòng đã duyệt trên tổng số dòng; hiện thành **thẻ yêu cầu báo giá nhiều dòng** trong chat | [16](../16-sales-chat/README.md), [14](../14-cart-and-order/README.md) |
| **Lịch sử quyết định** | Mọi bước của bản thiết kế (BR-13, §5.3) và của dự án (§5.4) | Điều phối sản xuất, Sale, kiểm toán |
| **Link tải file** | Link ngắn hạn, cấp theo quyền; file trong snapshot chỉ cấp khi checksum còn khớp | Khách, Sale, Quản lý kinh doanh, Điều phối sản xuất, đơn vị sản xuất (qua Đơn sản xuất) — mỗi người trong phạm vi BR-03 |

**Vai trò của file trong bộ file xuất:**

| Vai trò | Nội dung | Người dùng chính |
|---------|----------|------------------|
| `PRINT_READY` | File in **PDF/X** theo khuôn: đúng kích thước thật, có lề tràn | Đơn vị sản xuất (qua Đơn sản xuất) |
| `CUSTOMER_PREVIEW` | Ảnh xem trước 2D (và 3D nếu quy cách có model 3D) | Khách, Điều phối sản xuất, Sale |
| `TECHNICAL_SPEC` | Đặc tả kỹ thuật sinh từ cấu hình in và bố cục: quy cách, kích thước khuôn và vùng in, vị trí / kích thước từng phần tử, độ phân giải hiệu dụng của từng ảnh, phông và mã màu chữ | Điều phối sản xuất, đơn vị sản xuất |
| `PRODUCTION_INSTRUCTION` | Hướng dẫn sản xuất **sinh mặc định từ quy cách thùng** (số cái / thùng, kích thước, khối lượng thùng — [01](../../warehouse/01-product-creation/README.md)) và yêu cầu đóng thùng (đơn vị sản xuất đóng thùng trước khi giao về kho) | Đơn vị sản xuất |

> ⚠️ Giả định: file in xuất theo chuẩn **PDF/X-4**. Xem [open-questions B35](../../open-questions/README.md).

---

## 4. Luồng nghiệp vụ từng bước (Step-by-step)

### 4.1. Bắt đầu thiết kế

1. Trên trang chi tiết sản phẩm ([13](../13-catalog/README.md)), khách chọn một quy cách và bấm **"Thiết kế"**. Nút chỉ có ở quy cách `Active` **đã có khuôn in** (khuôn 2D + vùng in — [01 §4.10](../../warehouse/01-product-creation/README.md)) của sản phẩm còn dùng được.
2. Hệ thống tạo **bản thiết kế** ở `Draft`, chủ sở hữu là tài khoản khách đang đăng nhập (BR-01), gắn sản phẩm, quy cách và **dự án** khách chọn — một dự án `Active` của chính khách, còn chỗ (BR-23.2; khách chưa có dự án thì tạo dự án trước — §4.9); khách đặt tên (đổi được khi còn `Draft`).
3. Khách cũng tạo bản thiết kế mới từ **mẫu thiết kế của tôi** (chọn dự án, sản phẩm, quy cách), **ngay trong một dự án thiết kế** (§4.9), hoặc tạo **bản sửa** từ bản đã duyệt (§4.7).

> ⚠️ Giả định: mỗi khách có tối đa **50 bản `Draft`** cùng lúc; bản `Draft` lâu không hoạt động **không tự xoá**. Xem [open-questions B37](../../open-questions/README.md).

### 4.2. Thiết kế trên khuôn 2D & xem trước 3D

4. Trình thiết kế hiện **khuôn 2D** của quy cách theo kích thước thật, đánh dấu **vùng in**, **lề an toàn** và **lề tràn**.
5. Khách **tải ảnh / logo lên** (hệ thống quét mã độc, kiểm định dạng và dung lượng ngay — BR-04), đặt lên khuôn: **kéo, xoay, đổi kích thước**; **thêm chữ**.
6. Khách **xem trước 3D** bố cục trên **model 3D** của quy cách.

   > ⚠️ Giả định: xem trước 3D dựng **phía trình duyệt của khách**, có **ảnh tĩnh dựng sẵn** làm dự phòng khi thiết bị không dựng được; quy cách **chưa có model 3D** thì khách chỉ xem trước 2D, vẫn thiết kế và gửi duyệt được. Xem [open-questions B7, B36](../../open-questions/README.md).
7. Khách **lưu**. Mỗi lần lưu tăng **phiên bản** bản thiết kế (BR-11). Bản `Draft` sửa tự do, không giới hạn số lần lưu.
8. Khách xem lại mọi bản thiết kế của mình (mọi trạng thái trừ `Archived`) trong **mẫu thiết kế của tôi** ([13](../13-catalog/README.md)).
9. Khách **xoá bản nháp** → `Archived` (BR-16). Chỉ bản `Draft` xoá được.

### 4.3. Gửi duyệt & kiểm tự động (preflight)

10. Khách bấm **"Gửi duyệt"** (hoặc **"Gửi duyệt và yêu cầu báo giá"** — §4.8). Hệ thống chạy **preflight** trên phiên bản đang lưu (BR-05):
    - có ít nhất một phần tử trên khuôn;
    - mỗi ảnh đạt **độ phân giải tối thiểu ở kích thước in thật** (theo ràng buộc file của quy cách);
    - chữ, logo nằm **trong vùng in và lề an toàn**; không phần tử nào vượt ra ngoài **vùng in + lề tràn**;
    - mọi file đúng **định dạng**, **dung lượng**;
    - quy cách và sản phẩm **còn dùng được** (quy cách `Active`, sản phẩm `Approved` / `Published`) và quy cách **đã có quy cách thùng** ([01](../../warehouse/01-product-creation/README.md)) — ✅ đã chốt 2026-11-05, xem [open-questions B25](../../open-questions/README.md).
11. **Không đạt** → hệ thống báo **lỗi cụ thể** (phần tử nào, lỗi gì, ví dụ "logo.png chỉ đạt 180 DPI ở kích thước in, cần ≥ 300"); bản thiết kế **vẫn `Draft`**, khách sửa rồi gửi lại.
12. **Đạt** → `Pending Review`. Bản thiết kế **khoá sửa** (BR-07); hệ thống báo cho Điều phối sản xuất (hàng chờ duyệt).

### 4.4. Duyệt kỹ thuật & kiểm duyệt nội dung

> ✅ Chốt lại 2026-11-20: người duyệt là **Điều phối sản xuất** (trước đây Quản lý kinh doanh) — §1 bước 3, [open-questions B46](../../open-questions/README.md).

13. Điều phối sản xuất mở bản `Pending Review`: xem bố cục trên khuôn 2D, xem trước 3D, kết quả preflight, ảnh gốc khách tải lên. Kiểm:
    - **kỹ thuật** — khả năng in: vị trí, kích thước so với khuôn, độ rõ của ảnh, chữ quá nhỏ / quá sát mép;
    - **nội dung** — bản quyền, nhãn hiệu của bên thứ ba, nội dung không phù hợp ([open-questions B6](../../open-questions/README.md)).
14. Điều phối sản xuất chọn (BR-06):
    - **Duyệt** (nhận xét không bắt buộc) → §4.5;
    - **Trả lại** kèm **lý do** (bắt buộc, ≤ 2.000 ký tự) → bản thiết kế về `Draft`; khách thấy lý do ở mẫu thiết kế của tôi, sửa rồi gửi duyệt lại (preflight chạy lại).
15. Quyết định gắn với **đúng phiên bản** đang chờ duyệt (BR-11).

### 4.5. Chốt thiết kế: xuất bộ file, snapshot & SKU riêng

16. Khi Điều phối sản xuất **Duyệt**, hệ thống làm trong **cùng một bước** (BR-09):
    - **xuất bộ file** từ bố cục và cấu hình in: `PRINT_READY` (file in PDF/X theo khuôn, kích thước thật, lề tràn), `CUSTOMER_PREVIEW` (ảnh xem trước 2D / 3D), `TECHNICAL_SPEC`, `PRODUCTION_INSTRUCTION` (sinh mặc định từ quy cách thùng) — §3;
    - chốt **design snapshot**: bộ file xuất + nội dung thiết kế, tính checksum từng file và cả bộ, kiểm file lưu trữ khớp;
    - **sinh SKU thành phẩm riêng** thuộc sản phẩm và quy cách của bản thiết kế, gắn khách sở hữu và snapshot ([01 §4.8](../../warehouse/01-product-creation/README.md)); **quy cách thùng** (số cái chuẩn / thùng — cơ sở để khách đặt theo số thùng ở [14](../14-cart-and-order/README.md); kích thước, khối lượng) của SKU lấy theo quy cách, chỉnh được ở [01](../../warehouse/01-product-creation/README.md);
    - chuyển bản thiết kế → `Approved`. Bản thiết kế từ đây **không sửa được nữa** (BR-08).
17. Hệ thống báo khách: bản thiết kế đã duyệt, SKU riêng đã sinh; khách tải được file xem trước. Nếu khách đã gửi kèm yêu cầu báo giá (§4.8), Sale được báo là đã lập báo giá được.

> ✅ Đã chốt 2026-11-05: SKU riêng được sinh **mã tự động** (theo mã sản phẩm + mã khách + số thứ tự), ở trạng thái `Active` ngay (preflight và Quản lý kinh doanh duyệt — *✅ Chốt lại 2026-11-20: nay Điều phối sản xuất* — thay cho bước kích hoạt biến thể ở [01](../../warehouse/01-product-creation/README.md)). Xem [open-questions B20](../../open-questions/README.md).

SKU riêng **không hiện giá trên sàn** (sàn ẩn giá — [13](../13-catalog/README.md)). Giá có trên báo giá / đơn / hợp đồng do Sale lập ([14](../14-cart-and-order/README.md)); Sale tham khảo **giá nội bộ** do Quản lý kinh doanh khai báo ([13](../13-catalog/README.md) §4.2), nếu có. Lần sản xuất đầu của một SKU riêng (mẫu mới) có **dòng phí khuôn / bản in** trên báo giá và hợp đồng ([14](../14-cart-and-order/README.md)).

### 4.6. Dùng thiết kế đã duyệt

18. Sale tra **danh sách thiết kế đã duyệt và SKU riêng** của khách (lọc theo sản phẩm hoặc SKU) và xem trước, để chọn SKU riêng vào **dòng báo giá / đơn hàng** ([14](../14-cart-and-order/README.md)).
19. Khi đơn thiếu hàng của SKU riêng, Điều phối sản xuất lập và gửi **Đơn sản xuất** ([17](../17-order-management/README.md)) — không qua duyệt *(✅ Chốt lại 2026-11-20, 17 BR-23)*; lúc gửi hệ thống kiểm lại checksum file in; snapshot của SKU được **đính kèm** Đơn sản xuất, đơn vị sản xuất nhận bộ file chốt (`PRINT_READY`, `TECHNICAL_SPEC`, `PRODUCTION_INSTRUCTION`) qua link tải có kiểm checksum.
    - **Duyệt mẫu in** ([17](../17-order-management/README.md)) — khi Đơn sản xuất bật cờ "cần duyệt mẫu in" (mặc định bật cho lần sản xuất đầu của SKU), đơn vị sản xuất in mẫu thử **theo đúng bộ file của snapshot**; khách xem ảnh / video mẫu ở mẫu thiết kế của tôi ([13](../13-catalog/README.md)) hoặc thẻ trong [chat](../16-sales-chat/README.md), rồi **Duyệt** hoặc **Yêu cầu in lại**.
    - Duyệt mẫu in **không sửa snapshot** (BR-14). Mẫu lỗi in hoặc khách muốn in lại cùng file → in lại vòng mới ở 17. Khách cần **sửa file thiết kế** → tạo bản sửa (§4.7), sinh SKU riêng mới, đổi SKU dòng đơn theo quy trình ở [17](../17-order-management/README.md).
20. Mỗi lần gắn (báo giá, đơn, Đơn sản xuất) và mỗi lần cấp link tải, hệ thống kiểm (BR-10):
    - SKU riêng và snapshot thuộc **đúng khách** của chứng từ;
    - file lưu trữ **còn khớp checksum**.

### 4.7. Sửa thiết kế đã duyệt (tạo bản sửa)

21. Muốn đổi một thiết kế đã duyệt (đổi logo, đổi màu, nội dung mới, hoặc xem mẫu in thử rồi thấy cần sửa file): khách bấm **"Tạo bản sửa"** trên bản `Approved` ở mẫu thiết kế của tôi. Hệ thống tạo **bản thiết kế mới** ở `Draft`:
    - cùng khách, sản phẩm, quy cách;
    - **sao chép nội dung** (bố cục, ảnh) từ snapshot cũ;
    - ghi nguồn là snapshot cũ;
    - **thuộc cùng dự án** với bản gốc nếu dự án đó `Active` và còn chỗ; khách cũng chọn được một dự án `Active` khác của mình. Dự án của bản gốc đã đóng hoặc đã đủ bản thì khách **phải chọn** một dự án `Active` khác; chưa chọn thì hệ thống từ chối, không tạo bản sửa. Bản sửa không bao giờ đứng lẻ (BR-22, BR-23.3).
22. Bản sửa đi lại từ §4.2. Khi bản sửa được duyệt, hệ thống xuất bộ file mới, sinh **snapshot mới và SKU riêng mới**. Snapshot và SKU cũ giữ nguyên, nên báo giá, đơn, Đơn sản xuất và tồn kho đã dùng SKU cũ không bị ảnh hưởng (BR-12).

    **Nhân bản sang quy cách khác** *(✅ Đã chốt 2026-11-25)* — từ một bản thiết kế của mình (trừ bản đã xoá), khách bấm **"Nhân bản"** và chọn **một quy cách khác của cùng sản phẩm** (đã có khuôn in). Hệ thống tạo **bản thiết kế mới** ở `Draft`: sao chép ảnh, bố cục đã khớp vào vùng in của quy cách đích, ghi nguồn là bản gốc; bản gốc không đổi. Bản nhân bản theo **cùng quy tắc dự án như bản sửa** (bước 21): vào dự án của bản nguồn nếu `Active` và còn chỗ, không thì khách phải chọn một dự án `Active` khác (BR-23.3). Bản nhân bản đi lại từ §4.2 và được duyệt riêng.

> ⚠️ Giả định: SKU riêng cũ **vẫn `Active`** sau khi có bản sửa (còn tồn, còn đơn đang mở); khách chọn SKU nào khi yêu cầu báo giá. Quản lý kinh doanh ngừng SKU cũ theo quy trình ngừng biến thể ở [01](../../warehouse/01-product-creation/README.md). Đổi SKU của một đơn đã chốt sang SKU mới đi theo quy trình thay đổi đơn ở [17](../17-order-management/README.md), chỉ khi Đơn sản xuất liên quan chưa sang `In Production`. Xem [open-questions B5](../../open-questions/README.md).

### 4.8. Yêu cầu báo giá từ bản thiết kế

23. Từ một bản **`Approved`**, khách bấm **"Yêu cầu báo giá"**, nhập **số thùng** mong muốn (số nguyên ≥ 1) và **ngày giao mong muốn** (nếu có). Hệ thống mở **chat với Sale** gắn ngữ cảnh bản thiết kế / SKU riêng, kèm **thẻ yêu cầu báo giá** ([16](../16-sales-chat/README.md)).
24. Khách có thể **"Gửi duyệt và yêu cầu báo giá"** cùng lúc từ bản `Draft`:
    - preflight không đạt → không gửi gì, bản thiết kế vẫn `Draft`;
    - preflight đạt → bản thiết kế sang `Pending Review` và Sale **nhận yêu cầu ngay**, nhưng chỉ **lập báo giá** được khi bản đã `Approved` (SKU riêng đã sinh) (BR-15).
25. Sale lập báo giá / đơn / hợp đồng như ở [14](../14-cart-and-order/README.md): số thùng khách nêu là **gợi ý**; ngày giao mong muốn chép thành **ngày giao khách yêu cầu** của báo giá.
26. Khách **không tự đặt hàng**; sàn không hiện giá ([13](../13-catalog/README.md)).

Ràng buộc tương ứng ở module khác: [13](../13-catalog/README.md) BR-16 (yêu cầu báo giá từ mẫu thiết kế của tôi), [14](../14-cart-and-order/README.md) BR-22 (báo giá từ yêu cầu báo giá của khách), [16](../16-sales-chat/README.md) BR-11 (thẻ yêu cầu báo giá).

### 4.9. Dự án thiết kế & thư viện tài nguyên

27. Trong **mẫu thiết kế của tôi** ([13](../13-catalog/README.md)), khách **tạo dự án thiết kế**: tên (≤ 200 ký tự), mô tả (≤ 2.000 ký tự, không bắt buộc). Dự án ở `Active` (§5.4).
28. Khách đưa bản thiết kế vào dự án (BR-17, BR-23):
    - **tạo bản mới ngay trong dự án** (chọn sản phẩm, quy cách — như §4.1); hoặc
    - **chuyển bản sẵn có** của mình (không phải `Archived`) từ một dự án `Active` khác sang.

    **Không có thao tác "gỡ khỏi dự án"**: bản thiết kế chỉ rời dự án khi được chuyển sang một dự án `Active` khác của cùng khách (BR-23.4). Không chuyển được:
    - bản đang `Pending Review`;
    - bản đang nằm trong một **yêu cầu báo giá dự án đang mở** — khách bỏ dòng đó khỏi yêu cầu trước (§4.11);
    - bản thuộc dự án đã đóng.

    Chuyển **không đổi trạng thái** bản thiết kế; lịch sử bản thiết kế ghi cả dự án cũ lẫn dự án mới (§5.3).
29. **Thư viện tài nguyên** — khách tải logo / ảnh vào thư viện của dự án **một lần** (quét mã độc, kiểm định dạng và dung lượng — BR-18). Trong trình thiết kế của một bản thuộc dự án, khách chọn tài nguyên từ thư viện để đặt lên khuôn 2D; hệ thống **sao chép** file vào bản thiết kế đó.
    - **Bố cục vẫn làm riêng từng bản** (mỗi quy cách một khuôn 2D).
    - **Preflight vẫn chạy riêng từng bản**: cùng một logo có thể đủ độ phân giải trên ly nhỏ nhưng không đủ trên túi giấy lớn.
30. Màn hình dự án hiện các bản **cạnh nhau** (ảnh xem trước 2D / 3D, trạng thái) để khách so sự đồng bộ của bộ nhận diện.

> ⚠️ Giả định: mỗi dự án tối đa **20 bản thiết kế** (không tính `Archived`); dự án đã đủ thì không tạo bản mới, bản sửa hay chuyển bản vào được; dự án đã đóng (`Archived`) **không mở lại**. Xem [open-questions B42](../../open-questions/README.md).
>
> ⚠️ Giả định: thư viện tài nguyên tối đa **30 file** mỗi dự án; file tải vào thư viện kiểm theo ràng buộc file mặc định (định dạng PNG / JPG / SVG / PDF, ≤ 50 MB — giả định B34); khi đặt lên một bản, kiểm lại theo ràng buộc file của quy cách bản đó. Xem [open-questions B43](../../open-questions/README.md).

### 4.10. Gửi duyệt cả bộ (lượt duyệt dự án)

> ✅ Chốt lại 2026-11-20: lượt duyệt dự án do **Điều phối sản xuất** duyệt (trước đây Quản lý kinh doanh), như §4.4.

31. Khách chọn **từ 2 bản `Draft` trở lên** trong dự án và bấm **"Gửi duyệt cả bộ"**. Hệ thống chạy **preflight từng bản** (BR-05, BR-19):
    - **có bản không đạt** → **không gửi bản nào**; hệ thống báo lỗi cụ thể theo từng bản; mọi bản vẫn `Draft`;
    - **tất cả đạt** → mọi bản sang `Pending Review` cùng lúc (khoá sửa — BR-07), gắn vào một **lượt duyệt dự án**; hệ thống báo Điều phối sản xuất (lượt duyệt hiện là **một mục** trong hàng chờ duyệt).

    Khách có thể bật **kèm yêu cầu báo giá** cho chính lô gửi (BR-25.1): mỗi dòng là một bản **trong lô**, kèm số thùng mong muốn; một ngày giao mong muốn chung (nếu có). Yêu cầu được kiểm **trước** preflight; yêu cầu sai (bản ngoài lô, một bản nêu hai lần, số thùng hoặc ngày sai) thì **không gửi bản nào**. Gửi thành công thì hệ thống mở **yêu cầu báo giá dự án** cùng lúc với lượt duyệt và báo Sale (§4.11).

    Dự án chỉ có **một** bản `Draft` thì khách gửi duyệt lẻ (§4.3), có thể kèm yêu cầu báo giá (§4.8) — BR-23.7.
32. Điều phối sản xuất mở lượt duyệt, xem các bản **cạnh nhau**. Với từng bản, kiểm như §4.4; với cả bộ, kiểm thêm **sự đồng bộ** (logo, màu, chữ giữa các bản). Điều phối sản xuất **quyết định từng bản** (BR-20):
    - **Duyệt** → §4.5 cho bản đó (xuất bộ file, snapshot, SKU riêng);
    - **Trả lại** kèm lý do → bản đó về `Draft`;
    - **"Duyệt tất cả"** → hệ thống duyệt **lần lượt** từng bản còn chờ; bản nào không duyệt được (BR-09) thì ở lại `Pending Review`, các bản khác vẫn được duyệt.
33. Bản bị trả lại: khách sửa rồi gửi duyệt lại — **gửi lẻ** (§4.3) hoặc trong **lượt duyệt dự án mới**. Lượt duyệt **tự đóng** khi mọi bản trong lượt đã có quyết định (duyệt hoặc trả lại).

### 4.11. Yêu cầu báo giá cả dự án

34. Khách mở **yêu cầu báo giá cả dự án** theo một trong hai đường (BR-21, BR-25):
    - kèm khi **gửi duyệt cả bộ** (§4.10) — các dòng là các bản trong lô gửi;
    - trong một dự án `Active`, chọn **một hoặc nhiều bản** `Pending Review`, bị trả lại (`Draft`) hoặc `Approved` và bấm **"Yêu cầu báo giá cả dự án"**.

    Khách nhập **số thùng** mong muốn cho từng bản (số nguyên ≥ 1) và **một ngày giao mong muốn chung** (nếu có). Hệ thống mở **chat với Sale** gắn ngữ cảnh dự án, kèm **thẻ yêu cầu báo giá nhiều dòng**: mỗi dòng một bản thiết kế, kèm **trạng thái hiện tại** của bản (và lý do trả lại nếu có); thẻ hiện **"đã duyệt x/y"** ([16](../16-sales-chat/README.md)).
35. Yêu cầu mở tới khi Sale lập báo giá hoặc yêu cầu đóng (BR-25):
    - Sale chỉ **lập báo giá** khi **mọi dòng đã `Approved`**; không báo giá một phần. Bản cuối cùng của yêu cầu được duyệt thì hệ thống **báo Sale** yêu cầu đã **đủ điều kiện báo giá**.
    - Bản trong yêu cầu bị trả lại: yêu cầu **vẫn mở và chờ**; khách sửa, gửi duyệt lại (lẻ hoặc trong lượt duyệt mới) thì dòng đi tiếp theo bản đó.
    - Khách **bỏ dòng** khi Sale chưa lập báo giá: bản bị bỏ vẫn ở trong dự án, giữ trạng thái, sau này yêu cầu báo giá riêng được. Bỏ hết dòng thì yêu cầu đóng.
    - Khách **xoá** một bản nháp đang nằm trong yêu cầu thì dòng đó tự rơi; yêu cầu hết dòng thì tự đóng.
    - Khách tự đóng yêu cầu; **Sale đóng yêu cầu kèm lý do** (bắt buộc), ví dụ khi bản bị trả lại mà khách không sửa tiếp.
36. Sale lập **một báo giá** gồm các SKU riêng tương ứng như ở [14](../14-cart-and-order/README.md): số thùng khách nêu là **gợi ý**; ngày giao mong muốn chép thành **ngày giao khách yêu cầu** của báo giá. Một yêu cầu → một báo giá nhiều dòng → một đơn nhiều dòng. Đã có báo giá lập từ yêu cầu thì khách **không sửa yêu cầu nữa**; muốn đổi thì **yêu cầu sửa báo giá** ([14](../14-cart-and-order/README.md) §4.2).

---

## 5. Trạng thái / Vòng đời (Lifecycle)

### 5.1. Bản thiết kế (Design)

| Trạng thái | Ý nghĩa | Chuyển tiếp cho phép | Ai thực hiện |
|------------|---------|----------------------|--------------|
| `Draft` | Khách đang thiết kế: tải ảnh, đặt bố cục, xem trước, lưu; sửa tự do | → `Pending Review` (gửi duyệt lẻ hoặc **gửi duyệt cả bộ** từ dự án, **preflight đạt**); → `Archived` (khách xoá bản nháp) | Chủ sở hữu; Hệ thống (preflight) |
| `Pending Review` | Đã qua preflight, chờ Điều phối sản xuất duyệt kỹ thuật và kiểm duyệt nội dung; khoá sửa | → `Approved` (Duyệt — xuất bộ file, chốt snapshot, sinh SKU riêng); → `Draft` (Trả lại, kèm lý do) | Điều phối sản xuất; Hệ thống (xuất file, snapshot, SKU) |
| `Approved` | Đã duyệt; bộ file, snapshot và SKU riêng đã sinh; bất biến | *(cuối)* — đổi nội dung = tạo bản sửa (bản thiết kế mới); yêu cầu báo giá được | Chủ sở hữu (tạo bản sửa, yêu cầu báo giá) |
| `Archived` | Bản nháp khách đã xoá; không còn hiện ở mẫu thiết kế của tôi; giữ lịch sử | *(cuối)* | Chủ sở hữu |

```mermaid
stateDiagram-v2
    state "Pending Review" as PendingReview
    [*] --> Draft: Khách bấm Thiết kế / tạo bản sửa
    Draft --> Draft: Lưu / preflight không đạt
    Draft --> PendingReview: Gửi duyệt lẻ hoặc cả bộ, preflight đạt
    PendingReview --> Draft: Điều phối sản xuất trả lại, có lý do
    PendingReview --> Approved: Duyệt, xuất file PDF/X, chốt snapshot, sinh SKU riêng
    Draft --> Archived: Khách xoá bản nháp
    Approved --> [*]
    Archived --> [*]
```

Mọi bản thiết kế thuộc một dự án (BR-23) và đi đúng vòng đời này; dự án không thêm trạng thái cho bản thiết kế.

### 5.2. Design snapshot & SKU riêng

Snapshot **không có vòng đời**: sinh ra một lần khi bản thiết kế được duyệt và không bao giờ thay đổi. Mỗi bản thiết kế có tối đa một snapshot và một SKU riêng; mỗi SKU riêng gắn đúng một snapshot.

Vòng đời của SKU riêng (`Active` → `Obsolete`) theo trạng thái biến thể ở [01](../../warehouse/01-product-creation/README.md); bản sửa là một bản thiết kế khác, có nguồn là snapshot cũ và sinh SKU riêng mới.

### 5.3. Lịch sử quyết định

| Sự kiện | Khi nào |
|---------|---------|
| `CREATED` | Khách tạo bản thiết kế |
| `DERIVED_FROM_SNAPSHOT` | Bản sửa được tạo từ snapshot cũ |
| `PREFLIGHT_FAILED` | Gửi duyệt nhưng preflight không đạt (ghi danh sách lỗi) |
| `SUBMITTED_FOR_REVIEW` | Preflight đạt, bản thiết kế sang `Pending Review` (ghi mã **lượt duyệt dự án** nếu gửi cả bộ) |
| `RETURNED` | Người duyệt trả lại (ghi lý do) |
| `APPROVED` | Người duyệt duyệt (ghi mã snapshot, SKU riêng đã sinh và checksum bộ file) |
| `ARCHIVED` | Khách xoá bản nháp |
| `QUOTE_REQUESTED` | Khách gửi yêu cầu báo giá (ghi số thùng, ngày giao mong muốn; ghi dự án nếu yêu cầu cả dự án) |
| `ADDED_TO_PROJECT` / `REMOVED_FROM_PROJECT` | Bản thiết kế vào / rời một dự án (ghi dự án). Tạo bản mới, bản sửa, bản nhân bản ghi `ADDED_TO_PROJECT`; chuyển dự án ghi `REMOVED_FROM_PROJECT` (nêu dự án cũ) rồi `ADDED_TO_PROJECT` (nêu dự án mới) |

### 5.4. Dự án thiết kế (Design Project)

| Trạng thái | Ý nghĩa | Chuyển tiếp cho phép | Ai thực hiện |
|------------|---------|----------------------|--------------|
| `Active` | Dự án đang dùng: thêm / chuyển bản, quản lý thư viện tài nguyên, gửi duyệt cả bộ (có thể kèm yêu cầu báo giá), yêu cầu báo giá cả dự án | → `Archived` (khách đóng dự án; không còn bản `Pending Review` trong dự án, và mọi yêu cầu báo giá dự án đang mở chỉ còn dòng `Approved` — BR-22) | Chủ sở hữu |
| `Archived` | Dự án đã đóng: không thêm bản, không chuyển bản vào hay ra, không gửi duyệt cả bộ, không yêu cầu báo giá cả dự án; các bản thiết kế bên trong **giữ nguyên trạng thái**, vẫn thuộc dự án, xem / xoá bản nháp / yêu cầu báo giá từng bản được (BR-23.5); giữ lịch sử | *(cuối)* — không mở lại (⚠️ giả định B42) | Chủ sở hữu |

```mermaid
stateDiagram-v2
    [*] --> Active: Khách tạo dự án
    Active --> Active: Thêm / chuyển bản, thư viện, gửi duyệt cả bộ, yêu cầu báo giá cả dự án
    Active --> Archived: Khách đóng dự án, không còn bản Pending Review, yêu cầu báo giá dự án đang mở chỉ còn dòng Approved
    Archived --> [*]
```

**Lượt duyệt dự án** không có vòng đời riêng: mở khi khách gửi duyệt cả bộ thành công, **tự đóng** khi mọi bản trong lượt đã được duyệt hoặc trả lại (§4.10).

Lịch sử của dự án (BR-13):

| Sự kiện | Khi nào |
|---------|---------|
| `PROJECT_CREATED` | Khách tạo dự án |
| `ASSET_ADDED` / `ASSET_REMOVED` | Khách thêm / xoá tài nguyên trong thư viện |
| `BATCH_PREFLIGHT_FAILED` | Gửi duyệt cả bộ nhưng có bản không đạt preflight (ghi lỗi từng bản) |
| `BATCH_SUBMITTED` | Gửi duyệt cả bộ thành công, mở lượt duyệt (ghi các bản trong lượt) |
| `BATCH_CLOSED` | Lượt duyệt tự đóng (ghi kết quả từng bản) |
| `PROJECT_QUOTE_REQUESTED` | Khách yêu cầu báo giá cả dự án, kể cả kèm khi gửi duyệt cả bộ (ghi các bản, số thùng từng bản, ngày giao mong muốn) |
| `PROJECT_ARCHIVED` | Khách đóng dự án |

---

## 6. Ràng buộc nghiệp vụ (Business Rules) & Edge case

### Business rules

- **BR-01:** Mỗi bản thiết kế thuộc **đúng một khách**, **một sản phẩm `CUSTOMIZABLE`** và **một quy cách**. Chủ sở hữu là tài khoản khách hàng đang hoạt động **của chính khách đó** (mỗi khách một tài khoản đăng nhập) và là **người tạo duy nhất**; nhân viên không tạo, không sửa bản thiết kế của khách.
- **BR-02:** Chỉ bắt đầu thiết kế được trên quy cách `Active` của sản phẩm `Approved` / `Published` **đã có khuôn in** (khuôn 2D + vùng in — [01 §4.10](../../warehouse/01-product-creation/README.md)).
- **BR-03:** **Phạm vi nhìn thấy.** Bản `Draft` chỉ chủ sở hữu thấy. Bản `Pending Review` thêm Điều phối sản xuất (hàng chờ duyệt) và nhân viên xem qua **yêu cầu báo giá đang mở** gắn bản đó — Sale, Quản lý kinh doanh (chỉ xem) *(✅ Chốt lại 2026-11-20: Quản lý kinh doanh không còn thấy hàng chờ duyệt, chỉ còn đường yêu cầu báo giá)*. Thiết kế **đã duyệt** và SKU riêng thì Sale, Quản lý kinh doanh và Điều phối sản xuất tra được theo khách (để báo giá, lập đơn, lập Đơn sản xuất). Khách chỉ thấy thiết kế và SKU của chính mình; với người khác coi như không tồn tại.

  > ⚠️ Giả định: nhân viên **không xem** bản `Draft` của khách (khách cần hỗ trợ thì gửi ảnh chụp / mô tả qua chat). Xem [open-questions B38](../../open-questions/README.md).
- **BR-04:** File tải lên phải đúng **định dạng**, **dung lượng** theo ràng buộc file của quy cách và qua **quét mã độc**; sai thì từ chối ngay, không lưu. Chỉ bản `Draft` nhận file và sửa bố cục.
- **BR-05:** **Preflight bắt buộc** trước `Pending Review`: có phần tử; độ phân giải tối thiểu ở kích thước in thật; chữ / logo trong vùng in và lề an toàn, không vượt vùng in + lề tràn; định dạng, dung lượng; quy cách / sản phẩm còn dùng được và **quy cách đã có quy cách thùng** (✅ đã chốt 2026-11-05 — [open-questions B25](../../open-questions/README.md)). Không đạt thì báo lỗi cụ thể và bản thiết kế ở lại `Draft`.
- **BR-06:** **Preflight không thay cho duyệt người.** Chỉ Điều phối sản xuất đang hoạt động (quyền **Duyệt kỹ thuật thiết kế**, mặc định gán Điều phối sản xuất — *✅ Chốt lại 2026-11-20; trước đây Quản lý kinh doanh*) được Duyệt / Trả lại; mỗi lần duyệt gồm **duyệt kỹ thuật** và **kiểm duyệt nội dung**. Trả lại bắt buộc lý do.
- **BR-07:** Bản `Pending Review` **khoá sửa**: không nhận file, không đổi bố cục, không xoá. Muốn sửa thì chờ kết quả duyệt.
- **BR-08:** **Snapshot bất biến từ lúc duyệt.** Snapshot gồm bộ file xuất, nội dung thiết kế, checksum SHA-256 của từng file và của cả bộ. Bản `Approved` không nhận file, không sửa bố cục, không xoá.
- **BR-09:** **Duyệt = xuất bộ file + snapshot + SKU riêng, trong cùng một bước.** Không xuất được bộ file hoặc không sinh được SKU riêng (ví dụ trùng mã, sản phẩm / quy cách đã ngừng) thì việc duyệt bị từ chối, bản thiết kế ở lại `Pending Review`.
- **BR-10:** **Toàn vẹn thiết kế.**
  - SKU riêng chỉ được báo giá, đặt và đưa vào Đơn sản xuất cho **đúng khách sở hữu**.
  - Ở mọi lần dùng (gắn báo giá / đơn / Đơn sản xuất, tải file), file lưu trữ phải khớp checksum đã chốt.
  - Lệch thì từ chối; không bao giờ gửi sản xuất theo file chưa được duyệt.
- **BR-11:** Mọi thao tác ghi (lưu, gửi duyệt, duyệt, trả lại, xoá) gửi kèm **phiên bản** bản thiết kế người dùng đang xem. Ai đó đã đổi trước thì thao tác bị từ chối, người dùng tải lại.
- **BR-12:** Báo giá, đơn, Đơn sản xuất và tồn kho đã dùng một SKU riêng thì **giữ SKU (và snapshot) đó**; bản sửa sinh SKU mới, không ảnh hưởng tới chúng.
- **BR-13:** Mọi quyết định ghi **lịch sử**: người, nội dung / lý do, phiên bản, thời điểm (§5.3).
- **BR-14:** **Duyệt mẫu in không thay cho duyệt thiết kế và không sửa snapshot.** Mẫu in thử của Đơn sản xuất ([17](../17-order-management/README.md)) được in theo bộ file của snapshot đã chốt; khách chỉ **Duyệt** hoặc **Yêu cầu in lại** (cùng file). Mọi thay đổi file thiết kế đi qua bản sửa (§4.7) → snapshot mới + SKU riêng mới.
- **BR-15:** **Yêu cầu báo giá không phải đơn hàng.** Khách gửi yêu cầu báo giá từ bản `Approved`, hoặc kèm khi gửi duyệt; Sale chỉ **lập báo giá** khi bản đã `Approved` (SKU riêng đã sinh). Khách không tự đặt; sàn không hiện giá.
- **BR-16:** Chỉ bản `Draft` xoá được (→ `Archived`); `Archived` không khôi phục, không dùng lại. Bản `Pending Review` / `Approved` không xoá được.
- **BR-17:** **Dự án thiết kế thuộc đúng một khách**; chỉ chủ sở hữu tạo, đổi tên, thêm / chuyển bản, quản lý thư viện và đóng dự án. Một bản thiết kế thuộc **đúng một dự án** (BR-23) *(✅ Chốt lại 2026-11-25; trước đây dự án không bắt buộc)*, và chỉ bản **của cùng khách** mới thêm vào được; bản `Archived` không thêm được; bản `Pending Review` không chuyển dự án được. Thêm / chuyển không đổi trạng thái bản thiết kế. **Phạm vi nhìn thấy** như BR-03: dự án chỉ chủ sở hữu thấy; Điều phối sản xuất (người duyệt) và Sale chỉ thấy **tên dự án** làm ngữ cảnh trên lượt duyệt và thẻ yêu cầu báo giá.
- **BR-18:** **Thư viện tài nguyên.** File tải vào thư viện qua quét mã độc, kiểm định dạng và dung lượng như BR-04. Đặt tài nguyên lên một bản = **sao chép** file vào bản đó và kiểm lại theo ràng buộc file của quy cách bản đó; sai thì không đặt được. Xoá tài nguyên khỏi thư viện **không ảnh hưởng** bản đã dùng nó (kể cả snapshot — BR-08).
- **BR-19:** **Gửi duyệt cả bộ là tất cả hoặc không.** Chỉ gửi được từ **2 bản `Draft` trở lên** của một dự án `Active`. Preflight chạy từng bản; có bản không đạt thì **không bản nào** sang `Pending Review`. Tất cả đạt thì mọi bản sang `Pending Review` cùng lúc trong một **lượt duyệt dự án**.
- **BR-20:** **Duyệt chung nhưng quyết định từng bản.** Trong lượt duyệt dự án, mỗi bản vẫn được Duyệt / Trả lại riêng theo BR-06, BR-09, BR-11; "Duyệt tất cả" là duyệt lần lượt từng bản còn chờ. Bản không duyệt được ở lại `Pending Review`, không chặn các bản khác. Lượt duyệt tự đóng khi mọi bản đã có quyết định.
- **BR-21:** **Yêu cầu báo giá cả dự án** gồm các bản của một dự án `Active` (ít nhất một bản) theo BR-25 *(✅ Chốt lại 2026-11-25: trước đây chỉ gồm bản `Approved` và ở mức dự án không có "Gửi duyệt và yêu cầu báo giá")*; số thùng từng bản là **gợi ý**; **một ngày giao mong muốn chung** thành ngày giao khách yêu cầu của báo giá. Sale lập **một báo giá nhiều dòng** khi mọi dòng đã `Approved` (bỏ / thêm dòng theo thoả thuận với khách — [14](../14-cart-and-order/README.md) BR-22). Như BR-15: không phải đơn hàng, không giữ hàng, không có giá.
- **BR-22:** **Đóng dự án** (`Active` → `Archived`) chỉ khi không còn bản `Pending Review` trong dự án. Dự án có **yêu cầu báo giá dự án đang mở** thì chỉ đóng được khi **mọi dòng** của yêu cầu đã `Approved`; yêu cầu đó vẫn đi tiếp tới báo giá. Còn dòng chưa duyệt (chờ duyệt hoặc bị trả lại) thì hệ thống từ chối; khách bỏ dòng (BR-25.4) hoặc đóng yêu cầu (BR-25.3) trước *(✅ Đã chốt 2026-11-25)*. Dự án `Archived` không nhận bản mới, không cho chuyển bản vào hay ra, không gửi duyệt cả bộ, không yêu cầu báo giá cả dự án; các bản bên trong giữ trạng thái, vẫn thuộc dự án và yêu cầu báo giá từng bản được (BR-23.5). Bản sửa tạo từ bản thuộc dự án đã đóng phải vào **một dự án `Active` khác** của khách (BR-23.3) *(✅ Chốt lại 2026-11-25; trước đây bản sửa không thuộc dự án nào)*.
- **BR-23:** **Mọi bản thiết kế thuộc một dự án** *(✅ Đã chốt 2026-11-25)*.
  - **BR-23.1:** Mọi bản thiết kế luôn thuộc **đúng một** dự án của chính khách đó, ở mọi trạng thái; không có thiết kế lẻ.
  - **BR-23.2:** Tạo bản thiết kế bắt buộc chỉ định một dự án `Active` của khách đang đăng nhập, **còn chỗ** (dưới 20 bản — ⚠️ giả định B42).
  - **BR-23.3:** Bản sửa và bản nhân bản vào **cùng dự án với bản nguồn** nếu dự án đó `Active` và còn chỗ. Nếu không thì khách phải chỉ định một dự án `Active` khác của mình; không đứng lẻ.
  - **BR-23.4:** **Không có thao tác "gỡ khỏi dự án".** Bản thiết kế chỉ **chuyển** sang một dự án `Active` khác của cùng khách. Không chuyển được bản `Pending Review` (BR-17), bản thuộc dự án đã đóng (BR-22), và bản đang nằm trong một **yêu cầu báo giá dự án đang mở** — khách bỏ dòng đó khỏi yêu cầu trước (BR-25.4), để yêu cầu của dự án này không chứa bản đã sang dự án khác.
  - **BR-23.5:** Bản thiết kế trong dự án đã đóng vẫn thuộc dự án đó: xem, xoá bản nháp, yêu cầu báo giá từng bản như cũ (BR-22). Muốn sửa tiếp thì tạo bản sửa vào một dự án `Active` (BR-23.3).
  - **BR-23.6:** Bản thiết kế lẻ có từ trước ngày chốt là dữ liệu thử nghiệm: để nguyên tới khi nạp lại dữ liệu mẫu; sau đó hệ thống chặn hẳn bản thiết kế không có dự án.
  - **BR-23.7:** Dự án có đúng **một** bản `Draft` vẫn gửi duyệt được bằng đường gửi duyệt lẻ (§4.3, §4.8). Gửi duyệt cả bộ vẫn từ 2 bản (BR-19).
- **BR-25:** **Gửi duyệt cả bộ kèm yêu cầu báo giá; yêu cầu báo giá dự án chờ đủ bộ được duyệt** *(✅ Đã chốt 2026-11-25)*. Mã BR-24 để dành cho một quyết định khác.
  - **BR-25.1:** Gửi duyệt cả bộ được kèm **một yêu cầu báo giá dự án**: mỗi dòng là một bản **trong lô gửi**, kèm số thùng; một ngày giao mong muốn chung cho cả yêu cầu. Yêu cầu sai thì không bản nào được gửi (tất cả hoặc không — BR-19).
  - **BR-25.2:** Yêu cầu báo giá dự án chứa được bản `Pending Review`, bản bị trả lại (`Draft`) hoặc `Approved`. Sale **chỉ lập báo giá khi mọi dòng đã `Approved`**; không báo giá một phần. Bản cuối cùng của yêu cầu được duyệt thì hệ thống **báo Sale** yêu cầu đã **đủ điều kiện báo giá**; thẻ yêu cầu trong chat hiện **"đã duyệt x/y"** ([16](../16-sales-chat/README.md) BR-11).
  - **BR-25.3:** Bản trong yêu cầu bị trả lại: yêu cầu **vẫn mở và chờ**; khách sửa và gửi duyệt lại thì yêu cầu đi tiếp. Mỗi dòng hiện trạng thái hiện tại của bản và lý do trả lại (nếu có). Yêu cầu không treo mãi: khách tự đóng được, và **Sale đóng được kèm lý do** (bắt buộc).
  - **BR-25.4:** Khách được **bỏ một dòng** khỏi yêu cầu đang mở, khi Sale chưa lập báo giá từ yêu cầu đó. Bản bị bỏ vẫn ở trong dự án, không đổi trạng thái; sau này yêu cầu báo giá riêng, thành báo giá và đơn khác. Bỏ hết dòng thì yêu cầu đóng.
  - **BR-25.5:** Xoá (`Archived`) một bản đang nằm trong yêu cầu thì dòng đó tự rơi khỏi yêu cầu. Yêu cầu không còn dòng nào thì **tự đóng**.
  - **BR-25.6:** Một yêu cầu báo giá dự án → **một báo giá nhiều dòng** → một đơn nhiều dòng ([14](../14-cart-and-order/README.md)).
  - **BR-25.7:** Đã có báo giá lập từ yêu cầu thì khách **không sửa yêu cầu nữa**; muốn đổi thì yêu cầu sửa báo giá ([14](../14-cart-and-order/README.md) §4.2).

### Edge case

| Tình huống | Hướng xử lý |
|------------|-------------|
| Quy cách chưa có khuôn in (khuôn 2D + vùng in) | Không có nút "Thiết kế"; khách liên hệ Sale qua chat, Quản lý kinh doanh khai báo cấu hình in ở [01 §4.10](../../warehouse/01-product-creation/README.md). |
| Quy cách có khuôn nhưng chưa có model 3D | Khách chỉ xem trước 2D, vẫn thiết kế và gửi duyệt được (⚠️ giả định §4.2). |
| Thiết bị của khách không dựng được 3D | Hiện ảnh tĩnh dự phòng (⚠️ giả định §4.2). |
| Ảnh khách tải lên độ phân giải thấp | Lưu nháp được; khi gửi duyệt preflight báo lỗi phần tử đó (DPI thực tế ở kích thước in so với ngưỡng); bản thiết kế ở lại `Draft`. |
| Logo / chữ lấn ra ngoài lề an toàn | Preflight không đạt, chỉ rõ phần tử; khách kéo vào trong rồi gửi lại. |
| Quy cách chưa có quy cách thùng khi khách gửi duyệt | Preflight không đạt (BR-05); khách liên hệ Sale, Quản lý kho khai báo quy cách thùng ([01](../../warehouse/01-product-creation/README.md)). |
| Điều phối sản xuất trả lại (vi phạm bản quyền, chữ quá nhỏ…) | Về `Draft` kèm lý do; khách sửa, gửi duyệt lại (preflight chạy lại). |
| Khách muốn sửa khi bản đang `Pending Review` | Không sửa được (BR-07); chờ kết quả duyệt, hoặc tạo bản thiết kế mới. |
| Hai người cùng thao tác trên một bản thiết kế (khách mở hai tab; hai Điều phối sản xuất cùng duyệt) | Người sau bị từ chối vì phiên bản đã đổi (BR-11); tải lại rồi làm tiếp. |
| Quy cách / sản phẩm bị khoá hoặc ngừng khi bản đang `Pending Review` | Không sinh được SKU riêng → duyệt bị từ chối (BR-09); Điều phối sản xuất trả lại kèm lý do, khách chọn quy cách khác bằng bản thiết kế mới. |
| Xuất bộ file PDF/X lỗi khi duyệt | Duyệt bị từ chối, bản thiết kế ở lại `Pending Review` (BR-09); Điều phối sản xuất duyệt lại sau. |
| Khách đã đạt số bản nháp tối đa | Không tạo bản mới; khách xoá bớt bản nháp (⚠️ giả định §4.1). |
| "Gửi duyệt và yêu cầu báo giá" nhưng bản bị trả lại | Thẻ yêu cầu báo giá vẫn ở chat; Sale chờ tới khi bản đó được gửi lại và duyệt mới lập báo giá (BR-15). ⚠️ Giả định — xem [open-questions B40](../../open-questions/README.md). |
| Khách muốn đổi thiết kế sau khi đã duyệt | Tạo bản sửa từ snapshot (§4.7) → gửi duyệt → duyệt → SKU riêng mới. |
| Mẫu in thử bị lỗi in (lệch màu, nhoè, sai vị trí so với file) | Không đụng thiết kế: khách **Yêu cầu in lại**, vòng mẫu mới ghi lý do lỗi in của bên sản xuất ([17](../17-order-management/README.md)). |
| Xem mẫu in thử, khách muốn đổi logo / màu / nội dung file | Không sửa snapshot (BR-14): tạo bản sửa (§4.7) → duyệt → SKU riêng mới; Sale đổi SKU dòng đơn, Điều phối sản xuất xử lý Đơn sản xuất theo quy trình ở [17](../17-order-management/README.md). |
| Gắn SKU riêng của khách khác vào báo giá / đơn | Hệ thống từ chối (BR-10). |
| File trong kho lưu trữ bị thay / hỏng sau khi duyệt | Kiểm checksum thất bại: không gắn báo giá / đơn / Đơn sản xuất, không cấp link tải. Khách tạo bản sửa (Sale hướng dẫn qua chat), gửi duyệt lại → SKU mới. |
| File tải lên nhiễm mã độc / sai định dạng / quá dung lượng | Từ chối ngay, không lưu (BR-04). |
| "Gửi duyệt cả bộ" nhưng một bản không đạt preflight | Không gửi bản nào (BR-19); hệ thống báo lỗi theo từng bản. Khách sửa bản lỗi, hoặc bỏ chọn bản đó rồi gửi các bản còn lại. |
| "Duyệt tất cả" nhưng một bản không sinh được SKU riêng (quy cách vừa ngừng) | Bản đó ở lại `Pending Review` (BR-09), các bản khác vẫn được duyệt (BR-20); Điều phối sản xuất trả lại bản đó kèm lý do để lượt duyệt đóng. |
| Khách muốn chuyển bản đang `Pending Review` sang dự án khác, hoặc đóng dự án còn bản `Pending Review` | Không được (BR-17, BR-22, BR-23.4); chờ kết quả duyệt. |
| Khách đóng dự án khi còn yêu cầu báo giá dự án đang mở | Mọi dòng đã `Approved` → đóng được, yêu cầu vẫn đi tiếp tới báo giá. Còn dòng chờ duyệt hoặc bị trả lại → từ chối; khách bỏ dòng đó hoặc đóng yêu cầu trước (BR-22). |
| Khách nhân bản một bản thuộc dự án đã đóng hoặc đã đủ bản | Như bản sửa: khách phải chọn một dự án `Active` khác còn chỗ; chưa chọn thì từ chối (BR-23.3). |
| Khách muốn đưa một bản ra khỏi dự án thành bản lẻ | Không có thao tác này (BR-23.4); khách chuyển bản sang một dự án `Active` khác, hoặc xoá bản nháp. |
| Khách muốn chuyển một bản đang nằm trong yêu cầu báo giá dự án đang mở | Không được (BR-23.4); khách bỏ dòng đó khỏi yêu cầu (BR-25.4) rồi chuyển. |
| Khách muốn chuyển một bản thuộc dự án đã đóng | Không được (BR-22); bản vẫn thuộc dự án cũ (BR-23.5). Muốn sửa tiếp thì tạo bản sửa vào một dự án `Active`. |
| Dự án đích đã đủ 20 bản khi tạo bản mới, tạo bản sửa hoặc chuyển bản vào | Hệ thống từ chối (⚠️ giả định B42); khách chọn dự án khác hoặc tạo dự án mới. |
| Dự án chỉ có một bản `Draft` | Gửi duyệt lẻ (§4.3), có thể kèm yêu cầu báo giá (§4.8) — BR-23.7. |
| Khách xoá logo khỏi thư viện khi logo đã dùng trên các bản | Các bản giữ bản sao của file; snapshot đã chốt không đổi (BR-18). |
| Logo trong thư viện hợp lệ nhưng sai định dạng / quá dung lượng theo ràng buộc của một quy cách | Không đặt được lên bản thuộc quy cách đó; hệ thống báo lỗi (BR-18). |
| Logo đủ độ phân giải trên ly nhỏ nhưng thiếu trên túi giấy lớn | Preflight của bản túi giấy không đạt (preflight chạy riêng từng bản — §4.9); khách tải logo độ phân giải cao hơn. |
| Khách muốn yêu cầu báo giá cả dự án khi có bản chưa duyệt | Được: yêu cầu chứa được bản `Pending Review` hoặc bị trả lại (BR-25.2); Sale được báo khi yêu cầu đủ điều kiện báo giá. Muốn Sale báo giá ngay phần đã duyệt thì khách chỉ chọn bản `Approved`, hoặc bỏ các dòng chưa duyệt (BR-25.4). |
| Khách tạo bản sửa từ một bản thuộc dự án đã đóng hoặc đã đủ bản | Khách phải chọn một dự án `Active` khác còn chỗ; chưa chọn thì hệ thống từ chối, không tạo bản sửa (BR-22, BR-23.3). |
| Gửi duyệt cả bộ kèm yêu cầu báo giá, yêu cầu nêu một bản ngoài lô gửi | Không gửi bản nào, không mở yêu cầu (BR-25.1); khách sửa yêu cầu rồi gửi lại. |
| Gửi duyệt cả bộ kèm yêu cầu báo giá, một bản trong lô bị trả lại | Yêu cầu vẫn mở; dòng đó hiện `Draft` kèm lý do trả lại (BR-25.3). Khách sửa và gửi duyệt lại, hoặc bỏ dòng đó (BR-25.4); Sale lập báo giá khi mọi dòng đã `Approved`. |
| Sale lập báo giá khi yêu cầu còn dòng chưa `Approved` | Hệ thống từ chối, nêu các bản chưa duyệt (BR-25.2). |
| Bản bị trả lại mà khách không sửa tiếp, yêu cầu treo | Sale đóng yêu cầu kèm lý do; khách cũng tự đóng được (BR-25.3). |
| Khách xoá bản nháp đang nằm trong yêu cầu báo giá dự án | Dòng đó tự rơi; yêu cầu hết dòng thì tự đóng (BR-25.5). |
| Khách bỏ dòng sau khi Sale đã lập báo giá từ yêu cầu | Hệ thống từ chối vì yêu cầu đã có báo giá (BR-25.7); khách yêu cầu sửa báo giá ([14](../14-cart-and-order/README.md)). |
| Khách bỏ dòng đúng lúc Sale bấm lập báo giá | Thao tác nào tới trước thì thắng. Sale trước: khách bị từ chối như dòng trên. Khách trước: Sale thấy yêu cầu đã đổi, lập báo giá theo danh sách mới nếu mọi dòng còn lại đã `Approved`. |

---

## 7. Liên kết với các module khác

| Module | Quan hệ |
|--------|---------|
| [01 · Product Creation](../../warehouse/01-product-creation/README.md) | Sản phẩm `CUSTOMIZABLE`, quy cách và **cấu hình in** của quy cách (khuôn 2D, vùng in, lề, model 3D, ràng buộc file — §4.10) mà khách thiết kế trên đó; quy cách thùng là điều kiện preflight; SKU thành phẩm riêng của khách được sinh vào sản phẩm khi bản thiết kế được duyệt (§4.8) |
| [13 · Catalog](../13-catalog/README.md) | Nút **"Thiết kế"** trên quy cách có khuôn in; **mẫu thiết kế của tôi** — tạo, sửa, xem lại, gửi duyệt, xoá bản nháp, tạo bản sửa, yêu cầu báo giá; **dự án thiết kế** (mọi bản thiết kế thuộc một dự án; thư viện tài nguyên, chuyển bản, gửi duyệt cả bộ kèm yêu cầu báo giá, yêu cầu báo giá cả dự án); SKU riêng, tình trạng còn hàng và **mẫu in chờ duyệt** |
| [14 · Báo giá, Đơn hàng & Hợp đồng](../14-cart-and-order/README.md) | Sale lập báo giá từ yêu cầu báo giá (số thùng gợi ý, ngày giao khách yêu cầu) — **một báo giá nhiều dòng** khi yêu cầu cả dự án, lập khi mọi dòng đã `Approved` (BR-25); dòng báo giá / đơn / hợp đồng dùng SKU riêng mang snapshot; hệ thống kiểm khách và checksum khi gắn |
| [16 · Sales Chat](../16-sales-chat/README.md) | Hội thoại gắn ngữ cảnh bản thiết kế / SKU riêng (hoặc dự án) kèm **thẻ yêu cầu báo giá** (nhiều dòng khi yêu cầu cả dự án, mỗi dòng có trạng thái, "đã duyệt x/y"); thông báo Sale yêu cầu **đủ điều kiện báo giá**; thẻ duyệt mẫu in |
| [17 · Order Management](../17-order-management/README.md) | Đơn sản xuất đính kèm snapshot (file in PDF/X) của SKU riêng gửi đơn vị sản xuất; **duyệt mẫu in** theo snapshot (khách cần sửa file → quay về 12 tạo bản sửa, SKU mới); đổi SKU của đơn đã chốt theo quy trình thay đổi đơn |
| [18 · Customer Information](../18-customer-information/README.md) | Chủ sở hữu là tài khoản đăng nhập của khách; thiết kế và SKU riêng gắn theo khách |
| [03 · Receipt](../../warehouse/03-receipt/README.md) | Nhập kho thành phẩm theo SKU riêng khi đơn vị sản xuất giao hàng |
| [20 · Inventory Control](../../warehouse/20-inventory-control/README.md) | Tồn và ATP của SKU riêng (chỉ dành cho khách sở hữu) |
