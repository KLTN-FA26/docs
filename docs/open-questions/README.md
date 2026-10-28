# Câu hỏi mở (Open Questions)

Tổng hợp các điểm nghiệp vụ **chưa được đề bài xác định rõ**, cần **Product Owner / người phụ trách sản phẩm** xác nhận trước khi thiết kế chi tiết & triển khai. Mỗi mục ghi: bối cảnh, giả định tạm thời tài liệu đang dùng, và tác động nếu giả định sai.

Ký hiệu mức độ ảnh hưởng: 🔴 Cao (ảnh hưởng kiến trúc/nhiều module) · 🟡 Trung bình · 🟢 Thấp (chủ yếu cấu hình/nội dung).

---

## A. Xuyên suốt hệ thống (Cross-cutting)

| # | Câu hỏi | Giả định tạm thời | Ảnh hưởng | Module liên quan |
|---|---------|-------------------|-----------|------------------|
| A1 | Hệ thống hỗ trợ **đa công ty / đa pháp nhân (multi-entity)** hay chỉ một doanh nghiệp với nhiều kho? | ✅ **Đã chốt 2026-08-29:** một pháp nhân duy nhất, **nhiều kho vật lý (multi-warehouse)**. Không hỗ trợ multi-entity. Chuyển kho là luân chuyển tồn nội bộ, không phát sinh hoá đơn/công nợ nội bộ. | 🟢 | 00, 10, 04, 15 |
| A2 | Mô hình **phân quyền (RBAC)** chi tiết: danh sách vai trò chuẩn, một người có được nhiều vai trò, phạm vi dữ liệu theo kho/nhóm khách? | ✅ **Đã chốt một phần 2026-08-29:** danh sách **11 role chuẩn** (role thứ 11 **Production Staff** bổ sung 2026-09-28 cùng module [19](../warehouse/19-production/README.md), xác nhận lại 2026-10-06) + quyền theo ngữ cảnh (phê duyệt PO/TO/chi, giám sát chat) — xem [Role Registry](../00-system-overview/README.md); một tài khoản có thể mang nhiều vai trò. **Còn mở:** phạm vi dữ liệu theo kho/nhóm khách, hạn mức cụ thể, phân bổ quyền mặc định | 🔴 | Tất cả |
| A3 | Có **quy trình maker–checker (tạo / duyệt)** cho master data (sản phẩm, NCC, danh mục) không, hay Admin toàn quyền? | Có bước duyệt; tổ chức nhỏ có thể gộp | 🟡 | 01 |
| A4 | **Đơn vị tiền tệ**: chỉ một loại (VND) hay đa tiền tệ (mua ngoại tệ, bán ngoại tệ)? | Chủ yếu VND; PO theo tiền của NCC | 🟡 | 02, 04, 15 |
| A5 | Phạm vi **tích hợp kế toán**: hệ thống chỉ đối chiếu & đề nghị thanh toán, hay hạch toán sổ cái đầy đủ? | Chỉ AP matching + đề nghị thanh toán; sổ cái ở hệ thống ngoài | 🔴 | 04, 15 |
| A6 | Yêu cầu **tuân thủ dữ liệu cá nhân** áp dụng khung nào (Nghị định 13/2023 VN, GDPR…)? Thời hạn lưu chứng từ? | Ẩn danh khi có yêu cầu xoá, giữ chứng từ theo nghĩa vụ luật định | 🟡 | 18, 16, 15 |
| A7 | **Đơn vị đo lường & bản địa hoá**: hệ mét, định dạng địa chỉ VN nhiều cấp (tỉnh/huyện/xã), đa ngôn ngữ cổng khách hàng? | Hệ mét, địa chỉ VN, tiếng Việt là chính | 🟢 | 08, 09, 13, 14 |
| A8 | **Ngành hàng & khách hàng mục tiêu** của doanh nghiệp? | ✅ **Đã chốt 2026-10-09:** chuyên **ly và bao bì dùng một lần cho ngành đồ uống** gồm hai nhóm: **ly** (ly nhựa PP/PET, ly giấy) và **bao bì mang đi** (túi đựng ly, khay giấy, túi giấy); không kinh doanh phụ kiện (nắp, ống hút…); bán hàng trơn hoặc in logo theo yêu cầu tại xưởng nội bộ (gia công ngoài một phần khi thiếu công suất — B1); khách chủ yếu là quán / chuỗi đồ uống, **chỉ bán sỉ (B2B)** — G5. Không còn kinh doanh nguyên liệu đồ uống | 🔴 | 00, 01, 12, 13, 19 |
| A9 | **Đơn vị bán & quy đổi**: khách được đặt theo đơn vị nào (chỉ thùng, hay cả cây / cái lẻ)? Cấu hình theo biến thể hay theo nhóm khách? Có cho đặt số lẻ thùng không? | Tồn ghi theo đơn vị cơ sở (cái); mỗi SKU khai báo đơn vị quy đổi (ví dụ thùng = 1.000 cái); đơn vị bán cấu hình **theo biến thể**, khách đặt **số nguyên** đơn vị bán; hệ số đã dùng không được sửa | 🟡 | 01, 02, 03, 07, 13, 14 |

---

## B. Sản phẩm in & sản xuất (Print / Production)

| # | Câu hỏi | Giả định tạm thời | Ảnh hưởng | Module |
|---|---------|-------------------|-----------|--------|
| B1 | Có **module Sản xuất/In (production / print job)** riêng giữa Order và Picking không? Ai vận hành (xưởng nội bộ / gia công ngoài)? Trạng thái & năng lực (capacity) sản xuất quản lý thế nào? | ✅ **Đã chốt 2026-09-28, bổ sung 2026-10-06:** có module [19 · Production](../warehouse/19-production/README.md) do **xưởng in nội bộ** vận hành; hai loại lệnh sản xuất: **mẫu** (`SAMPLE`) và **đơn hàng** (`ORDER` — mỗi dòng đơn tùy chỉnh = một lệnh). Quản lý **hàng đợi + trạm in** theo kỹ thuật in; công suất/ngày chỉ dùng ước tính ngày xong, không lập lịch tự động (xem B10). Role **Production Staff**; Warehouse Manager điều phối xưởng; QC Staff kiểm mẫu và thành phẩm. **2026-10-06 thay quyết định "không gia công ngoài" của 2026-09-28:** khi thiếu công suất, Quản lý kho được **tách một phần LSX `ORDER` đưa gia công ngoài** (cả hai cách: mình cấp phôi / nhà gia công lo phôi); LSX `SAMPLE` luôn in nội bộ | 🔴 | 19, 07, 08, 14, 17, 12, 03, 02, 04 |
| B2 | Picking cho đơn in ấn: pick **vật tư/phôi** cho xưởng in, hay chỉ pick **thành phẩm** sau khi in xong, hay cả hai? | ✅ **Đã chốt 2026-09-28 (xác nhận 2026-10-06):** chỉ **pick phôi** từ bin lên khu `PRODUCTION`; thành phẩm đạt QC được **chuyển thẳng sang khu `PACKING`**, không cất lại lên bin và không pick thành phẩm. Đơn chỉ gồm hàng tùy chỉnh đi `Ready to Fulfill` → `Packed`, không qua `Picking` | 🟡 | 07, 08, 19, 01 |
| B3 | **Công thức tính giá** sản phẩm tùy chỉnh: theo diện tích phủ mực, số màu, kỹ thuật in, số lượng? Nguồn công thức? | Giá dòng phụ thuộc thông số in; công thức khai báo ở [01](../warehouse/01-product-creation/README.md) | 🟡 | 12, 14, 01 |
| B4 | Chính sách **đổi/trả với sản phẩm cá nhân hoá**: chỉ nhận khi lỗi sản xuất, hay có cho đổi ý trong X ngày? | Không đổi/trả vì đổi ý; chỉ nhận khi lỗi sản xuất | 🟡 | 17, 16 |
| B5 | **Yêu cầu thay đổi thiết kế sau khi đặt hàng**: cho phép tới mốc nào (trước khi vào sản xuất)? Có tính phí? | Cho tới khi bắt đầu sản xuất; sau đó từ chối hoặc tính phí | 🟡 | 12, 17 |
| B6 | **Kiểm duyệt nội dung** khách tải lên (bản quyền, nội dung không phù hợp): tự động, thủ công, hay kết hợp? Ai chịu trách nhiệm? | Kiểm tra cơ bản (định dạng/dung lượng) + rà nội dung theo chính sách. Hiện hệ thống chỉ kiểm định dạng, dung lượng và quét mã độc; QC duyệt kỹ thuật bộ file trước khi gửi khách ([12](../ecommerce/12-product-design/README.md)) | 🟡 | 12 |
| B7 | **Live 3D Preview**: render phía client (WebGL) hay server? Yêu cầu thiết bị tối thiểu? Fallback khi không render được? | Client-side, có fallback ảnh tĩnh render server. Phụ thuộc B18 | 🟢 | 12 |
| B8 | Nguồn **model 3D & template**: doanh nghiệp tự dựng cho từng sản phẩm, hay có thư viện dùng chung? | Khai báo theo sản phẩm ở [01](../warehouse/01-product-creation/README.md). Phụ thuộc B18 | 🟢 | 01, 12 |
| B9 | **Tồn kho của sản phẩm tùy chỉnh** theo dõi trên **phôi** (hàng chưa in) hay trên thành phẩm? Mỗi thiết kế của khách có sinh SKU mới không? (liên quan B2) | ✅ **Đã chốt 2026-10-06:** Inventory Item của SKU là **phôi**; thiết kế đi kèm dòng đơn (design snapshot), không sinh SKU mới; ATP tính theo phôi | 🟡 | 01, 12, 13, 07, 19 |
| B10 | **Xếp lịch xưởng in**: theo hạn hoàn thành, theo ưu tiên khách, hay tính công suất từng máy? | Bản đầu: hàng đợi theo hạn hoàn thành rồi thời điểm release; Quản lý kho gán trạm in thủ công; công suất/ngày của trạm chỉ dùng ước tính ngày xong — hệ thống chưa tự lập lịch theo công suất máy | 🟡 | 19 |
| B11 | Có **xuất dư phôi** để bù hao hụt in không, hay chỉ in bù khi thiếu? | Không xuất dư; hao hụt bù bằng vòng in bù (LSX quay về `Ready`) | 🟢 | 19, 06 |
| B12 | **Phí làm mẫu**: miễn phí, tính riêng, hay cộng vào báo giá? Giới hạn số lần làm mẫu? | Chưa chốt | 🟡 | 19, 14 |
| B13 | **Gửi mẫu cho khách**: bàn giao trực tiếp, chuyển phát, hay chỉ gửi ảnh? Có cần theo dõi trong hệ thống? | Gửi ngoài hệ thống; hệ thống chỉ ghi nhận đã gửi | 🟢 | 19, 16 |
| B14 | **Gia công ngoài — mức hao hụt phôi cho phép** khi mình cấp phôi: một mức chung hay theo từng nhà gia công? Bao nhiêu %? | Theo từng nhà gia công, mặc định 2%; vượt mức → Quản lý kho duyệt, trừ vào hoá đơn gia công | 🟡 | 19, 03, 04 |
| B15 | **Gia công ngoài — trả tiền theo số nào**: số thành phẩm nhận, hay số **đạt QC**? Nhà gia công có dùng hệ thống (cổng NCC) không? | Trả theo số đạt QC; nhà gia công không dùng hệ thống, nhân viên mình ghi nhận mọi trạng thái | 🟡 | 19, 04, 02 |
| B16 | Kho **không có xưởng in** có được chọn làm kho xuất cho dòng tùy chỉnh không (ví dụ in ở kho A rồi chuyển sang kho B giao)? | Không — dòng tùy chỉnh chỉ giao từ kho có khu `PRODUCTION` và trạm in `Active` đúng kỹ thuật in | 🟡 | 17, 19, 10 |
| B17 | **Ngưỡng in bù (rework)** của một lệnh sản xuất là bao nhiêu lần trước khi phải dừng chờ Manager? Cấu hình theo hệ thống hay theo sản phẩm? | Cấu hình theo hệ thống; vượt ngưỡng → lệnh `On Hold` | 🟢 | 19 |
| B18 | Có làm **trình thiết kế 2D cho khách tự thiết kế**, **xem trước 3D theo thời gian thực** và **preflight tự động** (`Pass` / `Warning` / `Fail`) không? | Phạm vi giai đoạn sau. Hiện tại nhân viên (Sale / Quản trị bán hàng) tải bộ file thiết kế, QC duyệt kỹ thuật, khách xác nhận trên cổng ([12](../ecommerce/12-product-design/README.md)) | 🟡 | 12, 01, 13 |

---

## C. Kho vận & Slotting

| # | Câu hỏi | Giả định tạm thời | Ảnh hưởng | Module |
|---|---------|-------------------|-----------|--------|
| C1 | **Tiêu chí & trọng số của thuật toán gợi ý vị trí (slotting)**: thứ tự ưu tiên giữa (gần khu đóng gói / gom cùng SKU / FEFO / cân bằng tải / công thái học / mật độ)? Cấu hình được không? | ✅ **Đã chốt một phần 2026-10-28:** gợi ý cất hàng hiện xếp hạng **cùng lô → cùng SKU → bin trống → còn chỗ**, rồi bin còn nhiều chỗ hơn, rồi mã vị trí; chỉ bin cho phép cất, cùng lớp lưu trữ, còn sức chứa / tải trọng ([05](../warehouse/05-putaway/README.md) §4.2). **Còn mở:** tiêu chí nâng cao (gần khu đóng gói, velocity / ABC, FEFO, cân bằng tải) và trọng số cấu hình — bộ tiêu chí mặc định trong [06](../warehouse/06-warehouse-map-slotting/README.md) §4.2 | 🔴 | 06, 05, 11 |
| C2 | Slotting là **rule-based** hay có **học từ dữ liệu (ML)**? Mức độ tự động vs cần người duyệt? | Rule-based + feedback loop; re-slotting cần Manager duyệt | 🟡 | 06 |
| C3 | Tồn khả dụng để bán (**ATP**) tính theo **tổng toàn hệ thống** hay theo **kho phục vụ khu vực giao** của khách? | Cấu hình được; mặc định tổng toàn hệ thống | 🔴 | 13, 14, 10 |
| C4 | Tồn **In-Transit** (chuyển kho) có được tính vào ATP của kho đích để bán trước không? | Không tính cho tới khi nhận & putaway | 🟡 | 10, 13, 14 |
| C5 | Tồn khả dụng cộng **ngay khi Receipt** hay **chỉ sau Putaway**? | Chỉ sau Putaway (`Inbound` chưa khả dụng) | 🟡 | 03, 05, 13 |
| C6 | Mức độ chi tiết vị trí: quản lý tới **ô/bin** hay tới **kệ/khu**? Có bắt buộc quét mã vị trí mọi thao tác? | ✅ **Đã chốt một phần 2026-09-17:** quản lý tới **bin** trên tầng kệ (Kho → Kệ → Tầng → Bin, Zone tùy chọn); hàng trên sàn gắn với **Area**; mô hình chi tiết ở [06](../warehouse/06-warehouse-map-slotting/README.md). Bỏ các quy tắc single-SKU/single-lot, nhóm tương thích, sức chứa theo thể tích. **Còn mở:** có bắt buộc quét mã vị trí mọi thao tác (giả định: có) | 🟡 | 05, 06, 07, 11 |
| C7 | **Định giá tồn kho** (costing): FIFO, bình quân gia quyền, hay đích danh? Chi phí vận chuyển chuyển kho có cộng vào giá vốn? | Theo chính sách kế toán, cấu hình hệ thống | 🟡 | 03, 10, 04 |
| C8 | Có nghiệp vụ **kiểm kê (cycle count / stocktake)** trong phạm vi sản phẩm không? | ✅ **Đã chốt 2026-10-26:** có kiểm kê, mô tả ở module riêng [20 · Inventory Control](../warehouse/20-inventory-control/README.md): phiếu kiểm theo phạm vi vị trí / SKU, mặc định kiểm mù, chênh lệch thành điều chỉnh chờ duyệt bốn mắt; short pick tự mở kiểm kê | 🟡 | 05, 07, 11, 20 |
| C9 | Quy tắc **over-receipt tolerance** và **price/quantity tolerance** ở Invoice: theo hệ thống, theo NCC, hay theo SKU? | ✅ **Đã chốt một phần 2026-10-28:** dung sai **nhận vượt** khai báo **theo từng NCC** (trống = 0), tính trên mọi phiếu nhận chưa huỷ của PO, vượt thì **luôn chặn** — không có duyệt ngoại lệ ([03](../warehouse/03-receipt/README.md) BR-02). **Còn mở:** dung sai giá / số lượng ở Invoice | 🟢 | 02, 03, 04 |
| C10 | Xử lý **hàng Quarantine/Rejected**: quy trình trả NCC (RTV), tiêu huỷ, hồ sơ chất lượng NCC — chi tiết đến đâu? | ✅ **Đã chốt một phần 2026-09-23:** hàng `Quarantine` và `Rejected` được chuyển sang khu cách ly (Area `QUARANTINE`); hàng `Rejected` giữ tồn trạng thái `Blocked` tới khi xuất trả NCC. **Còn mở:** quy trình RTV chi tiết, tiêu huỷ, hồ sơ chất lượng NCC; chưa có module RTV riêng | 🟡 | 03 |
| C11 | Có áp dụng **tồn an toàn (safety stock)** — giữ lại một lượng không bán — không? Cấu hình theo SKU hay theo kho? | ✅ **Đã chốt 2026-10-26:** có tồn an toàn, cấu hình **theo SKU** trên Inventory Item, **chỉ dùng để cảnh báo** (`SAFETY`), **không trừ vào ATP**; ATP = tồn khả dụng − phần giữ hàng. Xem [20 BR-13](../warehouse/20-inventory-control/README.md) | 🟡 | 01, 13, 14, 20 |
| C12 | SKU có phải **gán kho kinh doanh** trước khi dùng (PO, Receipt, bán) không, hay SKU `Active` dùng được ở mọi kho? | Không cần gán; SKU `Active` dùng được ở mọi kho | 🟡 | 01, 02, 03, 17 |
| C13 | **Loại ô (`Bin.type`: STANDARD / PALLET / SMALL_PARTS)** là ràng buộc cứng (hàng pallet chỉ vào ô `PALLET`) hay chỉ để mô tả & ưu tiên? | Chỉ mô tả & ưu tiên (soft scoring), không chặn | 🟢 | 06, 05, 11 |
| C14 | Chứng từ liên quan **hai kho** (Transfer Order) dùng **prefix** của kho nào? | Prefix của kho nguồn | 🟢 | 00, 10 |
| C15 | Hàng nghi hỏng khi kiểm đếm (thùng móp, nghi hụt) của **SKU không bật cờ Yêu cầu QC** có bắt buộc đi qua khu kiểm tra chất lượng không? | Có — dòng nhận có điều kiện luôn đi luồng 3 bước, bất kể cờ của SKU | 🟢 | 03, 05 |
| C16 | Có **kiểm kê theo lịch** (định kỳ) và **tự mở phiếu kiểm kê** khi lệch số lúc di chuyển trong kho / chuyển kho không? | Chưa: nguồn `SCHEDULED`, `MOVE_DISCREPANCY`, `TRANSFER_DISCREPANCY` mới được dành chỗ; hiện lập phiếu tay (nguồn `MANUAL`). Chỉ short pick tự mở phiếu | 🟢 | 10, 11, 20 |
| C17 | Có làm **lấy hàng theo đợt / theo zone** (wave, batch, zone picking), **tối ưu lộ trình** trên bản đồ kho, và **gợi ý thùng tự động** (cartonization) tại bàn đóng gói không? | Phạm vi giai đoạn sau. Hiện mỗi đơn một task, dòng lấy sắp theo mã vị trí; Packer tự chọn loại kiện và nhập kích thước ([07](../warehouse/07-picking/README.md), [08](../warehouse/08-packing/README.md)) | 🟢 | 07, 08, 06 |
| C18 | Cất hàng có cần **phân công** nhiệm vụ cho từng người, **khoá mềm** sức chứa bin khi nhận nhiệm vụ, **bắt buộc mã lý do** khi cất khác bin gợi ý, và **cất nhiều chặng** qua khu trung gian không? | Phạm vi giai đoạn sau. Hiện mọi nhân viên kho của kho thấy và cất mọi nhiệm vụ; cất khác gợi ý không cần lý do; ràng buộc cứng kiểm lại tại bin được quét ([05](../warehouse/05-putaway/README.md)) | 🟢 | 05, 06 |
| C19 | Có làm **Bảng đề xuất nhập hàng** do MRP tự sinh (chuyển thành PO nháp) và **nhận hàng không PO** (blind receipt) không? | Phạm vi giai đoạn sau. Hiện Procurement tạo PO trực tiếp từ cảnh báo tồn `REORDER` / `SAFETY`; mọi phiếu nhận phải tham chiếu một PO ([02](../warehouse/02-purchase-order/README.md), [03](../warehouse/03-receipt/README.md)) | 🟡 | 02, 03, 20 |
| C20 | **Hạn mức duyệt PO** cho từng role (theo tiền tệ, thời gian hiệu lực) do ai khai báo, mức mặc định bao nhiêu? | Chưa có màn hình / dữ liệu khai báo; vì vậy mọi PO đều vào `Pending Approval` và người duyệt không bị giới hạn số tiền ([02](../warehouse/02-purchase-order/README.md) §4.3) | 🟡 | 02, 00 |

---

## D. Bán hàng, Đơn hàng & Thanh toán

| # | Câu hỏi | Giả định tạm thời | Ảnh hưởng | Module |
|---|---------|-------------------|-----------|--------|
| D1 | **Danh sách phương thức thanh toán** cần hỗ trợ (cổng nào, ví nào, thẻ nội địa/quốc tế, chuyển khoản, COD, đặt cọc)? | ✅ **Đã chốt 2026-10-27:** khách sỉ chỉ **chuyển khoản**, theo ba điều khoản trả trước / đặt cọc / công nợ; không có cổng thẻ / ví, không COD bán lẻ. Tiền nhà vận chuyển thu hộ chỉ dùng cho phần còn lại `ON_DELIVERY` của đơn đặt cọc ([15](../ecommerce/15-payment/README.md)) | 🔴 | 15, 14 |
| D2 | Dùng **authorize–capture 2 bước** hay **capture ngay**? Nếu 2 bước: capture tại thời điểm nào (khi ship / khi confirmed)? | ✅ **Đã chốt 2026-10-27:** không áp dụng — không có cổng thanh toán; kế toán ghi nhận tiền theo sao kê | 🟡 | 15 |
| D3 | Điều kiện cho phép **COD**: hạn mức giá trị đơn, khu vực, lịch sử khách? | ✅ **Đã chốt 2026-10-06:** không có COD bán lẻ (chỉ bán sỉ). Thay bằng **công nợ có hạn mức** theo khách (D13) và thu phần còn lại khi giao với đơn đặt cọc | 🟡 | 15, 18 |
| D4 | **Đặt cọc**: áp dụng cho loại đơn nào, % cọc bao nhiêu, thu phần còn lại khi nào (trước giao / COD)? | Áp cho đơn giá trị lớn / in ấn; % cấu hình | 🟡 | 15, 14 |
| D5 | Chính sách **giữ hàng (reservation) & thời hạn thanh toán** cho đơn `Pending Payment`? | Reservation có thời hạn; hết hạn → huỷ & giải phóng | 🟡 | 14, 15 |
| D6 | Một khách được có **bao nhiêu giỏ Active** cùng lúc? Giỏ guest gộp vào tài khoản thế nào? | ✅ **Đã chốt 2026-10-06:** không có khách vãng lai nên không có giỏ guest; khách sỉ đặt nhanh / đặt lại trên cổng khách hàng | 🟢 | 14, 18 |
| D7 | **Khuyến mãi / voucher**: các loại (theo %/số tiền/free-ship/quà tặng), điều kiện xếp chồng (stacking), giới hạn lượt? | ⏸ **Tạm gác 2026-10-06:** bán sỉ dùng giá theo báo giá và giá riêng từng khách; voucher/khuyến mãi chưa làm | 🟡 | 14, 13 |
| D8 | **Thuế**: mô hình tính (VAT %, giá đã gồm thuế hay chưa), xuất hoá đơn điện tử (e-invoice) cho khách? | VAT theo cấu hình; e-invoice khách chưa xác định phạm vi | 🟡 | 14, 15 |
| D9 | Chính sách **giao một phần / tách kiện (split shipment)** cho đơn hỗn hợp (hàng thường + in ấn)? | ✅ **Đã chốt 2026-10-27:** **Sale quyết định cho phép giao một phần theo từng đơn** (khoá từ khi đơn `Shipped`). Có cho phép thì hàng sẵn được giao trước trong lúc dòng in còn sản xuất; không cho phép thì không được để lại hàng khi bàn giao ([17](../ecommerce/17-order-management/README.md) §4.2) | 🟡 | 08, 09, 17 |
| D10 | Chính sách **đổi/trả tổng quát** (ngoài sản phẩm in ấn): thời hạn, ai chịu phí ship trả, điều kiện tình trạng hàng? | Có luồng RMA; tham số chính sách chưa chốt | 🟡 | 17, 09 |
| D11 | **Consolidated shipment** (gộp nhiều đơn của một khách vào một kiện) có hỗ trợ không? | Mặc định: không; mỗi kiện một đơn | 🟢 | 08 |
| D12 | Đơn **`Completed` tự động** sau `Delivered` bao lâu (hết hạn khiếu nại), hay chờ khách xác nhận? | ✅ **Đã chốt 2026-10-27:** tự `Completed` sau **7 ngày** kể từ khi giao, khi đơn không còn tiền phải thu và không có yêu cầu trả hàng; tự `Closed` sau **30 ngày**; chạy mỗi đêm ([17](../ecommerce/17-order-management/README.md) §4.3) | 🟢 | 17 |
| D13 | **Công nợ khách sỉ**: hạn mức, kỳ hạn (ví dụ 30 ngày), ai duyệt đơn vượt hạn mức, xử lý khách quá hạn? | ✅ **Đã chốt một phần 2026-10-27:** mỗi khách có điều khoản được phép, hạn mức (≥ 0) và kỳ hạn (0–365 ngày) do **kế toán** duyệt. Đơn công nợ vượt hạn mức **hoặc khách có khoản quá hạn** → `On Hold`; kế toán duyệt, hoặc từ chối (huỷ / đổi trả trước / đặt cọc). Khách quá hạn không release được đơn công nợ ([15](../ecommerce/15-payment/README.md) §4.4). **Còn mở:** ngưỡng hạn mức / kỳ hạn mặc định | 🔴 | 15, 17, 18 |
| D14 | **Giá riêng theo khách**: bảng giá theo từng khách hay theo nhóm khách? Bậc giá theo số lượng? | Có bảng giá riêng theo khách + bậc số lượng; báo giá ghi đè cho đơn đặc biệt | 🟡 | 13, 14, 18 |
| D15 | Kế toán **ghi nhận lệnh hoàn tiền đã chuyển khoản xong** (hoặc hoàn lỗi) trong hệ thống ở đâu? | Hệ thống hiện chỉ tạo lệnh hoàn `Pending`; chuyển khoản hoàn làm ngoài hệ thống; màn hình đánh dấu `Completed` / `Failed` chưa có ([15](../ecommerce/15-payment/README.md) §4.7) | 🟡 | 15, 17 |
| D16 | Hồ sơ khách có cần: **nhiều người liên hệ / người đặt hàng** có quyền riêng trên cổng, **2FA**, **consent marketing**, **hồ sơ rủi ro**, **yêu cầu quyền riêng tư (tải / ẩn danh dữ liệu)**, **hợp nhất hồ sơ trùng**? | Phạm vi giai đoạn sau; hiện mỗi khách **một tài khoản đăng nhập**, trạng thái chỉ gồm `Pending Activation` / `Active` / `Blocked` / `Inactive` ([18](../ecommerce/18-customer-information/README.md)) | 🟡 | 18, 16 |

---

## E. Vận chuyển (Shipping / Carrier)

| # | Câu hỏi | Giả định tạm thời | Ảnh hưởng | Module |
|---|---------|-------------------|-----------|--------|
| E1 | **Danh sách hãng vận chuyển** tích hợp và phương thức tích hợp (API trực tiếp từng hãng / một aggregator)? | ✅ **Đã chốt 2026-10-06:** **không tích hợp API** hãng vận chuyển. Hãng là bên thứ ba ngoài hệ thống; hệ thống chỉ ghi nhận bàn giao và kết quả giao do nhân viên cập nhật — xem [09](../warehouse/09-shipping/README.md). Danh sách hãng là dữ liệu cấu hình (tên, liên hệ) để chọn khi đặt xe | 🟢 | 09, 08 |
| E2 | **Quy tắc chọn hãng/dịch vụ**: khách chọn khi checkout, hay hệ thống tự chọn (rẻ nhất / nhanh nhất / theo vùng)? | Kho chọn hãng khi lập kế hoạch xếp hàng và đặt xe (09); khách không chọn hãng. **Còn mở:** khách có chọn mức dịch vụ (tiêu chuẩn / nhanh) không | 🟡 | 14, 08, 09 |
| E3 | **Phí vận chuyển thu khách**: bảng giá cố định, theo cân nặng/vùng, hay lấy real-time từ hãng? | Không lấy real-time từ hãng (không tích hợp API, xem E1); nguồn còn lại là bảng phí cấu hình — chưa chốt cách tính (cố định / theo cân nặng / theo vùng) | 🟡 | 14, 09 |
| E4 | Cập nhật tracking từ hãng qua **webhook** hay **polling**? Tần suất? | ✅ **Đã chốt 2026-10-06:** **không theo dõi tracking**. Sau bàn giao, Order Coordinator cập nhật `Delivered` / `Delivery Failed` khi hãng báo lại — xem [09](../warehouse/09-shipping/README.md) | 🟢 | 09 |
| E5 | Xử lý **bồi thường** khi hãng làm mất/hỏng hàng: quy trình claim, ai chịu chi phí với khách trong lúc chờ? | Tra soát và claim với hãng xử lý **ngoài hệ thống**; trong hệ thống ghi `Delivery Failed` lý do "thất lạc" và giao lại hàng mới theo chính sách | 🟡 | 09, 17 |
| E6 | **Đối soát COD & cước** với hãng: chu kỳ, công cụ, xử lý chênh lệch? | Có bước reconciliation; chu kỳ chưa chốt | 🟡 | 09, 15 |
| E7 | **Giao nguyên chuyến cho đơn khách**: ai chỉ định đơn nào giao nguyên chuyến (khách chọn ở checkout hay nội bộ chỉ định)? Phí vận chuyển thu khách và cách thu phần còn lại (đơn đặt cọc) cho đơn giao nguyên chuyến? | Order Coordinator chỉ định khi release đơn sỉ khối lượng lớn; khách không chọn trên cổng khách hàng; phí & cách thu phần còn lại (đơn đặt cọc) chưa xác định | 🔴 | 09, 14, 15, 17 |
| E8 | **Đặt xe & bàn giao** cho giao nguyên chuyến: đặt xe trong hay ngoài hệ thống? Danh mục loại xe & số chiếc sẵn có khai báo theo từng kho hay toàn hệ thống? Theo dõi hành trình bằng gì? | Đặt xe ngoài hệ thống, hệ thống chỉ ghi thông tin xe (biển số, tài xế) cho từng xe của kế hoạch; Quản lý kho khai báo loại xe; trạng thái giao cập nhật thủ công | 🟡 | 09, 10 |
| E9 | Load Planning có cần **chọn xe theo chi phí thuê** (rẻ nhất) không? | Chưa hỗ trợ: chọn theo số xe & độ lấp đầy; giá thuê chỉ lưu tham khảo. Muốn chọn xe rẻ nhất phải bổ sung bảng giá cho từng loại xe | 🟡 | 09 |
| E10 | Load Planning có cần tính **phân bố trọng tâm, hàng dễ vỡ, giới hạn chồng tầng** không? | Chưa hỗ trợ | 🟡 | 09, 01, 08 |
| E11 | Chuyến nhiều điểm giao: có cần xếp theo **thứ tự điểm giao** (hàng giao trước đặt phía cửa xe) không? | Chưa hỗ trợ | 🟡 | 09 |
| E12 | Có cần cho phép **đặt kiện nghiêng** (không vuông góc cạnh thùng) không? | Không: chỉ đặt vuông góc với các cạnh thùng | 🟢 | 09 |
| E13 | **Cờ cho phép xoay**: mặc định của SKU là gì? Kiện chứa nhiều SKU có cờ khác nhau lấy giá trị nào? | Mặc định cho phép xoay; kiện lấy giá trị chặt nhất (một SKU không cho xoay → kiện không cho xoay); Packer được ghi đè | 🟢 | 01, 08, 09 |

---

## F. Sales Chat & CSKH

| # | Câu hỏi | Giả định tạm thời | Ảnh hưởng | Module |
|---|---------|-------------------|-----------|--------|
| F1 | Chat chỉ **trong cổng khách hàng (1–1 khách sỉ ↔ Sale)** hay tích hợp **kênh ngoài** (Messenger, Zalo, email, hotline)? | Chỉ trong storefront | 🟡 | 16 |
| F2 | Có **chatbot / trợ lý AI** trả lời câu hỏi cơ bản trước khi chuyển người không? | Chưa có; routing tới agent | 🟢 | 16 |
| F3 | **Giờ làm việc & SLA phản hồi**: trong giờ / ngoài giờ xử lý thế nào (offline message)? Mục tiêu FRT? | Có offline message ngoài giờ; FRT mục tiêu chưa chốt | 🟢 | 16 |
| F4 | **Hạn mức ưu đãi** mà Sale được tự áp trong chat (không cần duyệt)? | Có hạn mức; giá trị chưa chốt | 🟡 | 16, 14 |
| F5 | Quy tắc **định tuyến hội thoại** (skill-based, theo khách cũ, theo tải)? | Kết hợp; chi tiết chưa chốt | 🟢 | 16 |
| F6 | Lưu trữ & **quyền xem lịch sử chat**: ai được xem, thời hạn lưu, có tính vào yêu cầu xoá dữ liệu của khách? | Gắn hồ sơ khách; theo chính sách quyền riêng tư | 🟡 | 16, 18 |

---

## G. Dữ liệu & Tích hợp

| # | Câu hỏi | Giả định tạm thời | Ảnh hưởng | Module |
|---|---------|-------------------|-----------|--------|
| G1 | Ranh giới **Product Creation (back-office) ↔ Catalog (cổng khách hàng)**: cùng một thực thể sản phẩm hay hai lớp tách biệt đồng bộ? | Cùng nguồn; Catalog là lớp hiển thị tiêu thụ dữ liệu từ [01](../warehouse/01-product-creation/README.md) | 🟡 | 01, 13 |
| G2 | Có tích hợp **sàn TMĐT bên thứ ba** (Shopee, Lazada, TikTok Shop) đồng bộ đơn/tồn không, hay chỉ storefront riêng? | Chỉ storefront riêng của StockFlowCommerce | 🔴 | 13, 14, 17 |
| G3 | Hệ thống **nguồn sự thật của tồn kho**: chính StockFlowCommerce hay đồng bộ từ ERP ngoài? | StockFlowCommerce là nguồn sự thật | 🔴 | Toàn bộ kho vận |
| G4 | Yêu cầu **realtime** đến mức nào cho tồn kho hiển thị trên catalog (độ trễ chấp nhận được)? | "Gần thời gian thực"; ngưỡng trễ chưa chốt | 🟡 | 13, 05, 07 |
| G5 | Có cần **API mở cho đối tác** (đối tác đặt hàng B2B, tra cứu đơn) không? | ✅ **Đã chốt 2026-10-06:** hệ thống **chỉ bán sỉ (B2B)**: khách sỉ dùng cổng khách hàng (không phải API mở). API cho đối tác vẫn chưa làm | 🟢 | 14, 17 |
| G6 | **Xuất hoá đơn tài chính cho khách** (VAT e-invoice) nằm trong phạm vi hệ thống hay tích hợp phần mềm hoá đơn? | Chưa xác định phạm vi | 🟡 | 15, 14 |
| G7 | Có bán **combo / bundle** (một SKU gồm nhiều hàng tồn kho khác nhau) không? | ✅ **Đã chốt 2026-09-17:** không bán combo / bundle; mỗi SKU ứng với đúng một Inventory Item (1:1) | 🟢 | 01, 13, 14, 07 |

---

## H. Ưu tiên xác nhận (đề xuất)

Nhóm câu hỏi nên được trả lời **trước tiên** vì ảnh hưởng kiến trúc nhiều module:

1. **A2, G2, G3** — phạm vi hệ thống (phân quyền, có sàn bên thứ ba không, nguồn sự thật tồn kho). *(A1 — phạm vi đa kho/đa pháp nhân — đã chốt.)*
2. **D13** — công nợ khách sỉ: hạn mức, kỳ hạn, người duyệt (ảnh hưởng checkout, release đơn và sản xuất). *(B1 — module Sản xuất — đã chốt; G5 — chỉ bán sỉ — đã chốt.)*
3. **C1, C3** — tiêu chí slotting & cách tính ATP (ảnh hưởng vận hành kho + hiển thị bán hàng).
4. **D1** — danh sách phương thức thanh toán (ảnh hưởng thiết kế tích hợp). *(E1 — tích hợp hãng vận chuyển — đã chốt: không tích hợp.)*
5. **A5** — ranh giới với kế toán (ảnh hưởng phạm vi module Invoice & Payment).
