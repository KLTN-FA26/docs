# Sơ đồ luồng dữ liệu & trạng thái (Flow & State Diagrams)

Tất cả sơ đồ vẽ bằng **Mermaid**. Trên GitHub/GitLab và hầu hết trình xem Markdown, các khối ```mermaid``` được render tự động. Nếu trình xem không hỗ trợ, dán nội dung vào <https://mermaid.live>.

Mục lục:

1. [Luồng Procurement: PO → Receipt → Invoice → Putaway](#1-luồng-procurement)
2. [Luồng Fulfillment: Order → (Production) → Picking → Packing → Shipping](#2-luồng-fulfillment) — kèm vòng đời lệnh sản xuất (§2.3) và kế hoạch chuyến xe (§2.4)
3. [Luồng Transfer giữa các kho](#3-luồng-transfer-giữa-các-kho)
4. [Phụ lục: Luồng Storefront checkout & luồng Transfer nội bộ kho](#4-phụ-lục)

---

## 1. Luồng Procurement

### 1.1. Flowchart tổng thể

```mermaid
flowchart TD
    START([Nhu cầu bổ sung tồn kho]) --> MRP[MRP đối soát tồn\nsinh Replenishment Proposal]
    MRP --> PLAN_REV[Planner đánh giá\nConvert to Draft PO]
    PLAN_REV --> PO_CREATE[Tạo PO – Draft]
    PO_CREATE --> PO_APPROVE{Vượt hạn mức?}
    PO_APPROVE -- Có --> PO_PENDING[Pending Approval] --> PO_APPROVED[Approved]
    PO_APPROVE -- Không --> PO_APPROVED
    PO_APPROVED --> PO_CONFIRM[Chốt PO\nPO: Confirmed]
    PO_CONFIRM --> WAIT[Chờ hàng về]

    WAIT --> RC_CREATE[Tạo Receipt tham chiếu PO]
    RC_CREATE --> RC_COUNT[Kiểm đếm & quét mã\nGhi lô/hạn dùng/serial]
    RC_COUNT --> RC_CONFIRM[Xác nhận Receipt\nCập nhật SL đã nhận trên PO\nTồn Inbound tại WH/Input - RECEIVING]

    RC_CONFIRM --> PO_STATUS{PO đã nhận đủ?}
    PO_STATUS -- Chưa --> PO_PARTIAL[PO: Partially Received] --> WAIT
    PO_STATUS -- Đủ --> PO_RECEIVED[PO: Received]

    RC_CONFIRM --> RC_NEEDQC{SKU yêu cầu QC?}
    RC_NEEDQC -- Không, 2 bước --> PA_TASK
    RC_NEEDQC -- Có, 3 bước --> QC_MOVE[Chuyển hàng sang\nWH/Quality Control - QUALITY_CONTROL]
    QC_MOVE --> RC_QC{Kiểm tra chất lượng}
    RC_QC -- Rejected --> RTV[Chuyển sang khu QUARANTINE\nTồn Blocked, chờ trả NCC]
    RC_QC -- Quarantine --> QUAR[Chuyển sang khu QUARANTINE\nCách ly chờ kết luận]
    QUAR -- Kết luận Accepted --> PA_TASK
    QUAR -- Kết luận Rejected --> RTV
    RC_QC -- Accepted --> PA_TASK[Sinh Putaway task\ntừ khu chờ cất]

    PA_TASK --> PA_SUGGEST[Slotting Engine gợi ý vị trí]
    PA_SUGGEST --> PA_MOVE[NV kho di chuyển & quét vị trí đích]
    PA_MOVE --> PA_DONE[Putaway Completed - WH/Stock\nTồn: Inbound → Available]
    PA_DONE --> RC_CLOSE[Receipt: Closed]

    RC_CONFIRM --> INV_MATCH
    subgraph INVOICE [Đối chiếu hoá đơn - song song]
        INV_RECV[Nhận hoá đơn NCC] --> INV_MATCH[Three-way matching\nPO ↔ Receipt ↔ Invoice]
        INV_MATCH --> INV_RESULT{Khớp?}
        INV_RESULT -- Có --> INV_OK[Matched → Approved for Payment]
        INV_RESULT -- Không --> INV_EXC[Exception:\nchênh giá / SL / thuế]
        INV_EXC --> INV_DISPUTE[Xử lý / Dispute với NCC]
        INV_DISPUTE --> INV_MATCH
        INV_OK --> INV_PAID[Paid]
    end

    PO_RECEIVED --> PO_CLOSE{Hàng nhận đủ\n& hoá đơn đã đối chiếu?}
    INV_PAID --> PO_CLOSE
    PO_CLOSE -- Có --> PO_CLOSED([PO: Closed])
```

### 1.2. Sequence diagram (trình tự tác nhân)

```mermaid
sequenceDiagram
    autonumber
    actor PROC as Procurement
    participant PO as Purchase Order
    actor VENDOR as Nhà cung cấp
    actor WH as NV kho
    actor QC as NV QC
    participant RC as Receipt
    participant SLOT as Slotting Engine
    participant INVSYS as Inventory
    actor AP as Kế toán
    participant INV as Invoice

    PROC->>PO: Tạo PO (Draft) + dòng hàng
    PO->>PO: Duyệt (nếu vượt hạn mức) → Approved
    PO->>PO: Chốt PO → Confirmed
    VENDOR->>WH: Giao hàng + phiếu giao
    WH->>RC: Tạo Receipt tham chiếu PO
    WH->>RC: Kiểm đếm, ghi lô/hạn dùng, xác nhận
    RC->>INVSYS: Ghi tồn Inbound tại WH/Input (RECEIVING)
    RC->>PO: Cập nhật SL đã nhận / open qty
    opt SKU yêu cầu QC (luồng 3 bước)
        WH->>INVSYS: Chuyển hàng WH/Input → WH/Quality Control
        QC->>RC: Kết luận Accepted / Quarantine / Rejected
        RC->>INVSYS: Phần Quarantine / Rejected → khu QUARANTINE
    end
    RC->>WH: Sinh Putaway task (hàng Accepted / không cần QC)
    WH->>SLOT: Yêu cầu gợi ý vị trí
    SLOT-->>WH: Danh sách vị trí xếp hạng
    WH->>INVSYS: Đặt hàng vào bin (WH/Stock) → tồn Available
    RC->>RC: Closed
    VENDOR->>AP: Gửi hoá đơn
    AP->>INV: Nhập hoá đơn, chọn PO/Receipt
    INV->>INV: Three-way matching
    alt Khớp
        INV->>AP: Approved for Payment → Paid
    else Có chênh lệch
        INV->>PROC: Exception (chênh giá/SL)
        PROC->>VENDOR: Dispute / yêu cầu credit note
        VENDOR-->>INV: Hoá đơn điều chỉnh
        INV->>AP: Matched → Paid
    end
    AP->>PO: Hoá đơn đã đối chiếu
    PO->>PO: Closed
```

---

## 2. Luồng Fulfillment

### 2.1. Flowchart tổng thể

```mermaid
flowchart TD
    ORDER([Order: Confirmed]) --> SOURCE[Order Mgmt chọn kho xuất\ndòng tùy chỉnh: kho có xưởng in]
    SOURCE --> PRINT{Có dòng tùy chỉnh?}
    PRINT -- Không --> READY[Order: Ready to Fulfill\nrelease lệnh xuất]

    PRINT -- Có --> MO[Sinh lệnh sản xuất / dòng\nOrder: In Production]
    MO --> PREPRESS{Prepress file in đạt?}
    PREPRESS -- Không --> MO_HOLD[Lệnh On Hold\nSale liên hệ khách sửa file]
    MO_HOLD --> PREPRESS
    PREPRESS -- Có --> BLANK[Lấy phôi từ bin\n→ khu PRODUCTION]
    BLANK --> QUEUE[Hàng đợi → gán trạm in\ncùng kỹ thuật in]
    QUEUE --> PRINTING[Production Staff in\nbáo SL đạt / hỏng]
    PRINTING --> PQC{QC thành phẩm\nđủ SL đạt?}
    PQC -- Không --> REWORK[Phần hỏng xuất huỷ\nLệnh Rework]
    REWORK --> BLANK
    PQC -- Có --> TO_PACK[Thành phẩm → khu PACKING\nLệnh Completed]
    TO_PACK --> ALL_DONE{Mọi lệnh của đơn\nđã Completed?}
    ALL_DONE -- Chưa --> WAIT_MO[Chờ các lệnh còn lại]
    WAIT_MO --> ALL_DONE
    ALL_DONE -- Rồi --> READY_P[Order: Ready to Fulfill]
    MO -. đơn hỗn hợp: dòng hàng thường\npick song song .-> ALLOC
    READY_P --> HAS_STD{Còn dòng hàng thường\nchưa pick?}
    HAS_STD -- Có --> ALLOC
    HAS_STD -- Không --> PACK_CHECK

    READY --> ALLOC[Phân bổ tồn theo vị trí & lô\nTồn: Available → Allocated]
    ALLOC --> WAVE[Lập wave / batch\nSinh pick list + lộ trình]
    WAVE --> PICK[Picker lấy hàng\nquét vị trí + SKU/lô + số lượng]

    PICK --> SHORT{Đủ hàng?}
    SHORT -- Thiếu --> REALLOC[Phân bổ lại từ vị trí/lô khác]
    REALLOC --> PICK
    SHORT -- Vẫn thiếu --> PARTIAL[Short pick →\nbackorder / giao một phần]
    SHORT -- Đủ --> STAGE[Gom về khu đóng gói\nTồn: Picked → Staged]
    PARTIAL --> STAGE

    STAGE --> PACK_CHECK[Packing: kiểm đơn\nquét từng SKU / serial]
    PACK_CHECK --> PACK_DESIGN{Sản phẩm in ấn?}
    PACK_DESIGN -- Có --> PACK_VERIFY[Đối chiếu thành phẩm\nvới design snapshot]
    PACK_DESIGN -- Không --> PACK_BOX
    PACK_VERIFY --> PACK_FAIL{Khớp?}
    PACK_FAIL -- Không --> INVESTIGATE[Verification Failed →\nxuất huỷ thành phẩm lỗi,\nlệnh sản xuất Rework]
    PACK_FAIL -- Có --> PACK_BOX[Chọn thùng, đóng gói,\ncân & đo]
    INVESTIGATE --> REWORK

    PACK_BOX --> MODE{Hình thức giao?}
    MODE -- Gửi kiện qua hãng --> LABEL[Gọi Carrier API\nTạo shipping label + tracking]
    MODE -- Giao nguyên chuyến --> FTL_PACKED[Order: Packed\nNhãn kiện nội bộ\nShipment: Ready to Dispatch]
    FTL_PACKED --> LOADPLAN[Load Planning 3D\nKế hoạch chuyến xe:\nDraft → Planned → Confirmed]
    LOADPLAN --> LOADING[Đặt xe, xếp kiện theo sơ đồ\nmanifest theo từng xe\nShipment: Handed Over]
    LOADING --> SHIPPED
    LABEL --> LABEL_OK{Tạo nhãn OK?}
    LABEL_OK -- Không (địa chỉ sai / lỗi API) --> HOLD[Order: On Hold\nCSKH liên hệ khách]
    HOLD --> LABEL
    LABEL_OK -- Có --> PACKED[Order: Packed\nDán nhãn, niêm phong]

    PACKED --> HANDOVER[Bàn giao Carrier + manifest\nShipment: Handed Over]
    HANDOVER --> SHIPPED[Order: Shipped]
    SHIPPED --> TRANSIT[In Transit → Out for Delivery]
    TRANSIT --> DELIV{Giao thành công?}
    DELIV -- Có --> DELIVERED[Order: Delivered]
    DELIV -- Thất bại --> RETRY{Thử lại?}
    RETRY -- Có --> TRANSIT
    RETRY -- Không --> RTO[Returning → Returned\nNhập lại kho qua Receipt]

    DELIVERED --> COD{Đơn COD?}
    COD -- Có --> RECON[Đối soát tiền carrier\npayment_status → Paid]
    COD -- Không --> DONE
    RECON --> DONE([Order: Completed / Closed])
```

### 2.2. Sequence diagram (trình tự tác nhân)

```mermaid
sequenceDiagram
    autonumber
    actor CUST as Khách hàng
    participant OM as Order Management
    participant PAY as Payment
    participant PROD as Production (19)
    actor ADMIN as E-commerce Admin
    actor OPER as Production Staff
    actor QC as NV QC
    actor PICKER as Picker
    participant INVSYS as Inventory
    actor PACKER as Packer
    participant SHIP as Shipping
    participant CARRIER as Carrier (hãng vận chuyển)

    CUST->>PAY: Thanh toán đơn
    PAY-->>OM: payment_status = Paid / COD hợp lệ → Confirmed
    OM->>OM: Chọn kho xuất (dòng tùy chỉnh: kho có xưởng in)
    opt Đơn có dòng tùy chỉnh
        OM->>PROD: Sinh lệnh sản xuất / dòng (Order: In Production)
        ADMIN->>PROD: Prepress file in → Waiting Material
        PROD->>PICKER: Lệnh lấy phôi (đích khu PRODUCTION)
        PICKER->>INVSYS: Lấy phôi, bàn giao khu PRODUCTION (Allocated)
        OPER->>PROD: In tại trạm, báo SL đạt / hỏng → QC Pending
        QC->>PROD: QC thành phẩm
        alt Thiếu SL đạt
            PROD->>INVSYS: Xuất huỷ phần hỏng
            PROD->>PICKER: Cấp phôi phần thiếu (Rework)
        end
        PROD->>INVSYS: Thành phẩm → khu PACKING (Staged)
        PROD-->>OM: Mọi lệnh Completed → Ready to Fulfill
    end
    OM->>INVSYS: Phân bổ tồn dòng hàng thường (Allocated)
    OM->>PICKER: Release lệnh xuất / pick list (nếu còn dòng hàng thường)
    PICKER->>INVSYS: Lấy hàng (quét vị trí + SKU/lô)
    alt Thiếu hàng
        PICKER->>INVSYS: Re-allocate lô/vị trí khác
        PICKER-->>OM: Short pick → backorder/partial
    end
    PICKER->>PACKER: Bàn giao hàng đã pick (Staged)
    PACKER->>PACKER: Kiểm đơn + (dòng tùy chỉnh) đối chiếu thành phẩm với design snapshot
    alt Gửi kiện qua hãng
        PACKER->>SHIP: Yêu cầu tạo nhãn vận đơn
        SHIP->>CARRIER: Tạo label (API)
        CARRIER-->>SHIP: Tracking number + label
        PACKER-->>OM: Order = Packed
        PACKER->>CARRIER: Bàn giao kiện + manifest
    else Giao nguyên chuyến
        PACKER-->>OM: Order = Packed (nhãn kiện nội bộ)
        SHIP->>SHIP: Load Planning 3D → kế hoạch chuyến xe Confirmed
        SHIP->>CARRIER: Đặt xe, xếp kiện theo sơ đồ + manifest theo từng xe
    end
    CARRIER-->>OM: Shipped → In Transit → Out for Delivery
    CARRIER-->>CUST: Giao hàng
    alt Giao thành công
        CARRIER-->>OM: Delivered
        opt COD
            CARRIER->>PAY: Chuyển tiền thu hộ (đối soát)
            PAY-->>OM: payment_status = Paid
        end
        OM->>OM: Completed / Closed
    else Giao thất bại
        CARRIER-->>OM: Delivery Failed → Returning
        CARRIER->>SHIP: Hoàn hàng về kho (RTO)
        SHIP->>OM: Returned (nhập lại kho qua Receipt)
        opt Đã thu tiền
            OM->>PAY: Khởi tạo refund
        end
    end
```

### 2.3. Vòng đời lệnh sản xuất (Production Order)

Chi tiết trạng thái & điều kiện chuyển xem [19 · Production §5](../warehouse/19-production/README.md).

```mermaid
stateDiagram-v2
    [*] --> Created
    Created --> WaitingMaterial: Prepress đạt
    Created --> OnHold: File lỗi
    WaitingMaterial --> Queued: Phôi đã lên khu PRODUCTION
    WaitingMaterial --> OnHold: Thiếu phôi
    Queued --> Assigned: Gán trạm in
    Assigned --> Queued: Gán lại
    Assigned --> InProgress: Bắt đầu in
    InProgress --> QCPending: Báo SL đạt / hỏng
    QCPending --> Completed: Đủ SL đạt → khu PACKING
    QCPending --> Rework: Thiếu SL đạt
    Rework --> WaitingMaterial: Cấp phôi mới phần thiếu
    Rework --> OnHold: Vượt ngưỡng in lại
    OnHold --> Created
    OnHold --> WaitingMaterial
    OnHold --> Queued
    Created --> Cancelled
    WaitingMaterial --> Cancelled
    Queued --> Cancelled
    Assigned --> Cancelled
    OnHold --> Cancelled
    Completed --> [*]
    Cancelled --> [*]
```

### 2.4. Vòng đời kế hoạch chuyến xe (Load Plan — giao nguyên chuyến)

Chi tiết: [09 · Shipping §4.2](../warehouse/09-shipping/README.md#42-giao-nguyên-chuyến--lập-kế-hoạch-xếp-hàng-3d-load-planning). Dùng cho đơn khách giao nguyên chuyến và lệnh chuyển kho đi xe tải / container.

```mermaid
stateDiagram-v2
    [*] --> Draft
    Draft --> Planned: Tính phương án xếp
    Planned --> Planned: Chỉnh tay + tính lại xe
    Planned --> Draft: Đổi kiện / loại xe / chế độ
    Planned --> Confirmed: Chốt kế hoạch
    Confirmed --> Planned: Mở lại (chưa xếp lên xe)
    Confirmed --> Dispatched: Mọi xe bàn giao & rời kho
    Draft --> Cancelled
    Planned --> Cancelled
    Confirmed --> Cancelled
    Dispatched --> [*]
    Cancelled --> [*]
```

---

## 3. Luồng Transfer giữa các kho

### 3.1. Flowchart tổng thể

```mermaid
flowchart TD
    NEED([Nhu cầu cân bằng tồn / đưa hàng gần vùng bán]) --> TO_CREATE[Tạo Transfer Order – Draft\nKho nguồn → Kho đích + dòng SKU]
    TO_CREATE --> TO_CHECK{SL yêu cầu ≤ tồn Available\ntại kho nguồn?}
    TO_CHECK -- Không --> ADJUST[Điều chỉnh SL / chọn kho nguồn khác]
    ADJUST --> TO_CREATE
    TO_CHECK -- Có --> TO_APPROVE{Vượt ngưỡng duyệt?}
    TO_APPROVE -- Có --> TO_PENDING[Pending Approval] --> TO_APPROVED[Approved]
    TO_APPROVE -- Không --> TO_APPROVED

    TO_APPROVED --> SRC_PICK["Kho nguồn: phân bổ tồn + pick\n(FEFO theo lô)"]
    SRC_PICK --> SRC_PACK[Đóng kiện, ghi khối lượng]
    SRC_PACK --> TRUCK{Đi xe tải / container?}
    TRUCK -- Có --> SRC_LOADPLAN[Load Planning 3D\nchốt kế hoạch chuyến xe,\nxếp kiện theo sơ đồ]
    SRC_LOADPLAN --> DISPATCH
    TRUCK -- Không --> DISPATCH[Xuất kho nguồn – Dispatch\nTồn kho nguồn giảm → In-Transit]

    DISPATCH --> TRANSPORT["Vận chuyển giữa 2 kho\n(có/không tracking)"]
    TRANSPORT --> DST_RECV[Kho đích: tạo phiếu nhận\ntham chiếu Transfer Order]
    DST_RECV --> DST_COUNT[Kiểm đếm SL thực nhận + tình trạng]
    DST_COUNT --> DISCREPANCY{Khớp SL xuất?}
    DISCREPANCY -- Lệch --> INVESTIGATE["Ghi transfer discrepancy\nĐiều tra / claim hãng VC\nĐiều chỉnh In-Transit"]
    DISCREPANCY -- Khớp --> DST_PA
    INVESTIGATE --> DST_PA[Sinh Putaway task tại kho đích]

    DST_PA --> DST_SLOT[Slotting Engine kho đích gợi ý vị trí]
    DST_SLOT --> DST_MOVE[Đặt hàng vào vị trí\nTồn: In-Transit → Available tại kho đích]
    DST_MOVE --> TO_STATUS{Đã nhận & putaway hết dòng?}
    TO_STATUS -- Chưa --> DST_RECV
    TO_STATUS -- Rồi --> TO_DONE([Transfer Order: Completed / Closed])
```

### 3.2. Sequence diagram (trình tự tác nhân)

```mermaid
sequenceDiagram
    autonumber
    actor PLAN as Inventory Planner
    participant TO as Transfer Order
    actor SRC as Kho nguồn
    participant INV_SRC as Tồn kho nguồn
    participant TRANS as Vận chuyển
    actor DST as Kho đích
    participant SLOT as Slotting Engine (kho đích)
    participant INV_DST as Tồn kho đích

    PLAN->>TO: Tạo TO (Draft) – nguồn/đích + SKU/SL
    TO->>INV_SRC: Kiểm tra tồn Available tại kho nguồn
    TO->>TO: Duyệt (nếu vượt ngưỡng) → Approved
    TO->>SRC: Yêu cầu pick (phân bổ tồn, FEFO)
    SRC->>INV_SRC: Lấy hàng khỏi vị trí
    opt Đi xe tải / container
        SRC->>SRC: Load Planning 3D → kế hoạch chuyến xe, xếp kiện theo sơ đồ
    end
    SRC->>TRANS: Xuất kho nguồn (Dispatch)
    INV_SRC->>INV_SRC: Giảm tồn → trạng thái In-Transit (gắn TO)
    TRANS->>DST: Giao hàng tới kho đích
    DST->>TO: Tạo phiếu nhận tham chiếu TO
    DST->>DST: Kiểm đếm SL & tình trạng
    alt Lệch SL xuất/nhận
        DST->>TO: Ghi transfer discrepancy
        TO->>TRANS: Claim / điều tra
        DST->>INV_DST: Điều chỉnh In-Transit tương ứng
    end
    DST->>SLOT: Yêu cầu gợi ý vị trí (kho đích)
    SLOT-->>DST: Vị trí đích đề xuất
    DST->>INV_DST: Putaway → In-Transit chuyển thành Available (kho đích)
    TO->>TO: Completed / Closed
```

---

## 4. Phụ lục

### 4.1. Luồng Storefront checkout (Design/Catalog → Cart → Order → Payment)

```mermaid
flowchart TD
    ENTRY{Khách vào sàn} --> BROWSE[Catalog: duyệt / tìm / lọc]
    BROWSE --> KIND{Sản phẩm tùy chỉnh?}
    KIND -- Có --> DESIGN[Product Design 2D\n+ Live 3D Preview]
    DESIGN --> PREFLIGHT{Preflight đạt?}
    PREFLIGHT -- Fail --> DESIGN
    PREFLIGHT -- Pass/Warning --> CONFIRM_DSN["Chốt thiết kế\n→ Design snapshot"]
    CONFIRM_DSN --> ADDCART
    KIND -- Không --> ADDCART[Thêm vào giỏ]

    ADDCART --> CART[Giỏ hàng: sửa SL, áp voucher,\nước tính phí ship]
    CART --> CHECKOUT[Checkout: kiểm tra lần cuối\ntồn / giá / voucher]
    CHECKOUT --> ADDR[Địa chỉ giao + phương thức vận chuyển]
    ADDR --> TOTALS[Chốt giá cuối\n= subtotal + ship + thuế − giảm giá]
    TOTALS --> RESERVE["Giữ hàng (reservation)"]
    RESERVE --> CREATE_ORDER[Tạo Order: Pending Payment]
    CREATE_ORDER --> PAY[Payment: chọn phương thức]

    PAY --> METHOD{Loại thanh toán}
    METHOD -- Online gateway/ví/thẻ --> ONLINE["Xác thực qua cổng\n(webhook + verify)"]
    METHOD -- Chuyển khoản --> BANK[Awaiting Confirmation\nKế toán đối chiếu sao kê]
    METHOD -- COD --> CODCHK{Đủ điều kiện COD?}
    METHOD -- Đặt cọc --> DEPOSIT[Thu cọc → Partially Paid]

    ONLINE --> PAYRES{Kết quả}
    BANK --> PAYRES
    CODCHK -- Có --> CONFIRMED
    CODCHK -- Không --> METHOD
    DEPOSIT --> CONFIRMED
    PAYRES -- Thành công --> CONFIRMED[Order: Confirmed\nreservation → allocation]
    PAYRES -- Thất bại/hết hạn --> FAILED[Order: Payment Failed\ngiải phóng reservation]
    FAILED --> PAY

    CONFIRMED --> HANDOFF([→ Order Management → Fulfillment])
```

### 4.2. Luồng Transfer nội bộ một kho (Intra-warehouse — Replenishment)

```mermaid
flowchart TD
    MON[Giám sát tồn tại vị trí picking] --> LOW{Vị trí picking ≤ ngưỡng min?}
    LOW -- Không --> MON
    LOW -- Có --> GEN[Sinh replenishment task\nReserve → Picking bin, đưa về mức max]
    GEN --> ASSIGN["Phân công NV kho\n(ưu tiên cao nếu đang chặn picking)"]
    ASSIGN --> EXEC[Quét vị trí nguồn → SKU/lô →\nsố lượng → vị trí đích]
    EXEC --> CHK{Số đặt vào = số lấy ra?}
    CHK -- Lệch --> DISC[Ghi Discrepancy\nCycle count 2 vị trí + adjustment]
    CHK -- Khớp --> UPD[Cập nhật tồn theo vị trí\nTổng tồn kho KHÔNG đổi]
    DISC --> UPD
    UPD --> DONE([Move task: Completed])
```

---

## 5. Ghi chú về sơ đồ

- Các trạng thái viết hoa kiểu `Confirmed`, `In Transit`… là trạng thái chứng từ/nhiệm vụ, khớp với bảng vòng đời trong từng module.
- Sơ đồ mô tả **happy path + các nhánh ngoại lệ chính**; edge case chi tiết xem phần "Ràng buộc nghiệp vụ & Edge case" của module tương ứng.
- Các bước "Sản xuất/In" trong luồng Fulfillment mô tả xưởng in nội bộ của [19 · Production](../warehouse/19-production/README.md) cho dòng đơn tùy chỉnh.
