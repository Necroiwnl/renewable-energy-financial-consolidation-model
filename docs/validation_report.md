# Validation Report — Renewable Energy Financial Consolidation Model

This document describes every built-in validation rule in the model, the logic behind it, the sheet and cells where it is evaluated, and the expected result when all inputs are correct.

---

## Overview

The validation engine runs automatically every time the **Run Consolidation** button is clicked. Results are written to the **`CHECKS`** sheet. All checks must show **PASS** or **WARN** status before the consolidated financial statements are considered reliable for sign-off.

Checks are grouped into five categories:

1. Input Checks
2. FX Checks
3. Elimination Checks
4. Output Checks
5. Completeness Checks

---

## 1. Input Checks

### 1.1 Trial Balance in Balance

| Field | Detail |
|---|---|
| **Rule ID** | INP-001 |
| **Description** | For each entity's trial balance sheet, total debits must equal total credits. |
| **Formula logic** | `SUM(Debit column) = SUM(Credit column)` for both current and comparative periods. |
| **Tolerance** | Zero — any non-zero difference is a FAIL. |
| **CHECKS sheet row** | One row per entity. |
| **FAIL resolution** | Open the relevant `TB_[EntityCode]` sheet, review the debit/credit totals at the bottom, and find and correct the misposted account. |

### 1.2 No Blank Account Codes

| Field | Detail |
|---|---|
| **Rule ID** | INP-002 |
| **Description** | Every row in every TB sheet that contains a debit or credit amount must also contain an account code in column A. |
| **Formula logic** | `IF(OR(D_row<>0, E_row<>0), A_row<>"", TRUE)` |
| **CHECKS sheet row** | Aggregated count of blank-code violations per entity. |
| **FAIL resolution** | Enter the missing account code, or delete the row if it was entered in error. |

### 1.3 Account Codes in Chart of Accounts

| Field | Detail |
|---|---|
| **Rule ID** | INP-003 |
| **Description** | Every account code used in a TB sheet must exist in the `COA` (Chart of Accounts) sheet. |
| **Formula logic** | `COUNTIF(COA!A:A, TB_row_code) > 0` |
| **CHECKS sheet row** | List of unrecognised codes per entity. |
| **FAIL resolution** | Either correct the typo in the TB sheet, or add the missing code to the `COA` sheet. |

---

## 2. FX Checks

### 2.1 Exchange Rate Present for All Entity Currencies

| Field | Detail |
|---|---|
| **Rule ID** | FX-001 |
| **Description** | A closing rate and an average rate must be entered in `SETUP!FX_RATES` for every currency used by any active entity. |
| **Formula logic** | Cross-reference entity currency list from SETUP against FX_RATES table; flag any missing rows. |
| **CHECKS sheet row** | List of missing currency/rate pairs. |
| **FAIL resolution** | Add the missing exchange rate to the `FX_RATES` table in the `SETUP` sheet. |

### 2.2 Exchange Rates are Positive

| Field | Detail |
|---|---|
| **Rule ID** | FX-002 |
| **Description** | No exchange rate may be zero or negative. |
| **Formula logic** | `MIN(FX_RATES range) > 0` |
| **CHECKS sheet row** | Single pass/fail for the whole FX_RATES table. |
| **FAIL resolution** | Correct the invalid rate in the `SETUP` sheet. |

---

## 3. Elimination Checks

### 3.1 Intercompany Matrix is Symmetric

| Field | Detail |
|---|---|
| **Rule ID** | IC-001 |
| **Description** | For every entity pair (A, B), the receivable entered in cell (A,B) of the IC_MATRIX must equal the payable in cell (B,A). |
| **Formula logic** | `IC_MATRIX[A,B] = IC_MATRIX[B,A]` for all off-diagonal cells. |
| **Tolerance** | Zero. |
| **CHECKS sheet row** | List of asymmetric pairs. |
| **FAIL resolution** | Agree the intercompany balance with the counterpart entity and correct the IC_MATRIX. |

### 3.2 Total Eliminations Net to Zero

| Field | Detail |
|---|---|
| **Rule ID** | IC-002 |
| **Description** | The sum of all elimination journal entries (debits and credits) must net to zero in aggregate. |
| **Formula logic** | `SUM(all elimination DR) - SUM(all elimination CR) = 0` |
| **Tolerance** | Zero. |
| **CHECKS sheet row** | Single pass/fail with the net amount shown. |
| **FAIL resolution** | Review the manual adjustment section of the `CONSOLIDATION` sheet for entries that are not fully doubled up. |

### 3.3 All IC Balances Eliminated

| Field | Detail |
|---|---|
| **Rule ID** | IC-003 |
| **Description** | After eliminations, the consolidated balance on any account designated as "Intercompany" in the COA must be zero. |
| **Formula logic** | `SUMIF(COA, "IC", Consolidated_balance) = 0` |
| **Tolerance** | Amounts below the materiality threshold (set in SETUP cell B20) generate WARN not FAIL. |
| **CHECKS sheet row** | List of uneliminated IC accounts and residual balances. |
| **FAIL resolution** | Add a manual elimination entry on the `CONSOLIDATION` sheet for the residual amount. |

---

## 4. Output Checks

### 4.1 Balance Sheet Balances

| Field | Detail |
|---|---|
| **Rule ID** | OUT-001 |
| **Description** | Total assets must equal total liabilities plus total equity on the consolidated balance sheet. |
| **Formula logic** | `BS!Total_Assets = BS!Total_Liabilities + BS!Total_Equity` |
| **Tolerance** | Zero. |
| **CHECKS sheet row** | Single pass/fail with the difference amount shown. |
| **FAIL resolution** | This is typically caused by a failed elimination (IC-002 or IC-003) or an incorrect FX translation. Resolve upstream checks first. |

### 4.2 Retained Earnings Movement Reconciles

| Field | Detail |
|---|---|
| **Rule ID** | OUT-002 |
| **Description** | The movement in consolidated retained earnings from opening to closing balance must equal the consolidated net profit after tax. |
| **Formula logic** | `BS!Closing_RE - BS!Opening_RE = P&L!NPAT` |
| **Tolerance** | Amounts below the materiality threshold generate WARN. |
| **CHECKS sheet row** | Single pass/fail with the reconciling difference shown. |
| **FAIL resolution** | Check for dividend entries or prior-period adjustments that bypass the P&L. |

### 4.3 Cash Flow Reconciles to Balance Sheet Movement

| Field | Detail |
|---|---|
| **Rule ID** | OUT-003 |
| **Description** | The net change in cash per the cash flow statement must equal the movement in the cash and cash equivalents line on the balance sheet. |
| **Formula logic** | `CF!Net_Change_in_Cash = BS!Closing_Cash - BS!Opening_Cash` |
| **Tolerance** | Zero. |
| **CHECKS sheet row** | Single pass/fail with difference shown. |
| **FAIL resolution** | Review the cash flow derivation section on the `CF` sheet for any hardcoded overrides. |

---

## 5. Completeness Checks

### 5.1 All Active Entities Have Data

| Field | Detail |
|---|---|
| **Rule ID** | CMP-001 |
| **Description** | Every entity marked `YES` in the SETUP entity list must have at least one non-zero amount in its TB sheet. |
| **Formula logic** | `MAX(ABS(TB_[Entity] debit/credit columns)) > 0` |
| **CHECKS sheet row** | List of entities with no data entered. |
| **FAIL resolution** | Either enter the trial balance data or change the entity's Include flag to `NO` in SETUP. |

### 5.2 Reporting Period Dates Are Present

| Field | Detail |
|---|---|
| **Rule ID** | CMP-002 |
| **Description** | The period start date, end date, and label in the SETUP sheet must all be populated. |
| **Formula logic** | `AND(B2<>"", B3<>"", B4<>"")` |
| **CHECKS sheet row** | Single pass/fail. |
| **FAIL resolution** | Fill in the missing period fields on the `SETUP` sheet. |

### 5.3 Materiality Threshold Is Set

| Field | Detail |
|---|---|
| **Rule ID** | CMP-003 |
| **Description** | The materiality threshold used by WARN-level checks must be a positive number. |
| **Formula logic** | `SETUP!B20 > 0` |
| **CHECKS sheet row** | Single pass/fail. |
| **FAIL resolution** | Enter a positive numeric value in `SETUP!B20`. A common starting point is 0.1% of total group revenue. |

---

## Validation Run Summary

After a successful run with all checks passing, the top of the `CHECKS` sheet displays a summary banner:

```
✅ ALL CHECKS PASSED — [N] PASS   [0] FAIL   [W] WARN
   Run timestamp: YYYY-MM-DD HH:MM:SS
   Run by: [Excel user name]
```

This banner, together with the detailed check results below it, forms the validation evidence that should be retained as part of the financial close pack.
