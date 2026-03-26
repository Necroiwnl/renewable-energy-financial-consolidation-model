# USER MANUAL
## Renewable Consolidation Model

This manual explains how to use `Renewable_Consolidation_Model.xlsx` as it exists in production. The workbook is formula-driven, bounded to controlled input ranges, and monitored through a validation framework (`V01` to `V17`).

---

## Quick Start

1. Update source balances in `TB_Input` only within rows `5:23` and `25:200`.
2. Update intercompany records in `IC_Transactions` if group transactions changed.
3. Confirm helper formulas and totals populate automatically; do not type into formula cells.
4. Open `Validation` and confirm all checks return `PASS`.
5. Use `Output_BS`, `Output_PL`, `Output_CF`, and `Dashboard` only after validation is clear.

---

## Editable vs Non-Editable Sheets

The workbook is safest when users treat it as a controlled system rather than a free-form spreadsheet.

| Sheet or Group | Edit Status | Operating Guidance |
| --- | --- | --- |
| `TB_Input` | Editable | Routine data entry sheet. Enter source balances only in active rows `5:23` and `25:200`. Do not type into helper formulas. |
| `IC_Transactions` | Editable | Controlled transaction input sheet for intercompany entries. Update only when the underlying transactions change. |
| `Entity_Master` | Editable with control | Maintain only when entity structure, ownership, or consolidation method changes. Directly affects NCI and validation. |
| `Asset_Master`, `Reference_Lists`, `Period_Control` | Editable with control | Reference/admin sheets. Update only during controlled maintenance. |
| `Mapping_BS`, `Mapping_PL`, `Mapping_CF` | Controlled maintenance only | Do not change during routine reporting. Changes here belong to model maintenance, not month-end data entry. |
| `Eliminations` | Non-editable in normal use | Formula-driven support sheet. Manual edits can break traceability and are checked by `V10`. |
| `Consolidation_BS`, `Consolidation_PL`, `Consolidation_CF` | Non-editable | Core calculation engine. Do not type over formulas. |
| `Output_BS`, `Output_PL`, `Output_CF`, `Output_Segment` | Non-editable | Presentation layer only. These sheets should only display linked results. |
| `Validation` | Non-editable | Control sheet. Do not overwrite checks or statuses. |
| `Dashboard` | Non-editable | Summary sheet for KPI review and status monitoring. |
| `Instructions` | Reference only | Informational sheet; not part of the calculation flow. |

---

## 1. Introduction to the Model

### Technical Explanation

The workbook is an Excel-based financial consolidation model for a renewable energy group. It combines multiple entity trial balances into consolidated Balance Sheet, Profit and Loss, and Cash Flow outputs.

The primary reporting entities in the current file are:

- `E001` - Parent Renewable Ltd
- `E002` - Solar SPV 1 Ltd
- `E003` - Solar SPV 2 Ltd

The main processing path is:

```text
TB_Input -> Consolidation_BS / Consolidation_PL / Consolidation_CF
        -> Output_BS / Output_PL / Output_CF
        -> Dashboard
```

Supporting sheets provide:

- entity ownership and consolidation metadata (`Entity_Master`)
- intercompany transaction inputs (`IC_Transactions`)
- elimination feed lines (`Eliminations`)
- validation and control (`Validation`)

The model is designed around:

- bounded input and formula ranges
- non-volatile formulas
- traceable links between sheets
- validation-driven control before outputs are used

### Simple Explanation 

Think of the workbook as a controlled pipeline:

1. You load balances into one place.
2. Excel sorts them into the right financial statement buckets.
3. Group-only transactions are removed.
4. Clean reports are produced.
5. The validation sheet checks whether anything is broken.

It is not a static report. It is a calculation engine with a reporting layer on top.

### Example

Assume:

- `E001` revenue = `100`
- `E002` revenue = `80`
- `E003` revenue = `40`

The model:

- combines them into the consolidation layer
- subtracts intercompany revenue if applicable
- sends the post-elimination result to `Output_PL`
- shows the final KPI on the `Dashboard`

---

## 2. How to Input Data (`TB_Input`)

### Technical Explanation

`TB_Input` is the source sheet for entity-level trial balances.

#### Sheet layout

| Column | Purpose |
| --- | --- |
| `A` | `Account_Code` |
| `B` | `Account_Name` |
| `C` | `FS_Category` |
| `D` | `Account_Nature` (`Debit` or `Credit`) |
| `E:F:G` | `E001` Dr / Cr / Net |
| `H:I:J` | `E002` Dr / Cr / Net |
| `K:L:M` | `E003` Dr / Cr / Net |
| `N` | three-digit account prefix helper |

#### Formula behavior

The entity net columns are formula-driven. The pattern is:

```excel
=IF($D5="Debit",E5-F5,F5-E5)
```

This means:

- debit-nature accounts produce positive net values when debits exceed credits
- credit-nature accounts produce positive net values when credits exceed debits

Column `N` derives the prefix used by the model:

```excel
=LEFT(A5,3)
```

#### Active range

The supported active input range is:

- rows `5:23`
- rows `25:200`

Important structural notes:

- row `24` is reserved for the `TOTAL` line
- row `25` is part of the active calculation range and is already used in the delivered file
- add new accounts on the next blank active row, typically from row `26` onward

### Simple Explanation 

`TB_Input` is where you type the raw balances for each company.

You do not type the `Net` values manually. You only enter:

- the account code
- the account description
- whether the account is debit or credit in nature
- the debit and credit numbers for each entity

The workbook calculates the net figure automatically.

### Example

If row `6` contains:

- `Account_Code` = `110001`
- `Account_Name` = `PPE - Solar Plants`
- `Account_Nature` = `Debit`
- `E001 Dr` = `11000`
- `E001 Cr` = `1000`

Then `E001 Net` becomes:

```text
11000 - 1000 = 10000
```

---

## 3. Understanding Account Codes and Prefix Mapping

### Technical Explanation

The consolidation logic uses the first three digits of `Account_Code` to route balances into the correct financial statement line.

The model reads the first three digits through the prefix helper in `TB_Input!N` and uses that prefix in bounded `SUMPRODUCT` formulas.

Example calculation pattern:

```excel
=SUMPRODUCT((LEFT(TB_Input!$A$5:$A$200,3)="110")*(TB_Input!$G$5:$G$200))
```

This means all accounts beginning with `110` are aggregated into the same target line for the relevant entity.

#### Prefix examples used in the live model

| Prefix | Typical meaning in current model |
| --- | --- |
| `100`, `142` | Cash and cash equivalents |
| `110`, `115` | Property, plant and equipment |
| `111` | Capital work-in-progress |
| `120` | Investments |
| `122` | Loans |
| `141` | Trade receivables |
| `200` | Equity share capital |
| `300` | Borrowings (non-current) |
| `302`, `400` | Borrowings (current) |
| `402` | Trade payables |
| `500` to `540` | Operating revenue lines |
| `550`, `551` | Other income lines |
| `600` to `670` | Expense lines |
| `700`, `701` | Tax lines |

The mapping sheets (`Mapping_BS`, `Mapping_PL`, `Mapping_CF`) document the mapping structure. The live consolidation engine uses explicit formula logic aligned to that design.

### Simple Explanation 

The first three digits are the model's routing code.

If two accounts start with the same three digits, the model treats them as belonging to the same bucket.

This is why you can add a new account with an existing prefix and the model still picks it up automatically.

### Example

Both of these accounts roll into the same PPE bucket:

- `110001`
- `110045`

Because both begin with `110`, the model adds them together in the PPE line.

If you add account `115999`, it still flows into PPE because `115` is already mapped in the live model.

If you add account `999001`, the model does not know that prefix and `V11` will fail.

---

## 4. How Consolidation Works (`BS`, `PL`, `CF`)

### Technical Explanation

#### Balance Sheet and Profit and Loss

`Consolidation_BS` and `Consolidation_PL` follow the same structural pattern:

- entity columns (`E`, `F`, `G`) hold entity-level totals
- pre-elimination column (`H`) sums entity totals
- elimination column (`I`) pulls elimination amounts from `Eliminations`
- post-elimination column (`J`) calculates the consolidated result

The post-elimination formula pattern is:

```excel
=H4-I4
```

For `Consolidation_PL`, the current workbook also calculates attribution:

- `K` = non-controlling interest share
- `L` = amount attributable to owners

This uses ownership data from `Entity_Master`, including the `60% / 40%` split for `E003`.

#### Cash Flow

`Consolidation_CF` is not populated directly from `TB_Input`. It is derived from consolidated BS and PL figures.

Examples:

- Profit before tax comes from `Consolidation_PL`
- depreciation comes from `Consolidation_PL`
- receivable and payable movements come from `Consolidation_BS`
- closing cash is reconciled back to the consolidated BS cash line

### Simple Explanation 

The consolidation sheets do three things:

1. Add each company together.
2. Remove transactions that only exist inside the group.
3. Keep the clean group number.

The cash flow sheet is then built using the final BS and PL numbers rather than re-reading every TB line again.

### Example

Suppose a revenue line has:

- `E001` = `100`
- `E002` = `80`
- `E003` = `40`

Then:

- pre-elimination = `220`
- elimination = `30`
- post-elimination = `190`

That `190` is the number pushed into the output sheet.

---

## 5. How Eliminations Work

### Technical Explanation

Intercompany eliminations start in `IC_Transactions` and are converted into structured lines in `Eliminations`.

The current workbook contains elimination rows such as:

- `EPC_Revenue_Elim`
- `Profit_in_CWIP`
- `IC_Loan_Elim_Asset`
- `IC_Loan_Elim_Liab`
- `IC_Interest_Elim_Inc`
- `IC_Interest_Elim_Exp`

The `Eliminations` sheet stores:

- elimination type
- financial statement line code to debit or credit
- elimination amount

The consolidation sheets then pull those amounts using `SUMIF` on line codes, for example:

```excel
=IFERROR(SUMIF(Eliminations!$B$2:$B$100,A4,Eliminations!$E$2:$E$100),0)
 +IFERROR(SUMIF(Eliminations!$C$2:$C$100,A4,Eliminations!$E$2:$E$100),0)
```

This makes elimination logic traceable:

`IC_Transactions -> Eliminations -> Consolidation_*`

### Simple Explanation 

When one group company trades with another group company, the group as a whole has not made money from an outside party yet.

So the model removes those internal transactions before showing the final consolidated result.

### Example

Assume:

- Parent charges SPV `30,000` for EPC work
- Profit margin on that work is `10%`

The model creates elimination effects such as:

- remove `30,000` EPC revenue
- remove `3,000` unrealized profit from CWIP

That prevents the group from overstating revenue and assets.

---

## 6. Understanding the Output Sheets

### Technical Explanation

The output sheets are presentation layers:

- `Output_BS`
- `Output_PL`
- `Output_CF`

They do not perform source aggregation themselves. They link to the consolidation engine using `IFERROR(...)`.

Examples:

```excel
=IFERROR(Consolidation_BS!J2,0)
=IFERROR(Consolidation_PL!J25,0)
=IFERROR(Consolidation_CF!C26,0)
```

The output sheets present:

- reporting labels
- note numbers
- current period column
- comparison period column
- variance
- variance percentage

In the current workbook, the prior-period comparison columns are structurally present but formula-set to `0`.

### Simple Explanation 

The output sheets are the clean report pages.

They are meant for review, printing, and presentation. They are not the place where the core calculations happen.

### Example

If `Consolidation_PL!J25` equals `-23900`, then:

- `Output_PL!C27` shows `-23900`
- the dashboard can then read that same value for its PBT or PAT summary

---

## 7. Understanding the Dashboard

### Technical Explanation

The `Dashboard` sheet summarizes:

- validation status
- fail count
- key financial metrics

Current headline metrics shown on the dashboard:

- Total Revenue
- EBITDA
- Profit before Tax
- Profit after Tax
- Total Assets
- Total Equity

Validation status is driven by:

- `Dashboard!B12` = failed check count
- `Dashboard!C12` = status text

The dashboard pulls its KPI values from the output sheets, not from source data directly.

Important note on the current workbook:

- cells for current period, comparison period, display mode, and reporting currency are visible
- however, in this file version those cells are informational and are not active calculation drivers

### Simple Explanation 

The dashboard is the front page.

It tells you two things immediately:

1. Is the model healthy?
2. What are the main group numbers?

If the dashboard says `ISSUES DETECTED`, you should go to `Validation` before using any report.

### Example

If one check fails:

- `Dashboard!B12` becomes `1`
- `Dashboard!C12` becomes `ISSUES DETECTED`

If all checks pass:

- `Dashboard!B12` becomes `0`
- `Dashboard!C12` becomes `ALL CHECKS PASSED`

---

## 8. Validation System

### Technical Explanation

The `Validation` sheet is the workbook's control layer. Each check returns:

- expected value
- actual value
- status (`PASS` / `FAIL`)

#### Validation catalogue

| Code | Description | What it checks |
| --- | --- | --- |
| `V01` | BS Balance | Total assets vs total equity and liabilities |
| `V02` | IC Loan Asset = Loan Liability | Internal loan asset equals internal loan liability |
| `V03` | TB Debit = Credit (`E001`) | Trial balance integrity for entity `E001` |
| `V04` | TB Debit = Credit (`E002`) | Trial balance integrity for entity `E002` |
| `V05` | TB Debit = Credit (`E003`) | Trial balance integrity for entity `E003` |
| `V06` | Cash Flow Closing Cash = BS Cash | Cash flow ending cash matches BS cash |
| `V07` | NCI Attribution Consistent | Owner/NCI split is internally consistent |
| `V08` | Ownership % Valid | Ownership percentages remain between `0` and `1` |
| `V09` | No Self-Parent Relationships | Entity hierarchy is structurally valid |
| `V10` | IC Eliminations Complete and Aligned | Elimination completeness, formula integrity, mapping alignment, amount alignment |
| `V11` | Unmapped TB Prefixes | New or unsupported prefixes are flagged |
| `V12` | Entity Master Integrity | Required entity metadata is present in pairs |
| `V13` | No Formula Errors in Critical Sheets | Detects `#REF!`, `#VALUE!`, and other formula errors |
| `V14` | No Duplicate Entity_IDs | Duplicate entity IDs are blocked |
| `V15` | Output Layer Linked to Engine | Output cells still tie to consolidation engine |
| `V16` | Formula Integrity Check (Engine + TB) | Detects overwritten formulas in engine, output, and TB helper columns |
| `V17` | TB Input Range Capacity | Detects data below the supported input range |

#### Special focus: `V10`

`V10` is especially important because it no longer checks only row count. It now tests:

- whether the expected elimination rows exist
- whether `Eliminations!E` still contains formulas
- whether the elimination lines point to the correct FS codes
- whether the elimination amounts still agree with `IC_Transactions`

This protects against a false `PASS` when elimination amounts have been overwritten.

#### Special focus: `V16`

`V16` checks formula integrity in:

- `Consolidation_BS`
- `Consolidation_PL`
- `Consolidation_CF`
- `Output_BS`
- `Output_PL`
- `Output_CF`
- `TB_Input` helper columns `G`, `J`, `M`, and `N`

This is the main control that detects if someone pasted over formulas in critical calculation ranges.

#### Special focus: `V17`

`V17` checks:

```excel
=COUNTA(TB_Input!$A$201:$N$260)
```

This means the model will fail if users type data below the supported bounded range.

### Simple Explanation

The validation sheet is the workbook's internal audit list.

It is the model's way of saying:

- "the balance sheet still balances"
- "the eliminations still make sense"
- "nobody pasted over a formula"
- "nobody entered data outside the supported range"

If any row says `FAIL`, treat it as a real warning.

### Example

Three common examples:

1. If someone types a new account on row `201`, `V17` fails.
2. If someone overwrites `Eliminations!E2` with a number, `V10` fails.
3. If someone replaces `TB_Input!G5` with a hardcoded number, `V16` fails.

---

## 9. Common Mistakes and How to Avoid Them

### Technical Explanation

The most common user errors in this workbook are:

| Mistake | Effect | Prevention |
| --- | --- | --- |
| Typing into row `24` on `TB_Input` | Breaks the reserved `TOTAL` row | Never use row `24` for data entry |
| Adding data below row `200` | Data is outside the supported calculation range | Stay within rows `5:23` and `25:200` |
| Changing `Account_Nature` incorrectly | Reverses the sign logic in net columns | Use `Debit` for debit-nature accounts and `Credit` for credit-nature accounts |
| Using an unsupported new prefix | Data does not map to a statement line | Check `V11` and request mapping maintenance if needed |
| Overwriting formulas in helper or output cells | Breaks traceability | Enter data only in intended input cells |
| Editing elimination line codes manually | Misroutes eliminations | Treat `Eliminations` as a controlled sheet |

### Simple Explanation 

Most issues happen when users type in the wrong place or assume the model will "figure it out" automatically.

The safest habit is:

- only type in source input fields
- do not type over formula cells
- check `Validation` after every change

### Example

If you change an account from `Debit` to `Credit` by mistake:

- the same debit/credit numbers now produce the opposite net sign
- the consolidated result becomes wrong
- balance and P&L may shift in unexpected ways

---

## 10. What Happens When Something Breaks

### Technical Explanation

When the model detects an issue:

- the relevant validation row changes to `FAIL`
- total fail count increases
- the dashboard status changes to `ISSUES DETECTED`

#### Typical diagnosis path

| If this fails | Start here |
| --- | --- |
| `V03` to `V05` | Check entity debits and credits in `TB_Input` |
| `V10` | Check `IC_Transactions` and `Eliminations` |
| `V11` | Check new account codes and prefixes |
| `V16` | Check whether formulas were overwritten |
| `V17` | Check for data entered below row `200` |

### Simple Explanation 

The model does not usually fail quietly anymore.

If something important is broken, the validation sheet should tell you where to look first.

The workflow is:

1. See the dashboard warning.
2. Open `Validation`.
3. Find the first `FAIL`.
4. Trace the issue back to the relevant source sheet.

### Example

If `V11` fails:

- go to `TB_Input`
- check the new account codes
- compare the first three digits against supported prefixes

If you entered a new prefix, the model needs controlled maintenance before the output can be trusted.

---

## 11. How to Add New Accounts

### Technical Explanation

There are two different situations:

#### A. New account with an existing supported prefix

This is the easy case.

Steps:

1. Add the account on the next blank active row in `TB_Input`
2. Enter:
   - account code
   - account name
   - FS category
   - account nature
   - entity debits and credits
3. Let the helper formulas calculate net and prefix
4. Review `Validation`

Because the consolidation formulas aggregate by prefix, the new account is automatically included.

#### B. New account with a brand-new prefix

This is not a pure data-entry task.

If the prefix does not already exist in the live model:

- `V11` will fail
- the account will not map to a correct statement line
- model logic must be updated in a controlled way

### Simple Explanation 

If the first three digits already exist, the model usually knows where to put the new account.

If the first three digits are new, the model does not know what that code means. That is a design update, not just an input update.

### Example

If you add:

- `115999` - new PPE sub-account

it is picked up automatically because prefix `115` is already supported.

If you add:

- `888001`

the model flags it through `V11` because prefix `888` is not part of the supported mapping set.

---

## 12. How to Debug Issues

### Technical Explanation

A disciplined debug sequence is:

1. Start with `Dashboard`
2. Open `Validation`
3. Identify the failing check
4. Trace one layer backward at a time

Recommended trace path:

```text
Dashboard
-> Output sheet
-> Consolidation sheet
-> TB_Input or Eliminations / IC_Transactions
```

#### Practical debug rules

- If an output number is wrong, first confirm the linked consolidation cell
- If the consolidation number is wrong, test whether the entity totals or elimination column is wrong
- If the entity total is wrong, inspect the source rows in `TB_Input`
- If the elimination number is wrong, inspect `IC_Transactions` and `Eliminations`
- If the issue is structural, check `V16` and `V17`

### Simple Explanation 

Do not jump randomly between sheets.

Follow the number backward:

1. where do I see the wrong result?
2. where is that result linked from?
3. where does that linked number come from?
4. where was the original source entered?

This keeps debugging fast and controlled.

### Example

Suppose `Dashboard` shows the wrong Profit before Tax.

Trace it like this:

1. `Dashboard!B17`
2. `Output_PL!C25`
3. `Consolidation_PL!J22`
4. revenue / expense rows and elimination column in `Consolidation_PL`
5. relevant `TB_Input` rows and `Eliminations`

---

## 13. Best Practices

### Technical Explanation

Follow these operating rules:

- Enter data only in approved source areas
- Keep all trial balance lines within rows `5:23` and `25:200`
- Never use row `24` for entry
- Do not overwrite formula cells in helper, consolidation, output, validation, or dashboard ranges
- Review `Validation` after each major change
- Treat `V10`, `V16`, and `V17` as high-priority controls
- When adding accounts, prefer existing supported prefixes
- If a new prefix is unavoidable, treat it as a model change request
- Use `IC_Transactions` as the source of truth for intercompany eliminations

### Simple Explanation 

Use the workbook like a controlled system, not like a blank spreadsheet.

That means:

- type data where the model expects data
- let formulas do the math
- trust validation more than intuition

### Example

A good month-end routine is:

1. Update `TB_Input`
2. Review `IC_Transactions`
3. Check `Validation`
4. Confirm `Dashboard` says `ALL CHECKS PASSED`
5. Review `Output_BS`, `Output_PL`, and `Output_CF`

---

## 14. Safe Extension Guide

### Technical Explanation

This section is for controlled model maintenance, not routine month-end use.

#### A. How to increase `TB_Input` capacity safely

The current model is intentionally bounded to row `200`, with `V17` monitoring rows `201:260` for overflow. Extending capacity is possible, but it must be done as an engine change rather than a simple row insertion.

To extend capacity safely, all bounded references that rely on `5:200` must be reviewed and updated together, including:

- `TB_Input` totals on row `24`
- `Consolidation_BS` prefix aggregation formulas
- `Consolidation_PL` prefix aggregation formulas
- validation checks `V03`, `V04`, `V05`
- validation checks `V11`, `V16`, `V17`
- any helper formula pre-seeding below the current active area

Minimum safe extension procedure:

1. Increase the helper formula coverage in `TB_Input` for the new rows.
2. Update every bounded range that currently ends at row `200`.
3. Update `V17` so it monitors the new overflow zone rather than the old one.
4. Test one new account below the old limit and confirm it flows through to output.
5. Confirm `Validation` still returns `PASS` and the dashboard status is clean.

#### B. Risks of adding new months

Adding a new month is not only a data-entry task in the current workbook version.

The live model is structurally centered on the loaded reporting set, and the output comparison columns are currently formula-set to `0`. In addition, the visible period/display fields on `Dashboard` are informational rather than active calculation drivers.

That means a true multi-month extension requires controlled redesign of:

- comparative period sourcing
- period control logic
- output formulas
- dashboard linkage
- validation logic for cross-period consistency

If a new month is added informally, common risks are:

- stale prior-period columns
- incorrect dashboard comparisons
- hidden hardcoding in report copies
- false confidence from a dashboard that appears complete but is not time-aware

#### C. Why bounded ranges exist

Bounded ranges are a deliberate design choice. They exist to improve:

- stability
- traceability
- file performance
- formula auditability
- protection against accidental expansion into untested rows

The model avoids:

- full-column references
- volatile formulas
- dynamic open-ended ranges

This makes the workbook more predictable and easier to control in production.

### Simple Explanation 

The model has walls on purpose.

Those walls are not a weakness. They are there so the workbook does not quietly start reading extra rows, extra months, or half-built structures that nobody tested.

If you need more rows or more months, do not stretch the sheet casually. Treat that as a controlled upgrade to the model.

### Example

Suppose the current model supports rows up to `200`, and a user starts loading new accounts on row `215`.

If the model is not extended properly:

- the new rows may not reach consolidation
- outputs may stay unchanged
- the dashboard could be misleading without the correct control updates

The safe approach is:

1. extend the helper formulas
2. extend every `5:200` dependency together
3. move the overflow check boundary
4. test the new rows end to end

---

## Final Operating Note

This workbook is intentionally conservative:

- bounded ranges instead of open-ended ranges
- explicit formulas instead of volatile constructs
- validation before reporting

That design improves stability and traceability, but it also means users must stay within the defined structure. If the structure needs to change, update the model deliberately rather than working around it inside the workbook.
