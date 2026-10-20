# Output Template

```markdown
# DT Change Plan: <target_dynamic_table>

## 1. Summary
- PRD: <prd_path> (sheets: <list>)
- Target: <fqn>
- Verdict: Ready | Ready with defaults | Blocked — <one line why>

## 2. Current State
<sources unioned, key transforms, target lag, refresh mode — or "Not inspected: <reason>">

## 3. New Source Systems
| Source | Request ID | Cadence | Bronze table | Go-live | Status |
|---|---|---|---|---|---|

## 4. Column Mapping Changes
| Source | Source col | Target col | Transform | Change | Ref |
|---|---|---|---|---|---|

## 5. Business Rule Changes
| Rule | Effect on DT SQL | Bucket |
|---|---|---|

## 6. Proposed DT Change Outline
- <edit> [OQ-n if dependent]

## 7. Assumptions
| ID | Assumption | Why | Verify with |
|---|---|---|---|

## 8. Open Questions
| ID | Question | Ref | Blocking? | Owner | Proposed default |
|---|---|---|---|---|---|

## 9. Validation Plan
- <check>

## 10. Next Step
<action>
```
