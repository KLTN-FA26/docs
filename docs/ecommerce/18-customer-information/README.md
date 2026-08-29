# 18 — Quản lý thông tin khách hàng (Customer Information Management)

## 1. Mục đích (Purpose)

Quản lý **hồ sơ khách hàng (customer profile)** và dữ liệu liên quan: thông tin định danh & liên hệ, **sổ địa chỉ**, phương thức thanh toán đã lưu (token), **lịch sử đơn hàng & tương tác**, thiết kế đã lưu, nhóm khách / phân khúc, và **hồ sơ rủi ro**.

Đây là **nguồn dữ liệu khách hàng dùng chung** cho checkout ([14](../14-cart-and-order/README.md)), thanh toán ([15](../15-payment/README.md)), chăm sóc ([16](../16-sales-chat/README.md)), và điều hành đơn ([17](../17-order-management/README.md)); đồng thời là nơi thực thi **quyền riêng tư dữ liệu** của khách.

---

## 2. Actor liên quan

| Actor | Vai trò |
|-------|---------|
| **Khách hàng** | Tự quản lý hồ sơ, địa chỉ, phương thức thanh toán, mật khẩu, tuỳ chọn nhận tin; yêu cầu tải/xoá dữ liệu |
| **Nhân viên Sale (Sales Staff)** | Xem hồ sơ (theo quyền) để tư vấn & xử lý đơn; cập nhật thông tin thay khách khi được yêu cầu; thêm ghi chú |
| **Quản trị sàn (E-commerce Admin)** | Quản lý nhóm khách, phân khúc, chính sách dữ liệu, hợp nhất hồ sơ trùng; phân quyền truy cập hồ sơ (phối hợp System Admin) |
| **Hệ thống** | Xác thực, chống trùng, gắn dữ liệu đơn/tương tác vào hồ sơ, đánh giá rủi ro, thực thi yêu cầu quyền riêng tư |

---

## 3. Input / Output

### Input

| Nhóm | Dữ liệu | Nguồn |
|------|---------|-------|
| Định danh & liên hệ | Họ tên, email, số điện thoại, ngày sinh (tuỳ chọn), loại khách (cá nhân / doanh nghiệp), mã số thuế | Khách / Sale |
| Sổ địa chỉ | Nhiều địa chỉ giao & xuất hoá đơn; địa chỉ mặc định | Khách |
| Phương thức thanh toán đã lưu | Token thẻ/ví (không lưu số thẻ thật) | [15 Payment](../15-payment/README.md) |
| Tài khoản & bảo mật | Mật khẩu (hash), đăng nhập mạng xã hội, 2FA, phiên đăng nhập | Hệ thống |
| Tuỳ chọn (preferences) | Ngôn ngữ, nhận email/SMS marketing, kênh liên hệ ưu tiên | Khách |
| Dữ liệu phái sinh | Lịch sử đơn, tổng chi tiêu, tần suất mua, thiết kế đã lưu, lịch sử chat, điểm/hạng thành viên, hồ sơ rủi ro | 12, 14, 16, 17 |

### Output

| Kết quả | Dùng bởi |
|---------|----------|
| **Customer profile** hợp nhất | 14, 15, 16, 17, báo cáo |
| **Địa chỉ & người nhận** cho checkout | 14 Cart & Order |
| **Phân khúc / nhóm khách** (VIP, doanh nghiệp, khách mới…) | Pricing, khuyến mãi ([13](../13-catalog/README.md)), routing chat ([16](../16-sales-chat/README.md)) |
| **Hồ sơ rủi ro / blacklist** | 15 Payment, 17 Order Management |
| **Bản xuất dữ liệu cá nhân / xác nhận đã xoá** (theo yêu cầu quyền riêng tư) | Khách |
| **Lịch sử hoạt động (activity log)** của hồ sơ | Quản trị sàn, Sales Staff |

---

## 4. Luồng nghiệp vụ từng bước (Step-by-step)

### 4.1. Tạo & xác thực hồ sơ

1. **Đăng ký** — khách tạo tài khoản (email/SĐT + mật khẩu, hoặc social login). Hệ thống kiểm tra trùng (email/SĐT đã tồn tại?).
2. **Xác thực** — xác minh email/SĐT (OTP/link). Chưa xác thực vẫn mua được (guest → tài khoản) tuỳ chính sách.
3. **Guest checkout** — khách mua không đăng ký; hệ thống tạo **hồ sơ nhẹ (guest profile)** theo email; sau này khách đăng ký cùng email → **gắn kết lịch sử**.

### 4.2. Quản lý dữ liệu hồ sơ

4. **Sổ địa chỉ** — thêm/sửa/xoá địa chỉ; đặt mặc định giao & xuất hoá đơn; validate địa chỉ (định dạng, vùng phục vụ).
5. **Phương thức thanh toán** — lưu token qua [Payment](../15-payment/README.md) (không lưu số thẻ); đặt mặc định; xoá khi cần.
6. **Bảo mật** — đổi mật khẩu, bật 2FA, xem & thu hồi phiên đăng nhập, lịch sử đăng nhập bất thường.
7. **Tuỳ chọn nhận tin** — bật/tắt marketing theo kênh; lưu mốc thời gian đồng ý (consent) để tuân thủ quy định.

### 4.3. Dữ liệu phái sinh & phân khúc

8. **Gắn lịch sử** — mỗi đơn ([17](../17-order-management/README.md)), mỗi hội thoại ([16](../16-sales-chat/README.md)), mỗi thiết kế lưu ([12](../12-product-design/README.md)) tự động liên kết hồ sơ.
9. **Tính chỉ số** — tổng chi tiêu, số đơn, giá trị đơn trung bình, lần mua gần nhất, tỷ lệ huỷ/hoàn.
10. **Phân khúc / nhóm** — hệ thống/Quản trị sàn gán nhóm (VIP theo chi tiêu, khách doanh nghiệp, khách mới, khách rủi ro). Nhóm ảnh hưởng giá, ưu đãi, ưu tiên chăm sóc.
11. **Hồ sơ rủi ro** — cập nhật khi có chargeback, gian lận, tỷ lệ RTO cao, khiếu nại bất thường → cảnh báo cho [Payment](../15-payment/README.md) & [Order Management](../17-order-management/README.md).

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
| `Guest` | Hồ sơ nhẹ từ guest checkout, chưa có tài khoản | → `Active` (đăng ký cùng email) | Khách |
| `Pending Verification` | Đã đăng ký, chưa xác thực email/SĐT | → `Active` | Khách |
| `Active` | Bình thường | → `Suspended`, → `Closed`, → `Anonymized` | — |
| `Suspended` | Tạm khoá (gian lận, vi phạm) — không đăng nhập/mua | → `Active`, → `Closed` | Quản trị sàn |
| `Closed` | Khách yêu cầu đóng tài khoản; dữ liệu còn theo nghĩa vụ lưu trữ | → `Anonymized` | Khách / Quản trị sàn |
| `Anonymized` | Đã ẩn danh dữ liệu cá nhân theo yêu cầu xoá | *(cuối)* | Hệ thống |
| `Merged` | Đã gộp vào hồ sơ khác | *(cuối)* | Quản trị sàn |

---

## 6. Ràng buộc nghiệp vụ (Business Rules) & Edge case

### Business rules

- **BR-01:** Email (hoặc SĐT) là khoá định danh chính; không cho hai hồ sơ `Active` trùng khoá định danh.
- **BR-02:** **Không lưu** dữ liệu thẻ nhạy cảm; chỉ lưu token & 4 số cuối / thương hiệu thẻ (qua [Payment](../15-payment/README.md), tuân thủ PCI-DSS).
- **BR-03:** Truy cập hồ sơ khách theo **quyền tối thiểu**: Sales Staff chỉ xem dữ liệu cần cho công việc; mọi truy cập nhạy cảm được log.
- **BR-04:** Marketing chỉ gửi khi có **consent còn hiệu lực**; lưu mốc thời gian & nguồn consent.
- **BR-05:** Xoá dữ liệu: chứng từ đơn hàng/thanh toán phải giữ theo thời hạn luật định → thực hiện **ẩn danh** (tách thông tin cá nhân khỏi chứng từ), không xoá chứng từ.
- **BR-06:** Hồ sơ `Suspended`/`Closed` không thể tạo đơn mới; đơn đang chạy vẫn xử lý tới khi kết thúc.
- **BR-07:** Merge hồ sơ là thao tác **có kiểm soát & log**; không tự động merge khi chưa xác nhận.
- **BR-08:** Hồ sơ rủi ro chỉ ảnh hưởng **quy trình xử lý** (thêm bước kiểm tra), không tự động từ chối đơn trừ khi blacklist rõ ràng.

### Edge case

| Tình huống | Hướng xử lý |
|------------|-------------|
| Guest mua nhiều lần rồi mới đăng ký | Gắn kết mọi đơn cùng email vào hồ sơ mới; thông báo khách |
| Khách đổi email | Xác thực email mới; giữ email cũ như liên hệ phụ trong thời gian chuyển tiếp |
| Hai người cùng nhà dùng chung email | Địa chỉ & người nhận tách theo từng đơn; hồ sơ vẫn là một |
| Yêu cầu xoá dữ liệu khi còn đơn đang giao / còn công nợ | Hoãn ẩn danh phần liên quan tới khi đơn đóng & công nợ tất toán; ẩn danh phần còn lại |
| Nghi tài khoản bị chiếm (login lạ) | Buộc đổi mật khẩu, thu hồi phiên, tạm `Suspended` nếu cần |
| Trùng hồ sơ nhưng lịch sử mâu thuẫn (địa chỉ/tên khác) | Quản trị sàn review thủ công trước khi merge; giữ cả hai bộ địa chỉ |
| Khách doanh nghiệp nhiều người đặt hàng | Hồ sơ công ty + danh sách người liên hệ/đặt hàng; phân quyền theo người |
| Chargeback từ một khách "VIP theo chi tiêu" | Cập nhật hồ sơ rủi ro; nhóm phân khúc có thể bị hạ; cảnh báo OPS |

---

## 7. Liên kết với các module khác

| Module | Quan hệ |
|--------|---------|
| [14 · Cart & Order](../14-cart-and-order/README.md) | Cung cấp thông tin khách, sổ địa chỉ, người nhận cho checkout |
| [15 · Payment](../15-payment/README.md) | Lưu token phương thức thanh toán; nhận & cập nhật hồ sơ rủi ro/gian lận |
| [16 · Sales Chat](../16-sales-chat/README.md) | Hồ sơ & lịch sử tương tác hiển thị cho agent; lưu vết hội thoại vào hồ sơ |
| [17 · Order Management](../17-order-management/README.md) | Lịch sử đơn gắn hồ sơ; hồ sơ rủi ro ảnh hưởng cách xử lý đơn |
| [12 · Product Design](../12-product-design/README.md) | Thiết kế nháp/đã lưu gắn theo tài khoản khách |
| [13 · Catalog](../13-catalog/README.md) | Phân khúc khách ảnh hưởng giá & khuyến mãi hiển thị; wishlist |
| [00 · System Overview §3](../../00-system-overview/README.md) | Persona khách hàng |
