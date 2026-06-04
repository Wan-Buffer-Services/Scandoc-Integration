# ScanDoc Integration

> Production-ready OCR & document processing for Odoo — scan vendor bills into Purchase Orders, parse resumes into HR Applicants, process delivery challans on stock pickings, and turn unstructured documents into structured Odoo records, all powered by the external ScanDoc API.

Repository: https://github.com/Wan-Buffer-Services/Scandoc-Integration.git
Odoo module: `wb_scandoc_integration`

## Overview

**ScanDoc Integration** is an enterprise-grade document-management add-on that connects Odoo to the ScanDoc OCR service and uses the extracted data to create or enrich the right Odoo records — automatically. Drop a vendor bill PDF and the module can produce a draft Purchase Order. Upload a candidate's CV and it parses contact details and skills into an HR Applicant. Scan a delivery challan and it lines up the stock picking. Multi-currency, batch-capable, with explicit input validation, file-type/size checks, structured logging, and graceful error recovery.

It also ships dashboards so processing volumes and outcomes are visible at a glance.

## Key Features

- **OCR processing** for invoices, receipts and general documents, via the external ScanDoc service.
- **Vendor Bill → Purchase Order**: scan a vendor bill, generate a draft PO (`models/purchase_order.py`).
- **Resume parser**: extract candidate details and skills from CVs into HR Applicants (`wizards/resume_parser_wizard.py`, `models/hr_applicant.py`, depends on `hr_recruitment` + `hr_skills`).
- **Delivery challan / stock picking** document scanning (`models/stock_picking.py`).
- **Automatic currency detection & activation**: currencies referenced in documents are auto-activated on the company.
- **CRM-side cron** (`data/cron_crm_lead.xml`) for scheduled processing of CRM leads.
- **Dashboards**: dynamic + sentiment dashboards (`static/src/js/dynamic_dashboard.js`, `sentiment_dashboard.js`) visualize processing activity and outcomes.
- **Production hardening**: input validation, file-type & size limits, SQL-injection guards, optimized queries, structured logging, recoverable error handling.
- **Configurable feature toggles**: enable/disable Resume, Vendor Bill, and Delivery Challan scanning independently from Settings.
- **External Auth Key**: authenticated via a ScanDoc API key, validated against the ScanDoc service from the Settings page.

## Module Info

| | |
|---|---|
| Module | `wb_scandoc_integration` |
| Display name | ScanDoc Integration |
| Category | Document Management / OCR |
| Version | 18.0.1.0.0 |
| License | OPL-1 |
| Depends on | `base`, `base_setup`, `mail`, `purchase`, `stock`, `hr_recruitment`, `hr_skills`, `web`, `crm` |
| Author / Support | Wan Buffer Services — support@wanbuffer.com |

## Requirements

System / Python packages used by the OCR pipeline:

- **System**: `tesseract-ocr` (e.g. `sudo apt install tesseract-ocr`)
- **Python**: `requests`, `openpyxl`, `pytesseract`, `PyPDF2`, `Pillow`

Install the Python dependencies into your Odoo virtualenv:

```bash
pip install requests openpyxl pytesseract PyPDF2 Pillow
```

A **ScanDoc API account / Auth Key** is also required — generate one from the ScanDoc auth URL exposed by the module (see *Configuration*).

## Installation

1. Clone the repository into your Odoo addons path:
   ```bash
   git clone https://github.com/Wan-Buffer-Services/Scandoc-Integration.git
   ```
2. Install the Python + system requirements (see above).
3. Update the apps list and install **ScanDoc Integration**, or via CLI:
   ```bash
   odoo-bin -c <conf> -d <db> -i wb_scandoc_integration
   ```

## Configuration

Open **Settings → ScanDoc Integration** (added by `views/res_config_settings.xml`) and configure:

1. **ScanDoc Auth Key** — paste the API key. The module validates it against the ScanDoc service before saving.
2. **Generate Auth Key** — link/button that redirects to the ScanDoc auth-key generation endpoint (see `views/scandoc_auth_action.xml`).
3. **Feature toggles**:
   - **Resume Parsing** — enable to allow CV uploads on the Applicant form.
   - **Vendor Bill Scanning** — enable to allow vendor-bill OCR → Purchase Order generation.
   - **Delivery Challan Scanning** — enable to allow stock-picking document processing.

Settings are persisted via `ir.config_parameter` so they survive upgrades.

## Usage

**Resume Parsing**

1. Recruitment → an Applicant form → open the **Resume Parser** wizard (`wizards/resume_parser_wizard.py`).
2. Upload the CV. ScanDoc returns the structured fields; the wizard fills name, contact and skill records on the applicant.

**Vendor Bill → Purchase Order**

1. Purchase → Vendor Bills (or the ScanDoc menu).
2. Upload the bill scan; on success a draft Purchase Order is created from the extracted vendor / line items, ready for review and confirmation (`models/purchase_order.py`).

**Delivery Challan / Stock Picking**

1. Inventory → Transfers → an internal/incoming picking.
2. Use the ScanDoc action to upload a challan PDF; the picking is updated from the OCR output (`models/stock_picking.py`).

**Dashboards**

The ScanDoc menu (`views/menu_views.xml`) exposes dynamic and sentiment dashboards so administrators can track processing volume, success rates and per-document-type breakdowns.

## Security & Reliability

- File-type whitelist and size limit checks before forwarding to OCR.
- SQL-injection-safe ORM usage throughout.
- Structured `_logger` output makes failures debuggable in production.
- Errors during OCR / API calls fall back gracefully — the source record is preserved and the failure is recorded.

## Support

Wan Buffer Services — https://wanbuffer.com · support@wanbuffer.com · +91 9638442270
