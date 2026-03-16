---
name: emit-bug-change-request
description: Generate bug-focused internal change requests from QA to Dev for failed black-box test cases, including reproducible evidence and severity.
---

# Skill: emit-bug-change-request

**Version:** 0.2.0

---

## Purpose

Create a structured bug CR for failed QA test cases and write it to `spec/change-requests/`.
This skill is dedicated to QA-to-Dev bug feedback.

## Input

| Parameter             | Type     | Required | Description                                        |
| --------------------- | -------- | -------- | -------------------------------------------------- |
| `type`                | `string` | Yes      | Must be `"bug"`                                    |
| `from_layer`          | `string` | Yes      | Must be `"qa"`                                     |
| `to_layer`            | `string` | Yes      | Must be `"dev"`                                    |
| `test_plan_id`        | `string` | Yes      | Related test plan ID                               |
| `test_case_ids`       | `array`  | Yes      | One or more failing test case IDs                  |
| `story_or_release`    | `string` | Yes      | Story ID or release identifier                     |
| `observed_behavior`   | `string` | Yes      | Actual observed behavior                           |
| `expected_behavior`   | `string` | Yes      | Expected behavior                                  |
| `reproduction_steps`  | `array`  | Yes      | Ordered reproduction steps                         |
| `severity`            | `string` | Yes      | `critical`, `high`, `medium`, or `low`             |
| `violated_scope`      | `string` | No       | Violated Story AC or supplementary condition       |
| `possible_root_cause` | `string` | No       | Suspected ambiguity in Story/procedure/constraints |

## Output

```json
{
  "status": "ok" | "error",
  "cr_id": "CR-<YYYYMMDD>-<NNN>-qa-bug",
  "file_path": "spec/change-requests/<cr_id>.md",
  "error": "<error message or null>"
}
```

## Process

1. Validate layer/type fields (`type=bug`, `from_layer=qa`, `to_layer=dev`).
2. Group failing test cases by distinct root cause.
3. Generate one CR per root cause group; include all related `test_case_ids`.
4. Write each CR under `spec/change-requests/`.
5. Return generated CR identifiers and file paths.

## CR File Format

```markdown
# Change Request <cr_id>

**Date:** <YYYY-MM-DD>
**Type:** Bug
**From:** QA
**To:** Dev
**Status:** Open
**Test Plan:** TESTPLAN-<id>
**Scope:** STORY-<id> | Release <version>
**Severity:** critical | high | medium | low

---

## Failing Test Cases

- TC-<n>

## Violated Condition

<Story AC or supplementary condition>

## Observed Behavior

<observed behavior>

## Expected Behavior

<expected behavior>

## Reproduction Steps

1. <step>

## Possible Root Cause

<optional note for ambiguous Story/procedure/constraint>
```

## Rules

- Reject input if `type` is not `bug`.
- Reject input if `from_layer` is not `qa` or `to_layer` is not `dev`.
- Create one CR per distinct root cause; include multiple `test_case_ids` when failures share the same bug.
- Include reproducible evidence and clear expected vs observed behavior.

## Validation

| Check                   | Rule                                                |
| ----------------------- | --------------------------------------------------- |
| Required fields present | Required input fields must not be null or empty     |
| Layer routing           | Must be `qa -> dev`                                 |
| Severity enum           | Must be one of `critical`, `high`, `medium`, `low`  |
| Reproducibility         | `reproduction_steps` must contain at least one step |

## Error Codes

| Code                   | Meaning                                     |
| ---------------------- | ------------------------------------------- |
| `ERR_WRONG_TYPE`       | `type` is not `bug`                         |
| `ERR_WRONG_ROUTING`    | `from_layer` or `to_layer` value is invalid |
| `ERR_MISSING_FIELD`    | Required field is null or empty             |
| `ERR_INVALID_SEVERITY` | Severity is outside allowed enum            |
