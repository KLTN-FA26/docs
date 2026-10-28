# 18 — Quản lý thông tin khách hàng sỉ (Customer Information Management)

> Đã chốt 2026-10-06: hệ thống **chỉ bán sỉ (B2B)**. Khách là **doanh nghiệp đã liên hệ từ trước**, do Sale tạo hồ sơ; không có đăng ký tự do và không có khách vãng lai (guest).

## 1. Mục đích (Purpose)

Quản lý **hồ sơ khách sỉ (customer profile)** và dữ liệu liên quan: thông tin doanh nghiệp (tên, mã số thuế), **người liên hệ / người đặt hàng**, **sổ địa chỉ**, **điều khoản thương mại** (phương thức thanh toán được phép, **hạn mức công nợ, kỳ hạn**, **bảng giá riêng**), **lịch sử đơn hàng & tương tác**, thiết kế và **mẫu đã duyệt**, nhóm khách, và **hồ sơ rủi ro**.

Đây là **nguồn dữ liệu khách hàng dùng chung** cho báo giá & đặt hàng ([14](../14-cart-and-order/README.md)), thanh toán ([15](../15-payment/README.md)), chăm sóc ([16](../16-sales-chat/README.md)), và điều hành đơn ([17](../17-order-management/README.md)); đồng thời là nơi thực thi **quyền riêng tư dữ liệu** của khách.

---

## 2. Actor liên quan

| Actor | Vai trò |
|-------|---------|
| **Khách hàng sỉ** | Kích hoạt tài khoản từ lời mời; quản lý người liên hệ, địa chỉ, mật khẩu, tuỳ chọn nhận tin; xem công nợ — gồm hạn mức, dư nợ, hạn mức còn lại, kỳ hạn và cờ quá hạn của chính mình *(Quyết định 21/10/2026; hồ sơ rủi ro vẫn là dữ liệu nội bộ, không hiện cho khách)*; yêu cầu tải/xoá dữ liệu |
| **Nhân viên Sale (Sales Staff)** | **Tạo hồ sơ khách sỉ** và gửi lời mời kích hoạt; xem hồ sơ (theo quyền) để tư vấn & xử lý đơn; cập nhật thông tin thay khách; thêm ghi chú |
| **Người giữ quyền duyệt hạn mức** | Thiết lập / thay đổi phương thức thanh toán được phép, hạn mức công nợ, kỳ hạn của khách |
| **Quản trị sàn (E-commerce Admin)** | Quản lý nhóm khách, phân khúc, chính sách dữ liệu, hợp nhất hồ sơ trùng; phân quyền truy cập hồ sơ (phối hợp System Admin) |
| **Hệ thống** | Xác thực, chống trùng, gắn dữ liệu đơn/tương tác vào hồ sơ, đánh giá rủi ro, thực thi yêu cầu quyền riêng tư |

---

## 3. Input / Output

### Input

| Nhóm | Dữ liệu | Nguồn |
|------|---------|-------|
| Định danh doanh nghiệp | Tên công ty, mã số thuế, địa chỉ xuất hoá đơn | Sale |
| Người liên hệ / người đặt hàng | Họ tên, email, số điện thoại, quyền trên cổng (xem / đặt hàng / duyệt mẫu) | Sale / Khách |
| Điều khoản thương mại | Phương thức thanh toán được phép (trả trước / đặt cọc / công nợ), % cọc mặc định, **hạn mức công nợ**, **kỳ hạn**, **bảng giá riêng** | Sale + người duyệt hạn mức |
| Sổ địa chỉ | Nhiều địa chỉ giao & xuất hoá đơn; địa chỉ mặc định | Khách |
| Tài khoản & bảo mật | Mật khẩu (hash), đăng nhập mạng xã hội, 2FA, phiên đăng nhập | Hệ thống |
| Tuỳ chọn (preferences) | Ngôn ngữ, nhận email/SMS marketing, kênh liên hệ ưu tiên | Khách |
| Dữ liệu phái sinh | Lịch sử báo giá & đơn, tổng chi tiêu, **dư nợ & quá hạn**, thiết kế và mẫu đã duyệt, lịch sử chat, hồ sơ rủi ro | 12, 14, 15, 16, 17, 19 |

### Output

| Kết quả | Dùng bởi |
|---------|----------|
| **Customer profile** hợp nhất | 14, 15, 16, 17, báo cáo |
| **Địa chỉ & người nhận** cho đơn | 14 Quote & Order |
| **Điều khoản thương mại** (phương thức thanh toán, hạn mức, kỳ hạn, bảng giá) | 13 Catalog, 14 Quote & Order, 15 Payment |
| **Nhóm khách** (theo quy mô, ngành…) | Pricing mặc định khi chưa có bảng giá riêng ([13](../13-catalog/README.md)), routing chat ([16](../16-sales-chat/README.md)) |
| **Hồ sơ rủi ro / blacklist** | 15 Payment, 17 Order Management |
| **Bản xuất dữ liệu cá nhân / xác nhận đã xoá** (theo yêu cầu quyền riêng tư) | Khách |
| **Lịch sử hoạt động (activity log)** của hồ sơ | Quản trị sàn, Sales Staff |

---

## 4. Luồng nghiệp vụ từng bước (Step-by-step)

### 4.1. Tạo & xác thực hồ sơ

1. **Sale tạo hồ sơ khách sỉ** — sau khi khách liên hệ: thông tin doanh nghiệp, mã số thuế, người liên hệ, địa chỉ. Hệ thống kiểm tra trùng (mã số thuế / email). Hồ sơ ở `Pending Activation`.
2. **Thiết lập điều khoản thương mại** — phương thức thanh toán được phép, % cọc, hạn mức công nợ, kỳ hạn, bảng giá riêng; hạn mức và kỳ hạn do người có quyền duyệt.
3. **Mời kích hoạt** — hệ thống gửi lời mời tới email người liên hệ; người liên hệ đặt mật khẩu → hồ sơ `Active`. **Không có đăng ký tự do và không có guest checkout.**

### 4.2. Quản lý dữ liệu hồ sơ

4. **Sổ địa chỉ** — thêm/sửa/xoá địa chỉ; đặt mặc định giao & xuất hoá đơn; validate địa chỉ (định dạng, vùng phục vụ).
5. **Người liên hệ & quyền trên cổng** — thêm/bớt người đặt hàng, người duyệt mẫu của công ty khách.
6. **Bảo mật** — đổi mật khẩu, bật 2FA, xem & thu hồi phiên đăng nhập, lịch sử đăng nhập bất thường.
7. **Tuỳ chọn nhận tin** — bật/tắt marketing theo kênh; lưu mốc thời gian đồng ý (consent) để tuân thủ quy định.

### 4.3. Dữ liệu phái sinh & phân khúc

8. **Gắn lịch sử** — mỗi đơn ([17](../17-order-management/README.md)), mỗi hội thoại ([16](../16-sales-chat/README.md)), mỗi thiết kế lưu ([12](../12-product-design/README.md)) tự động liên kết hồ sơ.
9. **Tính chỉ số** — tổng chi tiêu, số đơn, giá trị đơn trung bình, lần mua gần nhất, tỷ lệ huỷ/hoàn.
10. **Nhóm khách** — Quản trị bán hàng gán nhóm (theo quy mô, ngành, khách rủi ro). Nhóm dùng làm giá mặc định khi chưa có bảng giá riêng, và ưu tiên chăm sóc.
    - **Dư nợ** — cập nhật từ [Payment](../15-payment/README.md); quá hạn → cảnh báo Sale và ảnh hưởng đơn công nợ mới.
11. **Hồ sơ rủi ro** — cập nhật khi quá hạn công nợ kéo dài, gian lận, khiếu nại bất thường → cảnh báo cho [Payment](../15-payment/README.md) & [Order Management](../17-order-management/README.md).

### 4.4. Quyền riêng tư dữ liệu (Data privacy)

12. **Yêu cầu truy cập/tải dữ liệu** — khách yêu cầu bản sao dữ liệu cá nhân → hệ thống xuất tập hợp dữ liệu.
13. **Yêu cầu chỉnh sửa** — khách sửa thông tin sai.
14. **Yêu cầu xoá (right to be forgotten)** — xoá/ẩn danh (anonymize) dữ liệu cá nhân **trong giới hạn nghĩa vụ lưu trữ** (chứng từ đơn hàng/kế toán phải giữ theo luật → ẩn danh thay vì xoá cứng).
15. **Rút lại đồng ý** — tắt marketing; ghi nhận thời điểm.

### 4.5. Hợp nhất hồ sơ trùng (Merge)

16. Phát hiện trùng (cùng email/SĐT/địa chỉ) → Quản trị sàn xem xét → **merge**: giữ một hồ sơ chính, gộp địa chỉ/đơn/tương tác, giữ log merge.

---

## 5. Trạng thái / Vòng đời (Lifecycle)

### Trạng thái hồ sơ khách

| Trạng thái | Ý nghĩa | Chuyển tiếp | Ai |
|------------|---------|-------------|----|
| `Pending Activation` | Sale đã tạo hồ sơ, chờ người liên hệ kích hoạt từ lời mời | → `Active`, → `Closed` | Sale / Khách |
| `Active` | Bình thường | → `Suspended`, → `Closed`, → `Anonymized` | — |
| `Suspended` | Tạm khoá (gian lận, vi phạm) — không đăng nhập/mua | → `Active`, → `Closed` | Quản trị sàn |
| `Closed` | Khách yêu cầu đóng tài khoản; dữ liệu còn theo nghĩa vụ lưu trữ | → `Anonymized` | Khách / Quản trị sàn |
| `Anonymized` | Đã ẩn danh dữ liệu cá nhân theo yêu cầu xoá | *(cuối)* | Hệ thống |
| `Merged` | Đã gộp vào hồ sơ khác | *(cuối)* | Quản trị sàn |

---

## 6. Ràng buộc nghiệp vụ (Business Rules) & Edge case

### Business rules

- **BR-01:** **Mã số thuế** là khoá định danh của khách doanh nghiệp; email là khoá đăng nhập của từng người liên hệ. Không cho hai hồ sơ `Active` trùng mã số thuế.
- **BR-01a:** Chỉ **Sale** tạo hồ sơ khách; không có đăng ký tự do và guest checkout. Chỉ hồ sơ `Active` mới xem giá, nhận báo giá và đặt hàng.
- **BR-02:** Hạn mức công nợ, kỳ hạn và phương thức thanh toán được phép chỉ do **người có quyền duyệt** thay đổi; mọi thay đổi được log.
- **BR-03:** Truy cập hồ sơ khách theo **quyền tối thiểu**: Sales Staff chỉ xem dữ liệu cần cho công việc; mọi truy cập nhạy cảm được log.
- **BR-04:** Marketing chỉ gửi khi có **consent còn hiệu lực**; lưu mốc thời gian & nguồn consent.
- **BR-05:** Xoá dữ liệu: chứng từ đơn hàng/thanh toán phải giữ theo thời hạn luật định → thực hiện **ẩn danh** (tách thông tin cá nhân khỏi chứng từ), không xoá chứng từ.
- **BR-06:** Hồ sơ `Suspended`/`Closed` không thể tạo đơn mới; đơn đang chạy vẫn xử lý tới khi kết thúc.
- **BR-07:** Merge hồ sơ là thao tác **có kiểm soát & log**; không tự động merge khi chưa xác nhận.
- **BR-08:** Hồ sơ rủi ro chỉ ảnh hưởng **quy trình xử lý** (thêm bước kiểm tra), không tự động từ chối đơn trừ khi blacklist rõ ràng.

### Edge case

| Tình huống | Hướng xử lý |
|------------|-------------|
| Người liên hệ của khách nghỉ việc | Khách / Sale vô hiệu hoá người liên hệ đó; thêm người mới; lịch sử đơn vẫn gắn công ty |
| Một người tự đăng ký trên cổng mà chưa liên hệ Sale | Không có đăng ký tự do: chuyển thành yêu cầu liên hệ cho Sale xử lý |
| Khách đổi email | Xác thực email mới; giữ email cũ như liên hệ phụ trong thời gian chuyển tiếp |
| Hai người cùng nhà dùng chung email | Địa chỉ & người nhận tách theo từng đơn; hồ sơ vẫn là một |
| Yêu cầu xoá dữ liệu khi còn đơn đang giao / còn công nợ | Hoãn ẩn danh phần liên quan tới khi đơn đóng & công nợ tất toán; ẩn danh phần còn lại |
| Nghi tài khoản bị chiếm (login lạ) | Buộc đổi mật khẩu, thu hồi phiên, tạm `Suspended` nếu cần |
| Trùng hồ sơ nhưng lịch sử mâu thuẫn (địa chỉ/tên khác) | Quản trị sàn review thủ công trước khi merge; giữ cả hai bộ địa chỉ |
| Khách doanh nghiệp nhiều người đặt hàng | Hồ sơ công ty + danh sách người liên hệ/đặt hàng; phân quyền theo người |
| Khách quá hạn công nợ kéo dài | Cập nhật hồ sơ rủi ro; người có quyền có thể hạ hạn mức hoặc chỉ cho trả trước; cảnh báo Sale |

---

## 7. Liên kết với các module khác

| Module | Quan hệ |
|--------|---------|
| [14 · Quote & Order](../14-cart-and-order/README.md) | Cung cấp thông tin khách, sổ địa chỉ, người nhận, phương thức thanh toán được phép, hạn mức |
| [15 · Payment](../15-payment/README.md) | Hạn mức, kỳ hạn, dư nợ; cập nhật hồ sơ rủi ro khi quá hạn |
| [16 · Sales Chat](../16-sales-chat/README.md) | Hồ sơ & lịch sử tương tác hiển thị cho agent; lưu vết hội thoại vào hồ sơ |
| [17 · Order Management](../17-order-management/README.md) | Lịch sử đơn gắn hồ sơ; hồ sơ rủi ro ảnh hưởng cách xử lý đơn |
| [12 · Product Design](../12-product-design/README.md) | Thiết kế nháp/đã lưu gắn theo tài khoản khách |
| [13 · Catalog](../13-catalog/README.md) | Bảng giá riêng / nhóm khách quyết định giá hiển thị |
| [00 · System Overview §3](../../00-system-overview/README.md) | Persona khách hàng |
