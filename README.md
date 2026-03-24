# Renewable Consolidation Model

`Renewable_Consolidation_Model.xlsx` is an Excel-based financial consolidation model for a renewable energy group. The workbook is production-ready, formula-driven, and designed to take multi-entity trial balances through consolidation, intercompany elimination, reporting, and validation without relying on volatile Excel functions or hardcoded consolidation values.

## Project Overview

The model consolidates financial data for three reporting entities:

- `E001` - Parent Renewable Ltd
- `E002` - Solar SPV 1 Ltd
- `E003` - Solar SPV 2 Ltd

It produces:

- Consolidated Balance Sheet
- Consolidated Statement of Profit and Loss
- Consolidated Cash Flow Statement
- Output reporting sheets
- Validation status and dashboard KPIs

The workbook is built around bounded, traceable formulas and a validation-first control layer.

## Key Features

- Formula-driven `TB_Input` netting for each entity (`Dr`, `Cr`, `Net`)
- Prefix-based aggregation into `Consolidation_BS` and `Consolidation_PL`
- Cash flow construction in `Consolidation_CF` from BS and PL links
- Intercompany elimination flow from `IC_Transactions` to `Eliminations`
- Output sheets linked to consolidation sheets with `IFERROR(...)`
- Validation framework `V01` to `V17`
- Dashboard with fail count and headline KPIs
- Non-controlling interest allocation for `E003` based on `Entity_Master`

## Model Architecture

### Core Flow

```text
TB_Input
  -> Consolidation_BS / Consolidation_PL / Consolidation_CF
  -> Output_BS / Output_PL / Output_CF
  -> Dashboard
```

### Supporting Layers

```text
Entity_Master -----------------------> Consolidation logic / NCI allocation
IC_Transactions -> Eliminations -----> Consolidation elimination columns
Validation --------------------------> Dashboard status
Mapping_BS / Mapping_PL / Mapping_CF -> Mapping reference structure
```

### Calculation Pattern

At the leaf level, BS and PL lines use bounded `SUMPRODUCT` formulas such as:

```excel
=SUMPRODUCT((LEFT(TB_Input!$A$5:$A$200,3)="110")*(TB_Input!$G$5:$G$200))
```

The general pattern is:

- Entity columns (`E:G`) = source totals by prefix and entity
- Pre-elimination column (`H`) = sum of entity columns
- Elimination column (`I`) = `SUMIF`-based pull from `Eliminations`
- Post-elimination column (`J`) = `H - I`

## Validation System

The workbook contains `V01` to `V17` on the `Validation` sheet. These checks cover:

- Balance integrity
- Trial balance integrity
- Ownership and entity master quality
- Intercompany elimination integrity
- Formula integrity
- Output-to-engine linkage
- Input range overflow

### High-Importance Checks

| Check | Purpose | Why it matters |
| --- | --- | --- |
| `V10` | Confirms eliminations are complete and aligned | Detects broken elimination amounts, overwritten formulas, and incorrect FS line mapping |
| `V16` | Formula integrity check (engine + TB) | Detects overwritten formulas in consolidation, output, and `TB_Input` helper columns |
| `V17` | TB input range capacity | Detects any data entered below the supported bounded range (`201:260`) |

### Validation Status Logic

- `Validation!G2` counts failed checks
- `Dashboard!B12` reads the fail count
- `Dashboard!C12` displays:
  - `ALL CHECKS PASSED`
  - `ISSUES DETECTED`

## How It Works

1. Users load or update account balances in `TB_Input`.
2. Net values are derived per entity using account nature (`Debit` / `Credit`).
3. BS and PL consolidation sheets aggregate balances by three-digit account prefix.
4. Intercompany items flow from `IC_Transactions` into `Eliminations`, then into the consolidation elimination columns.
5. `Consolidation_CF` derives cash flow lines from consolidated BS and PL outputs.
6. Output sheets present reporting layouts.
7. Validation checks test balance, mapping, formula integrity, and structural risks.
8. Dashboard summarizes validation status and key KPIs.

## File Structure

### Input and Reference Sheets

- `TB_Input` - trial balance input by entity
- `Entity_Master` - entity structure, ownership, consolidation method
- `Asset_Master` - asset reference data
- `Reference_Lists` - supporting lookup content
- `Mapping_BS`, `Mapping_PL`, `Mapping_CF` - mapping reference sheets
- `IC_Transactions` - intercompany transaction source
- `Eliminations` - elimination lines feeding consolidation
- `Period_Control` - period reference sheet

### Engine Sheets

- `Consolidation_BS`
- `Consolidation_PL`
- `Consolidation_CF`

### Reporting Sheets

- `Output_BS`
- `Output_PL`
- `Output_CF`
- `Output_Segment`
- `Dashboard`

### Control and Documentation Sheets

- `Validation`
- `Instructions`

## How to Use

1. Open `Renewable_Consolidation_Model.xlsx`.
2. Enter or update source balances in `TB_Input`.
3. Keep entries within the supported active range:
   - rows `5:23`
   - rows `25:200`
   - row `24` is reserved for `TOTAL`
4. Review `Validation` and confirm all checks return `PASS`.
5. Review `Output_BS`, `Output_PL`, `Output_CF`, and `Dashboard`.
6. If any validation fails, debug the issue before using the outputs.

## Key Design Decisions

- Bounded ranges only: the model uses explicit ranges such as `$A$5:$A$200`
- No volatile formulas: no `OFFSET`, `INDIRECT`, or similar functions
- No full-column references
- No hardcoded values in critical consolidation or output ranges
- `IFERROR(...)` used in reporting and dashboard layers
- Formula-driven elimination pull via `SUMIF` from `Eliminations`
- Validation-driven integrity control before report use

## Limitations

- `TB_Input` is intentionally bounded to row `200`; entries below that are not processed and are now flagged by `V17`
- Adding an account with an existing supported prefix is automatic; introducing a brand-new prefix requires controlled model maintenance
- The workbook contains comparison columns (`FY2023`) in the output sheets, but in the current file those prior-period values are structurally present and formula-set to `0`
- Dashboard period/display cells are informational labels in the current workbook version; they are not active calculation drivers
- The workbook package currently does not include native Excel `dataValidation` dropdown rules or sheet protection

## Screenshots

Add screenshots under `docs/images/` and reference them here.

Recommended placeholders:

- `docs/images/dashboard.png`
- `docs/images/validation.png`
- `docs/images/tb_input.png`
- `docs/images/output_pl.png`

Example markdown:

```md
![Dashboard](docs/images/dashboard.png)
```
