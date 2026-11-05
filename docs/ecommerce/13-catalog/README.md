# 13 — Catalog sản phẩm trên cổng khách hàng (Catalog)

> Đã chốt 2026-10-06: hệ thống **chỉ bán sỉ (B2B)**. Catalog nằm trong **cổng khách hàng**, chỉ người đã đăng nhập mới xem được; tình trạng còn hàng chỉ dành cho **khách sỉ đang hoạt động** (giá: xem mốc chốt 2026-11-05 bên dưới — sàn không hiển thị giá).
>
> Đã chốt 2026-11-04: catalog **chỉ để xem hàng và liên hệ Sale** — **khách không tự đặt hàng**; đơn do Sale chốt từ báo giá ([14](../14-cart-and-order/README.md)). **Không còn hàng trơn**: catalog hiển thị các **quy cách** in được của sản phẩm `CUSTOMIZABLE` (để khách xem và yêu cầu in qua chat), **mẫu thiết kế của khách** và **SKU thành phẩm riêng của khách** ([12](../12-product-design/README.md)). Không có giá gốc, không có giá nhóm.
>
> Đã chốt 2026-11-05: khách sỉ **đặt theo số thùng**, không đặt lẻ; **giá vẫn theo cái**. Bảng giá riêng ghi **đơn giá theo cái**, còn **MOQ và bậc giá tính theo số thùng**. Mỗi SKU riêng có **số cái chuẩn / thùng** cố định (quy cách thùng — [01](../../warehouse/01-product-creation/README.md)), hiển thị cho khách. Tình trạng còn hàng tính theo **số thùng** ([open-questions A9](../../open-questions/README.md)).
>
> Đã chốt 2026-11-05 (lượt 4): **sàn ẩn giá hoàn toàn** — catalog không hiển thị giá ở bất cứ màn hình nào, kể cả SKU riêng. Giá chỉ có trên **báo giá / đơn / hợp đồng** sau khi Sale thoả thuận với khách ([14](../14-cart-and-order/README.md)). **Bảng giá riêng** giữ lại làm **giá nội bộ**: do **Quản lý kinh doanh (Sales Manager)** khai báo, **chỉ nhân viên xem**, là **giá gợi ý** cho Sale khi lập báo giá. Mục **mẫu thiết kế của tôi** hiện thêm **mẫu in chờ duyệt** của Đơn sản xuất, với nút **Duyệt** / **Yêu cầu in lại** ([17](../17-order-management/README.md)).

## 1. Mục đích (Purpose)

Trưng bày ly và bao bì cho **khách sỉ đã đăng nhập** trên cổng khách hàng:

- **cây danh mục**, **trang danh sách** (PLP), **trang chi tiết sản phẩm** (PDP), bộ ảnh;
- **tìm kiếm** không dấu, theo tên hoặc mã, có **gợi ý** khi gõ;
- **quy cách in được** của từng sản phẩm `CUSTOMIZABLE` (dung tích, chất liệu…) — để khách tham khảo và **yêu cầu in theo mẫu** qua chat; quy cách không có giá, không có tồn;
- **mẫu thiết kế của tôi** — các bản thiết kế của khách và SKU riêng sinh ra từ đó ([12](../12-product-design/README.md)), kèm **mẫu in chờ duyệt** của Đơn sản xuất để khách **Duyệt** / **Yêu cầu in lại** ([17](../17-order-management/README.md));
- **SKU thành phẩm riêng của khách** (chỉ khách sở hữu thấy) kèm **số cái / thùng** — **không kèm giá**;
- **chỉ báo còn hàng** của SKU riêng (không bao giờ hiện số tồn);
- **mở chat với Sale** gắn ngữ cảnh sản phẩm / quy cách / SKU riêng / mẫu thiết kế / đơn cũ để hỏi hàng, yêu cầu in theo mẫu, yêu cầu báo giá hoặc đặt lại ([16](../16-sales-chat/README.md)).

Sàn **không hiển thị giá** (BR-04). Phía back-office, module quản lý **nội dung SEO / slug**, **giá nội bộ (bảng giá riêng) của khách cho SKU riêng**, cùng việc **xuất bản / ẩn** sản phẩm. Giá nội bộ chỉ nhân viên xem và là **giá gợi ý** khi Sale lập báo giá ([14](../14-cart-and-order/README.md)).

Catalog là **lớp hiển thị tiêu thụ dữ liệu** từ [Product Creation](../../warehouse/01-product-creation/README.md) (một nguồn dữ liệu — [open-questions G1](../../open-questions/README.md)) và **tồn khả dụng (ATP)** từ [Inventory Control](../../warehouse/20-inventory-control/README.md).

> ⚠️ Giả định: các phần sau thuộc **phạm vi giai đoạn sau** (xem [open-questions D17](../../open-questions/README.md)):
> - lọc theo thuộc tính, thương hiệu;
> - sắp xếp theo độ phổ biến;
> - thứ tự hiển thị và sản phẩm nổi bật do Quản lý kinh doanh đặt.
>
> Sàn ẩn giá (chốt 2026-11-05) nên **không có** lọc theo khoảng giá hay sắp xếp theo giá.

---

## 2. Actor liên quan

| Actor | Vai trò |
|-------|---------|
| **Khách hàng sỉ (B2B Customer)** | Đăng nhập cổng; duyệt danh mục, tìm kiếm, xem chi tiết sản phẩm và quy cách, xem **mẫu thiết kế của tôi** và **SKU riêng** của mình (không có giá). Hồ sơ `Active` mới xem được **tình trạng còn hàng** của SKU riêng. **Duyệt mẫu in**: xem ảnh / video mẫu in thử, bấm **Duyệt** hoặc **Yêu cầu in lại** kèm ghi chú. **Mở chat với Sale** từ sản phẩm, quy cách, SKU riêng, mẫu thiết kế hoặc đơn cũ. Không tự đặt hàng. |
| **Quản lý kinh doanh (Sales Manager)** | Sửa **slug và nội dung SEO**; **xuất bản / ẩn** sản phẩm; khai báo **giá nội bộ (bảng giá riêng)** của khách (đơn giá / cái, bậc theo số thùng) cho SKU riêng của chính khách đó |
| **Quản trị hệ thống (System Admin)** | Như Quản lý kinh doanh |
| **Nhân viên Sale (Sales Staff)** | **Xem** giá nội bộ làm giá gợi ý khi tư vấn và lập báo giá; nhận hội thoại khách mở từ catalog; gửi link sản phẩm / SKU riêng qua [chat](../16-sales-chat/README.md); tải ảnh / video mẫu in thử lên vòng mẫu của Đơn sản xuất ([17](../17-order-management/README.md)) để khách xem ở đây |
| **Hệ thống** | Dựng bản hiển thị (projection) và chỉ mục tìm kiếm theo sự kiện; lọc SKU riêng và mẫu in theo khách sở hữu; tính chỉ báo còn hàng từ ATP; gửi kết quả duyệt mẫu in về Đơn sản xuất ([17](../17-order-management/README.md)) |

---

## 3. Input / Output

### Input

| Nhóm | Dữ liệu | Nguồn |
|------|---------|-------|
| Sản phẩm & quy cách | Sản phẩm `CUSTOMIZABLE` `Published`: tên, mô tả, danh mục, các **quy cách** `Active` (dung tích, chất liệu…) — chỉ là khuôn để in, không có tồn, không có giá | [01 Product Creation](../../warehouse/01-product-creation/README.md) |
| **SKU thành phẩm riêng của khách** | SKU sinh khi khách xác nhận thiết kế: khách sở hữu, snapshot, sản phẩm & quy cách; **quy cách thùng** (số cái chuẩn / thùng) | [12](../12-product-design/README.md), [01](../../warehouse/01-product-creation/README.md) |
| **Mẫu thiết kế của khách** | Bản thiết kế mà khách là chủ sở hữu: trạng thái, file xem trước, SKU riêng (nếu đã xác nhận) | [12](../12-product-design/README.md) |
| **Mẫu in chờ duyệt** | Vòng mẫu in đang chờ khách của các Đơn sản xuất `Sample Pending` thuộc khách: SKU riêng, mã Đơn sản xuất, số vòng, **ảnh / video mẫu** Sale tải lên, ngày gửi mẫu vật (nếu có), kết quả các vòng trước | [17](../17-order-management/README.md) |
| Ảnh | Ảnh **đã đăng** (bốn mắt) của từng quy cách, ảnh bìa | [01](../../warehouse/01-product-creation/README.md) |
| Cây danh mục | Danh mục đang hoạt động, thứ tự, ảnh, SEO của danh mục | [01](../../warehouse/01-product-creation/README.md) |
| Nội dung catalog | **Slug** (khoá sau lần xuất bản đầu), **tiêu đề SEO** (≤ 255), **mô tả SEO** | Quản lý kinh doanh |
| **Giá nội bộ của khách (bảng giá riêng — customer price)** | Cho mỗi (khách, SKU riêng của khách đó): **1–20 bậc**, mỗi bậc = **số thùng tối thiểu** (≥ 1, không trùng) + **đơn giá theo cái** (VND nguyên dương). **Bậc đầu là MOQ gợi ý** (tính theo thùng). Chỉ nhân viên xem | Quản lý kinh doanh |
| Tồn khả dụng (ATP) | ATP **toàn hệ thống** của SKU riêng, theo **số thùng** | [20 · Inventory Control](../../warehouse/20-inventory-control/README.md) |
| Đơn cũ của khách | Danh sách đơn để khách mở chat "đặt lại" | [17](../17-order-management/README.md) |

### Output

| Kết quả | Dùng bởi |
|---------|----------|
| **Cây danh mục** | Khách |
| **Trang danh sách / kết quả tìm kiếm** — mỗi thẻ: tên, ảnh bìa, các quy cách, số SKU riêng của khách thuộc sản phẩm (nếu có); không kèm giá, tồn | Khách |
| **Gợi ý tìm kiếm** (tên, slug, mã khớp) | Khách |
| **Trang chi tiết** — tên, mô tả, SEO, các quy cách, bộ ảnh đã đăng, nút "Yêu cầu in theo mẫu"; SKU riêng của khách thuộc sản phẩm (nếu có) | Khách |
| **Mẫu thiết kế của tôi** — danh sách thiết kế của khách, trạng thái, file xem trước, SKU riêng kèm số cái / thùng và chỉ báo còn hàng (không có giá); **mẫu in chờ duyệt** | Khách |
| **Kết quả duyệt mẫu in** của khách — **Duyệt** hoặc **Yêu cầu in lại** kèm ghi chú, gắn đúng vòng mẫu | [17](../17-order-management/README.md) |
| **Giá nội bộ** cho từng SKU riêng (chỉ nhân viên) — số cái / thùng, các bậc (số thùng tối thiểu → đơn giá / cái), MOQ gợi ý (thùng) | Sale, Quản lý kinh doanh |
| **Chỉ báo còn hàng** cho từng SKU riêng: `In stock` / `Low stock` / `Out of stock` | Khách |
| **Hội thoại mở từ catalog** — gắn ngữ cảnh sản phẩm / quy cách / SKU riêng / mẫu thiết kế / đơn cũ | [16](../16-sales-chat/README.md) |
| Giá gợi ý (bậc giá: số thùng tối thiểu → đơn giá / cái) điền sẵn vào báo giá | [14](../14-cart-and-order/README.md) |

---

## 4. Luồng nghiệp vụ từng bước (Step-by-step)

### 4.1. Xuất bản & duy trì catalog (Quản lý kinh doanh)

1. **Nội dung catalog** — Quản lý kinh doanh sửa **slug**, **tiêu đề SEO**, **mô tả SEO** (gửi kèm phiên bản đang đọc). Tên và mô tả hiển thị lấy từ sản phẩm ở [01](../../warehouse/01-product-creation/README.md), không sửa ở đây. Sản phẩm `Discontinued` không sửa được.
   - Slug được chuẩn hoá (bỏ dấu, `đ` → `d`, chữ thường nối bằng `-`) và **khoá sau lần xuất bản đầu**.
2. **Xuất bản** — theo điều kiện của [01 §4.5](../../warehouse/01-product-creation/README.md) → sản phẩm `Published`, hiện trên catalog cùng các quy cách. Quy cách không có giá nên xuất bản không đòi điều kiện về giá. **Ẩn** → về `Approved`. Sản phẩm bị tạm ngưng (`Inactive`) cũng rời catalog; tiếp tục rồi phải xuất bản lại.
3. **Cập nhật hiển thị** — mỗi lần sửa nội dung, xuất bản, ẩn, tạm ngưng / tiếp tục, ngừng kinh doanh hoặc sinh SKU riêng mới, hệ thống dựng lại bản hiển thị của sản phẩm.
   - Bản nào cập nhật lỗi được thử lại sau mỗi 30 giây.
   - Khởi động lại thì dựng lại toàn bộ.
   - Tìm kiếm luôn kiểm lại trạng thái thật của sản phẩm, nên bản hiển thị chậm cũng không làm lộ sản phẩm đã ẩn.

### 4.2. Giá nội bộ của khách (Quản lý kinh doanh)

1. **Giá nội bộ (bảng giá riêng) của khách** — Quản lý kinh doanh khai báo 1–20 bậc cho (khách, SKU riêng); mỗi bậc gồm **số thùng tối thiểu** và **đơn giá theo cái**. Bậc có số thùng nhỏ nhất là **MOQ gợi ý** (số thùng) của khách cho SKU đó. SKU riêng chỉ đặt được giá cho **chính khách sở hữu**.
2. Đây là **nguồn giá gợi ý duy nhất**: quy cách không có giá; không có giá gốc, không có giá nhóm.
3. Sửa giá gửi kèm phiên bản (khoá lạc quan). Không có thời hạn hiệu lực, không cần duyệt; mọi thay đổi được ghi nhật ký kiểm toán.
4. **Chỉ nhân viên xem**; giá nội bộ **không bao giờ hiện trên sàn** (BR-04). Sale dùng làm **giá gợi ý** — điền sẵn bậc giá khi lập báo giá ([14](../14-cart-and-order/README.md)); giá thật được thoả thuận với khách và chốt trên báo giá / đơn / hợp đồng.
5. SKU riêng **chưa có giá nội bộ** (ví dụ vừa sinh) thì Sale tự nhập bậc giá trên báo giá ([14](../14-cart-and-order/README.md)); khách không thấy khác biệt vì sàn không hiện giá.

   > ✅ Đã chốt 2026-11-05: báo giá không bắt buộc SKU phải có giá nội bộ trước; giá nội bộ chỉ là gợi ý. Xem [open-questions](../../open-questions/README.md).

### 4.3. Khách duyệt & tìm hàng

1. **Đăng nhập cổng khách hàng.** Người chưa đăng nhập không xem được catalog (chỉ ảnh sản phẩm đã đăng là truy cập công khai).
2. **Cây danh mục** — danh mục đang hoạt động; danh mục ngưng ẩn cả nhánh con. Cùng cấp xếp theo thứ tự đặt rồi theo tên.
3. **Trang danh sách** — lọc theo **danh mục** (gồm mọi danh mục con) và/hoặc **từ khoá**. Sắp xếp:
   - **Liên quan** (mặc định khi có từ khoá);
   - **Mới nhất** (mặc định khi không có từ khoá) — theo lần xuất bản đầu;
   - **Tên**.

   Mỗi trang mặc định 20 sản phẩm, tối đa 200; kết quả xét trong 500 sản phẩm phù hợp nhất.
4. **Tìm kiếm:**
   - Không phân biệt dấu (bỏ dấu tiếng Việt cả từ khoá lẫn dữ liệu, `đ` → `d`); từ khoá tối đa 100 ký tự.
   - Khớp **tên** và **mô tả**; từ cuối khớp theo tiền tố (đang gõ dở); gõ sai chính tả nhẹ vẫn tìm được theo tên.
   - Khớp **mã** (mã quy cách, mã SKU riêng) chính xác hoặc theo tiền tố — chỉ trong các mã khách được xem (BR-02).
   - Xếp hạng "liên quan": khớp chữ + gần đúng + **khớp đúng mã được ưu tiên mạnh** + **SKU riêng còn hàng được cộng điểm**.
5. **Gợi ý khi gõ** — từ **2 ký tự** (chữ / số) trở lên; mặc định 5, tối đa **10** gợi ý; mỗi gợi ý có tên, đường dẫn, mã khớp. Giới hạn 30 lần / 10 giây mỗi người.

### 4.4. Trang chi tiết, quy cách và tình trạng còn hàng

1. **Trang chi tiết** — tên, mô tả, nội dung SEO, **các quy cách nhận in** (dung tích, chất liệu…), **bộ ảnh đã đăng** (theo quy cách, có ảnh bìa), nút **"Yêu cầu in theo mẫu"** mở chat với Sale gắn ngữ cảnh sản phẩm / quy cách ([16](../16-sales-chat/README.md)); quy trình thiết kế ở [12](../12-product-design/README.md).

   > Đã chốt 2026-11-04: quy cách của sản phẩm `CUSTOMIZABLE` chỉ là **khuôn để in** — không có tồn, không có giá, không báo giá / đặt trực tiếp. Hàng có tồn và bán được chỉ là **SKU thành phẩm riêng của khách** ([01](../../warehouse/01-product-creation/README.md), [open-questions B19](../../open-questions/README.md)).
2. **SKU riêng của khách** — khách sở hữu SKU riêng thuộc sản phẩm này thấy thêm các SKU đó (tên thiết kế, quy cách, ảnh xem trước, **số cái / thùng**) kèm chỉ báo còn hàng. Khách khác không thấy (BR-02).
3. **Không hiển thị giá** — trang chi tiết, danh sách, tìm kiếm và mẫu thiết kế của tôi đều không có giá, bậc giá hay MOQ (BR-04). Thay vào đó là nút **"Yêu cầu báo giá"** / **"Chat với Sale"**; khách thấy giá khi Sale gửi **báo giá** ([14](../14-cart-and-order/README.md)) — đơn giá / cái, bậc theo số thùng, thành tiền dòng = số thùng × số cái / thùng × đơn giá / cái ([14](../14-cart-and-order/README.md) BR-08).
4. **Tình trạng còn hàng** của SKU riêng (chỉ khách `Active`; tối đa 50 SKU mỗi lần hỏi):

   | Hiển thị | Điều kiện |
   |----------|-----------|
   | `In stock` — Còn hàng | ATP > ngưỡng |
   | `Low stock` — Sắp hết | 0 < ATP ≤ ngưỡng (mặc định **5 thùng**; đặt 0 thì không cảnh báo sắp hết) |
   | `Out of stock` — Hết hàng | ATP ≤ 0 |

   - **Không bao giờ hiện số ATP** cho khách.
   - ATP tính **toàn hệ thống** (cộng **mọi kho**), theo **số thùng** (mọi thùng đúng số cái chuẩn của SKU). Kết quả có thể trễ tối đa 10 giây.
   - **Quan hệ với kho xuất** *(✅ Đã chốt 2026-11-05)*: chỉ báo **không** cam kết hàng có sẵn ở một kho cụ thể. Đơn chỉ giữ hàng **tại kho xuất** Sale chọn khi đơn `Confirmed` ([17](../17-order-management/README.md) BR-22); khi chọn, hệ thống **ưu tiên gợi ý kho đang có đủ hàng** của các SKU trong đơn, rồi tới kho còn chỗ. Hàng nằm ở kho khác kho xuất không dùng cho đơn đó (không chuyển kho), nên "còn hàng" trên sàn có thể khác hàng có sẵn ở kho xuất.

   > ✅ Đã chốt 2026-11-05: ngưỡng "sắp hết" (mặc định 5) tính theo **số thùng**, cùng đơn vị với ATP hiển thị. Xem [open-questions D28](../../open-questions/README.md).
   - SKU riêng được sản xuất theo đơn, nên thường `Out of stock`; tồn có thể còn từ **hàng dư của đơn trước** hoặc **hàng trả** đã đưa lại tồn khả dụng ([17](../17-order-management/README.md)).
   - `Out of stock` không chặn khách liên hệ Sale: Sale vẫn có thể chốt đơn, phần thiếu được bổ sung qua Đơn sản xuất ([17](../17-order-management/README.md)).

### 4.5. Mẫu thiết kế của tôi

1. Khách mở mục **Mẫu thiết kế của tôi**: danh sách bản thiết kế mà khách là chủ sở hữu ([12](../12-product-design/README.md)), với trạng thái (`Draft` — đang thiết kế, `Submitted` — chờ khách xác nhận, `Approved` — đã xác nhận) và file xem trước.
2. Bản `Submitted`: khách vào trang thiết kế để **xác nhận** hoặc **yêu cầu sửa** theo [12 §4.5](../12-product-design/README.md).
3. Bản `Approved`: hiện **SKU riêng** sinh ra, kèm số cái / thùng và chỉ báo còn hàng (không có giá). SKU riêng vẫn hiện ở đây kể cả khi sản phẩm gốc đang `Unlisted`, chừng nào SKU còn `Active`.
4. **Mẫu in chờ duyệt** — với SKU riêng có Đơn sản xuất đang `Sample Pending` ([17](../17-order-management/README.md)), mục này hiện vòng mẫu in đang chờ khách: số vòng, **ảnh / video mẫu** Sale tải lên, ngày gửi mẫu vật (nếu có), kết quả các vòng trước. Khách chọn:
   - **Duyệt** → Đơn sản xuất sang `In Production` ([17](../17-order-management/README.md));
   - **Yêu cầu in lại** kèm **ghi chú** (bắt buộc) → Sale mở vòng mẫu mới và ghi lý do in lại ở [17](../17-order-management/README.md).

   Mẫu in được in theo đúng file thiết kế đã chốt; khách muốn **sửa file thiết kế** thì mở chat với Sale để tạo bản sửa ([12 §4.7](../12-product-design/README.md)) — không làm trong bước duyệt mẫu. Cùng thao tác có trên thẻ hành động trong [chat](../16-sales-chat/README.md); làm ở một nơi thì nơi kia hiện kết quả.
5. Từ mỗi mẫu, khách có thể **mở chat với Sale** gắn ngữ cảnh mẫu thiết kế (góp ý, đề nghị sửa, yêu cầu báo giá).

### 4.6. Liên hệ Sale từ catalog

Khách **không tự đặt hàng** trên catalog. Mọi nhu cầu mua đi qua Sale:

1. **Hỏi hàng / yêu cầu báo giá** — từ một SKU riêng (trên trang sản phẩm hoặc mẫu thiết kế của tôi), khách bấm **"Chat với Sale"** / **"Yêu cầu báo giá"**: hệ thống mở hội thoại gắn ngữ cảnh SKU riêng ([16](../16-sales-chat/README.md)). Sale lập báo giá ([14](../14-cart-and-order/README.md)) với giá gợi ý từ giá nội bộ của khách.
2. **Đặt lại** — từ một đơn cũ, khách bấm **"Đặt lại"**: hệ thống mở hội thoại gắn ngữ cảnh đơn đó. Sale lập báo giá mới, giá gợi ý theo giá nội bộ hiện hành ([14](../14-cart-and-order/README.md)).
3. **Yêu cầu in theo mẫu** — từ một sản phẩm / quy cách, hoặc từ mẫu thiết kế của tôi: mở hội thoại; khách gửi mẫu (logo, file, ảnh) qua chat; Sale ghi nhận yêu cầu thiết kế ([12 §4.1](../12-product-design/README.md)). Có SKU riêng (sau khi khách xác nhận thiết kế) thì mới báo giá được.

---

## 5. Trạng thái / Vòng đời (Lifecycle)

Catalog không có chứng từ. Một sản phẩm hiện trên catalog khi **`Published`** ở [01](../../warehouse/01-product-creation/README.md); mỗi SKU riêng mang một chỉ báo còn hàng. Mẫu in chờ duyệt chỉ hiện khi Đơn sản xuất ở `Sample Pending`; vòng đời Đơn sản xuất và các vòng mẫu in thuộc [17](../17-order-management/README.md).

| Trạng thái hiển thị | Ý nghĩa | Chuyển tiếp cho phép | Ai thực hiện |
|---------------------|---------|----------------------|--------------|
| `Listed` | Sản phẩm `Published`: hiện trong danh sách, tìm kiếm, trang chi tiết | → `Unlisted` (ẩn, tạm ngưng, ngừng kinh doanh) | Quản lý kinh doanh / System Admin |
| `Unlisted` | Sản phẩm `Approved` (chưa / thôi xuất bản), `Inactive` hoặc `Discontinued`: không hiện, link trực tiếp không xem được nữa | → `Listed` (xuất bản; với `Inactive` phải tiếp tục trước) | Quản lý kinh doanh / System Admin |

| Chỉ báo còn hàng (theo SKU riêng) | Điều kiện |
|-----------------------------------|-----------|
| `In stock` | ATP > ngưỡng (mặc định 5 thùng) |
| `Low stock` | 0 < ATP ≤ ngưỡng |
| `Out of stock` | ATP ≤ 0 |

```mermaid
stateDiagram-v2
    [*] --> Unlisted
    Unlisted --> Listed: Xuất bản (đủ điều kiện ở 01)
    Listed --> Unlisted: Ẩn / tạm ngưng / ngừng kinh doanh
```

---

## 6. Ràng buộc nghiệp vụ (Business Rules) & Edge case

### Business rules

- **BR-01:** Chỉ sản phẩm `Published` hiển thị; chỉ quy cách và SKU riêng `Active` được liệt kê. Catalog chỉ dành cho người đã đăng nhập.
- **BR-02:** **SKU thành phẩm riêng chỉ hiển thị với khách sở hữu** — trong danh sách, tìm kiếm, gợi ý, trang chi tiết và mẫu thiết kế của tôi. Với khách khác, SKU đó coi như không tồn tại.
- **BR-03:** **Tình trạng còn hàng và mẫu in chờ duyệt chỉ trả cho khách có hồ sơ `Active`**. Khách `Blocked` / `Inactive` không đăng nhập được ([18](../18-customer-information/README.md)); khách `Pending Activation` đăng nhập được nhưng không thấy tồn.
- **BR-04:** **Sàn không hiển thị giá** — không có giá, bậc giá hay MOQ ở bất cứ màn hình catalog nào (danh sách, tìm kiếm, gợi ý, trang chi tiết, mẫu thiết kế của tôi), kể cả với SKU riêng. Khách chỉ thấy giá trên **báo giá / đơn / hợp đồng** do Sale lập (trang chứng từ trên cổng hoặc thẻ trong chat — [14](../14-cart-and-order/README.md)). Giá nội bộ (bảng giá riêng của khách sở hữu cho SKU riêng) là **nguồn giá gợi ý duy nhất**; mỗi (khách, SKU riêng) có một bảng; không có giá gốc, không có giá nhóm; quy cách không có giá.
- **BR-05:** Giá nội bộ có **1–20 bậc** theo **số thùng**; bậc đầu là **MOQ gợi ý** (số thùng). Khi điền sẵn vào báo giá, đơn giá theo bậc chọn bằng **tổng số thùng của SKU trong báo giá**; MOQ ràng buộc là MOQ trên báo giá ([14](../14-cart-and-order/README.md) BR-07).
- **BR-06:** Mọi giá là **VND nguyên dương**, **theo cái**; thành tiền dòng = số thùng × số cái / thùng × đơn giá / cái ([14](../14-cart-and-order/README.md) BR-08). Không có thời hạn hiệu lực, không có duyệt giá nội bộ. Giá nội bộ chỉ là **giá gợi ý**; giá được chốt trên báo giá / đơn ([14](../14-cart-and-order/README.md)).
- **BR-07:** Tình trạng còn hàng tính từ **ATP toàn hệ thống** của SKU riêng (định nghĩa ở [Glossary](../../glossary/README.md), [20 BR-01](../../warehouse/20-inventory-control/README.md)); **không bao giờ hiện số ATP**. ✅ Đã chốt 2026-11-05 ([open-questions C3](../../open-questions/README.md)): chỉ báo **giữ tính theo mọi kho**, không theo kho phục vụ khu vực giao; giữ hàng cho đơn chỉ dùng tồn **tại kho xuất** ([17](../17-order-management/README.md) BR-22, [20 BR-01](../../warehouse/20-inventory-control/README.md)), và khi Sale chọn kho xuất hệ thống **ưu tiên gợi ý kho đang có đủ hàng** — chỉ báo trên sàn không cam kết hàng ở kho xuất (§4.4).
- **BR-08:** **Khách không đặt hàng trên catalog.** Mọi lối "đặt hàng", "yêu cầu báo giá", "đặt lại", "yêu cầu in theo mẫu" đều mở hội thoại với Sale gắn ngữ cảnh ([16](../16-sales-chat/README.md)); đơn do Sale tạo từ báo giá đã chấp nhận ([14](../14-cart-and-order/README.md)).
- **BR-09:** Slug khoá sau lần xuất bản đầu; nội dung catalog của sản phẩm `Discontinued` không sửa được.
- **BR-10:** Chỉ Quản lý kinh doanh / System Admin khai báo giá nội bộ và xuất bản; Sale chỉ xem giá nội bộ. Khách không bao giờ xem được giá nội bộ.
- **BR-11:** **Quy cách chỉ để xem và yêu cầu in**: không có giá, không có chỉ báo còn hàng, không được báo giá / đặt trực tiếp. Muốn mua, khách yêu cầu in theo mẫu qua chat; chỉ SKU riêng sinh từ thiết kế khách đã xác nhận mới được báo giá ([12](../12-product-design/README.md), [14](../14-cart-and-order/README.md)).
- **BR-12:** **Khách đặt theo số thùng, không đặt lẻ.** Mỗi SKU riêng hiển thị **số cái chuẩn / thùng** ([01](../../warehouse/01-product-creation/README.md)); mọi số lượng khách thấy trên catalog (tình trạng còn hàng) và trên báo giá (MOQ, bậc giá) tính theo thùng.
- **BR-13:** **Duyệt mẫu in trên sàn** chỉ dành cho **khách sở hữu** SKU của Đơn sản xuất, chỉ khi Đơn sản xuất ở `Sample Pending` và vòng mẫu đang chờ khách. Khách chỉ chọn **Duyệt** hoặc **Yêu cầu in lại** (bắt buộc ghi chú); mỗi vòng nhận đúng một kết quả, kết quả đã ghi (trên sàn hoặc qua thẻ chat) thì không đổi được. Quy tắc vòng mẫu, lý do in lại, số lần in mẫu miễn phí thuộc [17](../17-order-management/README.md). ✅ Đã chốt 2026-11-05: mỗi vòng mẫu chỉ nhận một kết quả, đã ghi thì không đổi — xem [open-questions](../../open-questions/README.md).

### Edge case

| Tình huống | Hướng xử lý |
|------------|-------------|
| Hết hàng ngay khi khách đang xem | Chỉ báo cập nhật sau tối đa 10 giây; Sale kiểm lại tồn khi lập đơn ([14](../14-cart-and-order/README.md)) |
| Sàn báo `In stock` nhưng hàng nằm rải ở nhiều kho, không kho nào đủ cho đơn | Chỉ báo tính theo mọi kho (BR-07). Khi đơn `Confirmed`, Sale chọn kho xuất theo gợi ý (ưu tiên kho đủ hàng, rồi kho còn chỗ); phần thiếu tại kho xuất đi Đơn sản xuất ([17](../17-order-management/README.md) §4.2) |
| Khách muốn mua SKU riêng `Out of stock` | Khách vẫn mở chat với Sale; Sale báo thời gian có hàng, đơn thiếu hàng được bổ sung qua Đơn sản xuất ([17](../17-order-management/README.md)) |
| Khách muốn đặt thẳng một quy cách (chưa có thiết kế) | Không có giá, không báo giá được (BR-11); khách bấm "Yêu cầu in theo mẫu" → luồng thiết kế ([12](../12-product-design/README.md)) |
| Khách chưa có SKU riêng nào | Catalog chỉ hiện sản phẩm và quy cách; mục mẫu thiết kế của tôi trống hoặc chỉ có bản chưa xác nhận; khách liên hệ Sale qua chat |
| Sản phẩm bị ẩn khi khách có link trực tiếp | Trang sản phẩm không xem được nữa; SKU riêng của khách vẫn xem được trong mẫu thiết kế của tôi |
| Quản lý kinh doanh đổi giá nội bộ sau khi Sale đã gửi báo giá | Báo giá đã gửi giữ giá của nó; giá nội bộ mới chỉ gợi ý cho báo giá lập sau ([14](../14-cart-and-order/README.md)) |
| Khách hỏi giá trên sàn | Sàn không hiện giá (BR-04); khách bấm "Yêu cầu báo giá" → chat với Sale, nhận báo giá |
| Khách có nhiều SKU riêng, một số hết hàng | Mỗi SKU riêng có chỉ báo riêng |
| SKU riêng vừa sinh, chưa có giá nội bộ | Sale tự nhập bậc giá trên báo giá; khách không thấy khác biệt trên sàn |
| Khách duyệt mẫu in trên sàn trong khi Sale vừa ghi kết quả qua thẻ chat | Vòng mẫu đã có kết quả; thao tác sau bị từ chối, màn hình hiện kết quả hiện tại (BR-13) |
| Khách xem mẫu in và muốn đổi logo / màu (không phải lỗi in) | Không chọn được "sửa file" ở bước duyệt mẫu; khách mở chat với Sale → bản sửa thiết kế ở [12](../12-product-design/README.md), SKU riêng mới, đổi SKU dòng đơn theo [17](../17-order-management/README.md) |
| Mẫu in chờ duyệt nhưng khách không phản hồi | Đơn sản xuất ở lại `Sample Pending`; Sale nhắc khách qua [chat](../16-sales-chat/README.md) |
| Khách cần một số cái không chia hết cho số cái / thùng | Không bán lẻ (BR-12); khách trao đổi với Sale qua chat để chọn số thùng nguyên |
| Khách gõ đúng mã SKU riêng của khách khác | Không tìm thấy (BR-02) |
| Khách mới chưa kích hoạt (`Pending Activation`) | Xem được catalog nhưng không thấy tình trạng còn hàng; vẫn mở chat với Sale được |
| Kết quả tìm kiếm rỗng | Gõ sai nhẹ vẫn tìm theo tên; nếu vẫn rỗng, bỏ bớt từ khoá hoặc danh mục |
| Bản hiển thị chưa kịp cập nhật sau khi ẩn | Tìm kiếm kiểm lại trạng thái thật; bản lỗi được thử lại mỗi 30 giây |
| Danh mục bị ngưng | Cả nhánh con biến khỏi cây; lọc theo danh mục đó báo không tìm thấy |

---

## 7. Liên kết với các module khác

| Module | Quan hệ |
|--------|---------|
| [01 · Product Creation](../../warehouse/01-product-creation/README.md) | Nguồn sản phẩm `CUSTOMIZABLE`, quy cách, ảnh đã đăng, cây danh mục; điều kiện xuất bản; trạng thái `Published`; SKU riêng của khách thuộc sản phẩm `CUSTOMIZABLE` và quy cách thùng (số cái chuẩn / thùng) |
| [20 · Inventory Control](../../warehouse/20-inventory-control/README.md) | ATP toàn hệ thống (mọi kho) của SKU riêng (theo số thùng) để tính chỉ báo còn hàng; giữ hàng cho đơn chỉ dùng tồn tại kho xuất |
| [12 · Quản lý mẫu & thiết kế](../12-product-design/README.md) | Mẫu thiết kế của khách và SKU riêng sinh khi khách xác nhận; khách xác nhận / yêu cầu sửa thiết kế từ mục mẫu thiết kế của tôi; khách cần sửa file sau khi xem mẫu in → bản sửa |
| [14 · Báo giá, Đơn hàng & Hợp đồng](../14-cart-and-order/README.md) | Giá nội bộ của khách làm giá gợi ý của báo giá; giá chỉ hiện với khách trên báo giá / đơn / hợp đồng; yêu cầu báo giá và đặt lại đi qua Sale |
| [16 · Sales Chat](../16-sales-chat/README.md) | Khách mở chat gắn ngữ cảnh sản phẩm / quy cách / SKU riêng / mẫu thiết kế / đơn cũ; Sale gửi link sản phẩm / SKU riêng; thẻ duyệt mẫu in (cùng thao tác với mục mẫu thiết kế của tôi) |
| [17 · Order Management](../17-order-management/README.md) | Đơn cũ để khách mở chat đặt lại; hàng hết được bổ sung qua Đơn sản xuất; **mẫu in chờ duyệt** của Đơn sản xuất `Sample Pending` và kết quả Duyệt / Yêu cầu in lại; hàng dư / hàng trả của SKU riêng làm tồn khả dụng |
| [18 · Customer Information](../18-customer-information/README.md) | Trạng thái hồ sơ khách quyết định xem tình trạng còn hàng; giá nội bộ gắn hồ sơ khách |
