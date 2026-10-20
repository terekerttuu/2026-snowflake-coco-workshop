---
name: prd-to-dynamic-table-plan
description: "Turn a PRD / business-requirements workbook (XLSX) into an implementation plan for changing a target Snowflake Dynamic Table (e.g. SILVER_AP_INVOICES). Use when: a PRD, requirements spreadsheet, source onboarding sheet, column mapping, or business rules file needs to be translated into Dynamic Table changes. Triggers: PRD to dynamic table, requirements to silver layer, onboard new source system, plan DT change from spreadsheet, business rules xlsx, column mapping workbook, update SILVER_AP_INVOICES."
---

# PRD → Dynamic Table Implementation Plan

Reads a requirements workbook, compares it against the **current** definition of a
target Dynamic Table, and returns a plan. It plans only — it never runs DDL.

## When to Use
- A stakeholder hands over a PRD-style XLSX (source onboarding, column mapping, business rules) that changes a Dynamic Table.
- You need to know what changes, what is already decided, and what must be clarified before writing SQL.

Do **not** use for: building the DT once the plan is approved (use the `dynamic-tables` skill), or for generic data analysis of the spreadsheet.

## Inputs
| Input | Required | Example | Notes |
|---|---|---|---|
| `prd_path` | Yes | `assets/ap_invoices_prd.xlsx` | XLSX only for now. If given `.csv`/`.docx`/`.pdf`, say it is unsupported and ask the user to convert or confirm a best-effort read. |
| `target_dynamic_table` | Yes | `COCO_WORKSHOP.PIPELINE_LAB.SILVER_AP_INVOICES` | Ask for the fully-qualified name if only a short name is given. |
| `layer_conventions` | No | "No FX conversion at Silver" | Extra house rules to apply. |
| `scope_filter` | No | "Baan only" | Limit the plan to some sources/requests. |

## Workflow

### Step 1: Validate inputs
- Confirm `prd_path` exists and ends in `.xlsx`/`.xlsm`. Otherwise stop and tell the user (see Inputs).
- Confirm `target_dynamic_table` is fully qualified.

### Step 2: Read every sheet of the workbook
- Use `read` on the file to list sheets and contents, then `read_tabular` with `sheet_name` per sheet for exact rows. Read **all** sheets; requirements are often split across tabs.
- Classify each sheet: *source onboarding*, *column mapping*, *business rules*, *other*. Classify by columns, not sheet name.
- Note each row's ID (e.g. `SRC-…`, `BR-…`) and its `Status`/`Decision` value so every later finding can cite its source.

### Step 3: Inspect the current target DT (read-only)
- `SHOW DYNAMIC TABLES LIKE '<name>' IN SCHEMA <db.schema>` → target lag, warehouse, refresh mode.
- `SELECT GET_DDL('TABLE', '<fqn>')` → current SQL, columns, existing UNION ALL branches, CASE mappings.
- `DESCRIBE TABLE <fqn>` → column names/types.
- For each new source in the PRD, check if its Bronze table exists (`SHOW TABLES LIKE …`). Missing = a blocker, not a guess.
- If the DT does not exist or access fails, continue the plan from the PRD alone and list this as a blocker.

### Step 4: Build the change set
Compare PRD vs. current DT and sort every item into exactly one bucket:
- **Confirmed** — status says confirmed/approved and no conflict with the DT or other rows.
- **Needs decision** — status open/pending, or the PRD itself offers options.
- **Conflict** — two PRD rows contradict, or the PRD contradicts the current DT.
- **Out of layer** — the PRD says it belongs elsewhere (Gold, DMF, Bronze). List it, don't plan it into the DT.

### Step 5: Present the plan (STOP)
Return the output below and **stop**. Do not write or run DDL. Ask the user to resolve open questions or approve the plan.

## Assumptions & Open Questions — rules
These matter more than the plan itself. A wrong silent guess costs more than a question.
1. **Never fill gaps silently.** If a value mapping, type, key, or rule is missing, raise an open question — don't invent a default.
2. **Cite every item** with its PRD row ID (or sheet + row) and, for DT facts, the DDL.
3. **Separate assumptions from facts.** Any assumption you need to keep planning goes in the Assumptions table with a "verify with" owner, using names from the PRD when they exist.
4. **Check for these on every run:**
   - unmapped enum/status values (what happens to values not listed?)
   - dedup keys: scope (per source or global), tie-breakers
   - sign conventions (credits/negatives) vs. "always positive" claims
   - currency/timezone handling vs. rules that need conversion
   - format drift (old vs. new code formats)
   - columns being dropped that downstream may need
   - sources with pending legal/approval status (go-live dependency)
   - type/length changes that would widen or break existing columns
   - refresh-lag impact of new source cadences
5. **Rank questions** by Blocking (can't write correct SQL) vs. Non-blocking (can ship with the stated default).
6. **Say what you couldn't verify** (e.g. "Bronze table not found", "no DT access").

## Output (always return all sections, in this order)
Use `references/output-template.md` exactly. Sections:
1. **Summary** — target DT, PRD file, sheets read, one-line verdict (Ready / Ready with defaults / Blocked).
2. **Current State** — what the DT does today (sources, key logic, target lag), or "not inspected" + why.
3. **New Source Systems** — table: source, request ID, cadence, Bronze table, go-live, status.
4. **Column Mapping Changes** — table: source col → target col, transform, new/changed/unchanged, PRD ref.
5. **Business Rule Changes** — table: rule ID, effect on the DT SQL, bucket (Confirmed / Needs decision / Conflict / Out of layer).
6. **Proposed DT Change Outline** — bullet outline of SQL edits (new UNION ALL branch, CASE additions, QUALIFY, dropped cols). Pseudocode only; mark lines that depend on open questions with `[OQ-n]`.
7. **Assumptions** — table: ID, assumption, why, verify with.
8. **Open Questions** — table: ID, question, PRD ref, Blocking?, suggested owner, proposed default.
9. **Validation Plan** — checks to run after the change (row counts per SOURCE_SYSTEM vs Bronze, key uniqueness, status value coverage, null checks on required cols).
10. **Next Step** — what the user should do next (answer OQs / approve → hand to `dynamic-tables` skill).

## Stopping Points
- ✋ Step 1 if the file is not XLSX or the DT name is ambiguous.
- ✋ Step 5 always — the plan must be reviewed before any implementation.

## Example
See `references/example-ap-invoices.md`.
