# Business Architecture Specification: System Mindmap & Use Case Diagrams

---

## 1. System Mindmap Diagram (3-Tier Hierarchical Structure)

The system is structured hierarchically: **Modules $\rightarrow$ Features $\rightarrow$ Functions / User Stories**.

### 1.1 Visual Mindmap (Rendered Image & Mermaid)

![FashionRev-Ops System Mindmap](screenshots/system_mindmap.png)

```mermaid
mindmap
  root((FashionRev-Ops<br/>Revenue & Landed Cost<br/>Control System))
    M01 Catalog & SKU Management
      Product & Variant Management
        Create Parent Products & Categories
        Define SKUs by Color & Size
        Prevent Duplicate SKUs & Barcodes
        Deactivate SKU preserving history
      Supplier Management
        Register garment workshops & vendors
        Track PO history by supplier
      SKU Reusability
        Quick SKU lookup across POs and Orders
    M02 Inbound & Landed Cost Engine
      Create DRAFT Purchase Orders
        Select supplier & add multiple SKUs
        Input truck freight & handling fees
        Draft saving without stock mutation
      Dynamic SKU Batch Management
        Add unplanned models on the fly
        Auto-redistribute freight across batch
      Landed Cost Allocation
        Unit landed cost distribution
        Penny rounding allocation rule
        Real-time calculation preview
      Confirm Receipt RECEIVE
        Freeze batch landed cost allocation
        Increment inventory & recalculate moving average COGS
    M03 Inventory & Cost of Goods Sold
      Inventory Ledger
        Record Inbound Outbound & Adjustment logs
        Trace stock balance to source documents
      Moving Weighted Average Costing
        Recalculate average COGS upon receipt
        Provide real-time unit COGS for fulfillment
      Stock Outbound Control
        Atomic lock preventing overselling
    M04 Multi-Channel Order Fulfillment
      Multi-Channel Order Ingestion
        Ingest orders from TikTok Shopee FB POS
        Support multi-item multi-line orders
      Order Lifecycle & Dispatch
        Dispatch SHIPPED deduct stock & freeze COGS snapshot
        Delivered DELIVERED recognize revenue
        Pre-dispatch Cancel CANCELLED release reservation
    M05 Platform Fees & Reconciliation
      Estimated Fees Strategy Pattern
        Calculate TikTok Shopee Freeship fees
        Handle zero fee valid values
      Actual Fee Ingestion & Audit
        Record actual settlement fees from platform wallet
        Variance analysis between estimated vs actual fees
    M06 Financial Analytics & P&L Reports
      Order Waterfall Ledger
        5-step deduction Gross Sales Fees COGS Packaging Net
        Classify orders High Margin Low Margin Loss
      Executive Financial Dashboard
        Track Revenue COGS and Contribution Margin
        Multi-dimensional filters by date and channel
      Data Export & Drillthrough
        Drillthrough from KPI to source order records
        Export reconciliation CSV files
    M07 System Administration & Governance
      Role-Based Access Control RBAC
        Roles for Shop Owner Ops Staff and Finance Manager
      Audit Logging & System Config
        Immutable audit log for inbound and dispatch
        Explicit Database environment configuration
```

---

## 2. UML Use Case Diagram

```mermaid
graph LR
    %% Actors
    UserOps([" Ops & Warehouse Staff<br>(Primary Actor)"])
    UserFinance([" Shop Owner / Finance<br>(Primary Actor)"])
    SystemEngine([" Financial Calculation Engine<br>(Secondary Actor)"])

    %% System Boundary
    subgraph SystemBoundary ["FASHIONREV-OPS SYSTEM BOUNDARY"]
        %% Group Inbound
        subgraph Group_Inbound ["1. Inbound & Landed Cost Subsystem"]
            UC_DraftPO("Create DRAFT Purchase Order")
            UC_DynamicSKU("Add Dynamic SKU on the Fly")
            UC_AllocateFreight("Allocate Freight & Penny Rounding")
            UC_ReceivePO("Confirm Inbound Receipt (RECEIVE)")
            UC_UpdateAvgCost("Recalculate Moving Average COGS")
        end

        %% Group Outbound
        subgraph Group_Outbound ["2. Order Fulfillment Subsystem"]
            UC_CreateOrder("Ingest Multi-Channel Order")
            UC_EstimateFee("Estimate Marketplace Fees (Strategy)")
            UC_ShipOrder("Confirm Dispatch (SHIPPED)")
            UC_FreezeSnapshot("Deduct Stock & Freeze COGS Snapshot")
        end

        %% Group Analytics
        subgraph Group_Analytics ["3. Fee Reconciliation & P&L Subsystem"]
            UC_RecordActualFee("Record Actual Settlement Fees")
            UC_ViewWaterfall("View 5-Step Waterfall Profit Ledger")
            UC_ViewDashboard("Monitor Executive KPIs & Drilldown")
            UC_ExportCSV("Export Reconciliation CSV")
        end
    end

    %% Ops Connections
    UserOps --> UC_DraftPO
    UserOps --> UC_ReceivePO
    UserOps --> UC_CreateOrder
    UserOps --> UC_ShipOrder

    %% Finance / Owner Connections
    UserFinance --> UC_RecordActualFee
    UserFinance --> UC_ViewWaterfall
    UserFinance --> UC_ViewDashboard

    %% Include & Extend Relationships
    UC_DraftPO -.->|"<<extend>>"| UC_DynamicSKU
    UC_DraftPO -->|"<<include>>"| UC_AllocateFreight
    UC_ReceivePO -->|"<<include>>"| UC_UpdateAvgCost

    UC_CreateOrder -->|"<<include>>"| UC_EstimateFee
    UC_ShipOrder -->|"<<include>>"| UC_FreezeSnapshot

    UC_ViewWaterfall -.->|"<<extend>>"| UC_RecordActualFee
    UC_ViewDashboard -.->|"<<extend>>"| UC_ExportCSV

    %% Engine Automation Connections
    UC_AllocateFreight --- SystemEngine
    UC_UpdateAvgCost --- SystemEngine
    UC_EstimateFee --- SystemEngine
    UC_FreezeSnapshot --- SystemEngine
    UC_ViewWaterfall --- SystemEngine

    %% Styling
    classDef default fill:#f8fafc,stroke:#64748b,stroke-width:1px;
    classDef engineTask fill:#ede9fe,stroke:#8b5cf6,stroke-width:1.5px;
    class UC_AllocateFreight,UC_UpdateAvgCost,UC_EstimateFee,UC_FreezeSnapshot engineTask;
```

---

## 3. Actor & Role Specifications

| Actor | Role Type | Core Responsibilities |
|---|---|---|
| **Ops & Warehouse Staff** *(UserOps)* | Primary Actor (Operations) | Creates PO drafts, enters truck shipping & handling expenses, adds unplanned SKUs upon truck unloading, confirms inbound physical receipt, dispatches outgoing orders. |
| **Finance Manager / Accountant** *(UserFinance)* | Primary Actor (Finance) | Inspects order waterfall deductions, records actual fees from platform payout statements, audits packaging expenses, exports periodic P&L reports. |
| **Shop Owner / Administrator** *(UserOwner)* | Primary Actor (Governance) | Monitors high-level KPI dashboard, approves manual stock adjustments, manages team roles, configures system environments. |
| **Financial Calculation Engine** *(SystemEngine)* | Secondary Actor (Automated Engine) | Automatically executes freight allocation algorithms, computes moving weighted average COGS, freezes immutable order snapshots, renders 5-step waterfall profit deductions. |

---

## 4. Functional Scope Matrix

| Mindmap Module | In-Scope Features (MVP Release) | Deliberately Out-of-Scope (Phase 2 / Excluded) |
|---|---|---|
| **1. Catalog & SKUs (M01)** | • Product, Color/Size Variant SKU, and Supplier CRUD.<br>• Duplicate SKU/Barcode prevention.<br>• Soft SKU deactivation preserving transaction integrity. | • Bill of Materials (BOM) garment production.<br>• Multi-brand & multi-subsidiary enterprise structures. |
| **2. Inbound & Landed Cost (M02)** | • DRAFT PO creation, Dynamic SKU modal.<br>• Quantity-based freight allocation with penny rounding.<br>• Live modal preview.<br>• Atomic & idempotent PO confirmation (RECEIVE). | • Partial inbound receipts across multiple dates.<br>• Advanced supplier accounts payable (AP) ledger. |
| **3. Inventory & COGS (M03)** | • Stock movement ledger (Inbound/Outbound/Adjustment).<br>• Moving Weighted Average costing.<br>• Frozen COGS snapshot upon SHIPPED.<br>• Concurrency-safe overselling prevention. | • Multi-warehouse physical locations and bin routing.<br>• Expiration & shelf-life tracking. |
| **4. Order Management (M04)** | • Multi-channel ingestion (TikTok Shop, Shopee, FB POS).<br>• Multi-line item orders.<br>• State machine: `PENDING` $\rightarrow$ `SHIPPED` $\rightarrow$ `DELIVERED` / `CANCELLED`. | • Real-time live open API webhook connectors.<br>• Automated warehouse wave picking. |
| **5. Platform Fees (M05)** | • Estimated fee strategies for Shopee/TikTok/FB POS.<br>• Actual wallet settlement fee ingestion.<br>• Zero fee handling (Shop subsidized free shipping). | • Automated bulk Excel payout statement parsing.<br>• Direct corporate bank API integrations. |
| **6. Analytics & P&L (M06)** | • 5-step order waterfall profit ledger.<br>• Profit tier classification (High Margin, Low Margin, Loss).<br>• Executive dashboard with date & channel filters.<br>• Source order drillthrough & CSV export. | • Company-wide net profit including overheads (M09 - Rent, Payroll, Ads).<br>• Machine Learning sales forecasting. |
| **7. Administration (M07)** | • 3 internal roles (Owner, Ops, Finance).<br>• Audit trail for inbound receipts and outbound dispatches.<br>• Explicit `.env` database configuration. | • Single Sign-On (Google SSO, LDAP).<br>• Multi-tenant SaaS architecture. |
| **8. Return Triage (M08)** | *Excluded from MVP* | • Returned garment triage, inspection, restock, or scrap write-off (Scheduled for Phase 2). |
