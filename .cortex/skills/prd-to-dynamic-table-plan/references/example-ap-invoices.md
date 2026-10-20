# Example: AP Invoices — Baan + Workday onboarding

## Invocation
```
$prd-to-dynamic-table-plan
prd_path: assets/ap_invoices_requirements.xlsx
target_dynamic_table: COCO_WORKSHOP.PIPELINE_LAB.SILVER_AP_INVOICES
```
Workbook sheets: `Source Onboarding`, `Column Mapping`, `Business Rules`.

## Abbreviated output

**1. Summary** — Verdict: *Ready with defaults* for Baan; *Blocked* for Workday (DPA-2025-0041 in legal review).

**2. Current State** — UNION ALL of BRONZE_SAP_AP_INVOICES and BRONZE_ORACLE_AP_INVOICES; SOURCE_SYSTEM literal per branch; TARGET_LAG = DOWNSTREAM.

**3. New Source Systems**
| Source | Request ID | Cadence | Bronze table | Go-live | Status |
|---|---|---|---|---|---|
| Baan IV | SRC-2025-003 | Nightly ~02:00 UTC | not found — verify | Q3 2025 | Approved |
| Workday | SRC-2025-004 | Hourly | not found — verify | Q3 2025 | Pending legal |

**5. Business Rule Changes (excerpt)**
| Rule | Effect on DT SQL | Bucket |
|---|---|---|
| BR-001 | Add POSTED→APPROVED, In Review→PENDING to CASE | Confirmed |
| BR-003 | QUALIFY ROW_NUMBER() on INVOICE_NUMBER | Conflict (scope) |
| BR-004 | >$500K flag | Out of layer (DMF) |
| BR-005 | Payment terms normalization | Needs decision |

**6. Outline (excerpt)**
- Add `'BAAN'` branch mapping BAN_* → Silver columns; drop BAN_COMPANY.
- Extend status CASE per BR-001; unmapped values → [OQ-3].
- Add QUALIFY dedup [OQ-1].
- Pass PAYMENT_TERMS through unchanged [OQ-2].

**8. Open Questions (excerpt)**
| ID | Question | Ref | Blocking? | Owner | Proposed default |
|---|---|---|---|---|---|
| OQ-1 | Dedup per source or global? Tie-breaker on equal CREATED_AT? | BR-003 | Yes | Karen van der Berg | PARTITION BY SOURCE_SYSTEM, INVOICE_NUMBER; tie-break INVOICE_ID |
| OQ-2 | Normalize payment terms at Silver or Gold? | BR-005 | No | Sarah Chen / David Kim | Leave as-is at Silver |
| OQ-3 | How to handle unlisted status values? | BR-001 | Yes | Sarah Chen | Map to NULL and alert |
| OQ-4 | Baan amounts "always positive" vs negative credit memos? | Mapping BAN_AMOUNT | Yes | Karen van der Berg | Keep sign as sent |
| OQ-5 | BR-004 needs USD equivalent but FX is out of scope | BR-002/BR-004 | No (DMF) | Tom Walsh | Threshold per native currency |

**10. Next Step** — Resolve OQ-1, OQ-3, OQ-4, then hand the approved plan to the `dynamic-tables` skill to implement the Baan branch.
