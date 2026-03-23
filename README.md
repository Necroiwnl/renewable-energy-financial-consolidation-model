# Renewable Energy Financial Consolidation Model

Production-ready Excel-based financial consolidation model with multi-entity aggregation, validation engine, and full traceability from trial balance to dashboard.

---

## Project Overview

This model is designed for renewable energy companies that need to consolidate financial data across multiple entities (e.g., solar farms, wind parks, holding companies) into a single group-level view. It provides a structured, auditable workflow from raw trial balance inputs to executive dashboards, without requiring any programming knowledge.

The workbook is self-contained: all data entry, calculations, validation checks, and reporting live within a single Excel file, making it easy to distribute, version-control, and audit.

---

## Features

| Feature | Description |
|---|---|
| **Multi-Entity Aggregation** | Consolidate up to 20 legal entities across currencies and reporting periods |
| **Automated Intercompany Elimination** | Flag and eliminate intercompany balances and transactions automatically |
| **Validation Engine** | Built-in checks ensure debits equal credits, eliminations net to zero, and no data is missing |
| **Trial Balance Import** | Structured input sheets accept data exported from any ERP or accounting system |
| **Income Statement & Balance Sheet** | Auto-populated consolidated financial statements in IFRS/GAAP layout |
| **Cash Flow Statement** | Indirect-method cash flow derived from balance sheet movements |
| **Dashboard** | One-page visual summary with KPIs, waterfall charts, and variance analysis |
| **Full Traceability** | Every consolidated line item traces back to its source entity and account |
| **Scenario Support** | Budget vs. Actual vs. Forecast comparison in a single workbook |
| **Audit Trail** | Change-log sheet records manual overrides with timestamp and user notes |

---

## Repository Structure

```
renewable-energy-financial-consolidation-model/
├── README.md                  # This file — project overview and quick-start
├── USER_MANUAL.md             # Step-by-step operating instructions
├── LICENSE
├── docs/
│   ├── validation_report.md   # Validation rules and expected check results
│   └── testing_report.md      # Test cases and acceptance criteria
└── screenshots/
    └── .gitkeep               # Placeholder — screenshots added during release
```

---

## Quick Start

1. **Download** the latest workbook from the [Releases](../../releases) page.
2. **Enable macros** when prompted (required for the validation engine and dashboard refresh).
3. Open the **`SETUP`** sheet and fill in your entity list, base currency, and reporting period.
4. Paste or type each entity's trial balance into the corresponding **`TB_[EntityName]`** input sheet.
5. Navigate to the **`CONSOLIDATION`** sheet and click **Run Consolidation**.
6. Review any red validation flags on the **`CHECKS`** sheet and resolve them.
7. View the final output on the **`P&L`**, **`BS`**, **`CF`**, and **`DASHBOARD`** sheets.

For full instructions, see [USER_MANUAL.md](USER_MANUAL.md).

---

## Requirements

- Microsoft Excel 2016 or later (Windows or macOS)
- Macros must be enabled
- No additional software, add-ins, or database connections required

---

## Documentation

| Document | Purpose |
|---|---|
| [USER_MANUAL.md](USER_MANUAL.md) | Detailed step-by-step instructions for all users |
| [docs/validation_report.md](docs/validation_report.md) | Description of all built-in validation rules |
| [docs/testing_report.md](docs/testing_report.md) | Test cases used to verify model accuracy |

---

## License

This project is licensed under the terms of the [LICENSE](LICENSE) file included in this repository.

