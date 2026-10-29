# Hướng dẫn cho Claude khi làm việc với repo này

Repo này chứa **tài liệu sản phẩm (Business Requirement Document) cho StockFlowCommerce** — một nền tảng hợp nhất WMS + Sàn TMĐT. Toàn bộ tài liệu nằm trong [`docs/`](docs/README.md). Không có mã nguồn ứng dụng.

**Nguyên tắc bao trùm:** đây là tài liệu nghiệp vụ mà nhiều người đọc và dựa vào. Mọi thay đổi nội dung nghiệp vụ đều phải **phân tích kỹ — tìm hết chỗ liên quan — hỏi người dùng — rồi mới sửa**. Không sửa mù, không tự quyết.

---

## Rule 1 — Phân tích & tìm kiếm TRƯỚC khi làm bất cứ gì

Trước khi tạo mới hoặc chỉnh sửa nội dung, BẮT BUỘC:

1. **Đọc file đích đầy đủ** (không chỉ đoạn cần sửa) để nắm ngữ cảnh, template, văn phong.
2. **Tìm mọi chỗ liên quan** bằng Grep/Glob:
   - Mục **"Liên kết với các module khác"** của module đang đụng tới — mở từng module được liệt kê.
   - Grep tên module / tên trạng thái / tên thuật ngữ trên **toàn bộ `docs/`** để thấy nó còn xuất hiện ở đâu (module khác, [diagrams/](docs/diagrams/README.md), [glossary/](docs/glossary/README.md), [open-questions/](docs/open-questions/README.md)).
   - Kiểm tra các **sơ đồ Mermaid** trong [docs/diagrams/README.md](docs/diagrams/README.md) và bảng vòng đời trong module có nhắc tới phần đang sửa không.
3. **Tóm tắt phát hiện** cho người dùng: nội dung hiện tại nói gì, nó liên kết/mâu thuẫn với chỗ nào, thay đổi sẽ lan tới đâu.

> Không được bắt đầu sửa khi chưa hoàn thành bước tìm kiếm này.

---

## Rule 2 — HỎI người dùng trước khi thay đổi nội dung nghiệp vụ

**Phải trình bày và chờ người dùng duyệt** trước khi:

- Thêm / sửa / xóa **business rule** (`BR-xx`), **trạng thái**, **bước trong luồng nghiệp vụ**, **edge case**.
- Thêm / đổi / bỏ **actor**, **Input/Output**, **liên kết giữa các module**.
- Thêm / sửa **sơ đồ Mermaid** (flowchart, sequence, stateDiagram).
- Đổi **cấu trúc thư mục**, tên module, thứ tự số module.
- Thêm module mới hoặc file mới ngoài cấu trúc hiện có.
- Đổi một **giả định** (`⚠️ Giả định`) hoặc câu hỏi trong [open-questions/](docs/open-questions/README.md).

Khi hỏi, nêu rõ 3 điều: **(a)** sẽ đổi gì, **(b)** ở file/mục nào, **(c)** những chỗ khác cần cập nhật đồng bộ theo (kết quả của Rule 1). Đề xuất phương án cụ thể, đừng hỏi chung chung.

**Được sửa trực tiếp KHÔNG cần hỏi** (chỉ các loại này):

- Lỗi chính tả, lỗi ngữ pháp, câu tối nghĩa mà không đổi nghĩa nghiệp vụ.
- Link Markdown hỏng, đường dẫn tương đối sai.
- Lỗi định dạng Markdown (bảng lệch, heading sai cấp, code fence thiếu).
- Cú pháp Mermaid không render được (sửa để render đúng **mà không đổi nội dung logic** sơ đồ).

Nếu không chắc một thay đổi thuộc nhóm "cần hỏi" hay "được sửa trực tiếp" → **coi như cần hỏi**.

---

## Rule 3 — Giữ tính nhất quán xuyên toàn bộ tài liệu

Tài liệu này tham chiếu chéo rất nhiều. Khi một thay đổi được duyệt, phải cập nhật đồng bộ **trong cùng một lần**:

| Khi đổi... | Phải rà & cập nhật... |
|------------|----------------------|
| Một **trạng thái** trong bảng vòng đời của module | `stateDiagram` của chính module đó; các sơ đồ trong [diagrams/](docs/diagrams/README.md); bảng "trạng thái thường gặp" trong [glossary/](docs/glossary/README.md); mọi module trỏ tới trạng thái này ở mục liên kết |
| Một **thuật ngữ / từ viết tắt** | [glossary/README.md](docs/glossary/README.md) (cả bảng nhóm lẫn bảng viết tắt); mọi nơi dùng thuật ngữ đó |
| **Tên / số / phạm vi một module** | [docs/README.md](docs/README.md); [00-system-overview](docs/00-system-overview/README.md) (module map + sơ đồ); mục "Liên kết" của mọi module liên quan; [README.md](README.md) gốc; đường dẫn tương đối trong link |
| Một **luồng nghiệp vụ** (thêm/bớt bước) | Sơ đồ tương ứng trong [diagrams/](docs/diagrams/README.md); bảng "Luồng nghiệp vụ chính" trong [00-system-overview](docs/00-system-overview/README.md) |
| Một **business rule** ảnh hưởng module khác | Mục Business rules / Edge case của (các) module bị ảnh hưởng |

Sau khi sửa xong, Grep lại từ khóa vừa đổi trên toàn `docs/` để chắc chắn không còn chỗ nào lệch.

---

## Rule 4 — Nội dung suy đoán phải được đánh dấu

- Bất kỳ nội dung nào **không suy ra trực tiếp từ yêu cầu đề bài / thông tin người dùng cung cấp** → đánh dấu bằng callout:
  `> ⚠️ Giả định: <nội dung>. Xem [open-questions](<đường-dẫn-tương-đối>/open-questions/README.md).`
- Đồng thời thêm một dòng tương ứng vào [docs/open-questions/README.md](docs/open-questions/README.md): đặt vào nhóm phù hợp (A–G), ghi **câu hỏi**, **giả định tạm thời**, **mức ảnh hưởng** (🔴/🟡/🟢), **module liên quan**.
- Không "âm thầm" biến giả định thành sự thật. Khi người dùng xác nhận một giả định, cập nhật cả nội dung module lẫn mục open-questions (đánh dấu đã chốt / xóa khỏi danh sách).

---

## Rule 5 — Bám template & văn phong

**Mỗi module** (`docs/warehouse/*/README.md`, `docs/ecommerce/*/README.md`) giữ đúng 7 mục, đúng thứ tự:

1. Mục đích (Purpose)
2. Actor liên quan
3. Input / Output
4. Luồng nghiệp vụ từng bước (Step-by-step)
5. Trạng thái / Vòng đời (Lifecycle) — bảng, kèm `stateDiagram-v2` nếu là chứng từ có vòng đời
6. Ràng buộc nghiệp vụ (Business Rules) & Edge case — rule đánh mã `BR-01`, `BR-02`…; edge case dạng bảng
7. Liên kết với các module khác — bảng, mỗi dòng có link tương đối

**Văn phong:**

- Viết bằng **tiếng Việt**, thuật ngữ tiếng Anh đặt trong ngoặc lần đầu xuất hiện: `Nhận hàng (Receipt)`.
- Chuẩn tài liệu nghiệp vụ / BRD: súc tích, khách quan, không marketing.
- Dùng **bảng** cho: trạng thái, thuộc tính, Input/Output, so sánh, edge case.
- Bảng trạng thái theo mẫu cột: *Trạng thái · Ý nghĩa · Chuyển tiếp cho phép · Ai thực hiện*.
- Sơ đồ vẽ bằng **Mermaid** trong code fence ```mermaid.

---

## Rule 6 — Cấu trúc file & thư mục

```
docs/
├── README.md                     Mục lục tổng
├── 00-system-overview/README.md
├── warehouse/NN-<slug>/README.md   NN = 01..11, 19, 20 (19-production, 20-inventory-control bổ sung sau)
├── ecommerce/NN-<slug>/README.md   NN = 12..18
├── diagrams/README.md
├── glossary/README.md
└── open-questions/README.md
```

- **Mỗi module = một thư mục** có prefix số thứ tự + `README.md` bên trong (để trình duyệt Git render khi mở thư mục).
- Prefix số `01`..`18` chạy xuyên suốt theo **thứ tự luồng nghiệp vụ**; ngoại lệ `19-production` (về luồng nằm giữa 17 và 07/08) và `20-inventory-control` (chạy song song, kiểm soát tồn) được thêm sau — không đổi số/đổi thứ tự khi chưa hỏi.
- Không tạo file `.md` rời ngoài cấu trúc này khi chưa được duyệt. Ảnh/tài nguyên đi kèm module đặt trong chính thư mục module đó.
- Link giữa các file dùng **đường dẫn tương đối** (ví dụ từ `warehouse/03-receipt/` tới payment là `../../ecommerce/15-payment/README.md`).

---

## Quy trình chuẩn khi có yêu cầu sửa docs

1. **Phân tích** yêu cầu → xác định (các) file & mục bị đụng tới.
2. **Tìm kiếm** (Rule 1): Grep/Glob/Read hết chỗ liên quan, kể cả diagrams & glossary.
3. **Báo cáo** cho người dùng: hiện trạng, các liên kết/mâu thuẫn phát hiện, phạm vi lan tỏa.
4. **Đề xuất** thay đổi cụ thể + danh sách file cần cập nhật đồng bộ → **chờ duyệt** (Rule 2).
5. **Thực hiện** sau khi được duyệt: sửa file chính + tất cả file đồng bộ (Rule 3) trong cùng lượt.
6. **Kiểm tra lại**: Grep từ khóa vừa đổi trên toàn `docs/`; kiểm tra link tương đối; kiểm tra Mermaid render được.
7. **Không commit** trừ khi người dùng yêu cầu.
