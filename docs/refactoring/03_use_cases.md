# Đặc tả Use Cases, Vai trò & Quy tắc Nghiệp vụ (v2 Baseline)

---

## 1. System Mindmap Diagram (Sơ đồ Tư duy 3 Tầng)

Cấu trúc phân rã chuẩn: **Modules $\rightarrow$ Features $\rightarrow$ Functions / User Stories**.

```mermaid
mindmap
  root((FashionRev-Ops<br/>Quản trị Doanh thu<br/>& Giá vốn))
    M01 Quản lý Danh mục & SKU
      Quản lý Sản phẩm & Biến thể
        Tạo sản phẩm cha & gán danh mục
        Khai báo SKU theo Màu và Size
        Chặn trùng mã SKU và Barcode
        Ngừng kích hoạt bảo toàn lịch sử
      Quản lý Nhà cung cấp
        Khai báo xưởng may và đầu mối
        Tra cứu lịch sử nhập theo xưởng
      Tái sử dụng SKU
        Tra cứu nhanh SKU xuyên suốt hệ thống
    M02 Nhập hàng & Landed Cost
      Lập phiếu nhập DRAFT
        Chọn xưởng may và thêm nhiều SKU
        Nhập cước xe tải và phụ phí bốc dỡ
        Lưu nháp không tác động tồn kho
      Thêm SKU động khi dỡ xe
        Khai báo mẫu phát sinh ngay trên modal
        Tự động san sẻ lại cước cho toàn lô
      Phân bổ giá vốn Landed Cost
        Phân bổ cước xe theo số lượng
        Áp dụng quy tắc bù phần dư Penny Rounding
        Xem trước giá vốn tức thời Live Preview
      Xác nhận nhập kho RECEIVE
        Khóa cố định giá vốn phân bổ
        Tăng tồn kho và tính lại giá vốn bình quân
    M03 Quản lý Tồn kho & Giá vốn
      Sổ biến động kho
        Ghi nhận lịch sử Nhập Xuất Điều chỉnh
        Truy vết số dư theo chứng từ gốc
      Giá vốn bình quân liên hoàn
        Tính lại Moving Weighted Average khi nhận hàng
        Cung cấp giá vốn tức thời cho xuất bán
      Kiểm soát xuất kho
        Chặn bán vượt tồn kho khả dụng
    M04 Quản lý Đơn bán hàng
      Ghi nhận đơn bán đa kênh
        Nhập đơn từ TikTok Shopee FB POS
        Hỗ trợ đơn nhiều dòng sản phẩm Multi-item
      Vòng đời đơn & Xuất kho
        Xuất kho SHIPPED Trừ tồn và Khóa Snapshot COGS
        Giao thành công DELIVERED Ghi nhận doanh thu
        Hủy đơn trước xuất CANCELLED Giải phóng hàng
    M05 Quản lý Phí sàn & Đối soát
      Ước tính phí sàn Strategy Pattern
        Tự động tính phí TikTok Shopee Freeship
        Xử lý phí 0đ hợp lệ
      Nhập phí thực tế & Đối soát
        Nhập phí thực nhận từ sao kê ví sàn
        Báo cáo chênh lệch phí Ước tính và Thực tế
    M06 Báo cáo & Phân tích P&L
      Thác nước Lợi nhuận Waterfall
        Bóc tách 5 bước Doanh thu Phí COGS Bao bì Lãi
        Phân loại đơn Lời Lỗ theo tỷ suất biên
      Bảng điều khiển tài chính Dashboard
        Theo dõi Doanh thu COGS và Lợi nhuận đóng góp
        Lọc đa chiều theo ngày và kênh bán
      Xuất dữ liệu & Truy vết Drillthrough
        Truy ngược từ KPI về danh sách đơn gốc
        Xuất file CSV đối soát số liệu
    M07 Hệ thống & Phân quyền
      Phân quyền theo vai trò RBAC
        Phân quyền Chủ shop Ops và Kế toán
      Nhật ký và Cấu hình
        Audit log thao tác nhận kho và xuất đơn
        Cấu hình môi trường Database
```

---

## 2. Use Case Diagram (Sơ đồ Use Case Chuẩn UML)

```mermaid
graph LR
    %% Actors
    UserOps([" Thủ kho / Vận hành<br>(Ops Staff)"])
    UserFinance([" Chủ shop / Kế toán<br>(Owner / Finance)"])
    SystemEngine([" Động cơ Tài chính<br>(Financial Engine)"])

    %% Boundary
    subgraph SystemBoundary ["HỆ THỐNG QUẢN TRỊ FASHIONREV-OPS"]
        subgraph Group_Inbound ["1. Phân hệ Nhập hàng & Landed Cost"]
            UC_DraftPO("Lập phiếu nhập hàng DRAFT")
            UC_DynamicSKU("Thêm SKU mới phát sinh khi dỡ xe")
            UC_AllocateFreight("Phân bổ cước xe & Bù phần dư lẻ")
            UC_ReceivePO("Xác nhận nhập kho RECEIVE")
            UC_UpdateAvgCost("Tính lại Giá vốn bình quân liên hoàn")
        end

        subgraph Group_Outbound ["2. Phân hệ Bán hàng & Giá vốn"]
            UC_CreateOrder("Tạo đơn bán hàng đa kênh")
            UC_EstimateFee("Ước tính phí sàn theo Strategy")
            UC_ShipOrder("Xác nhận xuất kho SHIPPED")
            UC_FreezeSnapshot("Trừ tồn kho & Khóa cứng Snapshot COGS")
        end

        subgraph Group_Analytics ["3. Phân hệ Phí sàn & Báo cáo P&L"]
            UC_RecordActualFee("Nhập phí thực tế từ sao kê sàn")
            UC_ViewWaterfall("Xem Thác Lợi Nhuận Waterfall 5 bước")
            UC_ViewDashboard("Xem Dashboard KPI & Drilldown đơn gốc")
            UC_ExportCSV("Xuất báo cáo CSV đối chiếu")
        end
    end

    UserOps --> UC_DraftPO
    UserOps --> UC_ReceivePO
    UserOps --> UC_CreateOrder
    UserOps --> UC_ShipOrder

    UserFinance --> UC_RecordActualFee
    UserFinance --> UC_ViewWaterfall
    UserFinance --> UC_ViewDashboard

    UC_DraftPO -.->|"<<extend>>"| UC_DynamicSKU
    UC_DraftPO -->|"<<include>>"| UC_AllocateFreight
    UC_ReceivePO -->|"<<include>>"| UC_UpdateAvgCost

    UC_CreateOrder -->|"<<include>>"| UC_EstimateFee
    UC_ShipOrder -->|"<<include>>"| UC_FreezeSnapshot

    UC_ViewWaterfall -.->|"<<extend>>"| UC_RecordActualFee
    UC_ViewDashboard -.->|"<<extend>>"| UC_ExportCSV

    UC_AllocateFreight --- SystemEngine
    UC_UpdateAvgCost --- SystemEngine
    UC_EstimateFee --- SystemEngine
    UC_FreezeSnapshot --- SystemEngine
    UC_ViewWaterfall --- SystemEngine

    classDef default fill:#f8fafc,stroke:#64748b,stroke-width:1px;
    classDef engineTask fill:#ede9fe,stroke:#8b5cf6,stroke-width:1.5px;
    class UC_AllocateFreight,UC_UpdateAvgCost,UC_EstimateFee,UC_FreezeSnapshot engineTask;
```

---

## 3. Bảng Phân Vai Tác Nhân (Actor & Role Specifications)

| Actor (Tác nhân) | Phân loại vai trò | Trách nhiệm chính trong hệ thống |
|---|---|---|
| **Thủ kho / Vận hành** *(Ops Staff)* | Primary Actor (Nghiệp vụ kho) | Khai báo SKU mới, lập phiếu nhập kho DRAFT, nhập cước xe, xác nhận nhận kho thực tế (RECEIVE), xuất đơn bán hàng (SHIPPED). |
| **Kế toán / Tài chính** *(Finance Manager)* | Primary Actor (Nghiệp vụ tài chính) | Giám sát Thác nước Lợi nhuận (Waterfall), nhập phí sàn thực tế từ sao kê ví để đối soát, kiểm tra chi phí bao bì, xuất báo cáo P&L. |
| **Chủ shop** *(Shop Owner / Admin)* | Primary Actor (Quản trị) | Giám sát Dashboard 4 chỉ số KPI tổng thể, phê duyệt điều chỉnh kho, phân quyền người dùng và quản trị hệ thống. |
| **Động cơ Tài chính** *(Financial Engine)* | Secondary Actor (Hệ thống tự động) | Tự động phân bổ cước xe (Penny Allocation), tính giá vốn bình quân (Moving Weighted Average), khóa snapshot COGS, bóc tách thác lợi nhuận 5 bước. |

---

## 4. Ma Trận Ranh Giới Phạm Vi Chức Năng (Functional Scope Matrix)

| Module trong Mindmap | Tính năng trong phạm vi MVP (In-Scope) | Chủ động loại trừ / Để Phase 2 (Out-of-Scope) |
|---|---|---|
| **1. Danh mục & SKU (M01)** | • CRUD Sản phẩm, Biến thể Màu/Size, Nhà cung cấp.<br>• Chặn trùng mã SKU/Barcode.<br>• Ngừng kích hoạt bảo toàn lịch sử. | • Quản lý định mức phụ liệu may mặc (BOM).<br>• Quản lý danh mục đa công ty / đa thương hiệu. |
| **2. Nhập hàng & Landed Cost (M02)** | • Lập phiếu DRAFT, Thêm SKU động khi dỡ xe.<br>• Phân bổ cước xe + bao bì nhập theo số lượng (Penny rounding).<br>• Live preview giá vốn tức thì trên modal.<br>• Xác nhận nhận kho 1 lần (Idempotent & Atomic). | • Quy trình nhận hàng từng phần (Partial Inbound).<br>• Quản lý công nợ mua hàng nhà cung cấp nâng cao. |
| **3. Tồn kho & Giá vốn (M03)** | • Sổ biến động kho (Nhập/Xuất/Điều chỉnh).<br>• Giá vốn bình quân liên hoàn (Moving Weighted Average).<br>• Khóa Snapshot COGS khi SHIPPED.<br>• Chặn xuất bán vượt tồn khả dụng. | • Quản lý đa kho vật lý / sơ đồ giá kệ.<br>• Phân tích hạn sử dụng và tuổi tồn chi tiết theo lô vị trí. |
| **4. Đơn bán hàng (M04)** | • Nhập thông tin đơn từ TikTok Shop, Shopee, FB POS.<br>• Đơn nhiều dòng sản phẩm (Multi-item Order).<br>• Vòng đời đơn: `PENDING` $\rightarrow$ `SHIPPED` $\rightarrow$ `DELIVERED` / `CANCELLED`. | • Đồng bộ tự động thời gian thực qua Open API / Webhook sàn.<br>• Phân luồng đóng gói tự động (Wave picking). |
| **5. Phí sàn & Đối soát (M05)** | • Ước tính phí sàn Shopee/TikTok/FB POS theo Strategy Pattern.<br>• Nhập phí thực tế đối soát từ sao kê ví.<br>• Xử lý phí 0đ hợp lệ. | • Tự động import và phân tích file Excel sao kê ví hàng loạt.<br>• Kết nối trực tiếp tài khoản ngân hàng doanh nghiệp. |
| **6. Báo cáo & Phân tích (M06)** | • Thác nước Lợi nhuận 5 bước theo từng đơn hàng.<br>• Phân loại đơn: Lãi cao (>40%), Lãi mỏng (<20%), Đơn lỗ (<0%).<br>• Dashboard KPI lọc theo ngày/kênh bán.<br>• Drillthrough truy vết đơn gốc và xuất CSV. | • Báo cáo Lợi nhuận ròng toàn shop sau Chi phí vận hành M09 (Mặt bằng, Lương, Quảng cáo).<br>• Dự báo doanh số bằng mô hình AI/ML. |
| **7. Hệ thống & Phân quyền (M07)** | • Phân quyền 3 vai trò: Chủ shop (Owner), Thủ kho (Ops), Kế toán (Finance).<br>• Ghi nhật ký Audit Log cho thao tác Nhập/Xuất kho.<br>• Tùy chọn cấu hình Database (.env) rõ ràng. | • Đăng nhập một chạm SSO (Google OAuth, LDAP).<br>• Đa người thuê (Multi-tenant SaaS). |
| **8. Hoàn hàng & Tổn thất (M08)** | *Loại trừ khỏi MVP* | • Toàn bộ quy trình tiếp nhận hàng hoàn, phân loại tái nhập kho hoặc ghi giảm giá trị tổn thất (Chuyển sang Phase 2). |

---

## 5. Bản đồ Vai trò & Ma trận Phân quyền (RBAC)

| Mã vai trò | Tên vai trò | Trách nhiệm chính | Màn hình truy cập chính |
|---|---|---|---|
| **ROLE_OWNER** | Chủ Shop / Quản trị | Toàn quyền kiểm soát tài chính, cấu hình hệ thống, duyệt điều chỉnh kho và xem báo cáo tổng thể. | Toàn bộ hệ thống, Dashboard P&L, Cấu hình. |
| **ROLE_OPS** | Thủ kho / Vận hành | Khai báo SKU, lập và nhận phiếu nhập kho, theo dõi tồn kho, tạo và xác nhận xuất đơn bán hàng. | Màn hình Nhập hàng, Danh mục SKU, Danh sách Đơn hàng. |
| **ROLE_FINANCE** | Kế toán / Tài chính | Nhập và đối soát phí sàn, kiểm tra chi phí bao bì, phân tích biên lợi nhuận đơn và chốt kỳ báo cáo. | Chi tiết Đơn hàng, Phí & Đối soát, Báo cáo Lợi nhuận Waterfall. |

### Ma trận quyền (Permission Matrix) theo Module

| Module / Hành động | Quyền API | OPS | FINANCE | OWNER | Ghi chú kiểm soát |
|---|---|:---:|:---:|:---:|---|
| **M01: Danh mục & SKU** | `catalog:read` / `write` |  /  |  /  |  /  | Ops tạo SKU; chỉ Owner có quyền ngừng sử dụng |
| **M02: Tạo phiếu nhập DRAFT** | `po:create_draft` |  |  |  | Lưu nháp chưa làm thay đổi tồn kho |
| **M02: Nhận kho PO (RECEIVE)** | `po:receive` |  |  |  | Tăng tồn, cập nhật giá vốn bình quân (Idempotent) |
| **M03: Điều chỉnh tồn thủ công** | `stock:adjust` |  |  |  | Bắt buộc có lý do và chứng từ điều chỉnh |
| **M04: Tạo & Sửa đơn bán** | `order:create` / `update` |  |  |  | Nhập thông tin đơn từ các kênh bán |
| **M04: Xuất kho đơn (SHIP)** | `order:ship` |  |  |  | Trừ tồn khả dụng, khóa cứng snapshot COGS |
| **M05: Nhập phí thực tế sàn** | `fee:record_actual` |  |  |  | Ghi đè biểu phí ước tính bằng số thực nhận |
| **M06: Xem báo cáo P&L / Margin**| `analytics:view` |  |  |  | Ops không xem biên lãi ròng chi tiết |
| **M07: Cấu hình shop & DB** | `admin:system` |  |  |  | Chỉ Owner có quyền đổi cấu hình |

---

## 2. Máy trạng thái (State Machines) & Vòng đời giao dịch

### 2.1 Vòng đời Phiếu Nhập Hàng (Purchase Order - PO)

```mermaid
stateDiagram-v2
    [*] --> DRAFT : Tạo phiếu nháp / Thêm SKU động
    DRAFT --> DRAFT : Chỉnh sửa số lượng, cước phí, preview
    DRAFT --> CANCELLED : Hủy phiếu nháp
    DRAFT --> RECEIVED : Xác nhận nhập kho (Atomic Transaction)
    RECEIVED --> [*] : Đã nhập kho (Không thể sửa trực tiếp)
```

* **DRAFT (Nháp):** Đang kiểm đếm, thêm bớt SKU, chỉnh sửa cước xe/phụ phí. **Chưa tác động vào số lượng tồn kho.**
* **RECEIVED (Đã nhận hàng):**
  * Tăng số lượng tồn kho `current_stock` theo từng SKU.
  * Tính lại **Giá vốn bình quân gia quyền liên hoàn** cho từng SKU.
  * Khóa cứng (freeze) toàn bộ chi phí phân bổ trên dòng PO (`landed_cost`).
  * Giao dịch là **Atomic** (nếu lỗi 1 dòng thì rollback toàn bộ) và **Idempotent** (gửi lại không cộng kho lần 2).
* **CANCELLED (Đã hủy):** Hủy phiếu nháp, không phát sinh giao dịch tồn kho.

---

### 2.2 Vòng đời Đơn Bán Hàng (Order)

```mermaid
stateDiagram-v2
    [*] --> PENDING : Nhập đơn mới (Đang chuẩn bị hàng)
    PENDING --> CANCELLED : Hủy đơn trước khi xuất
    PENDING --> SHIPPED : Xuất kho giao vận (Trừ tồn + Khóa COGS Snapshot)
    SHIPPED --> DELIVERED : Giao thành công (Ghi nhận Doanh thu & Lãi)
    SHIPPED --> RETURNED_TRIAGE : Khách từ chối nhận / Trả hàng (Phase 2)
    DELIVERED --> [*] : Hoàn tất đơn hàng
    CANCELLED --> [*] : Kết thúc
```

* **PENDING (Chờ xử lý / Đã chốt):** Đơn vừa nhập vào hệ thống. Có thể giữ chỗ tồn kho (Reserved) nếu cấu hình.
* **SHIPPED (Đã xuất kho / Đang giao):**
  * **Trừ tồn kho thực tế** (`current_stock = current_stock - quantity`).
  * **Khóa snapshot giá vốn:** Gán cố định `applied_landed_cogs = current_avg_cogs` tại đúng thời điểm xuất. Khi nhập lô hàng mới sau này với giá khác, snapshot này **không bao giờ bị thay đổi**.
* **DELIVERED (Đã giao hàng thành công):**
  * Ghi nhận Doanh thu chính thức (`gross_sales`) và ghi nhận vào Báo cáo Lợi nhuận kỳ này.
* **CANCELLED (Đã hủy trước xuất):** Hủy đơn, không phát sinh chi phí xuất kho và giải phóng tồn giữ chỗ.

---

## 3. Quy tắc Kế toán & Công thức Tính toán Tài chính

### 3.1 Công thức Phân bổ Giá vốn Nhập kho (Landed Cost Allocation)

Giá vốn nhập kho của một đơn vị SKU trong lô nhập gồm 3 thành phần:
$$\text{Landed Cost}_{\text{SKU}} = \text{Đơn giá mua} + \text{Cước phân bổ} + \text{Bao bì nhập}$$

Trong đó:
$$\text{Cước phân bổ mỗi chiếc} = \frac{\text{Tổng cước vận chuyển (Truck Freight)} + \text{Phụ phí bốc dỡ/kiểm đếm (Handling)}}{\sum \text{Số lượng toàn bộ lô nhập}}$$

#### Quy tắc làm tròn & Bù phần lẻ (Penny Allocation Rule):
* Đơn vị tiền tệ chuẩn: **VND (Số nguyên, không có số thập phân lẻ)**.
* Khi chia cước phát sinh số dư lẻ (ví dụ: cước 100.000đ chia cho 3 chiếc = 33.333,33đ/chiếc):
  * Dòng 1: `33.333đ`
  * Dòng 2: `33.333đ`
  * Dòng 3 (Dòng cuối): `33.334đ` (bù phần dư 1đ)
  * **Bất biến:** $\sum \text{Allocated Freight} = \text{Total Freight Inbound}$.

---

### 3.2 Thuật toán Giá vốn Bình quân Gia quyền Liên hoàn (Moving Weighted Average)

Mỗi khi phiếu nhập chuyển trạng thái `RECEIVED`, giá vốn bình quân mới của từng biến thể SKU được tính lại ngay lập tức:

$$\text{Avg Cost}_{\text{mới}} = \frac{(\text{Tồn trước nhập} \times \text{Avg Cost}_{\text{hiện tại}}) + (\text{Số lượng nhập} \times \text{Landed Cost}_{\text{lô mới}})}{\text{Tồn trước nhập} + \text{Số lượng nhập}}$$

#### Ví dụ chuẩn kiểm thử (Test Case Baseline):
1. **Lô 1:** Tồn kho ban đầu = 0. Nhập **100 chiếc** Áo Thun Đen size M với Landed Cost = **78.000đ/c**.  
   $\rightarrow$ Tồn kho = 100 chiếc, Giá vốn bình quân = **78.000đ/c**.
2. **Đơn bán 1:** Xuất bán **30 chiếc** (Trạng thái `SHIPPED`).  
   $\rightarrow$ Tồn kho còn = 70 chiếc. Snapshot giá vốn cho Đơn bán 1 = **78.000đ/c**.
3. **Lô 2:** Nhập thêm **80 chiếc** với Landed Cost mới (do tăng cước xe) = **85.000đ/c**.  
   $$\text{Avg Cost}_{\text{mới}} = \frac{(70 \times 78.000) + (80 \times 85.000)}{70 + 80} = \frac{5.460.000 + 6.800.000}{150} = \frac{12.260.000}{150} = \mathbf{81.733đ/c}$$
   $\rightarrow$ Tồn kho = 150 chiếc, Giá vốn bình quân mới = **81.733đ/c**.
4. **Kiểm tra tính bất biến:** Đơn bán 1 (đã xuất trước đó) **vẫn giữ nguyên snapshot 78.000đ/c**, không bị nhảy số liệu!

---

### 3.3 Thác Lợi Nhuận 5 Bước (Order Net Profit Waterfall)

Lợi nhuận thực nhận trên từng đơn hàng được bóc tách rành mạch qua 5 bước:

$$\begin{aligned}
\text{Bước 1 (Tổng thu)} &: \text{Tiền khách trả (Gross Sales)} \\
\text{Bước 2 (Trừ phí sàn \& trợ giá)} &: - \text{Phí hoa hồng / dịch vụ sàn (Platform Fee)} \\
& - \text{Voucher shop chịu (Voucher Discount)} \\
& - \text{Phí ship shop chịu (Shop Shipping Fee)} \\
\rightarrow \text{Số tiền sàn quyết toán} &: = \text{Net Settlement Amount} \\
\text{Bước 3 (Trừ giá vốn xuất)} &: - \sum (\text{Applied Landed COGS} \times \text{Quantity}) \\
\text{Bước 4 (Trừ bao bì đóng gói)} &: - \sum (\text{Packaging Box \& Tape Expense} \times \text{Quantity}) \\
\text{Bước 5 (Trừ tổn thất hoàn - P2)} &: - \text{Return Loss Cost (0đ trong MVP)} \\
\mathbf{\rightarrow \text{Lợi Nhuận Thực Nhận Vào Túi}} &: = \mathbf{\text{True Net Profit In Pocket}}
\end{aligned}$$

#### Quy tắc tính phí:
1. **Phí 0đ là hợp lệ:** Miễn phí sàn, Shop Freeship 0đ được ghi nhận rõ là `0.00`, tuyệt đối không dùng logic fallback gán giá trị mặc định.
2. **Tách biệt Ước tính vs Thực tế:**
   * Khi mới tạo đơn: Tính phí sàn theo công thức ước tính (Estimated Fee Strategy).
   * Khi có sao kê thực tế: Cho phép Kế toán nhập phí thực nhận (`actual_platform_fee`), hệ thống giữ lại lịch sử đối soát và tính lại Waterfall theo số thực.

---

## 4. Chi tiết Use Cases Trọng Yếu (Detailed Specs)

### UC-01: Nhập Kho Hàng Loạt & Phân Bổ Chi Phí (Receive Inbound PO)
* **Actor:** `ROLE_OPS`
* **Tiền điều kiện:** Phiếu nhập đang ở trạng thái `DRAFT`, có ít nhất 1 dòng sản phẩm, cước xe $\ge 0$.
* **Happy Path:**
  1. Ops nhấn "Xác nhận nhập kho".
  2. Backend mở Transaction:
     - Kiểm tra trạng thái hiện tại của PO là `DRAFT`.
     - Phân bổ cước xe và phụ phí theo số lượng (áp dụng Penny Allocation).
     - Cập nhật từng dòng `PurchaseOrderItem` với `landed_cost` chính xác.
     - Tăng `current_stock` của từng SKU trong bảng `product_variants`.
     - Tính và cập nhật lại `current_avg_cogs` cho từng SKU.
     - Chuyển trạng thái PO sang `RECEIVED`, ghi nhận `inbound_date = now()`.
     - Ghi nhật ký Audit Log (`user_id`, `action: RECEIVE_PO`, `po_id`).
  3. Commit Transaction.
  4. Trả về kết quả thành công và bảng phân bổ giá vốn chính thức.
* **Exception Paths:**
  * *E1 - PO đã được nhận trước đó:* Báo lỗi `409 Conflict: Phiếu nhập đã được xác nhận trước đó.` (Không cộng kho lặp lại).
  * *E2 - Lỗi hệ thống giữa chừng:* Rollback toàn bộ, không có dòng tồn kho nào bị tăng dở dang.

---

### UC-02: Xuất Kho & Đóng Băng Snapshot Đơn Bán (Ship Order & Freeze COGS)
* **Actor:** `ROLE_OPS`
* **Tiền điều kiện:** Đơn hàng ở trạng thái `PENDING`, các SKU có đủ tồn kho khả dụng.
* **Happy Path:**
  1. Ops nhấn "Xác nhận xuất kho (Giao vận)".
  2. Backend mở Transaction:
     - Kiểm tra `order_status == PENDING`.
     - Với từng dòng `OrderItem`:
       - Kiểm tra `current_stock >= quantity`. Nếu không đủ, raise ngoại lệ `422 Unprocessable: SKU [Mã] không đủ tồn kho`.
       - Lấy giá vốn bình quân hiện tại của SKU: `applied_cogs = variant.current_avg_cogs`.
       - Trừ tồn kho: `variant.current_stock -= item.quantity`.
       - Lưu snapshot: `item.applied_landed_cogs = applied_cogs`.
       - Tính biên lãi dự kiến của dòng: `net_margin = selling_price - applied_cogs - packaging_expense`.
     - Chuyển `order_status = SHIPPED`.
     - Ghi nhật ký Audit Log.
  3. Commit Transaction.
  4. Trả về thông tin đơn hàng đã xuất kèm snapshot giá vốn không thể thay đổi.
* **Exception Paths:**
  * *E1 - Bán vượt tồn:* Rollback toàn bộ, thông báo rõ danh sách SKU bị thiếu hàng và số lượng tồn hiện có.
  * *E2 - SKU chưa có giá vốn nhập:* Nếu `current_avg_cogs == 0` do chưa nhập kho bao giờ, hệ thống cảnh báo hoặc yêu cầu xác nhận ghi nhận giá vốn 0đ, không tự động đoán 40% giá bán.

---

## 5. Từ Điển Thuật Ngữ Nghiệp Vụ (Glossary)

* **SKU (Stock Keeping Unit):** Mã định danh biến thể sản phẩm duy nhất theo phân loại Màu - Size.
* **Landed Cost:** Tổng chi phí để đưa 1 sản phẩm về tới kệ kho (Giá mua sỉ + Cước xe phân bổ + Phụ phí + Bao bì nhập).
* **Frozen Landed COGS (Snapshot giá vốn):** Giá vốn bình quân tại thời điểm xuất kho, được ghi cứng vào dòng đơn hàng để bảo vệ tính bất biến của báo cáo tài chính lịch sử.
* **Net Settlement Amount (Tiền sàn quyết toán):** Số tiền sàn TMĐT thực tế chi trả về ví shop sau khi đã trừ phí hoa hồng, phí dịch vụ và voucher sàn.
* **True Net Profit In Pocket (Lãi thực nhận vào túi):** Số tiền lãi còn lại của đơn hàng sau khi lấy Tiền sàn quyết toán trừ đi Giá vốn nhập hàng và Chi phí bao bì đóng gói.
