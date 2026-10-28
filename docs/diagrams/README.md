# Sơ đồ luồng dữ liệu & trạng thái (Flow & State Diagrams)

Tất cả sơ đồ vẽ bằng **Mermaid**. Trên GitHub/GitLab và hầu hết trình xem Markdown, các khối ```mermaid``` được render tự động. Nếu trình xem không hỗ trợ, dán nội dung vào <https://mermaid.live>.

Mục lục:

1. [Luồng Procurement: PO → Receipt → Invoice → Putaway](#1-luồng-procurement)
2. [Luồng Fulfillment: Order → (Production) → Picking → Packing → Shipping](#2-luồng-fulfillment) — kèm vòng đời lệnh sản xuất (§2.3) và kế hoạch chuyến xe (§2.4)
3. [Luồng Transfer giữa các kho](#3-luồng-transfer-giữa-các-kho)
4. [Phụ lục: Luồng bán sỉ, luồng sản xuất (đơn hàng, gia công ngoài, mẫu) & luồng Transfer nội bộ kho](#4-phụ-lục)

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
    ORDER([Order: Confirmed]) --> SOURCE[Order Coordinator chọn kho xuất & release đơn\ndòng tùy chỉnh: kho có xưởng in]
    SOURCE --> PRINT{Có dòng tùy chỉnh?}
    PRINT -- Không --> READY[Order: Ready to Fulfill\nrelease lệnh xuất]

    PRINT -- Có --> MO[Sinh LSX ORDER / dòng — Pending Prepress\nOrder: In Production]
    MO --> PREPRESS{Prepress file in đạt?\nchecksum khớp}
    PREPRESS -- Không --> MO_HOLD[Lệnh On Hold\nSale liên hệ khách sửa file]
    MO_HOLD --> PREPRESS
    PREPRESS -- Có --> QUEUE[LSX Ready: vào hàng đợi\nWarehouse Manager gán trạm in\ncùng kỹ thuật in]
    QUEUE --> BLANK[NV kho quét xuất phôi từ bin\nsang khu PRODUCTION\nLSX Material Issued]
    BLANK --> PRINTING[Production Staff in tại trạm đã gán\nLSX Printing → báo SL in được / phế phẩm]
    PRINTING --> PQC{LSX QC: theo mẫu đã duyệt\nđủ SL đạt?}
    PQC -- Không --> REWORK[Phần hỏng xuất huỷ\nLSX quay về Ready để in bù]
    REWORK --> QUEUE
    PQC -- Có --> TO_PACK[Thành phẩm → khu PACKING\nLSX Completed]
    PREPRESS -. xưởng không kịp hạn:\ntách một phần gia công ngoài .-> SUBC[LSX con gia công\nxem §4.2]
    SUBC -. hàng về, QC đạt .-> TO_PACK
    TO_PACK --> ALL_DONE{Mọi lệnh của đơn\nđã Completed?}
    ALL_DONE -- Chưa --> WAIT_MO[Chờ các lệnh còn lại]
    WAIT_MO --> ALL_DONE
    ALL_DONE -- Rồi --> READY_P[Order: Ready to Fulfill]
    MO -. đơn hỗn hợp: dòng hàng thường\nlấy song song .-> TASK
    READY_P --> HAS_STD{Còn dòng hàng thường\nchưa lấy?}
    HAS_STD -- Có --> TASK
    HAS_STD -- Không --> PACK_CHECK

    READY --> TASK[Điều phối / Quản lý kho tạo task\ngiao người — dòng lấy theo bin & lô\nđã phân bổ khi release]
    TASK --> PICK[Picker lấy hàng\nquét vị trí + SKU + số lượng\ntồn bị trừ ngay khi quét]

    PICK --> SHORT{Đủ hàng?}
    SHORT -- Thiếu --> SHORT_HOLD[Báo thiếu: đơn On Hold INVENTORY\nkiểm kê mù vị trí đó]
    SHORT_HOLD --> REALLOC{Điều phối phân bổ lại\ntừ bin khác cùng kho?}
    REALLOC -- Được --> PICK
    REALLOC -- Không đủ hàng --> PARTIAL[Điều phối + Sale quyết:\ngiao một phần / đóng phần còn lại / huỷ]
    SHORT -- Đủ --> PACK_CHECK

    PACK_CHECK[Packing: Packer bắt đầu phiên\nmở kiện, cho hàng vào] --> PACK_BOX[Niêm phong kiện\ncân & đo bắt buộc\nkit: đủ N kiện]
    PACK_BOX --> LABEL[In nhãn kiện nội bộ\nmã đơn, mã kiện, địa chỉ,\nsố tiền thu khi giao nếu có]
    LABEL --> COMPLETE[Hoàn tất đóng gói\nthiếu hàng: chỉ khi được giao một phần]
    COMPLETE --> PACK_DESIGN{Có dòng in?}
    PACK_DESIGN -- Không --> PACKED
    PACK_DESIGN -- Có --> PACK_FAIL{Checksum thiết kế khớp?}
    PACK_FAIL -- Không --> DESIGN_HOLD[Phiên + đơn On Hold\nDESIGN_MISMATCH]
    DESIGN_HOLD --> REVERIFY[Quản lý kho xác minh lại]
    REVERIFY --> COMPLETE
    PACK_FAIL -- Có --> PACKED[Order: Packed / Partially Fulfilled\nShipment: Ready to Dispatch]

    PACKED --> MODE{Hình thức giao?}
    MODE -- Gửi kiện qua hãng --> PICKUP[Mở chuyến bàn giao cho một hãng\nquét đủ kiện]
    PICKUP --> HANDOVER[Ký bàn giao — tất cả hoặc không\nShipment: Handed Over]
    MODE -- Giao nguyên chuyến --> LOADPLAN[Load Planning 3D\nKế hoạch chuyến xe:\nDraft → Planned → Confirmed\nmỗi xe một chuyến bàn giao]
    LOADPLAN --> BOOK[Đặt xe với hãng ngoài hệ thống\nghi nhận đặt xe]
    BOOK --> LOAD[Bắt đầu xếp → khoá kế hoạch\nquét kiện theo sơ đồ từng xe]
    LOAD --> HANDOVER
    HANDOVER --> SHIPPED[Order: Shipped\nkhông theo dõi hành trình]
    SHIPPED --> DELIV{Hãng báo kết quả\nOrder Coordinator cập nhật}
    DELIV -- Thành công --> DELIVERED[Order: Delivered]
    DELIV -- Thất bại --> FAILED[Order: Delivery Failed]
    FAILED -- Hãng giao lại thành công --> DELIVERED
    FAILED -- Hàng hoàn về --> RTO[Returned\nNhập lại kho qua Receipt]

    DELIVERED --> BAL{Còn phải thu?\ncông nợ / phần sau cọc}
    BAL -- Có --> RECON[Ghi công nợ phải thu /\nthu phần còn lại → Paid]
    BAL -- Không --> DONE
    RECON --> DONE([Order: Completed / Closed])
```

### 2.2. Sequence diagram (trình tự tác nhân)

```mermaid
sequenceDiagram
    autonumber
    actor CUST as Khách hàng sỉ
    participant OM as Order Management
    participant PAY as Payment
    participant PROD as Sản xuất in (19)
    actor OPER as Production Staff
    actor QC as NV QC
    actor PICKER as Picker
    participant INVSYS as Inventory
    actor PACKER as Packer
    participant SHIP as Shipping
    participant CARRIER as Carrier (hãng vận chuyển)

    CUST->>PAY: Chuyển khoản trả trước / tiền cọc (hoặc đơn công nợ trong hạn mức)
    PAY-->>OM: Đủ điều kiện thanh toán → Confirmed
    OM->>OM: Coordinator chọn kho xuất (dòng tùy chỉnh: kho có xưởng in) + release
    opt Đơn có dòng tùy chỉnh
        OM->>PROD: Release dòng in → LSX ORDER / dòng (Order: In Production)
        OPER->>PROD: Prepress file in (checksum): Pending Prepress → Ready
        PROD->>PROD: Hàng đợi + Warehouse Manager gán trạm in (thuộc tính của lệnh Ready)
        PICKER->>INVSYS: NV kho quét xuất phôi sang khu PRODUCTION (tồn In Production) → Material Issued
        OPER->>PROD: In tại trạm (Printing), báo SL in được / phế phẩm → QC
        QC->>PROD: QC thành phẩm (đối chiếu mẫu đã duyệt)
        alt Thiếu SL đạt
            PROD->>INVSYS: Xuất huỷ phần hỏng
            PROD->>PICKER: LSX quay về Ready, quét xuất phôi phần thiếu (in bù)
        end
        PROD->>INVSYS: Thành phẩm → khu PACKING
        PROD-->>OM: Mọi lệnh Completed → Ready to Fulfill
    end
    OM->>INVSYS: Release: phân bổ dòng hàng thường vào bin & lô
    OM->>PICKER: Điều phối tạo task, giao picker (nếu còn dòng hàng thường)
    PICKER->>INVSYS: Lấy hàng (quét vị trí + SKU + số lượng) → trừ tồn
    alt Thiếu hàng
        PICKER-->>OM: Báo thiếu → đơn On Hold INVENTORY
        PICKER->>INVSYS: Kiểm kê mù vị trí đó
        OM->>INVSYS: Điều phối phân bổ lại từ bin khác cùng kho
    end
    PICKER->>PACKER: Task Picked → mở đóng gói
    PACKER->>PACKER: Đóng & niêm phong kiện (cân đo), in nhãn kiện nội bộ
    PACKER->>PACKER: Hoàn tất → hệ thống kiểm checksum thiết kế (dòng in)
    alt Lệch thiết kế
        PACKER-->>OM: Đơn On Hold DESIGN_MISMATCH → Quản lý kho xác minh lại
    end
    PACKER-->>OM: Order = Packed, shipment Ready to Dispatch
    alt Gửi kiện qua hãng
        SHIP->>CARRIER: Mở chuyến cho hãng, quét đủ kiện, ký bàn giao
    else Giao nguyên chuyến
        SHIP->>SHIP: Load Planning 3D → kế hoạch Confirmed, mỗi xe một chuyến
        SHIP->>CARRIER: Đặt xe (ngoài hệ thống), ghi nhận đặt xe
        SHIP->>CARRIER: Xếp kiện theo sơ đồ, quét, ký bàn giao từng xe
    end
    SHIP-->>OM: Shipped
    CARRIER-->>CUST: Giao hàng (ngoài hệ thống, không tracking)
    alt Giao thành công
        CARRIER-->>OM: Báo đã giao → Order Coordinator cập nhật Delivered
        opt Đơn công nợ / còn phần sau cọc
            CUST->>PAY: Chuyển khoản theo kỳ hạn (kế toán đối soát sao kê)
            PAY-->>OM: payment_status = Paid
        end
        OM->>OM: Completed / Closed
    else Giao thất bại
        CARRIER-->>OM: Báo giao thất bại → Order Coordinator cập nhật Delivery Failed
        CARRIER->>SHIP: Hoàn hàng về kho (RTO)
        SHIP->>OM: Returned (nhập lại kho qua Receipt)
        opt Đã thu tiền
            OM->>PAY: Khởi tạo refund
        end
    end
```

### 2.3. Vòng đời lệnh sản xuất (Production Order)

Chi tiết trạng thái & điều kiện chuyển xem [19 · Production §5](../warehouse/19-production/README.md). Áp dụng cho cả LSX `SAMPLE` và `ORDER`; nhánh `Subcontracted` chỉ dành cho LSX gia công ngoài. Hàng đợi và trạm in được gán là thuộc tính của lệnh `Ready`, không phải trạng thái.

```mermaid
stateDiagram-v2
    [*] --> PendingPrepress
    PendingPrepress --> Ready: File đạt, checksum khớp
    Ready --> MaterialIssued: Phôi đã lên khu PRODUCTION (in nội bộ)
    MaterialIssued --> Printing: Bắt đầu in tại trạm đã gán
    Printing --> QC: Ghi số in được, phế phẩm
    Ready --> Subcontracted: LSX gia công — gửi nhà gia công (PO đã duyệt)
    Subcontracted --> QC: Nhận hàng về, phiếu nhận Confirmed
    QC --> Completed: Đủ số đạt → khu PACKING / bàn giao mẫu
    QC --> Ready: Thiếu → in bù / gửi bù
    PendingPrepress --> OnHold
    Ready --> OnHold
    MaterialIssued --> OnHold
    Printing --> OnHold
    Subcontracted --> OnHold
    OnHold --> PendingPrepress: Mở lại (về trạng thái trước)
    OnHold --> Ready: Mở lại
    OnHold --> MaterialIssued: Mở lại
    OnHold --> Printing: Mở lại
    OnHold --> Subcontracted: Mở lại
    PendingPrepress --> Cancelled
    Ready --> Cancelled
    MaterialIssued --> Cancelled: Trả phôi về kho
    Subcontracted --> Cancelled: Thu hồi phôi, huỷ PO gia công
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

    DISPATCH --> TRANSPORT["Vận chuyển giữa 2 kho\n(không theo dõi hành trình)"]
    TRANSPORT --> DST_RECV[Kho đích: tạo phiếu nhận\ntham chiếu Transfer Order]
    DST_RECV --> DST_COUNT[Kiểm đếm SL thực nhận + tình trạng]
    DST_COUNT --> DISCREPANCY{Khớp SL xuất?}
    DISCREPANCY -- Lệch --> INVESTIGATE["Ghi transfer discrepancy\nĐiều tra / claim hãng VC (ngoài hệ thống)\nĐiều chỉnh In-Transit"]
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

### 4.1. Luồng bán sỉ (Báo giá → Mẫu → Đơn → Thanh toán)

```mermaid
flowchart TD
    ENTRY{Khách sỉ liên hệ\nhoặc đăng nhập cổng} --> PATH{Thiết kế mới?}
    PATH -- Có --> DESIGN["Thiết kế (12)\nSale tải bộ file"]
    DESIGN --> REVIEW{QC duyệt kỹ thuật?}
    REVIEW -- Không đạt --> DESIGN
    REVIEW -- Đạt --> CUST_OK{Khách xác nhận?}
    CUST_OK -- Yêu cầu sửa --> DESIGN
    CUST_OK -- Xác nhận --> CONFIRM_DSN["Thiết kế đã xác nhận\n→ Design snapshot"]
    CONFIRM_DSN --> QUOTE[Sale lập báo giá\nbậc số lượng, MOQ, hiệu lực]
    QUOTE --> SAMPLE["Yêu cầu làm mẫu (19)\nLSX SAMPLE"]
    SAMPLE --> SAMPLE_OK{Khách duyệt mẫu?}
    SAMPLE_OK -- Yêu cầu sửa --> FORK["Tạo bản sửa\ntừ snapshot"]
    FORK --> DESIGN
    SAMPLE_OK -- Duyệt --> ACCEPT[Khách chấp nhận báo giá]
    PATH -- "Không (hàng sẵn)" --> QUICK[Đặt nhanh hàng sẵn trên cổng\ngiá riêng của khách]
    PATH -- "Đặt lại đơn có thiết kế" --> REQUOTE[Hệ thống tạo báo giá mới\ntheo giá hiện hành] --> QUOTE
    ACCEPT --> CONVERT[Sale / Điều phối\nchuyển báo giá thành đơn]
    CONVERT --> ORDER[Tạo Order\nkiểm tồn phôi, MOQ, chốt giá]
    QUICK --> ORDER
    ORDER --> RESERVE["Giữ hàng (reservation)"]
    RESERVE --> METHOD{Điều khoản thanh toán\nđược phép cho khách}
    METHOD -- Trả trước 100% --> BANK[Pending Payment\nkế toán ghi nhận tiền theo sao kê]
    METHOD -- Đặt cọc --> DEPOSIT[Pending Payment\nchờ tiền cọc]
    METHOD -- Công nợ --> CREDIT{Không quá hạn và\ndư nợ + đơn ≤ hạn mức?}
    BANK -- Đủ tiền --> CONFIRMED
    DEPOSIT -- Đủ cọc --> CONFIRMED
    CREDIT -- Có --> CONFIRMED[Order: Confirmed]
    CREDIT -- Không --> HOLD[Order: On Hold\nkế toán duyệt tín dụng]
    HOLD -- Duyệt --> CONFIRMED
    HOLD -- "Từ chối: đổi trả trước / đặt cọc" --> DEPOSIT
    HOLD -- "Từ chối: huỷ" --> CANCEL
    BANK -- Hết hạn --> CANCEL[Cancelled\ngiải phóng reservation]
    DEPOSIT -- Hết hạn --> CANCEL

    CONFIRMED --> HANDOFF([→ Order Management release → Sản xuất / Fulfillment])
```

### 4.2. Luồng sản xuất đơn hàng và gia công ngoài

```mermaid
flowchart TD
    REL[Order Coordinator release đơn\nmỗi dòng in → 1 LSX ORDER] --> PRE{Prepress đạt?}
    PRE -- Không --> HOLD[LSX On Hold\nSale liên hệ khách]
    HOLD --> PRE
    PRE -- Có --> CAP{Xưởng nội bộ kịp hạn?}
    CAP -- Có --> QUEUE[LSX Ready: hàng đợi\ngán trạm in cùng kỹ thuật in]
    QUEUE --> ISSUE[NV kho quét xuất phôi\nsang khu PRODUCTION — Material Issued]
    CAP -- "Không — Quản lý kho tách một phần" --> SPLIT["LSX con gia công\n+ PO gia công (SUBCONTRACT)"]
    SPLIT -. phần còn lại .-> QUEUE
    ISSUE --> PRINT[Printing: in, ghi phế phẩm có lý do]
    PRINT --> QC
    SPLIT --> APPR{PO gia công Approved?}
    APPR -- Có --> MODE{Cách gia công}
    MODE -- Mình cấp phôi --> SEND_B["Xuất phôi gửi nhà gia công\nvị trí ảo, không tính ATP"]
    MODE -- Nhà gia công lo phôi --> SEND_F[Gửi file in + mẫu đã duyệt]
    SEND_B --> SUBC[LSX con: Subcontracted]
    SEND_F --> SUBC
    SUBC --> RCV["Nhận hàng theo PO gia công (03)\nđếm thành phẩm, phôi / phế phẩm trả về\nđối soát hao hụt"]
    RCV --> QC{QC theo mẫu đã duyệt\nđủ số đạt?}
    QC -- "Thiếu — in bù nội bộ (về Ready)" --> QUEUE
    QC -- "Thiếu — gửi bù nhà gia công (về Ready)" --> SUBC
    QC -- Đủ --> PACK[Thành phẩm sang khu PACKING\nLSX Completed]
    PACK --> ALL{Mọi LSX của đơn xong?}
    ALL -- Có --> RTF([Order: Ready to Fulfill → Packing])
    RCV -. "hoá đơn theo số đạt QC (04)" .-> AP[Đối chiếu ba chiều\ntrừ hao hụt đã duyệt]
```

### 4.3. Luồng sản xuất mẫu (Sample)

```mermaid
flowchart TD
    REQ[Sale tạo yêu cầu làm mẫu\nkhách, phôi, thiết kế, số lượng] --> SUB[Gửi xưởng → LSX SAMPLE\ngiữ chỗ phôi]
    SUB --> PRE{Prepress đạt?}
    PRE -- Không --> HOLD[LSX On Hold\nSale liên hệ khách sửa file]
    HOLD --> PRE
    PRE -- Có --> RDY[LSX Ready\nhàng đợi, gán trạm in]
    RDY --> ISSUE[NV kho quét xuất phôi\nsang khu PRODUCTION — Material Issued]
    ISSUE --> PRINT[Printing: in mẫu\nghi phế phẩm có lý do]
    PRINT --> QC{QC đạt đủ số mẫu?}
    QC -- Thiếu, in bù --> RDY
    QC -- Đủ --> READY[Yêu cầu mẫu: Ready\nphôi trừ tồn lý do SAMPLE]
    READY --> SENT[Gửi mẫu cho khách\nngoài hệ thống]
    SENT --> FB{Khách phản hồi}
    FB -- Duyệt --> APPROVED([Mẫu Approved\nkhoá thiết kế, chuẩn QC cho đơn])
    FB -- Yêu cầu sửa --> REV[Sửa thiết kế\nlàm mẫu lần tiếp theo]
    REV --> REQ
```

### 4.4. Luồng Transfer nội bộ một kho (Intra-warehouse — Replenishment)

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
- Bước "Sản xuất in" trong luồng Fulfillment và luồng sản xuất mẫu mô tả xưởng in nội bộ (kèm gia công ngoài một phần lệnh đơn hàng) của [19 · Production](../warehouse/19-production/README.md) (đã chốt 2026-10-06).
