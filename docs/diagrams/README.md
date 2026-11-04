# Sơ đồ luồng dữ liệu & trạng thái (Flow & State Diagrams)

Tất cả sơ đồ vẽ bằng **Mermaid**. Trên GitHub/GitLab và hầu hết trình xem Markdown, các khối ```mermaid``` được render tự động. Nếu trình xem không hỗ trợ, dán nội dung vào <https://mermaid.live>.

> **Đã chốt 2026-11-04:** kho chỉ là **kho thành phẩm**; **sản xuất nằm ngoài hệ thống** — hệ thống chỉ gửi **Yêu cầu sản xuất** ([17](../ecommerce/17-order-management/README.md)) và **nhập kho thành phẩm** ([03](../warehouse/03-receipt/README.md)); kho không kiểm tra chất lượng khi nhập, chỉ đếm đủ / thiếu / thừa. Mỗi đơn có một **hợp đồng mua bán**, khách thanh toán theo **các đợt của hợp đồng** ([14](../ecommerce/14-cart-and-order/README.md), [15](../ecommerce/15-payment/README.md)).
>
> **Đã chốt 2026-11-04 (bổ sung):** **không còn hàng trơn** — mọi dòng đơn là **SKU thành phẩm riêng của khách**; **Yêu cầu sản xuất chỉ phát sinh từ đơn** (Điều phối lập cho phần thiếu). Thành phẩm về kho đã được bộ phận sản xuất **đóng thùng sẵn (LPN)**: kho nhập, cất, di chuyển, lấy và xuất **nguyên thùng**, không mở thùng, không lấy lẻ. **Thùng LPN chính là kiện giao**: khâu đóng gói thay bằng **Kiểm thùng & Dán nhãn giao hàng** ([08](../warehouse/08-packing/README.md)) — không chọn loại kiện, không cân đo, không niêm phong.
>
> **Đã chốt 2026-11-05:** khách sỉ **đặt theo số thùng** (báo giá, đơn, hợp đồng, Yêu cầu sản xuất, lệnh chuyển kho ghi **số thùng**, kèm số cái); **giá vẫn theo cái**, MOQ và bậc giá theo số thùng ([01](../warehouse/01-product-creation/README.md) BR-11, [14](../ecommerce/14-cart-and-order/README.md) BR-08). **Mọi thùng đúng số cái chuẩn** của SKU: kho đếm số thùng, **thùng lệch số chuẩn không nhận** ([03](../warehouse/03-receipt/README.md) BR-13); không còn thùng lẻ. Giữ hàng / phân bổ một dòng = **chọn đủ N thùng** theo FEFO / FIFO; thiếu thùng thì phần thiếu (tính theo thùng) đi Yêu cầu sản xuất ([07](../warehouse/07-picking/README.md) BR-03).

Mục lục:

1. [Luồng nhập kho thành phẩm: Yêu cầu sản xuất → Nhập kho theo thùng LPN → Putaway](#1-luồng-nhập-kho-thành-phẩm)
2. [Luồng Fulfillment: Order → kiểm tồn (Yêu cầu sản xuất) → Picking nguyên thùng → (Khu chờ xuất) → Kiểm thùng & Dán nhãn giao hàng → Shipping](#2-luồng-fulfillment) — kèm vòng đời Yêu cầu sản xuất (§2.3), phiên kiểm thùng & thùng giao (§2.4) và kế hoạch chuyến xe (§2.5)
3. [Luồng Transfer giữa các kho](#3-luồng-transfer-giữa-các-kho)
4. [Phụ lục: Luồng bán sỉ theo hợp đồng, vòng đời đơn hàng, hợp đồng & đợt thanh toán, di chuyển nguyên thùng trong một kho](#4-phụ-lục)
5. [Ghi chú về sơ đồ](#5-ghi-chú-về-sơ-đồ)

---

## 1. Luồng nhập kho thành phẩm

Chi tiết: [17 · Order Management §4.3](../ecommerce/17-order-management/README.md) (Yêu cầu sản xuất), [03 · Nhập kho thành phẩm](../warehouse/03-receipt/README.md), [05 · Putaway](../warehouse/05-putaway/README.md). Nhập kho đi **2 bước**: WH/Input (khu `RECEIVING`) → WH/Stock (bin). Yêu cầu sản xuất **chỉ phát sinh từ đơn**: Điều phối lập cho phần thiếu sau khi đã kiểm tồn SKU riêng; Inventory Planner không lập Yêu cầu sản xuất. Kho chỉ nhập **theo thùng LPN** do bộ phận sản xuất đóng sẵn ([03](../warehouse/03-receipt/README.md) BR-12), đếm **số thùng**; thùng có số cái khác **số cái chuẩn / thùng** không được nhận ([03](../warehouse/03-receipt/README.md) BR-13).

### 1.1. Flowchart tổng thể

```mermaid
flowchart TD
    START_ORDER(["Đơn Confirmed thiếu thành phẩm<br/>sau khi kiểm tồn SKU riêng"]) --> PR_ORDER["Điều phối lập Yêu cầu sản xuất cho phần thiếu của đơn<br/>dòng thiếu điền sẵn — Draft"]
    PR_ORDER --> PR_LINE["Dòng: SKU riêng của khách, số thùng = phần thiếu, kèm số cái<br/>ngày cần hàng, kho nhận, đơn liên quan<br/>đính kèm design snapshot + bộ file đã chốt"]
    PR_LINE --> PR_SEND["Xuất phiếu yêu cầu PDF, gửi bộ phận sản xuất<br/>Yêu cầu sản xuất: Sent"]
    PR_SEND --> EXT[["Bộ phận sản xuất — ngoài hệ thống<br/>sản xuất, tự kiểm tra chất lượng, đóng thùng sẵn"]]
    EXT -.->|"báo đã bắt đầu"| PR_INPROD["Điều phối cập nhật tay<br/>Yêu cầu sản xuất: In Production"]
    EXT --> DELIVER["Giao các thùng thành phẩm về kho nhận<br/>kèm phiếu giao ghi số Yêu cầu sản xuất, số thùng"]

    DELIVER --> RC_CREATE["NV kho tạo phiếu nhập theo một Yêu cầu sản xuất<br/>Phiếu nhập: Draft"]
    RC_CREATE --> RC_STD{"Từng thùng: số cái<br/>đúng số chuẩn / thùng của SKU?"}
    RC_STD -->|"Lệch số chuẩn"| RC_REJ["Không nhận thùng lệch<br/>trả lại bộ phận sản xuất<br/>phần đó vẫn mở trên Yêu cầu sản xuất"]
    RC_REJ -.->|"tính là thiếu"| CMP
    RC_STD -->|"Đúng số chuẩn"| RC_COUNT["Đếm số thùng, từng dòng yêu cầu + lô<br/>ghi lô / hạn dùng, mỗi thùng mang số cái chuẩn<br/>cấp và dán nhãn LPN — thùng Receiving<br/>khu nhận hàng WH/Input — RECEIVING"]
    RC_COUNT --> CMP{"Số thùng nhận luỹ kế<br/>so với số thùng yêu cầu?"}
    CMP -->|Thừa| OVER["Chặn: không nhận thùng làm vượt số thùng còn mở<br/>trả nguyên thùng lại bộ phận sản xuất"]
    OVER --> RC_COUNT
    CMP -->|"Đủ / Thiếu"| RC_CONFIRM["Xác nhận phiếu nhập<br/>tồn Inbound trong từng thùng tại khu RECEIVING, thùng In Stock<br/>Phiếu nhập: In Putaway"]

    RC_CONFIRM --> PR_CHECK{"Yêu cầu sản xuất<br/>đã nhận đủ mọi dòng?"}
    PR_CHECK -->|Chưa| PR_PARTIAL["Yêu cầu sản xuất: Partially Received<br/>phần thiếu vẫn mở"]
    PR_PARTIAL -->|"giao tiếp"| DELIVER
    PR_PARTIAL -.->|"không giao nốt"| PR_CLOSE(["Điều phối đóng thiếu<br/>Yêu cầu sản xuất: Closed, có lý do"])
    PR_CHECK -->|Đủ| PR_RECEIVED["Yêu cầu sản xuất: Received"]

    RC_CONFIRM --> PA_TASK["Mở nhiệm vụ cất<br/>mỗi thùng LPN một nhiệm vụ — Pending"]
    PA_TASK --> PA_SUGGEST["Gợi ý tối đa 10 bin<br/>cùng lô → cùng SKU → bin trống → bin còn chỗ"]
    PA_SUGGEST --> PA_MOVE["NV kho quét LPN + quét bin đích, cất cả thùng<br/>hệ thống kiểm lại ràng buộc cứng"]
    PA_MOVE --> PA_DONE["Nhiệm vụ cất Done — WH/Stock<br/>tồn Inbound → Available"]
    PA_DONE --> RC_CLOSE(["Cất xong mọi thùng của phiếu<br/>Phiếu nhập: Closed"])
    PA_DONE --> HOLD["Thùng cất xong được giữ cho đơn liên quan — 17"]
    HOLD --> ORDER_CHECK{"Mọi Yêu cầu sản xuất của đơn<br/>đã Received và đã giữ đủ thùng?"}
    ORDER_CHECK -->|Chưa| WAIT(["Đơn tiếp tục Awaiting Stock"])
    ORDER_CHECK -->|Rồi| RTF(["Đơn: Ready to Fulfill → luồng §2"])
```

> Nhập kho theo Yêu cầu sản xuất còn ở `Sent` thì hệ thống chuyển yêu cầu sang `In Production` trước rồi mới ghi nhận số đã nhận ([17](../ecommerce/17-order-management/README.md) §4.3). Hàng giao rời, chưa đóng thùng thì không nhận. Thùng lỗi thấy rõ khi kiểm đếm không được nhận, trả nguyên thùng lại bộ phận sản xuất ngay; thùng lỗi phát hiện sau khi xác nhận phiếu được cách ly nguyên thùng sang khu `QUARANTINE` / điều chỉnh tồn ([20](../warehouse/20-inventory-control/README.md)) và đổi trả với bộ phận sản xuất ngoài hệ thống — ⚠️ giả định, xem [open-questions](../open-questions/README.md).

> **Đã chốt 2026-11-05:** bộ phận sản xuất đóng **mọi thùng đúng số cái chuẩn / thùng** của SKU; kho đếm **số thùng**, thùng lệch số chuẩn không nhận và phần đó vẫn mở trên Yêu cầu sản xuất ([03](../warehouse/03-receipt/README.md) BR-13, [open-questions B24](../open-questions/README.md)). ⚠️ Giả định: thùng phát hiện lệch số chuẩn **sau khi xác nhận nhập** không được phân bổ, chuyển sang khu `QUARANTINE` rồi ghi giảm cả thùng bằng điều chỉnh có duyệt — xem [open-questions C34](../open-questions/README.md).

### 1.2. Sequence diagram (trình tự tác nhân)

```mermaid
sequenceDiagram
    autonumber
    actor COORD as Điều phối
    participant PR as Yêu cầu sản xuất (17)
    participant PRODEXT as Bộ phận sản xuất (ngoài hệ thống)
    actor WH as NV kho
    participant RC as Phiếu nhập (03)
    participant SLOT as Gợi ý bin (05)
    participant INVSYS as Tồn kho (20)
    participant OM as Đơn hàng (17)

    COORD->>PR: Lập yêu cầu cho phần thiếu của đơn (Draft)
    COORD->>PR: Xuất phiếu PDF kèm bộ file thiết kế → Sent
    PR->>PRODEXT: Gửi phiếu yêu cầu (ngoài hệ thống)
    PRODEXT-->>COORD: Báo đã bắt đầu sản xuất
    COORD->>PR: Cập nhật tay → In Production
    PRODEXT->>WH: Giao các thùng đã đóng sẵn (đúng số cái chuẩn) + phiếu giao ghi số yêu cầu, số thùng
    WH->>RC: Tạo phiếu nhập theo một Yêu cầu sản xuất (Draft)
    opt Thùng có số cái khác số chuẩn / thùng
        WH->>PRODEXT: Không nhận, trả lại thùng lệch (phần đó vẫn mở trên yêu cầu)
    end
    WH->>RC: Đếm số thùng đúng chuẩn, ghi lô, hạn dùng
    RC-->>WH: Cấp mã LPN từng thùng, in nhãn để dán
    opt Nhận vượt số thùng yêu cầu
        RC-->>WH: Chặn — không nhận thùng làm vượt số thùng còn mở
        WH->>PRODEXT: Trả nguyên thùng thừa
    end
    WH->>RC: Xác nhận phiếu → In Putaway
    RC->>INVSYS: Ghi tồn Inbound trong từng thùng tại khu RECEIVING (thùng In Stock)
    RC->>PR: Cập nhật số thùng đã nhận → Partially Received / Received
    RC->>WH: Mở nhiệm vụ cất (mỗi thùng LPN một nhiệm vụ)
    WH->>SLOT: Xem nhiệm vụ cất
    SLOT-->>WH: Tối đa 10 bin xếp hạng
    WH->>INVSYS: Quét LPN + quét bin, cất cả thùng (WH/Stock) → tồn Available
    INVSYS->>OM: Giữ thùng cho đơn liên quan
    OM->>OM: Mọi Yêu cầu sản xuất đã Received, giữ đủ thùng → Ready to Fulfill
    RC->>RC: Cất hết mọi thùng → Closed
    opt Bộ phận sản xuất không giao nốt
        COORD->>PR: Đóng thiếu (Closed, có lý do)
    end
```

---

## 2. Luồng Fulfillment

Chi tiết: [17 · Order Management §4.2–4.6](../ecommerce/17-order-management/README.md), [07 · Picking](../warehouse/07-picking/README.md), [08 · Kiểm thùng & Dán nhãn giao hàng](../warehouse/08-packing/README.md), [09 · Shipping](../warehouse/09-shipping/README.md).

### 2.1. Flowchart tổng thể

> **Đã chốt 2026-11-05:** dòng đơn ghi **số thùng N**; mọi thùng đúng số cái chuẩn nên giữ hàng / phân bổ = **chọn đủ N thùng LPN** của SKU, ưu tiên FEFO / FIFO. Chỉ có sẵn k < N thùng thì giữ k thùng, **N − k thùng còn lại là thiếu** — Điều phối gửi Yêu cầu sản xuất cho phần thiếu (tính theo thùng) hoặc Sale thoả thuận lại số thùng với khách ([07](../warehouse/07-picking/README.md) BR-03, [17](../ecommerce/17-order-management/README.md) §4.2, BR-21, [open-questions C32](../open-questions/README.md)).

> ⚠️ Giả định: Điều phối quyết định có dùng **khu chờ xuất (`STAGING`)** cho đơn; lấy thùng sang `STAGING` chưa trừ tồn, lấy bổ sung vào cùng task, **chuyển cả tác vụ sang khu kiểm & dán nhãn** một lần cho cả task. Xem [open-questions](../open-questions/README.md).

```mermaid
flowchart TD
    CONF(["Đơn: Confirmed<br/>hợp đồng Signed, đủ đợt ON_SIGNING"]) --> CHECK["Hệ thống kiểm tồn SKU riêng khả dụng từng dòng, theo số thùng<br/>hàng dư từ đơn trước, hàng trả<br/>giữ đủ N thùng của dòng, hoặc số thùng có sẵn"]
    CHECK --> SETUP["Điều phối chọn kho xuất<br/>và cách giao: gửi kiện / giao nguyên chuyến"]
    SETUP --> ENOUGH{"Đủ hàng?<br/>mỗi dòng có đủ N thùng"}

    ENOUGH -->|"Thiếu thùng"| PR_SEND["Điều phối lập và gửi Yêu cầu sản xuất<br/>cho phần thiếu, tính theo số thùng — luồng §1"]
    PR_SEND --> AWAIT["Đơn: Awaiting Stock<br/>Sale báo khách ngày giao mới nếu trễ hợp đồng"]
    AWAIT --> AW_CHOICE{"Phần có sẵn<br/>xử lý thế nào?"}
    AW_CHOICE -->|"Chờ đủ hàng"| AW_BIN["Thùng nằm tại bin, đã giữ cho đơn"]
    AW_CHOICE -->|"Điều phối dùng khu chờ xuất: đích STAGING"| TASK
    AW_CHOICE -->|"Sale cho phép giao một phần: đích PACKING"| TASK
    AW_BIN -.->|"thành phẩm nhập kho, cất xong — §1"| RECV["Mọi Yêu cầu sản xuất của đơn đã Received, giữ đủ thùng<br/>Đơn: Ready to Fulfill<br/>đợt BEFORE_DELIVERY → Due"]

    ENOUGH -->|Đủ| RELEASE["Điều phối release, phân bổ đủ N thùng LPN cụ thể mỗi dòng — bin, lô<br/>FEFO / FIFO<br/>Đơn: Ready to Fulfill — đợt BEFORE_DELIVERY → Due"]
    RELEASE --> TASK
    RECV --> HAS_TASK{"Đơn đã có task?"}
    HAS_TASK -->|Chưa| TASK["Điều phối / Quản lý kho tạo task, giao người<br/>chọn đích lấy hàng: PACKING / STAGING<br/>mỗi thùng đã phân bổ một dòng lấy hàng<br/>đơn Ready to Fulfill → Picking, đơn Awaiting Stock giữ nguyên"]
    HAS_TASK -->|Có| TOPUP["Phân bổ bổ sung đủ số thùng còn thiếu, thêm dòng vào cùng task<br/>giữ đích của task — task và đơn → Picking"]
    TOPUP --> PICK
    TASK --> PICK["Picker quét vị trí + quét LPN của thùng<br/>đúng thùng của dòng, không nhập số lượng"]

    PICK --> SHORT{"Lấy được thùng<br/>của dòng?"}
    SHORT -->|"Không thấy / hư hỏng"| SHORT_HOLD["Báo thiếu: dòng Short, đơn On Hold INVENTORY<br/>tự mở kiểm kê mù thùng đó"]
    SHORT_HOLD --> REALLOC{"Phân bổ lại được đủ số thùng thiếu<br/>từ thùng khác của SKU, cùng kho?"}
    REALLOC -->|"Được — tự gỡ treo"| PICK
    REALLOC -->|"Không"| SHORT_DECIDE(["Điều phối + Sale quyết:<br/>Yêu cầu sản xuất / thoả thuận lại số thùng /<br/>giao một phần rồi đóng phần còn lại / huỷ"])
    SHORT -->|"Lấy được"| DEST{"Đích lấy hàng?"}

    DEST -->|STAGING| STAGED["Dòng Staged: thùng chuyển nguyên sang khu STAGING, sổ MOVE<br/>chưa trừ tồn, phân bổ vẫn Held, giữ riêng cho đơn<br/>Task: Staged"]
    STAGED --> READY_OUT{"Đơn sẵn sàng xuất?<br/>đủ hàng, hoặc được giao một phần"}
    READY_OUT -->|"Chưa: chờ hàng / chờ đợt BEFORE_DELIVERY"| STAGED
    STAGED -.->|"phần thiếu nhập kho"| RECV
    READY_OUT -->|Rồi| TO_CHECK["Điều phối / Quản lý kho: chuyển cả tác vụ sang khu kiểm và dán nhãn<br/>một lần cho cả task, đơn không treo<br/>trừ tồn, sổ PICK, thùng và phân bổ Consumed — Task: Picked"]
    DEST -->|PACKING| PICKED["Dòng Picked: trừ tồn ngay, sổ PICK, thùng Consumed<br/>Task: Picked"]
    TO_CHECK --> CHECK_START
    PICKED --> CHECK_START["NV kho bắt đầu phiên kiểm thùng tại khu PACKING<br/>quét LPN từng thùng: đúng đơn, đúng SKU — thùng Checked<br/>thùng hư hỏng: loại → Removed, cất lại vào QUARANTINE"]

    CHECK_START --> LABEL["Kích thước, khối lượng theo quy cách thùng của SKU<br/>in và dán nhãn giao hàng lên từng thùng — Labeled<br/>thùng đầu tiên dán nhãn: khoá huỷ đơn, cách giao, địa chỉ"]
    LABEL --> COMPLETE{"Hoàn tất kiểm thùng:<br/>đủ thùng?"}
    COMPLETE -->|Đủ| PACKED["Phiên kiểm thùng: Completed<br/>Đơn: Packed — đã kiểm đủ thùng và dán nhãn<br/>Shipment: Ready to Dispatch"]
    COMPLETE -->|"Thiếu, đơn được giao một phần"| PART["Đơn: Partially Fulfilled<br/>Shipment: Ready to Dispatch<br/>phần còn lại lấy bổ sung sau"]
    PART -.->|"phần còn lại về kho"| RECV
    PACKED --> PAY_WARN["Cảnh báo nếu đợt BEFORE_DELIVERY chưa thu<br/>không chặn kiểm thùng"]
    PART --> PAY_WARN

    PAY_WARN --> MODE{"Cách giao?"}
    MODE -->|"Gửi kiện qua hãng"| PARCEL["Gán hãng, mở chuyến bàn giao cho một hãng<br/>quét LPN đủ kiện lên xe — mỗi thùng một kiện"]
    MODE -->|"Giao nguyên chuyến"| FTL["Load Planning 3D theo quy cách thùng — kế hoạch Confirmed, mỗi xe một chuyến<br/>đặt xe ngoài hệ thống, ghi nhận đặt xe<br/>bắt đầu xếp, quét LPN theo sơ đồ từng xe"]
    PARCEL --> GATE
    FTL --> GATE{"Mọi đợt BEFORE_DELIVERY đã Paid?<br/>đơn không treo"}
    GATE -->|Chưa| BLOCK["Từ chối ký bàn giao cả chuyến<br/>bỏ shipment đó ra, chờ ở khu DISPATCH<br/>tới khi Kế toán ghi nhận đủ tiền"]
    BLOCK -.-> GATE
    GATE -->|Rồi| HANDOVER["Ký bàn giao — tất cả hoặc không<br/>Shipment: Handed Over"]
    HANDOVER --> SHIPPED["Đơn: Shipped, hoặc Partially Fulfilled nếu còn phần chưa giao<br/>không theo dõi hành trình, không thu tiền khi giao"]

    SHIPPED --> RESULT{"Hãng báo kết quả<br/>Điều phối cập nhật"}
    RESULT -->|"Thành công"| DELIVERED["Đơn: Delivered<br/>đợt AFTER_DELIVERY đến hạn — công nợ phải thu"]
    RESULT -->|"Thất bại, có lý do"| FAILED["Đơn: Delivery Failed"]
    FAILED -->|"Hãng giao lại thành công"| DELIVERED
    FAILED -->|"Hàng hoàn về kho"| RTO(["Đơn: Returned — nhập lại kho, xử lý hoàn tiền"])
    DELIVERED --> AUTO["Mỗi đêm: sau 7 ngày, mọi đợt Paid,<br/>không có yêu cầu trả hàng — Đơn: Completed"]
    AUTO --> CLOSED(["Sau 30 ngày: Closed"])
```

### 2.2. Sequence diagram (trình tự tác nhân)

```mermaid
sequenceDiagram
    autonumber
    actor SALE as Sale
    actor COORD as Điều phối
    participant OM as Đơn hàng (17)
    participant PR as Yêu cầu sản xuất (17)
    participant INVSYS as Tồn kho (20)
    actor PICKER as Picker
    actor CHECKER as NV kho kiểm thùng (08)
    actor DISP as Dispatch Staff
    participant PAY as Thanh toán (15)
    actor ACC as Kế toán
    participant CARRIER as Hãng vận chuyển (ngoài hệ thống)
    actor CUST as Khách hàng sỉ

    OM->>INVSYS: Đơn Confirmed — kiểm tồn SKU riêng theo số thùng, giữ đủ N thùng mỗi dòng (hoặc số thùng có sẵn)
    COORD->>OM: Chọn kho xuất, cách giao (gửi kiện / nguyên chuyến)
    alt Thiếu thùng
        COORD->>PR: Lập và gửi Yêu cầu sản xuất cho phần thiếu (tính theo số thùng)
        OM->>OM: Awaiting Stock
        SALE-->>CUST: Báo ngày giao dự kiến mới (nếu trễ hợp đồng)
        opt Điều phối dùng khu chờ xuất
            COORD->>PICKER: Tạo task đích STAGING cho phần có sẵn
            PICKER->>INVSYS: Lấy nguyên thùng về khu STAGING (MOVE, chưa trừ tồn) → task Staged
        end
        PR-->>OM: Thành phẩm nhập kho đủ (luồng §1) → Ready to Fulfill
    else Đủ hàng
        COORD->>OM: Release → Ready to Fulfill
    end
    OM->>PAY: Đợt BEFORE_DELIVERY chuyển Due
    COORD->>INVSYS: Phân bổ đủ N thùng LPN mỗi dòng (FEFO / FIFO)
    alt Đơn chưa có task
        COORD->>PICKER: Tạo task, chọn đích PACKING / STAGING, giao người → đơn Picking
    else Đơn đã có task Staged
        COORD->>PICKER: Phân bổ bổ sung, thêm dòng vào cùng task → task và đơn Picking
    end
    PICKER->>INVSYS: Quét vị trí + quét LPN của thùng (mỗi dòng một thùng)
    opt Không lấy được thùng
        PICKER-->>OM: Báo thiếu → đơn On Hold INVENTORY
        INVSYS->>INVSYS: Tự mở kiểm kê mù thùng đó
        COORD->>INVSYS: Phân bổ lại đủ số thùng thiếu từ thùng khác cùng kho → tự gỡ treo
    end
    alt Đích PACKING
        INVSYS->>INVSYS: Trừ tồn ngay (PICK), thùng Consumed → task Picked
    else Đích STAGING
        INVSYS->>INVSYS: Chuyển thùng sang khu STAGING (MOVE), phân bổ vẫn Held → task Staged
        COORD->>INVSYS: Chuyển cả tác vụ sang khu kiểm và dán nhãn → trừ tồn, thùng và phân bổ Consumed → task Picked
    end
    CHECKER->>CHECKER: Bắt đầu phiên kiểm thùng, quét LPN từng thùng (đúng đơn, đúng SKU)
    CHECKER->>CHECKER: In và dán nhãn giao hàng từng thùng (kích thước, khối lượng theo quy cách thùng)
    CHECKER-->>OM: Thùng đầu tiên dán nhãn → khoá huỷ đơn, đổi cách giao, đổi địa chỉ
    CHECKER->>OM: Hoàn tất → Packed (hoặc Partially Fulfilled), shipment Ready to Dispatch
    opt Đợt BEFORE_DELIVERY chưa thu
        PAY-->>CHECKER: Cảnh báo số tiền (không chặn kiểm thùng)
    end
    alt Gửi kiện qua hãng
        DISP->>DISP: Mở chuyến cho một hãng, quét LPN đủ kiện
    else Giao nguyên chuyến
        DISP->>DISP: Load Planning 3D → kế hoạch Confirmed, mỗi xe một chuyến
        DISP->>CARRIER: Đặt xe (ngoài hệ thống), ghi nhận đặt xe
        DISP->>DISP: Bắt đầu xếp, quét LPN theo sơ đồ từng xe
    end
    DISP->>PAY: Kiểm mọi đợt BEFORE_DELIVERY đã Paid
    opt Còn đợt BEFORE_DELIVERY chưa Paid
        PAY-->>DISP: Từ chối bàn giao (danh sách đơn, số tiền còn thiếu)
        CUST->>ACC: Chuyển khoản đợt BEFORE_DELIVERY
        ACC->>PAY: Ghi nhận theo sao kê, phân bổ → đợt Paid, mở chặn
    end
    DISP->>CARRIER: Ký bàn giao → shipment Handed Over
    DISP-->>OM: Shipped (không theo dõi hành trình)
    CARRIER-->>CUST: Giao hàng (ngoài hệ thống, không thu tiền)
    alt Giao thành công
        CARRIER-->>COORD: Báo đã giao
        COORD->>OM: Delivered
        OM->>PAY: Đợt AFTER_DELIVERY đến hạn (công nợ phải thu)
        CUST->>ACC: Chuyển khoản theo kỳ hạn
        ACC->>PAY: Ghi nhận theo sao kê → đợt Paid
        OM->>OM: Sau 7 ngày, mọi đợt Paid → Completed, sau 30 ngày → Closed
    else Giao thất bại
        CARRIER-->>COORD: Báo giao thất bại
        COORD->>OM: Delivery Failed (kèm lý do)
        CARRIER->>DISP: Hoàn hàng về kho (RTO)
        COORD->>OM: Returned → xử lý hoàn tiền (15)
    end
```

### 2.3. Vòng đời Yêu cầu sản xuất (Production Request)

Chi tiết trạng thái & điều kiện chuyển xem [17 · Order Management §5.3](../ecommerce/17-order-management/README.md). Yêu cầu sản xuất **chỉ phát sinh từ đơn**: Điều phối lập cho phần thiếu của đơn `Confirmed` sau khi đã kiểm tồn SKU riêng; không có yêu cầu bổ sung tồn. `Partially Received` / `Received` do hệ thống chuyển khi nhập kho theo thùng ([03](../warehouse/03-receipt/README.md)).

```mermaid
stateDiagram-v2
    state "In Production" as InProduction
    state "Partially Received" as PartiallyReceived
    [*] --> Draft: Điều phối lập cho phần thiếu của đơn
    Draft --> Sent: Xuất phiếu, gửi bộ phận sản xuất
    Sent --> InProduction: Bộ phận sản xuất bắt đầu
    InProduction --> PartiallyReceived: Nhập kho một phần
    InProduction --> Received: Nhập kho đủ
    PartiallyReceived --> Received: Nhập nốt
    PartiallyReceived --> Closed: Đóng thiếu (lý do)
    InProduction --> Closed: Dừng sản xuất, chưa nhận gì (lý do)
    Draft --> Cancelled
    Sent --> Cancelled: Chưa nhận gì (lý do)
    Received --> [*]
    Closed --> [*]
    Cancelled --> [*]
```

> ⚠️ Giả định: Yêu cầu sản xuất `In Production` mà bộ phận sản xuất dừng hẳn khi chưa giao gì thì Điều phối **đóng** (`Closed`, bắt buộc lý do), không huỷ. Xem [open-questions](../open-questions/README.md).

### 2.4. Vòng đời phiên kiểm thùng & thùng giao (Outbound Check & Labeling)

Chi tiết: [08 · Kiểm thùng & Dán nhãn giao hàng §5](../warehouse/08-packing/README.md). Thùng LPN là kiện giao: kho chỉ quét kiểm đủ thùng, dán nhãn giao hàng và gom thùng thành shipment — không chọn loại kiện, không cân đo, không niêm phong. Khi **thùng đầu tiên** của đơn được dán nhãn, đơn không huỷ được, không đổi cách giao, không đổi địa chỉ giao ([08](../warehouse/08-packing/README.md) BR-04).

**Phiên kiểm thùng (theo đơn):**

```mermaid
stateDiagram-v2
    state "In Progress" as InProgress
    [*] --> Pending: Tạo task lấy hàng (07)
    Pending --> InProgress: Bắt đầu (lấy hàng đã xong)
    InProgress --> InProgress: Hoàn tất một phần (đơn Partially Fulfilled)
    InProgress --> Completed: Hoàn tất đủ (đơn Packed)
    Pending --> Cancelled: Đơn huỷ
    InProgress --> Cancelled: Đơn huỷ (chưa thùng nào dán nhãn)
    Completed --> [*]
    Cancelled --> [*]
```

**Thùng giao (Outbound carton):**

```mermaid
stateDiagram-v2
    state "To Check" as ToCheck
    [*] --> ToCheck: Thùng đã lấy về khu PACKING
    ToCheck --> Checked: Quét đúng LPN
    Checked --> Labeled: In và dán nhãn giao hàng
    ToCheck --> Removed: Loại thùng / đơn huỷ
    Checked --> Removed: Loại thùng / đơn huỷ
    Labeled --> Removed: Loại thùng hư hỏng (chưa thuộc shipment)
    Labeled --> [*]: Gom vào shipment (09)
    Removed --> [*]
```

> Thùng `Removed` vào hàng chờ cất lại ([07](../warehouse/07-picking/README.md) §4.5); thùng hư hỏng được cất nguyên thùng vào khu `QUARANTINE` ([20](../warehouse/20-inventory-control/README.md) §4.8). Sau khi vào shipment, vòng đời của thùng theo shipment ở [09](../warehouse/09-shipping/README.md) §5.1.

### 2.5. Vòng đời kế hoạch chuyến xe (Load Plan — giao nguyên chuyến)

Chi tiết: [09 · Shipping §4.2, §5.3](../warehouse/09-shipping/README.md#42-giao-nguyên-chuyến--lập-kế-hoạch-xếp-hàng-3d-load-planning). Dùng cho đơn khách giao nguyên chuyến và lệnh chuyển kho nguyên chuyến ([10](../warehouse/10-inter-warehouse-transfer/README.md)); một kế hoạch không trộn đơn khách với lệnh chuyển kho. **Kiện là thùng LPN**, kích thước / khối lượng theo quy cách thùng của SKU ([09](../warehouse/09-shipping/README.md) BR-17). Khi chốt, mỗi xe là một chuyến bàn giao. Kế hoạch không có nhánh gỡ kiện vì huỷ đơn — đơn đã có thùng dán nhãn thì không huỷ được; bỏ kiện khỏi kế hoạch (thùng hư hỏng, đơn chưa thu đợt `BEFORE_DELIVERY`) thì mở lại về `Planned` khi chưa xe nào bắt đầu xếp.

```mermaid
stateDiagram-v2
    [*] --> Draft
    Draft --> Planned: Tính phương án xếp
    Planned --> Planned: Chỉnh tay + tính lại xe
    Planned --> Draft: Đổi kiện / loại xe / chế độ
    Planned --> Confirmed: Chốt kế hoạch
    Confirmed --> Planned: Mở lại (chưa xếp lên xe)
    Confirmed --> Dispatched: Mọi xe bàn giao và rời kho
    Draft --> Cancelled
    Planned --> Cancelled
    Confirmed --> Cancelled
    Dispatched --> [*]
    Cancelled --> [*]
```

---

## 3. Luồng Transfer giữa các kho

Chi tiết: [10 · Inter-warehouse Transfer](../warehouse/10-inter-warehouse-transfer/README.md). Lệnh chuyển kho **chuyển nguyên thùng**: mỗi dòng ghi **số thùng**, kho nguồn phải đủ số thùng tự do; phân bổ = chọn đủ N thùng theo FEFO / FIFO ([10](../warehouse/10-inter-warehouse-transfer/README.md) BR-13); thùng LPN là kiện chuyển đi.

> ⚠️ Giả định: ngưỡng duyệt lệnh chuyển kho tính theo **tổng số thùng**, mặc định **100 thùng** ([10](../warehouse/10-inter-warehouse-transfer/README.md) BR-08). Xem [open-questions C35](../open-questions/README.md).

### 3.1. Flowchart tổng thể

> ⚠️ Từ bước **Kho đích: tạo phiếu nhận** trở đi (nhận, đối chiếu chênh lệch, cất hàng, đóng lệnh) là **phạm vi giai đoạn sau** — lệnh hiện dừng ở `In Transit` ([10](../warehouse/10-inter-warehouse-transfer/README.md) §4 bước 8–11, [open-questions C26](../open-questions/README.md)). Các bước đó vẽ **nét đứt**.

```mermaid
flowchart TD
    NEED(["Nhu cầu cân bằng tồn / đưa hàng gần vùng bán"]) --> TO_CREATE["Inventory Planner / Quản lý kho tạo lệnh chuyển kho — Draft<br/>kho nguồn → kho đích, cách giao, dòng SKU và số thùng"]
    TO_CREATE --> TO_CHECK{"Mỗi dòng đủ số thùng Available tự do<br/>tại kho nguồn?"}
    TO_CHECK -->|Không| ADJUST["Từ chối lưu dòng đó<br/>giảm số thùng hoặc chọn kho nguồn khác"]
    ADJUST --> TO_CREATE
    TO_CHECK -->|Có| TO_SUBMIT{"Gửi duyệt: tổng số thùng<br/>vượt ngưỡng, mặc định 100 thùng?"}
    TO_SUBMIT -->|Vượt| TO_PENDING["Pending Approval<br/>Quản lý kho khác người gửi duyệt"]
    TO_PENDING -->|Duyệt| TO_APPROVED
    TO_PENDING -->|"Từ chối + lý do"| TO_CREATE
    TO_SUBMIT -->|"Không — tự duyệt"| TO_APPROVED["Approved"]

    TO_APPROVED --> MODE{"Cách giao?"}
    MODE -->|"Nguyên chuyến — mặc định"| FTL_CHECK["Mở phiên kiểm thùng chuyển kho → TO Picking<br/>phân bổ và giữ đủ N thùng mỗi dòng, FEFO / FIFO<br/>quét LPN kiểm từng thùng, dán nhãn chuyển kho như 08<br/>thùng không lấy được: giao một phần kèm ghi chú"]
    FTL_CHECK -.->|"Huỷ lấy hàng, chưa rời kho"| TO_APPROVED
    FTL_CHECK --> FTL_LP["Kế hoạch chuyến xe 3D cho đúng lệnh này<br/>đi nguyên lô, xếp và ký bàn giao từng xe"]
    FTL_LP --> DISPATCH["Xuất kho nguồn → TO In Transit<br/>tồn kho nguồn giảm theo nguyên thùng, sổ TRANSFER_OUT, thùng Consumed<br/>tồn In-Transit thuộc lệnh"]
    MODE -->|"Gửi kiện"| PCL["NV kho lấy thùng<br/>xác nhận xuất bằng quét LPN từng thùng gửi đi"]
    PCL --> DISPATCH

    DISPATCH --> TRANSPORT["Vận chuyển giữa 2 kho<br/>không theo dõi hành trình"]
    TRANSPORT --> DST_RECV["Kho đích: tạo phiếu nhận<br/>tham chiếu lệnh chuyển kho"]
    DST_RECV --> DST_COUNT["Kiểm đếm số thùng thực nhận + tình trạng"]
    DST_COUNT --> DISCREPANCY{"Khớp số thùng xuất?"}
    DISCREPANCY -->|Lệch| INVESTIGATE["Ghi transfer discrepancy<br/>điều tra / claim hãng vận chuyển ngoài hệ thống<br/>điều chỉnh In-Transit"]
    DISCREPANCY -->|Khớp| DST_PA
    INVESTIGATE --> DST_PA["Sinh nhiệm vụ cất tại kho đích"]

    DST_PA --> DST_SLOT["Gợi ý bin cất hàng tại kho đích — 05"]
    DST_SLOT --> DST_MOVE["Cất nguyên thùng vào vị trí<br/>tồn In-Transit → Available tại kho đích"]
    DST_MOVE --> TO_STATUS{"Đã nhận và cất hết dòng?"}
    TO_STATUS -->|Chưa| DST_RECV
    TO_STATUS -->|Rồi| TO_DONE(["Lệnh chuyển kho: Completed / Closed"])

    classDef later stroke-dasharray: 5 5
    class DST_RECV,DST_COUNT,DISCREPANCY,INVESTIGATE,DST_PA,DST_SLOT,DST_MOVE,TO_STATUS,TO_DONE later
```

### 3.2. Sequence diagram (trình tự tác nhân)

```mermaid
sequenceDiagram
    autonumber
    actor PLAN as Inventory Planner / Quản lý kho
    participant TO as Lệnh chuyển kho (TO)
    actor MGR as Quản lý kho (người duyệt)
    actor SRC as NV kho nguồn
    participant INV_SRC as Tồn kho nguồn
    participant LP as Kế hoạch chuyến xe (09)
    participant TRANS as Vận chuyển
    actor DST as NV kho đích
    participant SLOT as Gợi ý bin (kho đích)
    participant INV_DST as Tồn kho đích

    PLAN->>TO: Tạo TO (Draft) — nguồn / đích, cách giao, dòng SKU và số thùng
    TO->>INV_SRC: Kiểm mỗi dòng đủ số thùng Available tự do
    PLAN->>TO: Gửi duyệt
    alt Tổng số thùng không vượt ngưỡng (mặc định 100 thùng)
        TO->>TO: Tự duyệt → Approved
    else Vượt ngưỡng
        TO->>MGR: Pending Approval
        MGR->>TO: Duyệt → Approved (từ chối kèm lý do → Draft)
    end
    alt Nguyên chuyến (mặc định)
        SRC->>TO: Mở phiên kiểm thùng chuyển kho → Picking
        TO->>INV_SRC: Phân bổ và giữ đủ N thùng mỗi dòng (FEFO / FIFO)
        SRC->>SRC: Lấy thùng, quét LPN kiểm, dán nhãn chuyển kho (không mở thùng, không đóng lại hàng)
        SRC->>LP: Lập kế hoạch chuyến xe cho đúng lệnh này, xếp và ký từng xe
        LP-->>TO: Xe cuối cùng ký bàn giao → In Transit
    else Gửi kiện
        SRC->>SRC: Lấy thùng
        SRC->>TO: Xác nhận xuất bằng quét LPN từng thùng gửi đi → In Transit
    end
    INV_SRC->>INV_SRC: Giảm tồn theo nguyên thùng (TRANSFER_OUT, thùng Consumed) → In-Transit thuộc TO
    SRC->>TRANS: Bàn giao vận chuyển (không theo dõi hành trình)
    Note over DST,INV_DST: Phạm vi giai đoạn sau — chưa có trong hệ thống
    TRANS->>DST: Giao hàng tới kho đích
    DST->>TO: Tạo phiếu nhận tham chiếu TO
    DST->>DST: Kiểm đếm số thùng và tình trạng
    opt Lệch số thùng xuất / nhận
        DST->>TO: Ghi transfer discrepancy
        TO->>TRANS: Claim / điều tra
        DST->>INV_DST: Điều chỉnh In-Transit tương ứng
    end
    DST->>SLOT: Yêu cầu gợi ý vị trí (kho đích)
    SLOT-->>DST: Vị trí đích đề xuất
    DST->>INV_DST: Cất nguyên thùng → In-Transit chuyển thành Available (kho đích)
    TO->>TO: Completed / Closed
```

---

## 4. Phụ lục

### 4.1. Luồng bán sỉ theo hợp đồng (Mẫu & thiết kế → Báo giá → Đơn → Hợp đồng → Đợt thanh toán → Công nợ)

Chi tiết: [12 · Mẫu & thiết kế](../ecommerce/12-product-design/README.md), [14 · Báo giá, Đơn hàng & Hợp đồng](../ecommerce/14-cart-and-order/README.md), [15 · Thanh toán theo hợp đồng & Công nợ](../ecommerce/15-payment/README.md). Khách **không tự đặt hàng** trên sàn: đơn do Sale tạo từ báo giá đã chấp nhận. Dòng báo giá / đơn chỉ là **SKU thành phẩm riêng của khách** (không còn hàng trơn), ghi **số thùng** (kèm số cái / thùng) và **đơn giá theo cái**; MOQ và bậc giá theo số thùng; thành tiền dòng = số thùng × số cái / thùng × đơn giá / cái ([14](../ecommerce/14-cart-and-order/README.md) BR-07, BR-08).

```mermaid
flowchart TD
    ENTRY(["Khách sỉ xem quy cách trên sàn, liên hệ Sale<br/>chat / điện thoại / email"]) --> PATH{"Hàng in theo<br/>mẫu mới?"}
    PATH -->|Có| CUST_INPUT["Khách đưa mẫu: logo, file, ảnh, mẫu vật<br/>Sale ghi nhận yêu cầu thiết kế — Draft (12)"]
    CUST_INPUT --> DESIGN["Người thiết kế tải bộ file<br/>Sale / Quản trị bán hàng"]
    DESIGN --> REVIEW{"Quản trị bán hàng duyệt kỹ thuật<br/>người duyệt khác người thiết kế"}
    REVIEW -->|"Không đạt"| DESIGN
    REVIEW -->|Đạt| SUBMIT["Gửi khách xác nhận trên sàn — Submitted<br/>quy cách phải đã có quy cách thùng"]
    SUBMIT --> CUST_OK{"Khách xác nhận?"}
    CUST_OK -->|"Yêu cầu sửa"| DESIGN
    CUST_OK -->|"Xác nhận"| SNAP["Design snapshot bất biến<br/>+ sinh SKU thành phẩm riêng của khách, quy cách thùng theo quy cách — Approved"]
    PATH -->|"Không — SKU riêng của khách đã có"| QUOTE
    SNAP --> QUOTE["Sale lập báo giá (14)<br/>số thùng từng dòng, đơn giá / cái theo bảng giá riêng<br/>bậc giá và MOQ theo số thùng, hiệu lực — Sent"]

    QUOTE --> Q_RESP{"Khách phản hồi trên sàn<br/>hoặc thẻ hành động trong chat"}
    Q_RESP -->|"Yêu cầu sửa"| Q_REV["Changes Requested → bản sửa mới"]
    Q_REV --> QUOTE
    Q_RESP -->|"Hết hiệu lực"| Q_EXP(["Báo giá: Expired"])
    Q_RESP -->|"Chấp nhận, chọn số thùng ≥ MOQ"| ACCEPT["Báo giá: Accepted"]
    ACCEPT --> ORDER["Sale tạo đơn từ báo giá<br/>chốt số thùng, số cái / thùng, đơn giá / cái<br/>Đơn: Draft — báo giá Converted"]

    ORDER --> CONTRACT["Sale lập hợp đồng mua bán — Draft<br/>các bên, dòng hàng & giá, điều khoản giao<br/>lịch thanh toán: ON_SIGNING / BEFORE_DELIVERY / AFTER_DELIVERY"]
    CONTRACT --> SEND["Gửi hợp đồng — Sent<br/>Đơn: Pending Contract<br/>cảnh báo nếu vượt hạn mức / quá hạn"]
    SEND --> SIGN{"Khách ký?<br/>ngoài hệ thống"}
    SIGN -->|"Từ chối / cần sửa"| C_CANCEL["Huỷ hợp đồng, có lý do<br/>đơn về Draft để lập hợp đồng thay thế, hoặc huỷ đơn"]
    C_CANCEL -.-> CONTRACT
    SIGN -->|"Đã ký"| SIGNED["Sale tải bản scan → hợp đồng Signed<br/>sinh các đợt thanh toán — Not Due"]

    SIGNED --> CREDIT{"Có đợt AFTER_DELIVERY và khách<br/>quá hạn / vượt hạn mức?"}
    CREDIT -->|Có| HOLD["Đơn: On Hold — tín dụng<br/>Kế toán duyệt / từ chối"]
    HOLD -->|"Từ chối"| O_CANCEL(["Đơn: Cancelled — CREDIT_REJECTED"])
    HOLD -->|Duyệt| ONSIGN
    CREDIT -->|Không| ONSIGN{"Đợt ON_SIGNING đã thu đủ<br/>hoặc không có?"}
    ONSIGN -->|Chưa| PP["Đơn: Pending Payment<br/>khách chuyển khoản, Kế toán ghi nhận & phân bổ"]
    PP -->|"Thu đủ"| CONFIRMED
    ONSIGN -->|Rồi| CONFIRMED(["Đơn: Confirmed → kiểm tồn & Fulfillment — §2"])
    CONFIRMED -.-> AFTER["Đợt BEFORE_DELIVERY chặn bàn giao tới khi Paid<br/>sau giao: đợt AFTER_DELIVERY là công nợ phải thu<br/>mọi đợt Paid và đơn hoàn tất → hợp đồng Completed"]
```

### 4.2. Vòng đời đơn hàng (Order)

Chi tiết: [17 · Order Management §5.1](../ecommerce/17-order-management/README.md). Sơ đồ vẽ theo bảng chuyển tiếp của 17 §5.1.

```mermaid
stateDiagram-v2
    state "Pending Contract" as PendingContract
    state "Pending Payment" as PendingPayment
    state "Awaiting Stock" as AwaitingStock
    state "Ready to Fulfill" as ReadyToFulfill
    state "Partially Fulfilled" as PartiallyFulfilled
    state "On Hold" as OnHold
    state "Delivery Failed" as DeliveryFailed
    state "Return Requested" as ReturnRequested
    state "Partially Refunded" as PartiallyRefunded

    [*] --> Draft: Sale tạo từ báo giá
    Draft --> PendingContract: Gửi hợp đồng
    PendingContract --> Draft: Huỷ hợp đồng để lập lại
    PendingContract --> PendingPayment: Ký, còn đợt ON_SIGNING
    PendingContract --> Confirmed: Ký, đủ điều kiện
    PendingContract --> OnHold: Ký, tín dụng vượt / quá hạn
    OnHold --> PendingPayment: Kế toán duyệt, còn đợt ON_SIGNING
    OnHold --> Confirmed: Kế toán duyệt / gỡ treo
    PendingPayment --> Confirmed: Thu đủ đợt ON_SIGNING
    Confirmed --> ReadyToFulfill: Đủ hàng, release
    Confirmed --> AwaitingStock: Thiếu, gửi Yêu cầu sản xuất
    AwaitingStock --> ReadyToFulfill: Nhận đủ hàng
    AwaitingStock --> PartiallyFulfilled: Giao trước phần có sẵn
    ReadyToFulfill --> Picking: Tạo task lấy hàng
    Picking --> Packed: Kiểm đủ thùng và dán nhãn
    Picking --> PartiallyFulfilled
    Packed --> PartiallyFulfilled
    PartiallyFulfilled --> Packed
    Packed --> Shipped: Bàn giao (đợt BEFORE_DELIVERY đã thu)
    PartiallyFulfilled --> Shipped
    Shipped --> PartiallyFulfilled
    Shipped --> Delivered
    Shipped --> DeliveryFailed
    PartiallyFulfilled --> Delivered
    PartiallyFulfilled --> DeliveryFailed
    DeliveryFailed --> Delivered: Giao lại thành công
    DeliveryFailed --> PartiallyFulfilled
    DeliveryFailed --> Returned
    PartiallyFulfilled --> Returned
    Delivered --> Completed: Tự động sau 7 ngày, mọi đợt đã thu
    Delivered --> ReturnRequested
    ReturnRequested --> Delivered: Từ chối
    ReturnRequested --> Returned
    Returned --> Refunded
    Returned --> PartiallyRefunded
    Completed --> Closed: Sau 30 ngày
    Refunded --> Closed: Sau 30 ngày
    PartiallyRefunded --> Closed: Sau 30 ngày
    Draft --> Cancelled
    PendingContract --> Cancelled
    PendingPayment --> Cancelled
    Confirmed --> Cancelled
    AwaitingStock --> Cancelled
    ReadyToFulfill --> Cancelled
    Picking --> Cancelled
    OnHold --> Cancelled
    Closed --> [*]
    Cancelled --> [*]
```

> Treo tay (`On Hold`) được từ `Confirmed`, `Awaiting Stock`, `Ready to Fulfill`, `Picking`, `Partially Fulfilled`, `Packed`; gỡ treo quay về đúng trạng thái trước đó ([17](../ecommerce/17-order-management/README.md) §4.9). Sơ đồ chỉ vẽ nhánh treo tín dụng để dễ đọc. `Packed` = **đã kiểm đủ thùng & dán nhãn giao hàng** ([08](../warehouse/08-packing/README.md)). Mốc hết huỷ là khi **thùng đầu tiên của đơn được dán nhãn giao hàng** (08 BR-04): từ đó đơn không huỷ được, không đổi cách giao, không đổi địa chỉ giao — kể cả khi đơn còn `Picking`.

### 4.3. Vòng đời hợp đồng mua bán & đợt thanh toán

Chi tiết: [14 · §5.2 Hợp đồng mua bán](../ecommerce/14-cart-and-order/README.md), [15 · §5.1 Đợt thanh toán](../ecommerce/15-payment/README.md).

| Mốc đến hạn của đợt | Chuyển `Due` khi | Ảnh hưởng tới đơn |
|---------------------|------------------|-------------------|
| `ON_SIGNING` | Hợp đồng `Signed` | Chưa thu đủ → đơn `Pending Payment` |
| `BEFORE_DELIVERY` | Đơn `Ready to Fulfill` | Chưa `Paid` → chặn bàn giao vận chuyển ([09](../warehouse/09-shipping/README.md) BR-04) |
| `AFTER_DELIVERY` | Đơn `Delivered` (+ N ngày là hạn trả) | Công nợ phải thu có kỳ hạn |

**Hợp đồng mua bán (Sales Contract):**

```mermaid
stateDiagram-v2
    [*] --> Draft: Sale lập cho đơn Draft
    Draft --> Sent: Gửi khách (đơn → Pending Contract)
    Sent --> Signed: Tải bản scan đã ký
    Signed --> Completed: Đơn hoàn tất, mọi đợt đã thu
    Draft --> Cancelled
    Sent --> Cancelled: Khách từ chối / cần sửa / huỷ đơn
    Completed --> [*]
    Cancelled --> [*]
```

**Đợt thanh toán (Payment Installment):**

```mermaid
stateDiagram-v2
    state "Not Due" as NotDue
    state "Partially Paid" as PartiallyPaid
    [*] --> NotDue: Hợp đồng Signed
    NotDue --> Due: Tới mốc (ký / sẵn sàng xuất / giao xong)
    Due --> PartiallyPaid: Phân bổ một phần
    Due --> Paid: Phân bổ đủ
    Due --> Overdue: Quét đêm, quá hạn
    PartiallyPaid --> Paid
    PartiallyPaid --> Overdue
    Overdue --> Paid: Trả đủ
    NotDue --> Cancelled: Huỷ đơn
    Due --> Cancelled: Huỷ đơn
    PartiallyPaid --> Cancelled: Huỷ đơn
    Overdue --> Cancelled: Huỷ đơn
    Paid --> [*]
    Cancelled --> [*]
```

> ⚠️ Giả định: trạng thái `Cancelled` của đợt (khi huỷ đơn) bổ sung ngoài danh sách trạng thái đã chốt; đơn huỷ sau khi hợp đồng `Signed` thì hợp đồng giữ `Signed` và chuyển `Completed` khi phí huỷ / hoàn tiền xử lý xong. **Khoản phí huỷ** dùng cùng vòng đời đợt, bắt đầu từ `Due`. Xem [open-questions](../open-questions/README.md).

### 4.4. Luồng di chuyển nguyên thùng trong một kho (Intra-warehouse)

Chi tiết: [11 · Intra-warehouse Transfer](../warehouse/11-intra-warehouse-transfer/README.md). **Di chuyển nguyên thùng LPN bằng tay nằm trong phạm vi hiện tại** *(Đã chốt 2026-11-04)*: quét thùng, quét vị trí đích, ghi ngay; không mở thùng, không di chuyển một phần thùng. Tự động đưa thùng ra vùng lấy hàng, re-slotting và nhiệm vụ di chuyển là phạm vi giai đoạn sau ([C2, C24](../open-questions/README.md)). Lấy hàng về khu chờ xuất và cất lại thùng của đơn huỷ thuộc [07](../warehouse/07-picking/README.md), cách ly sang khu `QUARANTINE` thuộc [20](../warehouse/20-inventory-control/README.md) — không đi luồng này.

> ⚠️ Giả định: một lần xác nhận gồm **một hoặc nhiều thùng** tới **một vị trí đích**, mỗi thùng một dòng sổ `MOVE`; một thùng không hợp lệ thì cả lần không thực hiện. Thao tác không kiểm lớp lưu trữ, sức chứa, tải trọng của vị trí đích. Xem [open-questions C24](../open-questions/README.md).

```mermaid
flowchart TD
    START(["NV kho / Quản lý kho quét 1..n thùng LPN<br/>rồi quét vị trí đích — kiểm cho từng thùng"]) --> INSTOCK{"Thùng tồn tại<br/>và đang chứa hàng — In Stock?"}
    INSTOCK -->|Không| REJ
    INSTOCK -->|Có| SAME{"Đích khác vị trí hiện tại của thùng, cùng kho,<br/>cả hai vị trí thuộc kho được gán?"}
    SAME -->|"Khác kho"| REJ_TO(["Từ chối — khác kho thì dùng lệnh chuyển kho (10)"])
    SAME -->|Có| DEST{"Vị trí đích Active<br/>và không phải khu STAGING?"}
    DEST -->|Không| REJ
    DEST -->|Có| CNT{"Vị trí nguồn / đích đang bị phiếu kiểm kê giữ<br/>cho SKU / lô của thùng?"}
    CNT -->|Có| REJ
    CNT -->|Không| HELD{"Thùng đang giữ / phân bổ cho đơn<br/>kể cả ở khu chờ xuất, hoặc cho lệnh chuyển kho?"}
    HELD -->|Có| REJ(["Từ chối cả lần — không thùng nào được di chuyển<br/>báo lỗi theo từng thùng"])
    HELD -->|"Không — mọi thùng hợp lệ"| MOVE["Chuyển ngay nguyên thùng<br/>giữ SKU, lô, hạn dùng, ngày nhận, số trong thùng, trạng thái<br/>mỗi thùng một dòng sổ MOVE — tổng tồn kho không đổi"]
    MOVE --> DONE(["Xong"])
```

> Thùng không nằm ở vị trí sổ sách: không dùng thao tác này để sửa — lập phiếu kiểm kê, chênh lệch đi qua điều chỉnh tồn có duyệt ([20](../warehouse/20-inventory-control/README.md) §4.2–4.3). Thùng nghi lệch số cái chuẩn: cách ly sang khu `QUARANTINE`, không phân bổ, ghi giảm cả thùng bằng điều chỉnh có duyệt (⚠️ giả định — [open-questions C34](../open-questions/README.md)).

---

## 5. Ghi chú về sơ đồ

- Các trạng thái viết hoa kiểu `Confirmed`, `In Transit`… là trạng thái chứng từ / nhiệm vụ, khớp với bảng vòng đời trong từng module; khi lệch, **bảng vòng đời trong module là chuẩn**. Trong `stateDiagram-v2`, trạng thái nhiều từ được khai báo bằng mã không dấu cách kèm nhãn hiển thị (ví dụ `state "Pending Contract" as PendingContract`).
- Sơ đồ mô tả **happy path + các nhánh ngoại lệ chính**; edge case chi tiết xem phần "Ràng buộc nghiệp vụ & Edge case" của module tương ứng.
- **Sản xuất nằm ngoài hệ thống** *(Đã chốt 2026-11-04)*: bước của bộ phận sản xuất vẽ dạng khối "ngoài hệ thống"; hệ thống chỉ quản lý **Yêu cầu sản xuất** ([17](../ecommerce/17-order-management/README.md) §4.3, §5.3) và **nhập kho thành phẩm** ([03](../warehouse/03-receipt/README.md)). Kho không kiểm tra chất lượng khi nhập.
- **Thùng LPN** *(Đã chốt 2026-11-04)*: mọi tồn nằm trong thùng do bộ phận sản xuất đóng sẵn; nhập, cất, di chuyển, lấy, chuyển kho và kiểm kê đều **nguyên thùng**, xác nhận bằng quét LPN. Thùng LPN là **kiện giao** — "kiện" trong các sơ đồ giao hàng là thùng LPN, kích thước / khối lượng theo quy cách thùng của SKU ([01](../warehouse/01-product-creation/README.md), [08](../warehouse/08-packing/README.md) BR-02, BR-07).
- **Số thùng** *(Đã chốt 2026-11-05)*: "số lượng" của dòng báo giá, đơn, Yêu cầu sản xuất, phiếu nhập, lệnh chuyển kho trong các sơ đồ là **số thùng**; mọi thùng đúng số cái chuẩn của SKU (không có thùng lẻ), giá vẫn theo cái; phân bổ = chọn đủ N thùng FEFO / FIFO ([01](../warehouse/01-product-creation/README.md) BR-11, [07](../warehouse/07-picking/README.md) BR-03).
- **Đơn vị vận chuyển** cũng nằm ngoài hệ thống: không tích hợp API, không theo dõi hành trình, không thu tiền khi giao; kết quả giao do Điều phối cập nhật ([09](../warehouse/09-shipping/README.md)).
- Bước vẽ **nét đứt** (`classDef later`) là **phạm vi giai đoạn sau**; mũi tên nét đứt là nhánh phụ / sự kiện chờ.
