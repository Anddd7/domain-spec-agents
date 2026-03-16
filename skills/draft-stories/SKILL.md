---
name: draft-stories
description: Decompose feature intent into process-layer stories with acceptance criteria, architecture/procedure traceability, dependencies, and explicit scope exclusions.
---

# Skill: draft-stories

**Version:** 0.1.0

---

## Purpose

Convert feature-level intent into story artifacts that Dev can implement under architecture and procedure constraints.

---

## Interface

### Input

| Parameter                | Type     | Required | Description                                              |
| ------------------------ | -------- | -------- | -------------------------------------------------------- |
| `feature_intent`         | `string` | Yes      | Normalized feature/change description                    |
| `architecture_artifacts` | `object` | Yes      | Relevant service diagrams, contracts, and ADR references |
| `procedure_library`      | `array`  | Yes      | Existing procedures that may apply                       |
| `existing_stories`       | `array`  | No       | Existing stories for update or dependency linking        |
| `mode`                   | `string` | Yes      | `"initialize"` or `"update"`                             |

### Output

```json
{
  "status": "ok" | "error",
  "stories": [
    {
      "story_id": "STORY-<id>",
      "artifact": "<markdown string>"
    }
  ],
  "error": "<error message or null>"
}
```

---

## Artifact Structure

```markdown
# STORY-<id>: <Title>

**Status:** Draft | Ready | In Progress | Done
**Service:** <service-id>

## User Value

As a <role>, I want <capability>, so that <outcome>.

## Acceptance Criteria

<Numbered, verifiable conditions>

## Architecture References

<Relevant component diagrams, contracts, ADRs, and in-scope components>

## Procedure References

<Applicable PROC IDs or explicit "No applicable procedure yet">

## Dependencies

<Blocking STORY IDs or "None">

## Out of Scope

<Explicit exclusions>
```

---

## Process

1. Parse feature intent into independently deliverable outcomes.
2. Partition outcomes into one or more stories with clear service ownership.
3. Write ACs as verifiable conditions with observable results.
4. Attach architecture references that constrain implementation scope.
5. Attach applicable procedures; if none exist, mark the gap explicitly.
6. Define dependencies on prior stories if needed.
7. Write mandatory `Out of Scope` exclusions for each story.

---

## Rules

- Every story must include at least one architecture reference.
- Every story must include at least one procedure reference or an explicit procedure gap marker.
- `Out of Scope` is mandatory.
- Do not include implementation instructions.

---

## Validation

| Check                     | Rule                                                    |
| ------------------------- | ------------------------------------------------------- |
| AC non-empty              | Each story has at least one acceptance criterion        |
| AC verifiable             | ACs specify input/state, action, and observable outcome |
| Architecture traceability | At least one architecture reference exists              |
| Out-of-scope present      | `## Out of Scope` is non-empty                          |
| No implementation detail  | No code-level steps or framework/tool instructions      |

---

## Error Codes

| Code                        | Meaning                                     |
| --------------------------- | ------------------------------------------- |
| `ERR_NO_FEATURE_INTENT`     | `feature_intent` is null or empty           |
| `ERR_NO_ARCHITECTURE_INPUT` | Required architecture artifacts are missing |
| `ERR_MODE_INVALID`          | `mode` is not `"initialize"` or `"update"`  |
