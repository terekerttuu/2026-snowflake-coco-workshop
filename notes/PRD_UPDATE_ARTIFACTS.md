# SILVER_AP_INVOICES — PRD-Driven Update: Artifacts to Save

Change: add Baan IV and Workday as AP invoice sources to
`COCO_WORKSHOP.PIPELINE_LAB.SILVER_AP_INVOICES` (deployed 2026-10-20; 50 rows; all checks PASS).

Status key: **Exists** = already in repo · **To save** = content exists only in the session, write it to the path shown.

## 1. Inputs (the PRD)
| Artifact | Path | Status | Why |
|---|---|---|---|
| Source onboarding requests (SRC-2025-003/004) | `assets/sample_business_requirements_source_onboarding.csv` | Exists | Scope, go-live, legal dependency |
| Column mapping | `assets/sample_business_requirements_column_mapping.csv` | Exists | Source → Silver field mapping |
| Business rules (BR-001…BR-010) | `assets/sample_business_requirements_business_rules.csv` | Exists | Rules cited in the DDL comments |

## 2. Change plan & decisions
| Artifact | Path | Status | Why |
|---|---|---|---|
| DT change plan (10-section skill output) | `docs/silver_ap_invoices/change_plan.md` | To save | What changed and why, with PRD refs |
| Assumptions & open questions log | `docs/silver_ap_invoices/open_questions.md` | To save | Payment-terms format, empty SOURCE_ENTITY_CODE, Workday legal gate, dedup scope, unmapped statuses → NULL, Baan cost-center format, credit-memo signs, BR-004 DMF not built |
| Session plan file | `.cortex/plans/plan_2026-10-20_0709.md` | Exists | Raw planning history |

## 3. Implementation
| Artifact | Path | Status | Why |
|---|---|---|---|
| Deployed DT DDL (new version) | `sql/silver/SILVER_AP_INVOICES.sql` | To save | Source of truth for review / redeploy |
| Previous DT DDL (SAP + Oracle only) | `sql/silver/SILVER_AP_INVOICES.prev.sql` | To save | Rollback and diff for reviewers |
| Deploy notes | in `change_plan.md` | To save | Run `CREATE OR ALTER`, then `ALTER DYNAMIC TABLE … REFRESH`; settings unchanged (1h lag, INCREMENTAL, COCO_WORKSHOP_WH) |

> Tip: regenerate either DDL anytime with
> `SELECT GET_DDL('TABLE','COCO_WORKSHOP.PIPELINE_LAB.SILVER_AP_INVOICES');`

## 4. Validation (rerunnable checks)
| Artifact | Path | Status | Why |
|---|---|---|---|
| Original proof query (SAP + Oracle only) | `SILVER_AP_INVOICES_proof_query.sql` | Exists — **outdated** | Replace or keep as baseline |
| V1 row-count reconciliation, 4 sources | `sql/validation/v1_row_counts.sql` | To save | Silver = Bronze per source (Baan = distinct refs) |
| V2–V4 status domain / dup keys / required NULLs | `sql/validation/v2_v4_quality.sql` | To save | Expect all zeros |
| V5 refresh health | `sql/validation/v5_refresh_history.sql` | To save | Confirm next scheduled refresh is `INCREMENTAL` |
| Baseline results from deploy | `docs/silver_ap_invoices/validation_results_2026-10-20.md` | To save | V1 PASS (SAP 15, Oracle 15, Baan 10, Workday 10); V2–V4 = 0; V5 SUCCEEDED (REINITIALIZE) |

## 5. Reusable skill (PRD Evaluator)
| Artifact | Path | Status | Why |
|---|---|---|---|
| Skill definition | `.cortex/skills/prd-to-dynamic-table-plan/SKILL.md` | Exists | Invoke with `$prd-to-dynamic-table-plan` |
| Output template | `.cortex/skills/prd-to-dynamic-table-plan/references/output-template.md` | Exists | Fixed 10-section output |
| Worked example | `.cortex/skills/prd-to-dynamic-table-plan/references/example-ap-invoices.md` | Exists — update | Replace illustrative values (XLSX name, "Bronze not found") with this real run |
| Skill improvements backlog | in `open_questions.md` or a skill TODO | To save | Accept CSV input; add `EXPLAIN CHANGES` and change-tracking checks |

## 6. Review handoff
- [ ] Commit `.cortex/`, `assets/`, `docs/`, `sql/` (all currently untracked)
- [ ] Reviewer diffs `SILVER_AP_INVOICES.prev.sql` vs `SILVER_AP_INVOICES.sql`
- [ ] Reviewer reruns V1–V5 and compares to the baseline results
- [ ] Engineering signs off open questions (payment terms, SOURCE_ENTITY_CODE, Workday legal gate)
