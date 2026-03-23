# User Manual — Renewable Energy Financial Consolidation Model

This manual provides step-by-step instructions for setting up, populating, running, and interpreting the Excel-based financial consolidation model.

---

## Table of Contents

1. [Before You Begin](#1-before-you-begin)
2. [Workbook Overview](#2-workbook-overview)
3. [Initial Setup](#3-initial-setup)
4. [Entering Trial Balance Data](#4-entering-trial-balance-data)
5. [Running the Consolidation](#5-running-the-consolidation)
6. [Reviewing Validation Checks](#6-reviewing-validation-checks)
7. [Intercompany Eliminations](#7-intercompany-eliminations)
8. [Reading the Financial Statements](#8-reading-the-financial-statements)
9. [Using the Dashboard](#9-using-the-dashboard)
10. [Scenario Analysis (Budget / Actual / Forecast)](#10-scenario-analysis-budget--actual--forecast)
11. [Audit Trail](#11-audit-trail)
12. [Saving and Versioning](#12-saving-and-versioning)
13. [Troubleshooting](#13-troubleshooting)
14. [Glossary](#14-glossary)

---

## 1. Before You Begin

### System Requirements

- **Microsoft Excel 2016 or later** on Windows or macOS.
- Macros (VBA) must be **enabled**. When you open the workbook you will see a yellow security bar at the top of the screen — click **Enable Content**.
- No internet connection, database, or add-in is required.

### Enabling Macros

1. Open the workbook.
2. If a yellow bar appears at the top reading *"SECURITY WARNING: Macros have been disabled"*, click **Enable Content**.
3. If no bar appears and buttons do not respond, go to **File → Options → Trust Center → Trust Center Settings → Macro Settings** and select **Enable all macros**.

> ⚠️ Only open workbooks received from a trusted source. Enabling macros in unknown files is a security risk.

---

## 2. Workbook Overview

The workbook contains the following sheets (tabs), displayed in workflow order:

| Sheet | Colour | Purpose |
|---|---|---|
| `SETUP` | Blue | Entity list, currency, and period configuration |
| `TB_[Entity]` | Green | One trial balance input sheet per entity |
| `IC_MATRIX` | Orange | Intercompany balance matrix for elimination |
| `CONSOLIDATION` | Yellow | Aggregation engine and elimination entries |
| `CHECKS` | Red/Green | Validation results — must show all green before sign-off |
| `P&L` | White | Consolidated income statement |
| `BS` | White | Consolidated balance sheet |
| `CF` | White | Consolidated cash flow statement |
| `DASHBOARD` | Dark | Executive one-pager with charts and KPIs |
| `SCENARIOS` | Purple | Budget vs. Actual vs. Forecast comparison |
| `AUDIT_LOG` | Grey | Record of manual overrides |

---

## 3. Initial Setup

Open the **`SETUP`** sheet and complete each section before entering any trial balance data.

### 3.1 Reporting Period

| Cell | Field | Example |
|---|---|---|
| B2 | Reporting period label | `FY 2025` |
| B3 | Period start date | `01/01/2025` |
| B4 | Period end date | `31/12/2025` |
| B5 | Comparative period label | `FY 2024` |

### 3.2 Entity List

Fill in the entity table starting at row 9:

| Column | Field | Notes |
|---|---|---|
| A | Entity code | Short unique identifier, e.g. `SOLAR_UK` |
| B | Entity name | Full legal name |
| C | Functional currency | ISO 4217 code, e.g. `GBP`, `EUR`, `USD` |
| D | Ownership % | Percentage held by the group (0–100) |
| E | Consolidation method | `Full`, `Proportionate`, or `Equity` |
| F | Include in run | `YES` or `NO` — use `NO` to exclude temporarily |

### 3.3 Base Currency

Enter the group's presentation currency in cell B7 (e.g. `EUR`). Exchange rates for translating each entity's figures are entered in the `FX_RATES` table starting at row 30.

---

## 4. Entering Trial Balance Data

Each entity that is marked `YES` in the SETUP sheet will have its own **`TB_[EntityCode]`** sheet auto-generated. Navigate to the relevant sheet and fill in the trial balance.

### 4.1 Column Layout

| Column | Field | Notes |
|---|---|---|
| A | Account code | Must match the chart of accounts in the `COA` sheet |
| B | Account description | Free text — for reference only |
| C | Account type | `BS` (balance sheet) or `PL` (profit & loss) |
| D | Current period debit | Positive numbers only |
| E | Current period credit | Positive numbers only |
| F | Comparative period debit | Prior year figures |
| G | Comparative period credit | Prior year figures |
| H | Notes | Optional free-text annotation |

### 4.2 Entering Data

- Enter all amounts in the **entity's functional currency** (not the group currency — translation is automatic).
- Do **not** insert or delete rows. Use only the pre-formatted data entry rows (rows 5 onwards).
- Totals at the bottom of the sheet will automatically verify that debits equal credits.

### 4.3 Importing from an ERP

If your ERP can export a trial balance to CSV:
1. Open the CSV in a separate Excel window.
2. Copy the relevant columns.
3. Paste into the `TB_[EntityCode]` sheet using **Paste Special → Values only** (Ctrl+Shift+V) to avoid overwriting cell formatting.

---

## 5. Running the Consolidation

Once all trial balances are entered and the `CHECKS` sheet shows no errors in the input section:

1. Navigate to the **`CONSOLIDATION`** sheet.
2. Click the **"Run Consolidation"** button (top-left of the sheet).
3. A progress bar will appear while the macro aggregates entity data, applies FX translation, and posts elimination entries.
4. When complete, a dialog box will confirm success or list any blocking errors.

> 💡 The consolidation can be re-run at any time. It always overwrites the previous run's output.

---

## 6. Reviewing Validation Checks

Navigate to the **`CHECKS`** sheet after each consolidation run.

### Check Categories

| Section | Checks Performed |
|---|---|
| **Input Checks** | Each entity's TB balances (DR = CR); no blank account codes |
| **FX Checks** | Exchange rates present for all entity currencies |
| **Elimination Checks** | All intercompany balances are fully eliminated; net elimination = zero |
| **Output Checks** | Balance sheet balances (Assets = Liabilities + Equity); retained earnings movement reconciles |
| **Completeness Checks** | All `YES` entities have data entered; no period dates are missing |

### Status Indicators

- 🟢 **PASS** — Check passed. No action needed.
- 🔴 **FAIL** — Check failed. The adjacent cell contains a description of the problem and the sheet/cell reference to investigate.
- 🟡 **WARN** — Warning. The model can proceed, but the issue should be reviewed (e.g. a non-material rounding difference).

**The model should not be signed off until all FAIL statuses are resolved.**

---

## 7. Intercompany Eliminations

### 7.1 Entering Intercompany Balances

Navigate to the **`IC_MATRIX`** sheet. The matrix lists all entities on both axes.

- For each pair of entities that have intercompany balances, enter the gross receivable in the relevant cell.
- The matrix is symmetric — entering a receivable on one side automatically populates the payable on the other.

### 7.2 Common Elimination Entries

The model automatically generates elimination journal entries for:

- Intercompany receivables and payables
- Intercompany revenue and cost of sales
- Dividends paid/received between group entities
- Investment in subsidiary vs. equity of subsidiary (on acquisition)

### 7.3 Manual Elimination Adjustments

If a standard elimination entry needs to be modified:
1. Go to the **`CONSOLIDATION`** sheet.
2. Find the relevant elimination entry in the "Manual Adjustments" section (rows 200 onwards).
3. Enter the adjustment debit/credit and a mandatory description.
4. All manual entries are automatically logged in the `AUDIT_LOG` sheet.

---

## 8. Reading the Financial Statements

### 8.1 Income Statement (`P&L` sheet)

- Figures are presented in the group's presentation currency.
- Both current period and comparative period columns are shown.
- Subtotals (Gross Profit, EBITDA, EBIT, PBT, PAT) are auto-calculated.
- An entity-level split is available in the section below the consolidated totals.

### 8.2 Balance Sheet (`BS` sheet)

- Presented in the standard Assets / Liabilities / Equity format.
- The balance sheet check cell (top-right) must show **BALANCED** before sign-off.
- Non-controlling interests (NCI) are shown separately within equity.

### 8.3 Cash Flow Statement (`CF` sheet)

- Derived automatically using the indirect method from balance sheet movements and P&L.
- Three sections: Operating, Investing, Financing.
- Net change in cash reconciles to the movement in the cash line on the balance sheet.

---

## 9. Using the Dashboard

Navigate to the **`DASHBOARD`** sheet for a one-page executive summary.

### Available Visuals

| Visual | Description |
|---|---|
| Revenue waterfall | Bridge from prior year to current year revenue by entity |
| EBITDA margin trend | Monthly or quarterly margin progression |
| Debt / EBITDA gauge | Leverage ratio vs. covenant threshold |
| Entity contribution table | Each entity's % contribution to group revenue and EBITDA |
| FX sensitivity | Impact of ±5% move in each currency |

### Refreshing the Dashboard

Click the **"Refresh Dashboard"** button at the top of the sheet. Charts and KPIs update automatically from the financial statement sheets.

---

## 10. Scenario Analysis (Budget / Actual / Forecast)

Navigate to the **`SCENARIOS`** sheet.

1. In the **Scenario Selector** (cell B2), choose `Actual`, `Budget`, or `Forecast` from the dropdown.
2. The financial statement sheets and dashboard will switch to showing the selected scenario.
3. To enter budget or forecast figures, select the scenario in B2 and then enter data in the trial balance sheets as usual — the model keeps scenarios in separate memory columns.

---

## 11. Audit Trail

The **`AUDIT_LOG`** sheet automatically records:

- The date and time of each consolidation run.
- The user name (read from the Excel environment).
- Any manual adjustment entries with their descriptions.
- Any changes to the SETUP sheet (entity list or period dates).

Do not edit the `AUDIT_LOG` sheet directly. It is write-protected; only the macro can write to it.

---

## 12. Saving and Versioning

- Save the workbook as **`.xlsm`** (macro-enabled) to preserve VBA code. Saving as `.xlsx` will strip out all macros.
- Use a clear naming convention: `REFCM_[Period]_[Version]_[Status].xlsm`, e.g. `REFCM_FY2025_v3_DRAFT.xlsm`.
- Store versions in a shared document management system or version-controlled repository.
- Never overwrite a file that has been shared for review — always increment the version number.

---

## 13. Troubleshooting

| Symptom | Likely Cause | Resolution |
|---|---|---|
| Buttons do not respond | Macros are disabled | Enable macros (see Section 1) |
| `#REF!` errors on output sheets | Row was inserted or deleted in a TB sheet | Undo the structural change; use only pre-formatted rows |
| Balance sheet does not balance | Missing or incorrect elimination entry | Review `CHECKS` sheet for FAIL items |
| FX translation error | Exchange rate missing for an entity currency | Add the rate to the `FX_RATES` table in `SETUP` |
| Dashboard charts show no data | Consolidation has not been run | Click "Run Consolidation" on the `CONSOLIDATION` sheet |
| Consolidation run is very slow | Large number of entities with dense TB data | Close other workbooks and disable screen updating in Excel options |

---

## 14. Glossary

| Term | Definition |
|---|---|
| **Trial Balance (TB)** | A list of all general ledger account balances at a point in time, with debits and credits |
| **Intercompany (IC)** | Transactions or balances between two entities within the same group |
| **Elimination** | Journal entry that removes the effect of intercompany transactions from the consolidated view |
| **Functional Currency** | The currency of the primary economic environment in which an entity operates |
| **Presentation Currency** | The currency in which the consolidated financial statements are presented |
| **NCI** | Non-Controlling Interest — the portion of a subsidiary not owned by the parent |
| **EBITDA** | Earnings Before Interest, Tax, Depreciation, and Amortisation |
| **FX Translation** | Converting an entity's figures from its functional currency to the group presentation currency |
| **IFRS** | International Financial Reporting Standards |
| **GAAP** | Generally Accepted Accounting Principles |
