# 15 — Thanh toán & Công nợ khách sỉ (Payment & Receivables)

> Đã chốt 2026-10-06: hệ thống **chỉ bán sỉ (B2B)**. Khách sỉ thanh toán bằng **chuyển khoản**, theo một trong ba phương thức được cấu hình cho từng khách: **trả trước 100%**, **đặt cọc**, **công nợ có hạn mức**. Cổng thanh toán online (thẻ / ví) và COD bán lẻ **tạm gác**.

## 1. Mục đích (Purpose)

Xử lý **thu tiền cho đơn bán sỉ** và **công nợ phải thu** của khách: xác nhận tiền chuyển khoản (trả trước, tiền cọc, phần còn lại), ghi nhận và theo dõi **công nợ có hạn mức & kỳ hạn**, các luồng ngược (**hoàn tiền**) và **đối soát** với sao kê ngân hàng.

Payment quyết định thời điểm đơn hàng **đủ điều kiện thanh toán** để [xác nhận](../14-cart-and-order/README.md); đơn đặt cọc chỉ được release sang [sản xuất](../../warehouse/19-production/README.md) khi tiền cọc đã về.

---

## 2. Actor liên quan

| Actor | Vai trò |
|-------|---------|
| **Khách hàng sỉ** | Chuyển khoản theo đơn / theo kỳ công nợ; xem công nợ và lịch sử thanh toán trên cổng khách hàng |
| **Nhân viên Sale (Sales Staff)** | Gửi thông tin thanh toán cho khách, nhắc thanh toán / nhắc công nợ qua [chat](../16-sales-chat/README.md) |
| **Kế toán (Accountant)** | Đối chiếu sao kê, xác nhận tiền về, theo dõi công nợ phải thu, duyệt hoàn tiền |
| **Người giữ quyền duyệt hạn mức** | Duyệt đơn công nợ vượt hạn mức; cấu hình hạn mức, kỳ hạn cho khách |
| **Hệ thống (Payment Service)** | Tạo yêu cầu thanh toán, ghi giao dịch, cập nhật `payment_status`, tính dư nợ, cảnh báo quá hạn |

---

## 3. Input / Output

### Input

| Trường | Mô tả | Bắt buộc |
|--------|-------|----------|
| Order + số tiền phải thu | Từ [14 Quote & Order](../14-cart-and-order/README.md) | Có |
| Phương thức thanh toán của đơn | Trả trước 100% / đặt cọc / công nợ — phải nằm trong các phương thức được phép của khách ([18](../18-customer-information/README.md)) | Có |
| % cọc | Theo cấu hình khách hoặc báo giá | Với đặt cọc |
| Hạn mức & kỳ hạn công nợ | Theo hồ sơ khách | Với công nợ |
| Sao kê / mã tham chiếu chuyển khoản | Nội dung chuyển khoản = mã đơn (hoặc mã kỳ công nợ) | Có (khi xác nhận tiền) |
| Yêu cầu hoàn tiền | Order, số tiền, lý do | Khi refund |

### Output

| Kết quả | Dùng bởi |
|---------|----------|
| **Payment transaction** (mã giao dịch, số tiền, loại: trả trước / cọc / phần còn lại / thanh toán công nợ / hoàn) | 17 Order Management, kế toán |
| **Cập nhật trạng thái đơn**: `Confirmed` / `Payment Failed`; `payment_status` | 14, 17 |
| **Khoản công nợ phải thu** theo đơn công nợ (số tiền, ngày đến hạn) | Kế toán, khách (trên cổng) |
| **Dư nợ của khách** và cảnh báo quá hạn | Kiểm hạn mức khi đặt đơn mới ([14](../14-cart-and-order/README.md)), Sale |
| **Refund transaction** | 17, kế toán |
| **Dữ liệu đối soát** với sao kê ngân hàng | Kế toán |

---

## 4. Các luồng thanh toán (Payment Flows)

### 4.1. Trả trước 100% bằng chuyển khoản

1. Order `Pending Payment` → hệ thống tạo **yêu cầu thanh toán**, hiển thị thông tin tài khoản & **nội dung chuyển khoản = mã đơn**; transaction `Awaiting Confirmation`.
2. Khách chuyển khoản.
3. Kế toán **đối chiếu sao kê**, xác nhận đã nhận đủ → transaction `Captured` → `payment_status = Paid` → Order `Confirmed`.
4. Quá hạn chưa nhận tiền → nhắc khách; hết thời hạn giữ đơn → Order `Cancelled`, giải phóng hàng.

### 4.2. Đặt cọc (Deposit)

1. Order `Pending Payment` → yêu cầu **tiền cọc** = % giá trị đơn (theo cấu hình khách hoặc báo giá).
2. Kế toán xác nhận tiền cọc về → transaction cọc `Captured` → `payment_status = Partially Paid` → Order `Confirmed`. **Chỉ khi đó** đơn mới được release sang [sản xuất](../../warehouse/19-production/README.md).
3. **Phần còn lại** thu theo chính sách của khách: chuyển khoản trước khi giao, thu khi giao (ghi trên nhãn kiện / biên bản bàn giao), hoặc chuyển thành **công nợ** nếu khách được phép.
4. Đủ tiền → `payment_status = Paid`.

### 4.3. Công nợ có hạn mức (Credit / Receivable)

1. Đơn chọn phương thức công nợ → hệ thống kiểm **dư nợ hiện tại + giá trị đơn ≤ hạn mức** ([14](../14-cart-and-order/README.md) BR-06).
   - Trong hạn mức → Order `Confirmed`, `payment_status = On Credit`.
   - Vượt → Order `On Hold`; người có quyền duyệt chấp thuận (→ `Confirmed`) hoặc từ chối (khách chuyển sang trả trước / đặt cọc, hoặc huỷ).
2. Khi giao hàng (`Delivered`) → ghi **khoản công nợ phải thu** cho đơn, **ngày đến hạn = ngày giao + kỳ hạn** của khách.
3. Khách chuyển khoản trả nợ (theo đơn hoặc gộp nhiều đơn); kế toán đối chiếu → phân bổ vào các khoản → khoản `Paid` / `Partially Paid`.
4. Quá hạn → khoản `Overdue`; nhắc khách; theo chính sách có thể **chặn đơn công nợ mới** cho tới khi thanh toán phần quá hạn.

### 4.4. Hoàn tiền (Refund)

1. Nguồn phát sinh (bán sỉ, sản phẩm in theo thiết kế):
   - Khách huỷ đơn **trước khi in** → hoàn tiền cọc / tiền trả trước (trừ phí theo chính sách nếu có);
   - Huỷ khi **đang in** → theo chính sách [17](../17-order-management/README.md) (từ chối huỷ, hoặc huỷ có trừ phí sản xuất);
   - Hàng **lỗi sản xuất** / giao thiếu được duyệt → hoàn một phần hoặc làm lại hàng.
2. Hoàn bằng chuyển khoản về tài khoản của khách; với đơn công nợ chưa thu → **ghi giảm công nợ** thay vì chuyển tiền.
3. Refund vượt hạn mức → cần kế toán / người có quyền duyệt.
4. Cập nhật đơn: `Refunded` / `Partially Refunded`; ghi log & thông báo khách.

### 4.5. Đối soát (Reconciliation)

- **Sao kê ngân hàng:** đối chiếu từng khoản tiền về với yêu cầu thanh toán / khoản công nợ theo mã tham chiếu.
- **Tiền thu khi giao** (phần còn lại của đơn đặt cọc, nếu có): đối chiếu với biên bản bàn giao.
- Chênh lệch (thiếu, thừa, sai nội dung) → tạo ticket xử lý; không đóng tài chính đơn cho tới khi khớp.

> ⚠️ Giả định: ngưỡng hạn mức, kỳ hạn, % cọc mặc định, chính sách chặn khi quá hạn chưa chốt — câu hỏi D13. Xem [open-questions](../../open-questions/README.md).

---

## 5. Trạng thái / Vòng đời (Lifecycle)

### 5.1. Trạng thái Payment Transaction

| Trạng thái | Ý nghĩa |
|------------|---------|
| `Awaiting Confirmation` | Đã phát hành yêu cầu thanh toán, chờ kế toán xác nhận tiền về |
| `Captured` | Đã nhận tiền |
| `Cancelled` | Hết hạn / huỷ trước khi nhận tiền |
| `Refunded` / `Partially Refunded` | Đã hoàn toàn phần / một phần |

```mermaid
stateDiagram-v2
    [*] --> AwaitingConfirmation: Phát hành yêu cầu thanh toán
    AwaitingConfirmation --> Captured: Kế toán xác nhận tiền về
    AwaitingConfirmation --> Cancelled: Hết hạn
    Captured --> Refunded: Hoàn toàn phần
    Captured --> PartiallyRefunded: Hoàn một phần
    Captured --> [*]
    Cancelled --> [*]
    Refunded --> [*]
```

### 5.2. Trạng thái thanh toán của Order (payment_status)

| Trạng thái | Ý nghĩa |
|------------|---------|
| `Unpaid` | Chưa thu (đơn trả trước / đặt cọc vừa tạo) |
| `Partially Paid` | Đã nhận cọc, còn phần còn lại |
| `On Credit` | Đơn công nợ; tiền thu theo kỳ hạn |
| `Paid` | Đã thu đủ |
| `Refunded` / `Partially Refunded` | Đã hoàn |

### 5.3. Trạng thái khoản công nợ phải thu (Receivable)

| Trạng thái | Ý nghĩa | Chuyển tiếp |
|------------|---------|-------------|
| `Open` | Đã giao, chưa tới hạn | → `Partially Paid`, → `Paid`, → `Overdue` |
| `Partially Paid` | Đã trả một phần | → `Paid`, → `Overdue` |
| `Overdue` | Quá hạn chưa trả đủ | → `Partially Paid`, → `Paid` |
| `Paid` | Đã tất toán | *(cuối)* |

```mermaid
stateDiagram-v2
    [*] --> Open: Giao hàng đơn công nợ
    Open --> PartiallyPaid: Trả một phần
    Open --> Paid: Trả đủ
    Open --> Overdue: Quá hạn
    PartiallyPaid --> Paid
    PartiallyPaid --> Overdue
    Overdue --> PartiallyPaid
    Overdue --> Paid
    Paid --> [*]
```

---

## 6. Ràng buộc nghiệp vụ (Business Rules) & Edge case

### Business rules

- **BR-01:** Đơn chỉ dùng phương thức thanh toán **được phép cho khách** đó.
- **BR-02:** Đơn trả trước chỉ `Confirmed` khi đã nhận đủ tiền; đơn đặt cọc chỉ `Confirmed` (và chỉ được release sang sản xuất) khi **đã nhận đủ tiền cọc**.
- **BR-03:** Đơn công nợ chỉ `Confirmed` khi **dư nợ + giá trị đơn ≤ hạn mức**, hoặc được người có quyền duyệt vượt hạn mức.
- **BR-04:** Tiền chuyển khoản chỉ được ghi nhận khi kế toán **đối chiếu sao kê**; không ghi nhận theo lời khách báo.
- **BR-05:** Refund không vượt số tiền đã thu của đơn; với đơn công nợ chưa thu thì ghi giảm công nợ.
- **BR-06:** Mọi giao dịch (thu & hoàn) là **bất biến**, chỉ thêm giao dịch mới để điều chỉnh; đầy đủ audit log.
- **BR-07:** Idempotency: một khoản tiền về (cùng mã tham chiếu sao kê) không được ghi nhận hai lần.
- **BR-08:** Số tiền thu khi giao (nếu có) ghi trên nhãn kiện / biên bản bàn giao = phần còn phải thu của đơn (đã trừ cọc).
- **BR-09:** Tiền khách trả công nợ được phân bổ vào **khoản đến hạn sớm nhất trước**, trừ khi khách ghi rõ trả cho đơn nào.

### Edge case

| Tình huống | Hướng xử lý |
|------------|-------------|
| Khách chuyển thiếu so với tiền cọc / trả trước | Ghi nhận phần đã nhận; đơn vẫn chờ; nhắc khách chuyển nốt |
| Khách chuyển thừa | Ghi có cho khách: trừ vào đơn sau / công nợ, hoặc hoàn lại theo yêu cầu |
| Nội dung chuyển khoản sai / thiếu mã đơn | Kế toán đối chiếu theo số tiền + tên khách; xác nhận với khách trước khi ghi nhận |
| Tiền về sau khi đơn đã hết hạn & bị huỷ | Hỏi khách: tạo lại đơn hoặc hoàn tiền; không "giữ tiền không đơn" |
| Khách quá hạn công nợ muốn đặt đơn công nợ mới | Chặn hoặc `On Hold` theo chính sách D13; Sale đề xuất trả trước / đặt cọc |
| Khách huỷ đơn đã cọc khi chưa in | Hoàn cọc (trừ phí nếu chính sách quy định) |
| Khách huỷ đơn đã cọc khi đang in | Theo chính sách huỷ khi đang in ([17](../17-order-management/README.md)): giữ cọc bù phí sản xuất hoặc từ chối huỷ |
| Hàng lỗi sản xuất phát hiện sau giao | Làm lại hàng hoặc hoàn tiền / ghi giảm công nợ phần lỗi |

---

## 7. Liên kết với các module khác

| Module | Quan hệ |
|--------|---------|
| [14 · Quote & Order](../14-cart-and-order/README.md) | Nhận Order + số tiền + phương thức; kiểm hạn mức công nợ; trả kết quả → `Confirmed` / `On Hold` / `Payment Failed` |
| [17 · Order Management](../17-order-management/README.md) | Đồng bộ `payment_status`; điều kiện release; khởi tạo refund khi huỷ / lỗi sản xuất |
| [19 · Production](../../warehouse/19-production/README.md) | Đơn đặt cọc chỉ release sang sản xuất khi tiền cọc đã về |
| [09 · Shipping](../../warehouse/09-shipping/README.md) | Số tiền thu khi giao (nếu có) trên nhãn kiện / biên bản bàn giao; giao hàng làm phát sinh khoản công nợ |
| [16 · Sales Chat](../16-sales-chat/README.md) | Sale gửi thông tin thanh toán, nhắc thanh toán / công nợ |
| [18 · Customer Information](../18-customer-information/README.md) | Phương thức thanh toán được phép, hạn mức, kỳ hạn, dư nợ của khách |
| Kế toán / Ngân hàng | Đối soát sao kê |
