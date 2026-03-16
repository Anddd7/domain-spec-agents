---
name: generate-test-code
description: Generate executable black-box test suites from a confirmed QA test plan with one-to-one case traceability and type-based test organization.
---

# Skill: generate-test-code

**Version:** 0.1.0

---

## Purpose

Generate QA test code from a confirmed test plan while preserving strict traceability and black-box scope.

---

## Interface

### Input

| Parameter             | Type     | Required | Description                                                           |
| --------------------- | -------- | -------- | --------------------------------------------------------------------- |
| `confirmed_test_plan` | `object` | Yes      | Confirmed `TESTPLAN-<id>` artifact                                    |
| `test_directories`    | `object` | Yes      | Mapping of test type to path (`integration`, `e2e`, `contract`, etc.) |
| `blocked_cases`       | `array`  | No       | Test case IDs blocked by open CRs                                     |

### Output

```json
{
  "status": "ok" | "error",
  "generated_files": ["<path>", "..."],
  "mapped_cases": ["TC-<n>", "..."],
  "skipped_cases": ["TC-<n>", "..."],
  "error": "<error message or null>"
}
```

---

## Process

1. Read confirmed test plan and enumerate test cases.
2. Generate test code by test type into configured directories.
3. Map each non-blocked test case to exactly one test function/block.
4. Add TC IDs as comments/annotations for traceability.
5. Skip blocked cases and record them in `skipped_cases`.

---

## Rules

- Follow the confirmed test plan exactly.
- Keep one-to-one mapping between test cases and test blocks.
- Preserve black-box behavior (no implementation internals).
- Do not generate code for blocked test cases.
- Organize tests by type using these default paths when not overridden:
  - `tests/integration/`
  - `tests/e2e/`
  - `tests/contract/`
  - `tests/regression/`
  - `tests/performance/`
  - `tests/security/`

---

## Validation

| Check               | Rule                                         |
| ------------------- | -------------------------------------------- |
| Plan status         | Test plan must be confirmed                  |
| Case traceability   | Every generated test includes a TC ID marker |
| One-to-one mapping  | Each test case maps to one test block        |
| Blocker enforcement | Blocked cases are not generated              |

---

## Error Codes

| Code                         | Meaning                                      |
| ---------------------------- | -------------------------------------------- |
| `ERR_TESTPLAN_NOT_CONFIRMED` | Test plan status is not confirmed            |
| `ERR_NO_TEST_DIR`            | Required target directory mapping is missing |
| `ERR_CASE_MAPPING_FAILED`    | One or more cases could not be mapped        |
