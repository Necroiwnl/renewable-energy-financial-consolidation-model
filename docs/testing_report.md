# Testing Report — Renewable Energy Financial Consolidation Model

This document records the test cases used to verify the accuracy, reliability, and robustness of the financial consolidation model. Each test case specifies the scenario being tested, the input data used, the expected output, and the acceptance criteria.

---

## Overview

Testing is performed on a reference copy of the workbook populated with synthetic (non-real) data. The test dataset covers a group of five fictional renewable energy entities across three currencies. All test cases must pass before a new version of the workbook is released.

### Test Dataset — Entity Summary

| Entity Code | Entity Name | Currency | Ownership % | Method |
|---|---|---|---|---|
| `SOLAR_UK` | Sunridge Solar Ltd | GBP | 100% | Full |
| `WIND_DE` | NordWind GmbH | EUR | 100% | Full |
| `HYDRO_NO` | Fjord Hydro AS | NOK | 75% | Full |
| `SOLAR_ES` | Sol Mediterráneo SA | EUR | 60% | Full |
| `HOLD_LU` | RE Holdings Sàrl | EUR | 100% | Full |

Presentation currency: **EUR**

---

## Test Categories

1. [Input Validation Tests](#1-input-validation-tests)
2. [FX Translation Tests](#2-fx-translation-tests)
3. [Consolidation Aggregation Tests](#3-consolidation-aggregation-tests)
4. [Intercompany Elimination Tests](#4-intercompany-elimination-tests)
5. [Financial Statement Output Tests](#5-financial-statement-output-tests)
6. [Dashboard Tests](#6-dashboard-tests)
7. [Scenario Analysis Tests](#7-scenario-analysis-tests)
8. [Edge Case and Stress Tests](#8-edge-case-and-stress-tests)

---

## 1. Input Validation Tests

### TC-INP-001: Balanced trial balance accepted

| Field | Detail |
|---|---|
| **Test Case ID** | TC-INP-001 |
| **Description** | A trial balance where total debits equal total credits should pass check INP-001. |
| **Input** | `SOLAR_UK` TB with total DR = GBP 10,000,000 and total CR = GBP 10,000,000. |
| **Expected result** | `CHECKS` sheet row for `SOLAR_UK` INP-001 shows **PASS**. |
| **Acceptance criteria** | Status cell = "PASS"; no error message. |

### TC-INP-002: Out-of-balance trial balance rejected

| Field | Detail |
|---|---|
| **Test Case ID** | TC-INP-002 |
| **Description** | A trial balance with debits ≠ credits should fail check INP-001. |
| **Input** | `SOLAR_UK` TB modified so total DR = GBP 10,000,100 and total CR = GBP 10,000,000. |
| **Expected result** | `CHECKS` sheet row for `SOLAR_UK` INP-001 shows **FAIL** with difference amount GBP 100. |
| **Acceptance criteria** | Status cell = "FAIL"; adjacent cell shows "DR/CR difference: GBP 100". |

### TC-INP-003: Unrecognised account code flagged

| Field | Detail |
|---|---|
| **Test Case ID** | TC-INP-003 |
| **Description** | An account code not present in the COA sheet should fail check INP-003. |
| **Input** | Add a row to `WIND_DE` TB with account code `9999` (not in COA). |
| **Expected result** | `CHECKS` sheet shows FAIL for INP-003 with code `9999` listed. |
| **Acceptance criteria** | Status cell = "FAIL"; unknown code `9999` appears in the detail list. |

---

## 2. FX Translation Tests

### TC-FX-001: GBP to EUR translation — income statement

| Field | Detail |
|---|---|
| **Test Case ID** | TC-FX-001 |
| **Description** | P&L items for a GBP entity should be translated at the average rate. |
| **Input** | `SOLAR_UK` revenue = GBP 5,000,000; average rate EUR/GBP = 0.8500 (i.e. 1 GBP = 1.1765 EUR). |
| **Expected result** | Consolidated revenue contribution from `SOLAR_UK` = EUR 5,882,353 (= 5,000,000 / 0.8500). |
| **Acceptance criteria** | Difference from expected < EUR 1 (rounding tolerance). |

### TC-FX-002: NOK to EUR translation — balance sheet

| Field | Detail |
|---|---|
| **Test Case ID** | TC-FX-002 |
| **Description** | Balance sheet items for a NOK entity should be translated at the closing rate. |
| **Input** | `HYDRO_NO` total assets = NOK 200,000,000; closing rate EUR/NOK = 11.50. |
| **Expected result** | Consolidated total assets contribution from `HYDRO_NO` = EUR 17,391,304 (= 200,000,000 / 11.50). |
| **Acceptance criteria** | Difference from expected < EUR 1 (rounding tolerance). |

### TC-FX-003: Translation difference posted to equity

| Field | Detail |
|---|---|
| **Test Case ID** | TC-FX-003 |
| **Description** | The difference arising from translating BS items at closing rate and P&L items at average rate should be posted to the Foreign Currency Translation Reserve in equity. |
| **Input** | Rates differ between average and closing (as in TC-FX-001 and TC-FX-002). |
| **Expected result** | BS balances (Assets = Liabilities + Equity including FCTR); FCTR line is non-zero and equals the calculated translation difference. |
| **Acceptance criteria** | OUT-001 shows PASS (balance sheet balances); FCTR balance matches manual calculation within EUR 1. |

---

## 3. Consolidation Aggregation Tests

### TC-AGG-001: Simple aggregation without eliminations

| Field | Detail |
|---|---|
| **Test Case ID** | TC-AGG-001 |
| **Description** | With no intercompany balances, consolidated revenue should equal the sum of all entity revenues translated to EUR. |
| **Input** | All IC balances set to zero in `IC_MATRIX`. Each entity has a known revenue figure. |
| **Expected result** | Consolidated revenue = sum of each entity's revenue translated at its average rate. |
| **Acceptance criteria** | Difference from manually calculated sum < EUR 5 (rounding tolerance across 5 entities). |

### TC-AGG-002: Non-controlling interest calculation

| Field | Detail |
|---|---|
| **Test Case ID** | TC-AGG-002 |
| **Description** | For `HYDRO_NO` (75% owned), 25% of net assets and profit should be attributed to NCI. |
| **Input** | `HYDRO_NO` net assets = EUR 10,000,000; net profit = EUR 1,000,000. |
| **Expected result** | NCI on BS = EUR 2,500,000; NCI in P&L = EUR 250,000. |
| **Acceptance criteria** | NCI values match expected within EUR 1. |

---

## 4. Intercompany Elimination Tests

### TC-IC-001: Elimination of intercompany loan

| Field | Detail |
|---|---|
| **Test Case ID** | TC-IC-001 |
| **Description** | A loan from `HOLD_LU` to `SOLAR_UK` must be fully eliminated from consolidated BS. |
| **Input** | `HOLD_LU` intercompany receivable = EUR 5,000,000; `SOLAR_UK` intercompany payable = EUR 5,000,000 (translated). Both mapped to IC account codes. |
| **Expected result** | After consolidation, consolidated IC receivable = 0 and consolidated IC payable = 0. |
| **Acceptance criteria** | IC-003 shows PASS; both accounts net to zero on `BS` sheet. |

### TC-IC-002: Elimination of intercompany revenue and cost

| Field | Detail |
|---|---|
| **Test Case ID** | TC-IC-002 |
| **Description** | Management fees charged by `HOLD_LU` to subsidiaries must be eliminated from consolidated revenue and cost. |
| **Input** | `HOLD_LU` IC revenue = EUR 500,000; subsidiaries' IC cost = EUR 500,000 in aggregate. |
| **Expected result** | Consolidated revenue and cost each reduced by EUR 500,000; net P&L impact = zero. |
| **Acceptance criteria** | Consolidated IC revenue = 0; consolidated IC cost = 0; NPAT unchanged vs. scenario with no IC fees. |

### TC-IC-003: Asymmetric IC balance generates FAIL

| Field | Detail |
|---|---|
| **Test Case ID** | TC-IC-003 |
| **Description** | If one entity records an IC balance not matched by the counterpart, check IC-001 should FAIL. |
| **Input** | Set `HOLD_LU` receivable from `SOLAR_UK` = EUR 5,000,000 but `SOLAR_UK` payable to `HOLD_LU` = EUR 4,900,000 (EUR 100,000 difference). |
| **Expected result** | `CHECKS` sheet IC-001 shows FAIL; detail shows pair `HOLD_LU / SOLAR_UK` with difference EUR 100,000. |
| **Acceptance criteria** | Status = "FAIL"; correct entity pair and amount shown in detail. |

---

## 5. Financial Statement Output Tests

### TC-OUT-001: Balance sheet balances after full consolidation

| Field | Detail |
|---|---|
| **Test Case ID** | TC-OUT-001 |
| **Description** | After a full consolidation run with no errors, the balance sheet must balance. |
| **Input** | Full test dataset with all IC balances correctly matched. |
| **Expected result** | `CHECKS` OUT-001 shows PASS. `BS` sheet balance check cell shows "BALANCED". |
| **Acceptance criteria** | Assets = Liabilities + Equity (difference = 0). |

### TC-OUT-002: Retained earnings movement reconciles

| Field | Detail |
|---|---|
| **Test Case ID** | TC-OUT-002 |
| **Description** | Movement in retained earnings must equal consolidated NPAT. |
| **Input** | Full test dataset; no dividends declared during the period. |
| **Expected result** | `CHECKS` OUT-002 shows PASS. |
| **Acceptance criteria** | `BS!Closing_RE - BS!Opening_RE = P&L!NPAT` (difference = 0). |

### TC-OUT-003: Cash flow reconciles to balance sheet

| Field | Detail |
|---|---|
| **Test Case ID** | TC-OUT-003 |
| **Description** | Net change in cash per CF statement must equal balance sheet cash movement. |
| **Input** | Full test dataset. |
| **Expected result** | `CHECKS` OUT-003 shows PASS. |
| **Acceptance criteria** | Difference = 0. |

---

## 6. Dashboard Tests

### TC-DASH-001: Revenue waterfall totals match P&L

| Field | Detail |
|---|---|
| **Test Case ID** | TC-DASH-001 |
| **Description** | The sum of all bars in the revenue waterfall chart must equal consolidated revenue on the P&L sheet. |
| **Input** | Full test dataset. |
| **Expected result** | Waterfall total = `P&L!Total_Revenue`. |
| **Acceptance criteria** | Values match exactly (no rounding difference). |

### TC-DASH-002: Dashboard refreshes after data change

| Field | Detail |
|---|---|
| **Test Case ID** | TC-DASH-002 |
| **Description** | After changing a revenue figure and re-running the consolidation, the dashboard must update. |
| **Input** | Increase `SOLAR_UK` revenue by GBP 1,000,000 and re-run. |
| **Expected result** | Dashboard KPI and waterfall reflect the updated figure. |
| **Acceptance criteria** | Dashboard revenue figure increases by approximately EUR 1,176,471 (GBP 1M at test rate). |

---

## 7. Scenario Analysis Tests

### TC-SCEN-001: Switching scenarios changes output

| Field | Detail |
|---|---|
| **Test Case ID** | TC-SCEN-001 |
| **Description** | Switching the scenario selector from Actual to Budget should change displayed figures to budget data. |
| **Input** | Different revenue figures entered under Actual and Budget scenarios. |
| **Expected result** | `P&L` sheet shows budget revenue when Budget is selected; reverts to actual when Actual is selected. |
| **Acceptance criteria** | Both scenario values are correct and neither overwrites the other. |

---

## 8. Edge Case and Stress Tests

### TC-EDGE-001: Entity with zero balances throughout

| Field | Detail |
|---|---|
| **Test Case ID** | TC-EDGE-001 |
| **Description** | An entity with all-zero TB entries should be included with no errors if marked YES. |
| **Input** | `SOLAR_ES` TB all zeros. |
| **Expected result** | CMP-001 shows FAIL (no data entered). After changing `SOLAR_ES` to `NO` in SETUP, CMP-001 shows PASS. |
| **Acceptance criteria** | Correct FAIL/PASS status transitions as described. |

### TC-EDGE-002: All entities in the same currency

| Field | Detail |
|---|---|
| **Test Case ID** | TC-EDGE-002 |
| **Description** | When all entities use EUR and the presentation currency is EUR, no FX translation should occur and the FCTR should be zero. |
| **Input** | Override all entity currencies to EUR and set all rates to 1.0. |
| **Expected result** | FCTR = 0; consolidated figures equal arithmetic sum of entity figures. |
| **Acceptance criteria** | FCTR balance = 0; sum check passes within EUR 1. |

### TC-EDGE-003: Maximum entity count

| Field | Detail |
|---|---|
| **Test Case ID** | TC-EDGE-003 |
| **Description** | The model should run without error when populated with the maximum supported entity count (20 entities). |
| **Input** | 20 entities, each with a 50-line trial balance. All IC balances zero. |
| **Expected result** | Consolidation completes without error; all CHECKS show PASS or WARN. |
| **Acceptance criteria** | No macro errors; CHECKS sheet has no FAIL rows; run completes in under 60 seconds. |

---

## Test Results Summary

The table below is updated each time a new version of the workbook is tested.

| Version | Date Tested | Tester | TC Pass | TC Fail | TC Warn | Overall Status |
|---|---|---|---|---|---|---|
| v1.0 | — | — | — | — | — | Not yet tested |

---

## Defect Log

No defects recorded. Defects discovered during testing will be logged here with the following fields: Defect ID, TC reference, description, severity (Critical / Major / Minor), status (Open / Fixed / Won't Fix), and fix version.
