# Bảng thuật ngữ (Glossary)

Giải thích các thuật ngữ nghiệp vụ dùng trong tài liệu StockFlowCommerce. Sắp xếp theo nhóm để dễ tra; trong mỗi nhóm theo thứ tự bảng chữ cái tiếng Anh.

---

## 1. Thuật ngữ chung về hàng hoá & tồn kho

| Thuật ngữ | Tiếng Việt / Giải thích |
|-----------|--------------------------|
| **SKU** (Stock Keeping Unit) | Đơn vị lưu kho — mã định danh **nhỏ nhất** để quản lý tồn và giao dịch. Mỗi biến thể sản phẩm (ví dụ "Áo thun – Đỏ – Size M") là một SKU riêng. |
| **Product / Sản phẩm** | Thực thể "cha" gom nhiều SKU biến thể (ví dụ "Áo thun" có nhiều màu/size). |
| **Variant / Biến thể** | Một tổ hợp thuộc tính cụ thể của sản phẩm (màu × size × chất liệu…), tương ứng một SKU. |
| **Inventory Item / Hàng tồn kho** | Hồ sơ **hàng vật lý** của một SKU: đơn vị tính, thông số logistics, lô/hạn dùng/serial, ngưỡng tồn, thông tin mua hàng. Quan hệ **1:1** với SKU, hệ thống tạo tự động khi sinh SKU; không chứa nội dung bán hàng. Với sản phẩm tùy chỉnh là phôi (hàng chưa in) — giả định, xem [open-questions B9](../open-questions/README.md). |
| **UoM** (Unit of Measure) | Đơn vị tính (cái, hộp, kg, mét…). |
| **Batch / Lot / Lô** | Nhóm hàng cùng đợt sản xuất/nhập, thường quản lý chung hạn sử dụng. |
| **Serial** | Số định danh **duy nhất cho từng đơn vị** hàng (thường với hàng giá trị cao / điện tử). |
| **Expiry date / Hạn sử dụng** | Ngày hết hạn dùng của lô hàng. |
| **Inventory / Tồn kho** | Số lượng hàng hoá đang có, luôn gắn với **vị trí** hoặc trạng thái quá cảnh. |
| **On-hand / Tồn thực tế** | Tổng số lượng vật lý đang nằm trong kho ở mọi trạng thái (`Inbound`, `Available`, `Allocated`, `Blocked`, `Quarantine`…); không gồm hàng `In-Transit` (thuộc lệnh chuyển kho). |
| **Inventory Level / Tồn theo kho** | Số lượng tồn của một Inventory Item tại **một kho**, **tổng hợp** từ tồn các vị trí trong kho đó (theo từng trạng thái). Là số tính ra, không nhập/sửa trực tiếp; tồn toàn hệ thống = tổng các kho. |
| **Available stock / Tồn khả dụng** | Tồn ở trạng thái `Available`: đã putaway vào vị trí được phép xuất, **chưa phân bổ** (allocate) cho đơn/lệnh nào, không bị khoá/cách ly. **Chưa trừ** phần giữ hàng (reservation). |
| **ATP** (Available To Promise) | Số lượng có thể "hứa bán" cho khách: **ATP = Tồn khả dụng − Phần đang giữ hàng (reservation)**. Không trừ thêm phần đã phân bổ vì hàng `Allocated` không nằm trong tồn khả dụng. Phạm vi: toàn hệ thống hoặc theo kho (câu hỏi C3). |
| **On-order / Incoming stock** | Hàng đã đặt qua PO nhưng **chưa về kho**. |
| **In-Transit / Đang quá cảnh** | Hàng đã rời kho nguồn nhưng chưa tới kho đích (trong luồng chuyển kho). |
| **Reorder point / Ngưỡng đặt lại** | Mức tồn tối thiểu; chạm ngưỡng thì cần lập PO bổ sung. |
| **Replenishment Proposal / Bảng Đề xuất Nhập hàng** | Danh sách SKU chạm ngưỡng tái đặt hàng do hệ thống MRP sinh, chờ Planner đánh giá & chuyển thành PO. |
| **Reservation / Giữ hàng** | Tạm giữ một số lượng tồn cho một đơn cụ thể trong thời hạn chờ thanh toán, **chưa gắn vị trí/lô**. Làm giảm ATP; hết hạn thì giải phóng; khi đơn được phân bổ thì chuyển thành allocation. |
| **Allocation / Phân bổ** | Gán chính thức tồn ở **vị trí & lô cụ thể** cho một đơn để đi lấy hàng. Số lượng chuyển trạng thái `Available` → `Allocated`. |
| **Inventory adjustment / Điều chỉnh tồn** | Sửa số lượng tồn có kiểm soát (kèm lý do, phê duyệt) khi phát hiện chênh lệch. |
| **Cycle count / Kiểm kê định kỳ** | Đếm lại tồn ở một số vị trí theo lịch, không dừng toàn kho. |
| **FIFO** (First-In-First-Out) | Nhập trước xuất trước. |
| **FEFO** (First-Expired-First-Out) | Hạn dùng gần thì xuất trước. |
| **ABC / XYZ analysis** | Phân loại SKU theo mức độ quan trọng (ABC: theo giá trị/tần suất xuất; XYZ: theo độ ổn định nhu cầu). |

---

## 2. Thuật ngữ mua hàng & nhập kho (Procurement / Inbound)

| Thuật ngữ | Tiếng Việt / Giải thích |
|-----------|--------------------------|
| **Procurement / Mua hàng** | Quy trình mua hàng từ nhà cung cấp về kho. |
| **Supplier / Vendor / Nhà cung cấp (NCC)** | Đối tác bán hàng hoá cho doanh nghiệp. |
| **PO** (Purchase Order) — **Đơn đặt hàng** | Chứng từ cam kết mua: SKU nào, số lượng, giá, giao về kho nào, khi nào. |
| **PO line / Dòng hàng PO** | Một dòng trong PO: một SKU + số lượng + đơn giá. |
| **Open quantity / Số lượng còn chờ nhận** | SL đặt − SL đã nhận của một dòng PO. |
| **PO revision / amendment** | Bản chỉnh sửa PO sau khi đã `Confirmed` (tăng version, có log). |
| **MOQ** (Minimum Order Quantity) | Số lượng đặt hàng tối thiểu NCC yêu cầu. |
| **Lead time / Thời gian giao** | Khoảng thời gian từ khi đặt đến khi hàng về. |
| **Receipt / Goods Receipt — Nhận hàng** | Ghi nhận hàng thực tế về kho theo PO: kiểm đếm, ghi lô, đưa vào khu nhận hàng; với SKU yêu cầu QC thì chuyển sang khu QC và ghi kết quả QC. |
| **Delivery note / Packing slip / Phiếu giao hàng** | Chứng từ NCC gửi kèm lô hàng, liệt kê nội dung giao. |
| **QC** (Quality Control) — **Kiểm tra chất lượng** | Đánh giá hàng nhận tại khu kiểm tra chất lượng: đạt (Accepted) / cách ly (Quarantine) / từ chối (Rejected). Chỉ áp dụng cho SKU bật cờ **Yêu cầu QC khi nhận**. |
| **Nhập kho 2 bước / 3 bước (2-step / 3-step reception)** | 3 bước: NCC → khu nhận hàng (WH/Input) → khu kiểm tra chất lượng (WH/Quality Control) → bin (WH/Stock), cho SKU yêu cầu QC. 2 bước: NCC → khu nhận hàng → bin, cho SKU không yêu cầu QC. |
| **QC area / Khu kiểm tra chất lượng** | Khu chứa hàng đã nhận đang chờ QC kiểm — Area loại `QUALITY_CONTROL`. Khác khu cách ly (`QUARANTINE`) là nơi chứa hàng **sau khi** QC kết luận cách ly hoặc không đạt. |
| **Quarantine / Cách ly** | Trạng thái giữ hàng chờ kết luận chất lượng lại, chưa cho vào tồn khả dụng; hàng nằm ở Area loại `QUARANTINE`. |
| **RTV** (Return To Vendor) — **Trả hàng NCC** | Trả lại hàng không đạt cho nhà cung cấp. |
| **Over-receipt tolerance / Dung sai nhận vượt** | Tỷ lệ % cho phép nhận nhiều hơn số đặt. |
| **Invoice / Vendor Invoice / AP Invoice — Hoá đơn (mua hàng)** | Yêu cầu thanh toán từ NCC cho hàng đã giao. |
| **Three-way matching / Đối chiếu ba chiều** | So khớp PO (đặt) ↔ Receipt (nhận) ↔ Invoice (đòi tiền) trước khi thanh toán. |
| **Price / Quantity variance / Chênh lệch giá / số lượng** | Sai khác giữa hoá đơn và PO/Receipt. |
| **Credit note / Debit note** | Chứng từ NCC/doanh nghiệp phát hành để điều chỉnh giảm/tăng số tiền hoá đơn. |
| **AP** (Accounts Payable) — **Công nợ phải trả** | Khoản doanh nghiệp nợ nhà cung cấp. |
| **Putaway / Cất hàng vào kho** | Di chuyển hàng từ khu chờ cất (khu QC với hàng đạt QC, hoặc khu nhận hàng với SKU không yêu cầu QC) lên vị trí lưu trữ chính thức. |
| **Inbound area / Staging area / Khu nhận hàng** | Khu tạm chứa hàng vừa nhận (WH/Input), chờ chuyển sang khu QC hoặc chờ putaway — Area loại `RECEIVING`. |

---

## 3. Thuật ngữ vận hành kho (Warehouse Operations)

| Thuật ngữ | Tiếng Việt / Giải thích |
|-----------|--------------------------|
| **WMS** (Warehouse Management System) | Hệ thống quản lý kho. |
| **Warehouse Map / Bản đồ kho** | Mô hình không gian kho (2D mặt bằng / 3D khối kệ) kèm trạng thái lấp đầy. |
| **Phân cấp không gian kho** | **Kho (Warehouse) → Kệ (Shelf) → Tầng kệ (Shelf Level) → Ô (Bin)**; kệ và khu vực có thể thuộc một **Zone** (tùy chọn). Hàng trên sàn nằm ở **Area**. Không khai báo dãy/lối đi riêng. |
| **Zone** | Nhóm logic **tùy chọn** gom kệ/khu vực (có màu hiển thị), dùng cho hiển thị và zone picking; không mang quy tắc lưu trữ. |
| **Shelf / Kệ** | Khối kệ đặt trên bản đồ kho (toạ độ, kích thước, góc xoay), có mặt lấy hàng và lớp lưu trữ mặc định. |
| **Shelf Level / Tầng kệ** | Một tầng của kệ; có cao độ, chiều cao lọt và tải trọng tối đa. |
| **Bin / Ô** | Điểm chứa hàng nhỏ nhất trên tầng kệ; có sức chứa theo đơn vị, cờ cho phép lấy hàng / cất hàng, loại ô, lớp lưu trữ ghi đè. |
| **Picking bin / Reserve bin — Bin picking / Bin dự trữ** | Bin cho phép lấy hàng (`is_pickable` = true) / bin chỉ để dự trữ, phải bổ sung sang bin picking trước khi lấy. |
| **Bin autogenerate / Tự sinh bin** | Tạo hàng loạt bin cho các tầng kệ chưa có bin bằng cách chia đều mặt kệ theo số hàng × số cột; mã bin đặt theo kiểu `SEQUENTIAL` (`01`, `02`…) hoặc `ROW_LETTER_COLUMN` (`A01`, `B01`…). |
| **Area / Khu vực** | Vùng trên sàn không phải kệ, có mã khu vực riêng: `RECEIVING` (khu nhận), `QUALITY_CONTROL` (kiểm tra chất lượng), `QUARANTINE` (cách ly), `PRODUCTION` (xưởng in), `PACKING` (đóng gói), `DISPATCH` (chờ giao), `OVERFLOW` (khu tràn), `NON_STORAGE` (không chứa hàng). |
| **Boundary / Ranh giới** | Đoạn tường (`WALL`) hoặc cửa (`DOOR`) trên bản đồ; cửa có thể đi qua hoặc không, mở hoặc đóng. |
| **Obstacle / Vật cản** | Kệ hoặc khu vực đánh dấu không đi xuyên được khi tính đường đi. |
| **Pick face / Mặt lấy hàng** | Phía của kệ (bắc/đông/nam/tây) mà NV kho tiếp cận để lấy/cất hàng. |
| **Storage class / Lớp lưu trữ** | Phân loại điều kiện lưu trữ: `NORMAL`, `COLD`, `HAZMAT`, `FRAGILE`, `OVERSIZE`. SKU chỉ được cất vào bin có lớp lưu trữ hiệu lực trùng lớp của SKU. |
| **Location / Vị trí** | Nơi tồn kho được ghi nhận: một **Bin** hoặc một **Area chứa hàng**. |
| **Location barcode / Mã vị trí** | Mã quét để xác nhận đúng vị trí khi cất/lấy hàng, do hệ thống ghép khi tạo vị trí và không đổi: bin = prefix kho + mã kệ + tầng + mã bin (ví dụ `HN-A01-2-03`); khu vực = prefix kho + mã khu vực (ví dụ `HN-RCV01`). Duy nhất toàn hệ thống. |
| **Warehouse prefix / Tiền tố kho** | Mã ngắn duy nhất của kho, dùng làm tiền tố cho mã vị trí và mã chứng từ phát sinh tại kho. Không sửa được sau khi tạo. |
| **Capacity / Sức chứa** | Giới hạn của vị trí: số đơn vị tối đa của bin; tải trọng tối đa & chiều cao lọt của tầng kệ. |
| **Slotting / Slotting Suggestion / Gợi ý vị trí** | Đề xuất vị trí lưu trữ tối ưu cho một SKU dựa trên quy tắc & dữ liệu. |
| **Putaway recommendation** | Tên gọi khác của gợi ý vị trí khi cất hàng. |
| **Re-slotting / Tối ưu vị trí** | Sắp xếp lại vị trí SKU định kỳ khi nhu cầu/tần suất xuất thay đổi. |
| **Golden zone** | Khu vực dễ với tay nhất (ngang hông–vai), ưu tiên cho hàng bán chạy. |
| **Honeycombing** | Tình trạng vị trí bị chiếm dở dang, còn trống nhưng không dùng được → lãng phí chỗ. |
| **Velocity / Tần suất xuất** | Mức độ một SKU được lấy ra để giao (nhanh/chậm). |
| **Hard constraint / Ràng buộc cứng** | Điều kiện không được vi phạm khi cất hàng: trạng thái vị trí, cho phép cất, lớp lưu trữ, sức chứa, tải trọng & chiều cao lọt của tầng. |
| **Hazmat** (Hazardous materials) | Hàng nguy hiểm, có khu vực & quy tắc lưu trữ riêng. |
| **Picking / Lấy hàng** | Lấy hàng từ các vị trí để gom thành đơn chuẩn bị giao. |
| **Pick list / Danh sách lấy hàng** | Danh sách SKU–vị trí–số lượng cần lấy cho một hoặc nhiều đơn. |
| **Pick task / Nhiệm vụ lấy hàng** | Đơn vị công việc picking giao cho một NV kho. |
| **Wave / Batch picking** | Gom nhiều đơn thành một đợt lấy hàng để tối ưu di chuyển. |
| **Zone picking** | Mỗi NV kho phụ trách lấy hàng trong một zone; đơn đi qua nhiều zone. |
| **Short pick / Lấy thiếu** | Không lấy đủ số lượng do vị trí không đủ hàng. |
| **Backorder / Đặt hàng chờ** | Phần đơn chưa có hàng, sẽ giao sau khi bổ sung tồn. |
| **Staging (outbound) / Khu chờ giao** | Khu gom hàng đã pick và thành phẩm in đạt QC chờ đóng gói (Area `PACKING`) / kiện chờ bàn giao vận chuyển (Area `DISPATCH`). |
| **Put-back / Trả hàng về vị trí** | Đưa hàng đã lấy nhưng không dùng (đơn huỷ) trở lại vị trí lưu trữ. |
| **Replenishment / Bổ sung vùng picking** | Chuyển hàng từ bin dự trữ sang bin picking khi sắp cạn. |
| **Packing / Đóng gói** | Kiểm đơn lần cuối, đóng thành kiện, gắn nhãn. |
| **Cartonization** | Gợi ý loại thùng/hộp & cách chia kiện phù hợp. |
| **Packing slip / Phiếu đóng gói** | Phiếu liệt kê nội dung kiện, đặt kèm trong hoặc ngoài kiện. |
| **Dimensional weight / Khối lượng quy đổi** | Khối lượng tính cước theo thể tích kiện (khi hàng cồng kềnh nhẹ cân). |
| **Shipping / Giao cho đơn vị vận chuyển** | Bàn giao kiện cho hãng vận chuyển ngoài để giao đến khách. |
| **Carrier / 3PL / Đơn vị vận chuyển** | Bên thứ ba thực hiện vận chuyển (giao nhận). |
| **AWB** (Air Waybill) / **Shipping label** / **Vận đơn / Nhãn vận chuyển** | Chứng từ + nhãn dán lên kiện, chứa mã theo dõi. |
| **Tracking number / Mã theo dõi** | Mã để tra cứu hành trình kiện hàng. |
| **Manifest / Bảng kê bàn giao** | Danh sách kiện giao cho hãng vận chuyển trong một lần, hai bên ký. |
| **POD** (Proof of Delivery) — **Bằng chứng giao hàng** | Chữ ký/ảnh/thời gian xác nhận đã giao. |
| **RTO** (Return To Origin) — **Hoàn hàng về kho** | Kiện không giao được, quay về kho gửi. |
| **Shipment** | Nhóm 1..n kiện của một đơn, có trạng thái & lịch sử tracking chung. |
| **Inter-warehouse Transfer / Chuyển kho** | Di chuyển tồn giữa hai kho khác địa điểm. |
| **Intra-warehouse Transfer / Chuyển nội bộ kho** | Di chuyển tồn giữa các vị trí trong cùng một kho. |
| **Transfer Order (TO) / Lệnh chuyển kho** | Chứng từ yêu cầu chuyển hàng giữa/trong kho. |
| **Transfer discrepancy / Chênh lệch chuyển kho** | Sai khác giữa số xuất ở kho nguồn và số nhận ở kho đích. |

---

## 3a. Thuật ngữ sản xuất / in theo đơn (Production)

| Thuật ngữ | Tiếng Việt / Giải thích |
|-----------|--------------------------|
| **Blank / Phôi** | Hàng chưa in của sản phẩm tùy chỉnh (áo trơn, cốc trắng…); là Inventory Item của SKU tùy chỉnh, được cấp cho xưởng theo lệnh sản xuất. |
| **Print method / Kỹ thuật in** | Công nghệ in của sản phẩm (DTG, in lụa, UV, sublimation…), khai báo ở [01](../warehouse/01-product-creation/README.md); quyết định trạm in được nhận lệnh. |
| **Print Station / Trạm in** | Máy/vị trí in tại xưởng nội bộ, có kỹ thuật in, công suất/ngày và trạng thái (`Active` / `Maintenance` / `Inactive`). |
| **Production area / Khu xưởng in** | Area loại `PRODUCTION`: nơi chứa phôi đã cấp, hàng đang in và thành phẩm chờ QC. |
| **Production Order (MO) / Lệnh sản xuất** | Chứng từ sản xuất sinh từ **một dòng đơn tùy chỉnh**: SKU phôi, số lượng, design snapshot, print-ready file, kỹ thuật in, trạm in, hạn cần xong. Mã theo prefix kho, ví dụ `HN-MO-000123`. Xem [19](../warehouse/19-production/README.md). |
| **Production queue / Hàng đợi sản xuất** | Danh sách lệnh đã có phôi, chờ gán trạm; ưu tiên theo hạn giao rồi thời điểm xác nhận đơn. |
| **Prepress / Kiểm file trước sản xuất** | E-commerce Admin kiểm print-ready file của lệnh sản xuất trước khi cấp phôi. Khác *Preflight* (kiểm tự động khi khách thiết kế). |
| **Rework / In lại** | Cấp phôi mới và in lại **phần còn thiếu** khi in hỏng hoặc QC thành phẩm không đạt. |
| **Scrap / Xuất huỷ** | Giảm tồn phôi in hỏng / thành phẩm lỗi bằng chứng từ điều chỉnh có lệnh sản xuất làm nguồn. |

---

## 4. Thuật ngữ sàn TMĐT (E-commerce / Storefront)

| Thuật ngữ | Tiếng Việt / Giải thích |
|-----------|--------------------------|
| **Storefront** | Mặt tiền bán hàng online mà khách nhìn thấy. |
| **Catalog / Danh mục sản phẩm** | Tập hợp sản phẩm trưng bày để khách duyệt & mua. |
| **PLP** (Product Listing Page) — **Trang danh sách sản phẩm** | Trang liệt kê nhiều sản phẩm (có lọc, sắp xếp). |
| **PDP** (Product Detail Page) — **Trang chi tiết sản phẩm** | Trang một sản phẩm: hình ảnh, mô tả, chọn biến thể, giá, tình trạng còn hàng. |
| **Publish / Xuất bản** | Đưa sản phẩm lên catalog cho khách thấy. |
| **Print-on-demand / In theo yêu cầu** | Sản phẩm chỉ được in/sản xuất sau khi có đơn của khách. |
| **Customization / Personalization / Tùy chỉnh cá nhân hoá** | Khách tự thêm nội dung (ảnh, chữ) lên sản phẩm. |
| **Print area / Vùng in** | Khu vực cho phép in nội dung trên sản phẩm. |
| **Safe area / Vùng an toàn** | Phần bên trong vùng in, tránh bị xén khi sản xuất. |
| **Bleed / Vùng tràn lề** | Phần mở rộng ngoài mép để tránh viền trắng khi cắt. |
| **DPI** (Dots Per Inch) | Độ phân giải in; DPI thấp → ảnh in bị mờ/vỡ. |
| **RGB / CMYK** | Hệ màu màn hình (RGB) vs hệ màu in (CMYK); cần chuyển đổi khi in. |
| **Live 3D Preview / Xem trước 3D trực tiếp** | Hiển thị thiết kế 2D "dán" lên mô hình 3D, cập nhật realtime khi khách chỉnh. |
| **Design draft / Thiết kế nháp** | Bản thiết kế đang chỉnh, lưu tạm. |
| **Design snapshot / artifact / Bản thiết kế chốt** | Bản mô tả đầy đủ & **bất biến** của thiết kế khách đã xác nhận, đi kèm đơn để sản xuất. |
| **Preflight / Kiểm tra tiền in ấn** | Kiểm tra file thiết kế đạt chuẩn in (DPI, hệ màu, font, lề). |
| **Print-ready file / File sẵn sàng in** | File xuất bản in đúng chuẩn kỹ thuật cho xưởng. |
| **Cart / Giỏ hàng** | Nơi tạm gom sản phẩm trước khi đặt hàng. |
| **Cart line / Dòng giỏ hàng** | Một mục trong giỏ: SKU + số lượng (+ design snapshot nếu tùy chỉnh). |
| **Checkout / Thanh toán (bước đặt hàng)** | Quy trình biến giỏ hàng thành đơn hàng: địa chỉ, vận chuyển, thanh toán. |
| **Abandoned cart / Giỏ hàng bị bỏ** | Giỏ quá hạn không thao tác, chưa chuyển thành đơn. |
| **Order / Đơn hàng** | Chứng từ bán hàng chính thức sau khi khách đặt. |
| **Order number / Mã đơn hàng** | Mã định danh duy nhất của đơn. |
| **Order line / Dòng đơn hàng** | Một mục hàng trong đơn với giá đã chốt. |
| **Price lock / Chốt giá** | Cố định giá dòng hàng vào đơn tại thời điểm đặt; không đổi theo giá catalog sau đó. |
| **Fulfillment / Hoàn tất đơn** | Toàn bộ quá trình từ đơn confirmed đến khi giao hàng cho khách (pick–pack–ship). |
| **Fulfillment sourcing / Chọn kho xuất** | Quyết định đơn được giao từ kho nào. |
| **Split fulfillment / shipment / Tách giao** | Một đơn giao thành nhiều đợt / từ nhiều kho. |
| **Assisted order / Đặt hàng hộ** | Nhân viên Sale tạo/hoàn tất đơn thay khách. |
| **Payment / Thanh toán** | Thu tiền cho đơn hàng và các luồng tài chính liên quan. |
| **Payment gateway / Cổng thanh toán** | Dịch vụ trung gian xử lý giao dịch online (thẻ, ví, QR…). |
| **Authorize / Capture / Void** | Giữ tiền (authorize) → trừ tiền (capture); huỷ khoản đã giữ chưa trừ (void). |
| **COD** (Cash On Delivery) — **Thanh toán khi nhận hàng** | Khách trả tiền cho người giao khi nhận hàng. |
| **Prepaid / Trả trước** | Khách thanh toán trước khi hàng được giao. |
| **Deposit / Partial payment — Đặt cọc / Trả một phần** | Trả trước một phần (thường % giá trị đơn) để đơn vào sản xuất. |
| **Refund / Hoàn tiền** | Trả lại tiền cho khách (toàn phần / một phần). |
| **Chargeback** | Khách khiếu nại giao dịch với ngân hàng/cổng để đòi lại tiền. |
| **Reconciliation / Đối soát** | So khớp giao dịch hệ thống với báo cáo của cổng thanh toán / hãng vận chuyển. |
| **Settlement report** | Báo cáo quyết toán tiền từ cổng thanh toán về doanh nghiệp. |
| **COD remittance** | Tiền hãng vận chuyển thu hộ (COD) chuyển trả về doanh nghiệp. |
| **PCI-DSS** | Bộ tiêu chuẩn bảo mật dữ liệu thẻ thanh toán. |
| **Order Management / Quản lý đơn hàng** | Lớp điều phối & theo dõi vòng đời đơn (không tự pick/pack/ship). |
| **Order timeline / Dòng thời gian đơn** | Chuỗi sự kiện của đơn theo thời gian. |
| **Order amendment / Sửa đơn** | Thay đổi nội dung đơn sau khi tạo (theo quy tắc trạng thái). |
| **RMA** (Return Merchandise Authorization) — **Phiếu cho phép trả hàng** | Chứng từ chấp thuận & hướng dẫn khách trả hàng. |
| **Exchange / Đổi hàng** | Trả hàng cũ + nhận hàng mới (thường xử lý như trả + đơn mới). |
| **Sales Chat / Messaging / Nhắn tin với Sale** | Kênh hội thoại trực tiếp khách ↔ nhân viên Sale. |
| **Conversation / Thread / Hội thoại** | Một chuỗi tin nhắn giữa khách và Sale, gắn ngữ cảnh (sản phẩm/đơn/thiết kế). |
| **Routing / Định tuyến** | Gán hội thoại đến agent phù hợp (kỹ năng, tải, lịch sử). |
| **SLA** (Service Level Agreement) | Cam kết mức dịch vụ (ví dụ thời gian phản hồi tối đa). |
| **FRT** (First Response Time) — **Thời gian phản hồi đầu tiên** | Khoảng thời gian từ khi khách nhắn đến khi Sale trả lời lần đầu. |
| **CSAT** (Customer Satisfaction) — **Mức hài lòng khách hàng** | Điểm khách chấm sau khi được hỗ trợ. |
| **Escalate / Chuyển cấp** | Đưa vấn đề vượt thẩm quyền lên người giữ quyền giám sát chat / cấp quản lý. |
| **Internal note / Ghi chú nội bộ** | Ghi chú chỉ nhân viên thấy, khách không thấy. |
| **Customer profile / Hồ sơ khách hàng** | Bộ dữ liệu về một khách: định danh, địa chỉ, lịch sử, phân khúc, rủi ro. |
| **Guest checkout / Mua không đăng ký** | Khách đặt hàng mà không tạo tài khoản. |
| **Address book / Sổ địa chỉ** | Danh sách địa chỉ giao & xuất hoá đơn của khách. |
| **Segment / Nhóm khách / Phân khúc** | Nhóm khách theo tiêu chí (VIP, doanh nghiệp, mới, rủi ro…) để áp giá/ưu đãi. |
| **Consent / Đồng ý (nhận tin)** | Sự cho phép của khách để gửi marketing, có lưu thời điểm. |
| **Right to be forgotten / Quyền được xoá** | Quyền của khách yêu cầu xoá/ẩn danh dữ liệu cá nhân. |
| **Anonymize / Ẩn danh** | Tách bỏ thông tin cá nhân khỏi dữ liệu, giữ lại phần chứng từ bắt buộc. |
| **Merge / Hợp nhất hồ sơ** | Gộp hai hồ sơ khách trùng thành một. |
| **Risk profile / blacklist — Hồ sơ rủi ro / danh sách chặn** | Đánh dấu khách có rủi ro (chargeback, gian lận, RTO cao). |

---

## 5. Thuật ngữ trạng thái thường gặp

| Trạng thái | Xuất hiện ở | Ý nghĩa ngắn |
|------------|-------------|--------------|
| `Draft` | PO, Receipt, Invoice, TO, Design | Bản nháp, sửa tự do, chưa có hiệu lực |
| `Pending Approval` | PO, TO | Chờ phê duyệt |
| `Confirmed` | PO, Order | Đã chốt nội bộ, khoá sửa nội dung chính |
| `Partially Received` | PO, TO | Đã nhận một phần |
| `Received` | PO, TO | Đã nhận đủ |
| `Closed` / `Completed` | Hầu hết chứng từ | Đã hoàn tất & đối chiếu xong |
| `Cancelled` | Hầu hết chứng từ | Bị huỷ trước khi hoàn tất |
| `In QC` | Receipt | Còn dòng hàng cần QC chưa có kết luận |
| `Quarantine` | Receipt (hàng) | Cách ly chờ kết luận chất lượng |
| `Reserved` | Order (giữ hàng) | Đã giữ số lượng cho đơn, chưa gắn vị trí/lô; làm giảm ATP |
| `Allocated` | Tồn kho (theo vị trí) | Đã phân bổ tồn ở vị trí & lô cho một đơn/lệnh cụ thể |
| `In-Transit` | Transfer, Shipment | Đang trên đường |
| `Pending Payment` | Order | Đã tạo đơn, chờ thanh toán |
| `Paid` | Order (payment_status) | Đã thu đủ tiền |
| `In Production` | Order | Đang sản xuất/in (đơn tùy chỉnh); rời trạng thái khi mọi lệnh sản xuất của đơn `Completed` |
| `Waiting Material` | Production Order | Đã qua prepress, chờ cấp phôi lên khu `PRODUCTION` |
| `Queued` / `Assigned` | Production Order | Chờ gán trạm in / đã gán trạm |
| `QC Pending` | Production Order | In xong, chờ QC thành phẩm |
| `Rework` | Production Order | Thiếu thành phẩm đạt, cần cấp phôi in lại phần thiếu |
| `Ready to Fulfill` | Order | Đã sẵn sàng cho kho xử lý |
| `Shipped` / `Delivered` | Order, Shipment | Đã gửi đi / đã giao đến khách |
| `Refunded` / `Partially Refunded` | Order, Payment | Đã hoàn tiền toàn phần / một phần |

---

## 6. Từ viết tắt nhanh

| Viết tắt | Đầy đủ | Nghĩa |
|----------|--------|-------|
| WMS | Warehouse Management System | Hệ thống quản lý kho |
| SKU | Stock Keeping Unit | Đơn vị lưu kho |
| PO | Purchase Order | Đơn đặt hàng NCC |
| TO | Transfer Order | Lệnh chuyển kho |
| MO | Manufacturing / Production Order | Lệnh sản xuất |
| RMA | Return Merchandise Authorization | Phiếu cho phép trả hàng |
| ATP | Available To Promise | Số lượng có thể hứa giao |
| UoM | Unit of Measure | Đơn vị tính |
| MOQ | Minimum Order Quantity | Số lượng đặt tối thiểu |
| QC | Quality Control | Kiểm tra chất lượng |
| RTV | Return To Vendor | Trả hàng nhà cung cấp |
| RTO | Return To Origin | Hoàn hàng về kho |
| POD | Proof of Delivery | Bằng chứng giao hàng |
| AWB | Air Waybill | Vận đơn |
| COD | Cash On Delivery | Thanh toán khi nhận hàng |
| AP | Accounts Payable | Công nợ phải trả |
| FIFO / FEFO | First-In / First-Expired-First-Out | Nhập trước / hết hạn trước, xuất trước |
| PLP / PDP | Product Listing / Detail Page | Trang danh sách / chi tiết sản phẩm |
| DPI | Dots Per Inch | Độ phân giải in |
| SLA | Service Level Agreement | Cam kết mức dịch vụ |
| FRT | First Response Time | Thời gian phản hồi đầu |
| CSAT | Customer Satisfaction | Mức hài lòng khách hàng |
| PCI-DSS | Payment Card Industry Data Security Standard | Tiêu chuẩn bảo mật dữ liệu thẻ |

---

## 7. Vai trò (Roles)

Danh sách chuẩn & định nghĩa đầy đủ xem tại [Role Registry — 00 System Overview §2](../00-system-overview/README.md).

| Thuật ngữ | Tiếng Việt / Giải thích |
|-----------|--------------------------|
| **System Admin / Quản trị hệ thống** | Cấu hình toàn hệ thống, khai báo kho, phân quyền, quản lý dữ liệu gốc sản phẩm. |
| **Procurement Staff / Nhân viên mua hàng** | Lập & theo dõi PO, chốt PO, xử lý tranh chấp hoá đơn. |
| **Warehouse Staff / Nhân viên kho** | Thao tác vật lý trong kho; gọi theo công đoạn được phân công: Receiver, Putaway Operator, Picker, Packer, Dispatch Staff (vai thao tác, không phải role riêng). |
| **Warehouse Manager / Quản lý kho** | Giám sát KPI, duyệt điều chỉnh tồn & phiếu chuyển kho, xử lý ngoại lệ; điều phối xưởng in (gán lệnh sản xuất cho trạm, đổi ưu tiên). |
| **Production Staff / Nhân viên sản xuất** | Vận hành trạm in tại xưởng nội bộ: in theo lệnh sản xuất, báo số lượng đạt / hỏng, bàn giao thành phẩm cho QC. |
| **Inventory Planner / Nhân viên kế hoạch tồn kho** | Theo dõi mức tồn giữa các kho, đề xuất & tạo lệnh chuyển kho (TO) để cân bằng tồn; đánh giá Replenishment Proposal & chuyển thành Draft PO. |
| **QC Staff / Nhân viên QC** | Kiểm tra chất lượng hàng nhập tại khu kiểm tra chất lượng (accept / reject / quarantine) & thành phẩm in tại xưởng (đạt / không đạt). |
| **Accountant / Kế toán** | Đối chiếu ba chiều PO–Receipt–Invoice, công nợ NCC, đối soát thanh toán / COD / hoàn tiền. |
| **E-commerce Admin / Quản trị sàn** | Quản lý catalog, giá, khuyến mãi, nội dung, chính sách; cấu hình template thiết kế & kiểm tra file in (prepress). |
| **Sales Staff / Nhân viên Sale** | Tư vấn & chăm sóc khách hàng (CSKH) qua chat, tạo đơn hộ, chốt thiết kế, xử lý khiếu nại. |
| **Order Coordinator / Điều phối đơn hàng** | Release đơn cho kho, xử lý đơn treo, chọn kho xuất, gộp/tách đơn. |
| **Approver / Người duyệt** | Không phải chức danh — cách gọi tắt của *người giữ quyền phê duyệt* (PO / TO / duyệt chi) được gán cho role qua RBAC. |
| **Chat Supervision / Quyền giám sát chat** | Quyền phân công hội thoại, giám sát SLA, nhận escalate — gán cho Quản trị sàn hoặc Sales Staff được chỉ định. |
