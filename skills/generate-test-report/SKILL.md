---
name: generate-test-report
description: Generate structured QA test reports from execution outcomes, including summary metrics, case-level status, issued CR links, and delivery decision.
---

# Skill: generate-test-report

**Version:** 0.1.0

---

## Purpose

Produce standardized QA test reports from execution results for story-level or release-level validation.

---

## Interface

### Input

| Parameter            | Type     | Required | Description                                                         |
| -------------------- | -------- | -------- | ------------------------------------------------------------------- |
| `test_plan_id`       | `string` | Yes      | Source `TESTPLAN-<id>` reference                                    |
| `mode`               | `string` | Yes      | `"story"` or `"release"`                                            |
| `execution_metadata` | `object` | Yes      | Timestamp, environment, executor (`user` or `CI`)                   |
| `case_results`       | `array`  | Yes      | Per-case results: TC ID, type, status, notes, optional linked CR ID |

### Output

```json
{
  "status": "ok" | "error",
  "report_id": "TESTREPORT-<id>-<timestamp>",
  "artifact": "<markdown string>",
  "delivery_status": "ready for release" | "blocked",
  "error": "<error message or null>"
}
```

---

## Report Structure

```markdown
# TESTREPORT-<story-id | release-version>-<timestamp>: <Title>

**Test Plan:** TESTPLAN-<id>
**Executed:** <timestamp>
**Environment:** <environment description>
**Executed by:** <user | CI>

## Summary

| Total | Pass | Fail | Blocked |
| ----- | ---- | ---- | ------- |
| <n>   | <n>  | <n>  | <n>     |

## Results by Test Case

| TC ID  | Type   | Status | CR ID (if fail) | Notes   |
| ------ | ------ | ------ | --------------- | ------- | ------- | ------- |
| TC-<n> | <type> | pass   | fail            | blocked | CR-<id> | <notes> |

## Blocked Cases

<TC-level blocked reasons>

## CRs Issued

<CR references for failures>

## Delivery Status

**ready for release** | **blocked**
<Rationale>
```

---

## Process

1. Aggregate execution results into pass/fail/blocked counters.
2. Build case-level results table with CR linkage for failed cases.
3. Build blocked-case list with reasons.
4. Build CR-issued section from failed case mappings.
5. Derive delivery status:
   - `ready for release` only if all cases pass and no open CRs remain.
   - otherwise `blocked`.

---

## Validation

| Check                      | Rule                                               |
| -------------------------- | -------------------------------------------------- |
| Case completeness          | All executed cases from input are represented      |
| Status validity            | Case status must be `pass`, `fail`, or `blocked`   |
| Delivery decision validity | `ready for release` only when fail=0 and blocked=0 |

---

## Error Codes

| Code                        | Meaning                                |
| --------------------------- | -------------------------------------- |
| `ERR_NO_TEST_PLAN_ID`       | `test_plan_id` is missing              |
| `ERR_NO_EXECUTION_METADATA` | Required execution metadata is missing |
| `ERR_NO_CASE_RESULTS`       | `case_results` is empty                |
| `ERR_MODE_INVALID`          | `mode` is not `"story"` or `"release"` |
