---
name: validate-process-consistency
description: Validate consistency between process artifacts and architecture references before writing procedures or stories.
---

# Skill: validate-process-consistency

**Version:** 0.1.0

---

## Purpose

Validate consistency between procedures, stories, and architecture artifacts before any artifact is written to disk. Mandatory gate — no artifact is written if any blocking check fails.

---

## Interface

### Input

| Parameter                | Type     | Required | Description                                                                                  |
| ------------------------ | -------- | -------- | -------------------------------------------------------------------------------------------- |
| `architecture_artifacts` | `object` | Yes      | Parsed `spec/architecture/` — service decomposition, component diagrams, interface contracts |
| `procedures`             | `array`  | No       | Procedure entries being written or updated in this session                                   |
| `stories`                | `array`  | No       | Story entries being written or updated in this session                                       |
| `existing_procedures`    | `array`  | No       | Current `spec/process/procedures/` — for cross-reference checks                              |
| `existing_stories`       | `array`  | No       | Current `spec/process/stories/` — for dependency and reference checks                        |

### Output

```json
{
  "status": "pass" | "fail",
  "blocking_failures": [
    {
      "check": "<check name>",
      "severity": "blocking",
      "detail": "<what failed and where>"
    }
  ],
  "warnings": [
    {
      "check": "<check name>",
      "severity": "warning",
      "detail": "<what was flagged>"
    }
  ]
}
```

- `status: pass` — all blocking checks passed; warnings may still be present.
- `status: fail` — one or more blocking checks failed; no artifacts are written.

---

## Checks

### Group 1 — Procedure Integrity (Blocking)

| Check                              | Rule                                                                                                                            | Failure message                                                     |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| **Architecture reference present** | Every procedure must reference at least one architecture artifact (component diagram, interface contract, or ADR)               | `PROC-<id> has no architecture reference`                           |
| **Steps are non-empty**            | Every procedure must have at least one step                                                                                     | `PROC-<id> has no steps defined`                                    |
| **Verification is non-empty**      | Every procedure must have a verification section                                                                                | `PROC-<id> has no verification method defined`                      |
| **Referenced components exist**    | Every architectural component named in a procedure must exist in `spec/architecture/services/<service-id>/component-diagram.md` | `PROC-<id> references unknown component "<name>"`                   |
| **No code or framework specifics** | Procedure steps must not contain language-specific syntax, framework names, or package names                                    | `PROC-<id> contains code-level detail in steps — move to Dev layer` |

### Group 2 — Story Integrity (Blocking)

| Check                              | Rule                                                                                           | Failure message                                                          |
| ---------------------------------- | ---------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| **Architecture reference present** | Every story must reference at least one architecture artifact                                  | `STORY-<id> has no architecture reference`                               |
| **Acceptance Criteria non-empty**  | Every story must have at least one AC                                                          | `STORY-<id> has no Acceptance Criteria`                                  |
| **AC is verifiable**               | Each AC must specify a given/when/then or equivalent input→output structure                    | `STORY-<id> AC-<n> is not verifiable — missing input or expected output` |
| **Out of Scope non-empty**         | Every story must have an explicit Out of Scope section                                         | `STORY-<id> is missing Out of Scope`                                     |
| **Service exists**                 | The `service` field must match an entry in `spec/architecture/system/service-decomposition.md` | `STORY-<id> references unknown service "<name>"`                         |
| **No implementation steps**        | Stories must not contain implementation instructions                                           | `STORY-<id> contains implementation steps — move to Dev layer`           |

### Group 3 — Cross-Reference Integrity (Blocking)

| Check                              | Rule                                                                           | Failure message                                                   |
| ---------------------------------- | ------------------------------------------------------------------------------ | ----------------------------------------------------------------- |
| **Procedure references valid**     | Every PROC-<id> referenced in a Story must exist in `spec/process/procedures/` | `STORY-<id> references unknown PROC-<id>`                         |
| **Story dependencies valid**       | Every STORY-<id> listed as a dependency must exist in `spec/process/stories/`  | `STORY-<id> depends on unknown STORY-<id>`                        |
| **No circular story dependencies** | Story dependency graph must be acyclic                                         | `Circular dependency detected: STORY-<A> → STORY-<B> → STORY-<A>` |

### Group 4 — Chesterton Check (Blocking)

| Check                             | Rule                                                                                  | Failure message                                      |
| --------------------------------- | ------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| **Reason field on modifications** | Any procedure or story modified from a previous version must include a `reason` field | `Modified artifact "<id>" is missing a reason field` |

### Group 5 — Warnings (Non-Blocking)

| Check                               | Condition                                                                               | Warning message                                                    |
| ----------------------------------- | --------------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| **No procedure reference in story** | Story has no PROC reference and does not explicitly state "No applicable procedure yet" | `⚠️ STORY-<id> has no procedure reference — consider creating one` |
| **Deprecated procedure referenced** | A story references a procedure with `Status: Deprecated`                                | `⚠️ STORY-<id> references deprecated PROC-<id>`                    |
| **Open CR blocker**                 | A story or procedure is marked with an open CR blocker                                  | `⚠️ <id> is blocked by open CR-<id>`                               |
| **Single-AC story**                 | A story has only one AC — may indicate under-specified scope                            | `⚠️ STORY-<id> has only one AC — verify scope is complete`         |

---

## Behavior

1. Run all blocking checks first. Collect all failures before returning.
2. Run all warning checks.
3. Return output object.
4. If `status: fail`:
   - Agent must not write any artifact.
   - Agent presents all `blocking_failures` to user with plain-language explanations.
   - Agent asks how to resolve before retrying.
5. If `status: pass` with warnings:
   - Agent may write artifacts.
   - Agent presents warnings to user after writing.

---

## Error Codes

| Code                            | Meaning                                                    |
| ------------------------------- | ---------------------------------------------------------- |
| `ERR_NO_ARCHITECTURE_ARTIFACTS` | `architecture_artifacts` is null or missing required files |
| `ERR_NOTHING_TO_VALIDATE`       | Both `procedures` and `stories` are null or empty          |
