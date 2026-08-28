# 16 — Nhắn tin trực tiếp với Sale (Sales Chat / Messaging)

## 1. Mục đích (Purpose)

Kênh **hội thoại trực tiếp giữa khách hàng và nhân viên Sale** để: tư vấn sản phẩm, hỗ trợ **tinh chỉnh & chốt thiết kế** ([12](../12-product-design/README.md)), báo giá, **đặt hàng hộ** ([14](../14-cart-and-order/README.md)), gửi **link thanh toán** ([15](../15-payment/README.md)), xử lý phát sinh sau bán (đổi/huỷ, khiếu nại giao hàng).

Chat gắn ngữ cảnh (context-aware): mỗi hội thoại có thể tham chiếu sản phẩm, thiết kế, giỏ hàng, hoặc đơn hàng cụ thể.

---

## 2. Actor liên quan

| Actor | Vai trò |
|-------|---------|
| **Khách hàng** | Mở hội thoại, gửi câu hỏi/ảnh/link thiết kế, nhận tư vấn & link hành động |
| **Nhân viên Sale (Agent)** | Trả lời, tư vấn, chỉnh thiết kế hộ, tạo đơn hộ, gửi báo giá & link thanh toán, chuyển tiếp nội bộ |
| **Trưởng nhóm Sale / Supervisor** | Phân công hội thoại, giám sát SLA, tham gia khi cần escalate |
| **Hệ thống (Routing + Notification)** | Định tuyến hội thoại đến agent phù hợp/rảnh; nhắc SLA; lưu lịch sử; (tuỳ chọn) chatbot trả lời câu hỏi cơ bản |

> ⚠️ Giả định: Phạm vi là **chat 1–1 khách ↔ Sale trong nội bộ storefront**. Việc tích hợp kênh ngoài (Facebook Messenger, Zalo, email) và chatbot AI chưa được đề bài xác định — xem [open-questions](../../open-questions/README.md).

---

## 3. Input / Output

### Input

| Trường | Mô tả | Bắt buộc |
|--------|-------|----------|
| Người khởi tạo | Khách (từ PDP, trang thiết kế, trang đơn hàng, nút "Chat với Sale") hoặc Sale (chủ động) | Có |
| Ngữ cảnh đính kèm | product ID / design ID / cart ID / order ID | Không |
| Nội dung tin nhắn | Text, ảnh, file (ảnh thiết kế, ảnh sản phẩm lỗi), link | Có |
| Thông tin khách | Từ [18 Customer Information](../18-customer-information/README.md) (nếu đăng nhập) | Không |

### Output

| Kết quả | Dùng bởi |
|---------|----------|
| **Conversation / thread** — lịch sử tin nhắn, người tham gia, ngữ cảnh, trạng thái | Sale, khách, Supervisor |
| **Hành động phát sinh:** link thiết kế đã chỉnh, giỏ/đơn tạo hộ, link thanh toán, phiếu yêu cầu đổi/huỷ | 12, 14, 15, 17 |
| **Ghi chú nội bộ (internal note)** — chỉ Sale thấy | Sale, Supervisor |
| **Chỉ số:** thời gian phản hồi đầu (FRT), thời gian xử lý, mức hài lòng (CSAT) | Supervisor, báo cáo |
| **Lịch sử tương tác** gắn hồ sơ khách | [18 Customer Information](../18-customer-information/README.md) |

---

## 4. Luồng nghiệp vụ từng bước (Step-by-step)

1. **Khởi tạo hội thoại** — khách bấm "Chat với Sale" từ một trang (PDP / trình thiết kế / trang đơn). Hệ thống tạo thread, đính **ngữ cảnh** trang đó.
2. **Định tuyến (Routing)** — hệ thống gán agent: theo hàng đợi, kỹ năng (ví dụ nhóm chuyên sản phẩm in ấn), tải hiện tại, hoặc agent đã từng chăm sóc khách này.
3. **Chờ & phản hồi** — nếu ngoài giờ / không có agent: hiển thị thời gian phản hồi dự kiến, thu thập email để trả lời sau (offline message). Trong giờ: agent nhận thông báo, mở thread thấy toàn bộ ngữ cảnh + hồ sơ khách + lịch sử.
4. **Tư vấn / trao đổi** — hai bên nhắn tin, gửi ảnh/file. Agent có thể:
   - Gửi **link sản phẩm** ([13](../13-catalog/README.md)).
   - Mở **thiết kế của khách** ([12](../12-product-design/README.md)), chỉnh hộ, gửi lại phiên bản mới để khách duyệt.
   - Lập **báo giá** cho yêu cầu đặc thù (số lượng lớn, kỹ thuật in đặc biệt).
5. **Chốt đơn:**
   - Khách tự đặt: agent hướng dẫn; hoặc
   - **Đặt hộ (assisted order):** agent tạo giỏ/đơn ([14](../14-cart-and-order/README.md)), gắn thiết kế đã chốt, áp ưu đãi trong hạn mức, gửi khách **link thanh toán** ([15](../15-payment/README.md)) hoặc xác nhận COD.
6. **Hỗ trợ sau bán** — khách nhắn về đơn đang giao / bị lỗi: agent tra [Order Management](../17-order-management/README.md), chia sẻ tracking, tạo **yêu cầu đổi/huỷ/hoàn** hoặc **yêu cầu thay đổi thiết kế** (nếu chưa sản xuất).
7. **Escalate** — vấn đề vượt thẩm quyền (giảm giá lớn, bồi thường): chuyển Supervisor; thread ghi nhận lý do & người xử lý tiếp.
8. **Kết thúc & đánh giá** — agent đóng thread khi giải quyết xong; gửi khảo sát CSAT. Thread có thể **mở lại (reopen)** nếu khách nhắn tiếp trong thời hạn.
9. **Lưu vết** — toàn bộ hội thoại + hành động gắn vào [hồ sơ khách](../18-customer-information/README.md) và (nếu có) vào đơn liên quan.

---

## 5. Trạng thái / Vòng đời (Lifecycle)

### Trạng thái Conversation

| Trạng thái | Ý nghĩa | Chuyển tiếp | Ai |
|------------|---------|-------------|----|
| `New` | Vừa tạo, chưa gán agent | → `Assigned`, → `Queued` | System |
| `Queued` | Chờ agent rảnh / ngoài giờ | → `Assigned` | System |
| `Assigned` | Đã gán agent, chưa trả lời | → `Active` | System / Agent |
| `Active` | Đang trao đổi | → `Waiting on Customer`, → `Waiting on Internal`, → `Resolved` | Agent |
| `Waiting on Customer` | Chờ khách phản hồi | → `Active`, → `Auto-Closed` (quá hạn im lặng) | System |
| `Waiting on Internal` | Chờ bộ phận khác (kho, kỹ thuật, sản xuất) | → `Active` | Agent |
| `Escalated` | Đã chuyển Supervisor | → `Active`, → `Resolved` | Supervisor |
| `Resolved` | Đã xử lý xong | → `Reopened`, → `Closed` | Agent |
| `Reopened` | Khách nhắn lại sau khi resolved | → `Active` | System |
| `Closed` | Đóng vĩnh viễn (hết thời hạn reopen) | *(cuối)* | System |

---

## 6. Ràng buộc nghiệp vụ (Business Rules) & Edge case

### Business rules

- **BR-01:** Mọi hành động có hệ quả (tạo đơn hộ, áp giảm giá, gửi link thanh toán, tạo yêu cầu hoàn) phải **ghi log gắn thread & gắn người thực hiện**.
- **BR-02:** Sale chỉ áp giảm giá / ưu đãi **trong hạn mức** được cấp; vượt phải escalate.
- **BR-03:** Sửa thiết kế hộ khách: chỉ tạo **phiên bản mới**, khách phải **duyệt** trước khi dùng để đặt hàng (không tự thay bản khách đang có).
- **BR-04:** Nội bộ note không hiển thị cho khách; cần tách biệt rõ trong giao diện & dữ liệu.
- **BR-05:** Thông tin nhạy cảm (số thẻ, mật khẩu) không được yêu cầu/nhập qua chat; hệ thống nên chặn/che (masking) các mẫu nhạy cảm.
- **BR-06:** SLA phản hồi (FRT) & thời gian giải quyết được đo và cảnh báo; hội thoại quá hạn nổi bật trong hàng đợi.
- **BR-07:** Yêu cầu đổi/huỷ/hoàn tạo từ chat phải đi qua quy trình & trạng thái của [Order Management](../17-order-management/README.md) — chat không tự thay đổi đơn.

### Edge case

| Tình huống | Hướng xử lý |
|------------|-------------|
| Khách rời trang giữa chừng | Lưu thread; gửi phần trả lời qua email; khách mở lại thấy lịch sử |
| Nhiều agent cùng mở một thread | Khoá quyền trả lời cho agent đang phụ trách; người khác ở chế độ xem |
| Khách mở nhiều thread cho cùng một vấn đề | Cho phép gộp thread; giữ một thread chính, tham chiếu chéo |
| Agent nghỉ giữa hội thoại đang mở | Re-route sang agent khác; chuyển toàn bộ ngữ cảnh & note |
| Khách gửi file quá lớn / định dạng lạ | Giới hạn dung lượng & loại file; hướng dẫn nén hoặc dùng trình thiết kế |
| Khách yêu cầu điều không thể (đổi thiết kế khi đã in) | Agent giải thích chính sách; ghi nhận; đề xuất phương án (đơn mới, chiết khấu) |
| Tranh chấp "đã hứa giảm giá qua chat" | Lịch sử thread là bằng chứng; ưu đãi phải được ghi nhận chính thức trên đơn mới có hiệu lực |
| Spam / lạm dụng | Chặn/blacklist theo tài khoản/thiết bị; cảnh báo Supervisor |

---

## 7. Liên kết với các module khác

| Module | Quan hệ |
|--------|---------|
| [12 · Product Design](../12-product-design/README.md) | Chia sẻ & chỉnh thiết kế hộ khách; khách duyệt phiên bản mới |
| [13 · Catalog](../13-catalog/README.md) | Agent gửi link sản phẩm; khách hỏi về sản phẩm đang xem |
| [14 · Cart & Order](../14-cart-and-order/README.md) | Đặt hàng hộ (assisted order); gắn thiết kế; áp ưu đãi |
| [15 · Payment](../15-payment/README.md) | Gửi link thanh toán; hỗ trợ khi lỗi thanh toán |
| [17 · Order Management](../17-order-management/README.md) | Tra cứu đơn, chia sẻ tracking, tạo yêu cầu đổi/huỷ/hoàn |
| [18 · Customer Information](../18-customer-information/README.md) | Hồ sơ khách, lịch sử tương tác & mua hàng; lưu vết hội thoại |
| [09 · Shipping](../../warehouse/09-shipping/README.md) | Xử lý khiếu nại giao hàng, đơn trễ |
