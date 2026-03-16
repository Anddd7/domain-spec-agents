---
name: draft-test-plan
description: Draft story-level or release-level QA test plans from Story ACs, supplementary acceptance conditions, and impact boundaries. Use when preparing black-box validation before test code generation.
---

# Skill: draft-test-plan

**Version:** 0.1.0

---

## Purpose

Produce a structured Test Plan that covers Story ACs, QA supplementary conditions, and impact-driven regression scope.

---

## Interface

### Input

| Parameter                  | Type     | Required | Description                            |
| -------------------------- | -------- | -------- | -------------------------------------- |
| `mode`                     | `string` | Yes      | `story` or `release`                   |
| `story_or_release_id`      | `string` | Yes      | Story ID or release identifier         |
| `validation_baseline_ac`   | `array`  | Yes      | Story AC list copied verbatim          |
| `supplementary_conditions` | `array`  | No       | QA-defined conditions beyond Story ACs |
| `impact_boundary`          | `array`  | Yes      | Output from `perform-impact-analysis`  |
| `existing_test_plan`       | `object` | No       | Existing test plan for update flows    |

### Output

```json
{
  "status": "ok" | "error",
  "test_plan_id": "TESTPLAN-<story-id | release-version>",
  "artifact": "<markdown string>",
  "coverage_summary": {
    "ac_covered": true,
    "supplementary_covered": true,
    "regression_boundary_covered": true
  },
  "error": "<error message or null>"
}
```

---

## Artifact Structure

```markdown
# TESTPLAN-<story-id | release-version>: <Title>

**Story / Release:** STORY-<id> | Release <version>
**Mode:** story | release
**Version:** <semver>
**Status:** Draft | Confirmed | Superseded

## Validation Baseline

<List Story ACs verbatim>

## Supplementary Acceptance Conditions

<QA-defined conditions>

## Impact Boundary

<Components/flows and regression requirement>

## Test Cases

<TC list with coverage mapping>

## Completion Criteria

- All test cases status: pass
- No open CRs issued by this Test Plan
- [release mode] All Stories in release scope are QA-approved
```

---

## Process

1. Load validation baseline ACs and preserve wording verbatim.
2. Add supplementary conditions only when needed for integration-level confidence.
3. Expand impact boundary entries into regression scope.
4. Generate test cases that cover:
   - every Story AC,
   - every supplementary condition,
   - every impact-boundary entry marked regression-required.
5. Assign test type per rule table.
6. Populate completion criteria according to mode.

---

## Test Type Selection Rules

| Story contains                          | Test types to include                 |
| --------------------------------------- | ------------------------------------- |
| Functional requirement                  | Functional integration test, E2E test |
| API or service interaction              | Contract test                         |
| Shared module or cross-Story dependency | Regression test                       |
| Explicit performance requirement        | Performance test                      |
| Explicit security requirement           | Security test                         |

Do not add performance or security tests unless explicitly required by Story scope. In release mode, include them if any Story in release scope requires them.

---

## Rules

- Every Story AC must be covered by at least one test case.
- Every supplementary condition must be covered by at least one test case.
- Every impact-boundary item marked regression-required must be covered by at least one regression case.
- `Completion Criteria` section is mandatory.

---

## Validation

| Check                       | Rule                                               |
| --------------------------- | -------------------------------------------------- |
| Baseline AC fidelity        | ACs are copied verbatim from Story                 |
| AC coverage                 | All ACs have at least one mapped test case         |
| Supplementary coverage      | All supplementary conditions have mapped test case |
| Regression coverage         | All required boundary items have regression cases  |
| Completion criteria present | Section exists and is non-empty                    |

---

## Error Codes

| Code                      | Meaning                                            |
| ------------------------- | -------------------------------------------------- |
| `ERR_MODE_INVALID`        | `mode` is not `story` or `release`                 |
| `ERR_NO_BASELINE_AC`      | Validation baseline AC list is empty               |
| `ERR_NO_IMPACT_BOUNDARY`  | Impact boundary is missing                         |
| `ERR_INCOMPLETE_COVERAGE` | One or more required coverage mappings are missing |
