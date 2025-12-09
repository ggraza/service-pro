## Overview

**Service Pro** is a comprehensive Frappe/ERPNext application designed for managing hydraulic cylinder service and manufacturing operations. It provides end-to-end management of workshop services, manufacturing processes, on-site field services, sales with margin control, inter-company stock movements, and financial tracking.

| Property | Value |
|----------|-------|
| App Name | service_pro |
| App Title | Service Pro |
| Module | Service Pro |
| License | MIT |
| Framework | Frappe Framework / ERPNext |

---

## Application Purpose

Service Pro is built for businesses that:
- **Service and repair hydraulic cylinders** - Complete workshop service management
- **Manufacture/assemble hydraulic equipment** - Production and assembly operations
- **Manage workshop and on-site field services** - Both in-shop and mobile service tracking
- **Handle sales operations** - Products and services with margin controls
- **Manage inter-company operations** - Stock transfers between multiple companies

### Post-Installation
After installation, the following fixtures are automatically loaded:
- Custom Fields
- Server Scripts
- Property Setters
- Client Scripts

## Architecture

### Directory Structure

```
service_pro/
├── service_pro/                    # Main module directory
│   ├── doctype/                   # 90 DocType definitions
│   │   ├── production/            # Production management
│   │   ├── inspection/            # Cylinder inspection
│   │   ├── estimation/            # Cost estimation
│   │   ├── service_order_form/    # Service orders
│   │   ├── service_receipt_note/  # Receipt notes
│   │   ├── inter_company_stock_transfer/
│   │   └── ... (86 more)
│   ├── doc_events/                # Document event handlers
│   │   ├── sales_invoice.py
│   │   ├── sales_order.py
│   │   ├── delivery_note.py
│   │   ├── quotation.py
│   │   ├── material_request.py
│   │   └── dashboard/
│   ├── public/js/                 # Client-side JavaScript (19 files)
│   ├── config/                    # Module configuration
│   ├── workspace/                 # Workspace definitions
│   ├── templates/                 # HTML templates
│   ├── overrides/                 # Report overrides
│   └── fixtures/                  # Custom fields, scripts
├── hooks.py                       # Application hooks
└── README.md
```

### DocType Categories

| Category | DocTypes | Purpose |
|----------|----------|---------|
| Service Workflow | Service Receipt Note, Inspection, Estimation, Production | Core service lifecycle |
| Production | Production, Raw Material, Scoop of Work, Additional Cost | Manufacturing management |
| Site Service | Site Visit Report, Site Job Report, Job Completion Report | Field service operations |
| Inter-Company | Inter Company Stock Transfer, Material Request, Journal Entry | Multi-company operations |
| Financial | Payment Scheduler, Sales Partner Payments, Petty Cash Request | Financial management |
| Configuration | Production Settings, Inspection Settings, Cylinder Dimensions | System configuration |

---

## Core DocTypes

### 1. Service Receipt Note (SRN)

The entry point for workshop services when a customer brings items for service.

**Key Fields:**
- `company` - Operating company
- `customer` - Customer link
- `project` - Associated project
- `materials` - Table of items received (Service Receipt Note Item)
- `status` - Workflow status (To Inspection, To Estimation, To Production)
- `posting_date` - Date of receipt
- `sales_team` - Associated sales team

**Statuses:**
- To Inspection
- To Estimation
- To Production
- Closed

---

### 2. Inspection

Detailed technical inspection of hydraulic cylinders with comprehensive image documentation.

**Key Features:**
- **40+ image attachment fields** for documenting cylinder components
- Parts inspection checklist
- Size specifications (Piston Rod, Tube)
- Road condition assessment

**Assessment Sections:**
| Section | Description |
|---------|-------------|
| Arrived/Offloaded Cylinder | Initial condition photos |
| Stripping from Barrel | Disassembly documentation |
| Seal Kit Assessment | Seal condition evaluation |
| Rod Assessment | Piston rod condition |
| Tube Assessment | Cylinder tube condition |
| Gland Assessment | Gland component evaluation |
| Sleeve Assessment | Sleeve condition |
| Piston Assessment | Piston condition |
| Clevis Assessment | Clevis component |
| Counter Balance Valve | Valve assessment |
| Manifold Block | Block assessment |
| Piston Rod Thread | Thread condition |
| Clevis Bush | Bush condition |
| O-Ring Groove | Groove assessment |

**Linked DocTypes:**
- `Service Receipt Note` (parent reference)
- `Parts Inspection` (child table)
- `Inspection Checklist` (child table)
- `Road Condition` (child tables for piston and tube)

---

### 3. Estimation

Cost estimation for repair and service work based on inspection findings.

**Purpose:**
- Calculate workshop service costs
- Prepare quotations
- Create production orders

**Key Methods:**
- `create_production()` - Convert estimation to production order
- `calculate_cost()` - Compute total service cost
- Rate fetching from price lists

---

### 4. Production

Central DocType for managing manufacturing, assembly, and service work orders.

**Production Types:**
| Type | Description |
|------|-------------|
| Assemble | Building new products |
| Disassemble | Breaking down products |
| Service | Servicing existing items |
| Re-Service | Re-servicing previously serviced items |

**Key Fields:**
- `company` - Operating company
- `customer` - Customer link
- `type` - Production type (Assemble/Disassemble/Service/Re-Service)
- `status` - Production status
- `item_code_prod` - Finished item code
- `raw_material` - Raw materials table
- `scoop_of_work` - Labor/machine cost table
- `additional_cost` - Additional expenses table
- `advance_payment` - Advance payment table

**Tabs:**
1. **Details** - Basic information, customer, dates
2. **Finished Item** - Output item details, warehouse, rates
3. **Raw Materials** - Input materials with costs
4. **Scoop of Work** - Labor and machine costs
5. **Advance** - Advance payments
6. **Additional Cost** - Extra expenses

**Key Methods:**
- `generate_item()` - Auto-create finished goods item
- `generate_se()` - Create stock entry for manufacturing
- `generate_si()` - Create sales invoice
- `generate_dn()` - Create delivery note
- `generate_so()` - Create sales order
- `generate_jv()` - Create journal entry for advances
- `calculate_total_selling_rate()` - Calculate selling prices
- `set_available_qty()` - Check stock availability
- `get_valuation_rate_from_sle()` - Get valuation from Stock Ledger

**Status Flow:**
```
In Progress → Partially Delivered → Completed
           → Partially Sales Order → To Deliver and Bill
           → To Deliver → To Bill → Closed
```

---

### 5. Service Order Form (SOF)

Custom quotation system for service operations.

**Key Features:**
- Service-specific quotation format
- Automatic expiry checking (daily scheduler)
- Conversion to Sales Order
- Money-in-words calculation

**Key Methods:**
- `create_sales_order()` - Convert SOF to Sales Order
- `check_service_order_expiry()` - Daily scheduler to expire old orders

---

### 6. Inter Company Stock Transfer

Manages stock transfers between multiple companies.

**Workflow:**
1. Create transfer request
2. Generate stock entry from source company (Issue)
3. Track in transit warehouse
4. Generate stock entry at destination (Receipt)
5. Create inter-company journal entry

**Related DocTypes:**
- Inter Company Stock Transfer Item
- Inter Company Stock Transfer From
- Inter Company Stock Transfer To
- Inter Company Stock Transfer Template

---

### 7. Site Visit Report

Track field service visits to customer locations.

**Key Fields:**
- Site visit details
- Job cards generation
- Team members assignment
- Job completion tracking

---

### 8. Site Job Report

On-site job tracking and documentation.

**Key Fields:**
- Site job scope of work
- Job status tracking
- Production linking

---

### 9. Payment Scheduler

Payment scheduling with interest calculation.

**Features:**
- Payment schedule details
- Interest calculation
- Payment tracking

---

### 10. Sales Partner Payments

Commission and incentive payment management.

**Features:**
- Sales partner commission tracking
- Paid/unpaid status
- Incentive calculations

---

## Workflows

### 1. Workshop Service Workflow

```
┌─────────────────────────────────────────────────────────────────────┐
│                    WORKSHOP SERVICE WORKFLOW                         │
└─────────────────────────────────────────────────────────────────────┘

Customer brings item
        │
        ▼
┌───────────────────┐
│ Service Receipt   │  ← Create SRN with item details
│ Note (SRN)        │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ Inspection        │  ← Detailed inspection with 40+ images
│                   │    Parts assessment, size documentation
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ Estimation        │  ← Cost calculation for repairs
│                   │    Material and labor costs
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ Quotation         │  ← Customer approval
│                   │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ Sales Order       │  ← Order confirmation
│                   │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ Production        │  ← Service type production
│ (Type: Service)   │    Raw material consumption
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ Stock Entry       │  ← Material consumption entry
│                   │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ Delivery Note     │  ← Deliver to customer
│                   │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ Sales Invoice     │  ← Invoice and payment
│                   │
└─────────┴─────────┘
          │
          ▼
     Status: Completed
```

### 2. Manufacturing Workflow

```
┌─────────────────────────────────────────────────────────────────────┐
│                    MANUFACTURING WORKFLOW                            │
└─────────────────────────────────────────────────────────────────────┘

Sales Order
     │
     ▼
┌───────────────────┐
│ Production        │  ← Type: Assemble
│ (Assemble)        │
└─────────┬─────────┘
     │
     ├─────────────────────────────────┐
     │                                 │
     ▼                                 ▼
┌───────────────────┐         ┌───────────────────┐
│ Raw Materials     │         │ Scoop of Work     │
│ (Add items)       │         │ (Labor/Machine)   │
└─────────┬─────────┘         └─────────┬─────────┘
     │                                 │
     └─────────────────┬───────────────┘
                       │
                       ▼
              ┌───────────────────┐
              │ Stock Entry       │  ← Manufacture type
              │ (Manufacture)     │    Consume raw materials
              └─────────┬─────────┘
                       │
                       ▼
              ┌───────────────────┐
              │ Finished Good     │  ← Create finished item
              │ Created           │
              └─────────┬─────────┘
                       │
                       ▼
              ┌───────────────────┐
              │ Delivery Note     │
              │                   │
              └─────────┬─────────┘
                       │
                       ▼
              ┌───────────────────┐
              │ Sales Invoice     │
              │                   │
              └─────────┴─────────┘
```

### 3. Site Service Workflow

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SITE SERVICE WORKFLOW                             │
└─────────────────────────────────────────────────────────────────────┘

Customer Request
     │
     ▼
┌───────────────────┐
│ Site Visit        │  ← Schedule site visit
│ Report            │    Team assignment
└─────────┬─────────┘
     │
     ▼
┌───────────────────┐
│ Job Card          │  ← Generate job cards
│ Generated         │
└─────────┬─────────┘
     │
     ▼
┌───────────────────┐
│ Site Job Report   │  ← Track on-site work
│                   │    Scope of work
└─────────┬─────────┘
     │
     ▼
┌───────────────────┐
│ Production        │  ← Create if needed
│ (Service type)    │
└─────────┬─────────┘
     │
     ▼
┌───────────────────┐
│ Job Completion    │  ← Document completion
│ Report            │
└─────────┬─────────┘
     │
     ▼
┌───────────────────┐
│ Invoicing         │
│                   │
└─────────┴─────────┘
```

### 4. Inter-Company Transfer Workflow

```
┌─────────────────────────────────────────────────────────────────────┐
│                    INTER-COMPANY TRANSFER WORKFLOW                   │
└─────────────────────────────────────────────────────────────────────┘

┌───────────────────┐
│ Inter Company     │  ← Request materials from another company
│ Material Request  │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│ Inter Company     │  ← Create transfer document
│ Stock Transfer    │
└─────────┬─────────┘
          │
          ├─────────────────────────────────────────────┐
          │                                             │
          ▼                                             ▼
┌───────────────────┐                         ┌───────────────────┐
│ Stock Entry       │                         │ In Transit        │
│ (From Company)    │  ────────────────────►  │ Warehouse         │
│ Material Issue    │                         │                   │
└───────────────────┘                         └─────────┬─────────┘
                                                        │
                                                        ▼
                                              ┌───────────────────┐
                                              │ Stock Entry       │
                                              │ (To Company)      │
                                              │ Material Receipt  │
                                              └─────────┬─────────┘
                                                        │
                                                        ▼
                                              ┌───────────────────┐
                                              │ Inter Company     │
                                              │ Journal Entry     │
                                              └───────────────────┘
```

---

## Features

### 1. Workshop Service Management

- **Complete Service Lifecycle**: Receipt → Inspection → Estimation → Production → Delivery → Invoicing
- **Detailed Inspection**: 40+ image attachment fields for comprehensive cylinder component documentation
- **Component Assessment**:
  - Gland, Piston, Rod, Tube, Sleeve
  - Clevis, Counter Balance Valve, Manifold Block
  - Piston Rod Thread, Clevis Bush, O-Ring Groove
- **Automatic Status Updates**: Status propagation across related documents

### 2. Production Management

- **Multiple Production Types**: Assemble, Disassemble, Service, Re-Service
- **Automatic Item Generation**: Create finished goods items automatically
- **Raw Material Tracking**: Complete material consumption tracking
- **Scoop of Work**: Labor and machine cost tracking
- **Additional Cost Management**: Track extra expenses
- **Stock Entry Generation**: Automatic stock entries for material consumption
- **Batch Tracking Support**: Track materials by batch
- **Valuation Rate Calculations**: Pull rates from Stock Ledger Entries

### 3. Sales & Margin Management

- **Service Order Form**: Custom quotation system for services
- **User-Specific Margins**: Enforce minimum margin rates per user
- **Margin Rate Validation**: Prevent selling below minimum margins
- **Role-Based Approvals**: Margin approver system
- **Automatic Calculations**: Margin computation from valuation rates
- **Sales Partner Incentives**: Track paid/unpaid incentives

### 4. Site Service Operations

- **Site Visit Tracking**: Plan and track field service visits
- **Job Card Generation**: Create job cards for field work
- **Job Completion Reports**: Document job completion
- **Mobile Support**: Support for mobile field service

### 5. Inter-Company Operations

- **Stock Transfer**: Transfer stock between companies
- **In-Transit Warehouses**: Track goods in transit
- **Material Requests**: Request materials across companies
- **Automatic Journal Entries**: Create inter-company accounting entries
- **Value Reconciliation**: Debit/Credit value matching

### 6. Financial Features

- **Payment Scheduling**: Schedule payments with interest calculation
- **Sales Partner Commissions**: Manage commission payments
- **Agent Payment Requests**: Process agent payments
- **Petty Cash Management**: Track petty cash requests
- **Automatic Journal Entries**: Generate accounting entries
- **Related Party Transactions**: Track related party transactions

---

## API Reference

### Whitelisted Methods

The application exposes 50+ whitelisted API methods:

#### Production Methods

```python
# Get item rate
@frappe.whitelist()
def get_rate(item_code, price_list=None):
    """Fetch item rate from price list or valuation"""

# Get available quantity
@frappe.whitelist()
def get_available_qty(item_code, warehouse):
    """Check stock availability"""

# Compute selling price
@frappe.whitelist()
def compute_selling_price(item_code, margin_rate):
    """Calculate selling price with margin"""

# Create delivery note
@frappe.whitelist()
def create_delivery_note(production):
    """Create DN from production"""
```

#### Estimation Methods

```python
@frappe.whitelist()
def get_estimation_raw_material(estimation):
    """Fetch raw materials from estimation"""
```

#### Service Order Form Methods

```python
@frappe.whitelist()
def create_sales_order(sof_name):
    """Convert Service Order Form to Sales Order"""
```

### Document Event Hooks

#### Sales Invoice Events

| Event | Method | Purpose |
|-------|--------|---------|
| on_submit | `on_submit_si` | Sales partner payments, production status |
| onload | `set_margin_rate_on_load` | Set user-specific margins |
| before_save | `before_save_si` | Pre-save validations |
| validate | `validate_margin_rate_with_rate` | Enforce margin rules |
| validate | `validate_and_calculate_rates` | Rate calculations |
| before_submit | `validate_so`, `validate_permission` | Pre-submit checks |
| on_cancel | `on_cancel_si` | Handle cancellation |

#### Sales Order Events

| Event | Method | Purpose |
|-------|--------|---------|
| on_submit | `create_qr_code` | Generate QR codes |
| on_submit | `sales_order_submit` | Post-submission logic |
| before_submit | `validate_permission` | Permission checks |

#### Delivery Note Events

| Event | Method | Purpose |
|-------|--------|---------|
| on_submit | `change_status` | Update production status |
| on_cancel | `change_status_cancel` | Handle cancellation |
| before_insert | `set_production_reference` | Link to production |

#### Other Events

| DocType | Event | Method |
|---------|-------|--------|
| Journal Entry | on_submit | `submit_jv` |
| Journal Entry | on_cancel | `cancel_related_party_entry` |
| Stock Ledger Entry | on_submit | `update_item_valuation_rate` |
| Quotation | on_submit | `on_submit_quotation` |
| Quotation | validate | `validate_item` |
| Purchase Receipt | before_submit | `update_supplier_packing_slip` |

---

## Client-Side Customizations

### JavaScript Files (19 files)

| File | Purpose | Size |
|------|---------|------|
| `sales_invoice.js` | Extensive SI customizations | 19KB |
| `quotation.js` | Quotation form logic | - |
| `sales_order.js` | Sales order customizations | - |
| `delivery_note.js` | Delivery note logic | - |
| `purchase_order.js` | Purchase customizations | - |
| `material_request.js` | Material request logic | - |
| `item.js` | Item master customizations | - |
| `employee.js` | Employee customizations | - |
| `default_value.js` | Default value population | - |
| `customer.js` | Customer form logic | - |
| `purchase_receipt.js` | Purchase receipt logic | - |
| `additional_salary.js` | Additional salary logic | - |
| `landed_cost_voucher.js` | LCV customizations | - |
| `project.js` | Project customizations | - |
| `supplier_quotation.js` | Supplier quotation logic | - |
| `delivery_trip.js` | Delivery trip customizations | - |
| `purchase_invoice.js` | Purchase invoice logic | - |

### Key UI Features

- **Custom Buttons**: Generate SE, DN, SI, SO, JV from Production
- **Automatic Rate Calculation**: Based on user margin settings
- **Interactive Tables**: Auto-calculations in child tables
- **Image Upload**: 40+ fields for inspection images
- **QR Code Generation**: For sales orders
- **Dashboard Customizations**: Custom dashboard views
- **Custom Filters**: Enhanced query filters

### DocType JavaScript Includes

```python
doctype_js = {
    "Quotation": "public/js/quotation.js",
    "Customer": "public/js/customer.js",
    "Sales Invoice": "public/js/sales_invoice.js",
    "Sales Order": ["public/js/sales_order.js", "public/js/default_value.js"],
    "Purchase Receipt": "public/js/purchase_receipt.js",
    "Material Request": "public/js/material_request.js",
    "Delivery Note": ["public/js/delivery_note.js", "public/js/default_value.js"],
    "Additional Salary": "public/js/additional_salary.js",
    "Landed Cost Voucher": "public/js/landed_cost_voucher.js",
    "Item": "public/js/item.js",
    "Employee": "public/js/employee.js",
    "Purchase Invoice": ["public/js/purchase_invoice.js", "public/js/default_value.js"],
    "Project": "public/js/project.js",
    "Purchase Order": "public/js/purchase_order.js",
    "Supplier Quotation": "public/js/supplier_quotation.js",
    "Delivery Trip": "public/js/delivery_trip.js"
}
```

---

## Configuration

### Production Settings

Global settings for production module configuration.

**Settings Include:**
- Default warehouses
- Default income accounts
- Series configuration
- Default cost centers

### Inspection Settings

Configuration for inspection module.

**Settings Include:**
- Default checklist items
- Terms and conditions
- Image requirements

### Cylinder Dimensions

Standard cylinder dimension definitions.

**Fields:**
- Size specifications
- Rod diameters
- Tube sizes
- Length specifications

### Default Sales Margin Percentage

User-specific margin settings.

**Purpose:**
- Set minimum margins per user
- Control selling price calculations
- Enforce pricing policies

---

## Scheduled Tasks

### Daily Tasks

```python
scheduler_events = {
    "daily": [
        "service_pro.service_pro.doctype.service_order_form.service_order_form.check_service_order_expiry",
        "service_pro.doc_events.utils.delete_old_logs"
    ],
}
```

| Task | Method | Purpose |
|------|--------|---------|
| Check SOF Expiry | `check_service_order_expiry` | Expire old service order forms |
| Delete Old Logs | `delete_old_logs` | Clean up old log entries |

---

## Report Overrides

### Accounts Receivable

Custom report override for accounts receivable with:
- Custom HTML template
- Custom JavaScript
- Custom Python logic

**Files:**
- `overrides/reports/html/accounts_receivable.html`
- `overrides/reports/js/accounts_receivable.js`
- `service_pro.overrides.reports.account_receivable.execute`

### Accounts Payable

Custom report override for accounts payable with:
- Custom HTML template
- Custom JavaScript
- Custom Python logic

**Files:**
- `overrides/reports/html/accounts_payable.html`
- `overrides/reports/js/accounts_payable.js`
- `service_pro.overrides.reports.account_payable.execute`

### Statement of Accounts

Custom templates for statement of accounts:
- `templates/process_statement_of_accounts.html` (General Ledger)
- `templates/process_statement_of_accounts_accounts_receivable.html` (AR)

---

## Dashboard Overrides

The application overrides several standard dashboards:

```python
override_doctype_dashboards = {
    "Stock Entry": "service_pro.doc_events.stock_entry_dashboard.get_stock_entry_data",
    "Purchase Order": "service_pro.doc_events.dashboard.dashboard.purchase_order_dashboard",
    "Purchase Receipt": "service_pro.doc_events.dashboard.dashboard.purchase_receipt",
    "Project": "service_pro.doc_events.dashboard.project_dashboard.project_dashboard",
    "Sales Order": "service_pro.doc_events.dashboard.dashboard.sales_order_dashboard",
    "Quotation": "service_pro.doc_events.dashboard.quotation_dashboard.quotation_dashboard",
}
```

---

## Method Overrides

### Material Request to Purchase Order

```python
override_whitelisted_methods = {
    "erpnext.stock.doctype.material_request.material_request.make_purchase_order":
        "service_pro.doc_events.material_request.make_purchase_order",
    "frappe.desk.query_report.get_script":
        "service_pro.overrides.report.get_script",
}
```

---

## Fixtures

The application exports the following fixtures:

```python
fixtures = [
    {
        "doctype": "Custom Field",
        "filters": [["module", "in", "Service Pro"]]
    },
    {
        "dt": "Server Script",
        "filters": [["module", "=", "Service Pro"]]
    },
    {
        "doctype": "Property Setter",
        "filters": [["module", "in", "Service Pro"]]
    },
    {
        "doctype": "Client Script",
        "filters": [["module", "in", "Service Pro"]]
    },
]
```

---

## Complete DocType List (90 DocTypes)

### Core DocTypes
1. Production
2. Inspection
3. Estimation
4. Service Receipt Note
5. Service Order Form

### Child Tables
6. Raw Material
7. Scoop of Work
8. Additional Cost
9. Advance Payment
10. Service Receipt Note Item
11. Service Order Form Item
12. Parts Inspection
13. Inspection Checklist
14. Inspection Table
15. Road Condition
16. Linked Productions
17. Item Selling Price List
18. Inspection Other Components

### Site Service
19. Site Visit Report
20. Site Visit Report Jobs
21. Site Job Report
22. Site Job Report Settings
23. Site Job SOW
24. Job Completion Report

### Inter-Company
25. Inter Company Stock Transfer
26. Inter Company Stock Transfer Item
27. Inter Company Stock Transfer From
28. Inter Company Stock Transfer To
29. Inter Company Stock Transfer Template
30. Inter Company Material Request
31. Inter Company Material Request Item
32. Inter Company Journal Entry
33. Inter Company Journal Entry Table
34. Inter Company Transfer

### Financial
35. Payment Scheduler
36. Payment Scheduler Details
37. Payment Scheduler Details Interest
38. Sales Partner Payments
39. Sales Partner Payments Details
40. Agent Payment Request
41. Agent Payment Request Table
42. Agent Payment Request Defaults
43. Petty Cash Request
44. Related Party Entry
45. Related Entry Account
46. Incentive Journal

### Configuration
47. Production Settings
48. Inspection Settings
49. Cylinder Dimensions
50. Size
51. Tube Size
52. Rod Dia
53. R Length
54. T Length
55. Machine
56. Worker
57. Work
58. Material
59. Condition

### Item Related
60. Cylinder Item
61. Item Specification
62. Item Size Specification
63. Item Pricing Entry
64. Pricing Item Group
65. Finish Good Defaults
66. Raw Material Defaults

### Sales
67. Default Sales Margin Percentage
68. Maximum User Discount
69. Target Item
70. Tax Template
71. Tax Charges

### HR
72. Employee Rejoining
73. Salary Certificate
74. Salary Certificate Template
75. Salary Seal
76. Team Members

### Supplier
77. Supplier Packing Slip
78. Supplier Packing Slip Item

### Company
79. Company Bank Detailes
80. Company Expense Ledger
81. Bank Details
82. Total Expense

### Other
83. Defaults Session
84. Defaults Session Settings
85. Repacking
86. Repack Item
87. Sales Invoice Production
88. Workshop Details
89. Other Check List

---

## Support

For issues and feature requests, contact:
- Email: janlloydangeles@gmail.com
- Report issues in the application's issue tracker

---

## License

MIT License

---

*Documentation generated for Service Pro v15.01*

