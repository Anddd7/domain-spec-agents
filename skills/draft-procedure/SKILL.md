---
name: draft-procedure
description: Draft or update reusable process procedures from architecture and workflow intent. Use when creating procedure definitions that describe module interactions, sequencing, and verification without implementation details.
---

# Skill: draft-procedure

**Version:** 0.1.0

---

## Purpose

Draft or update process procedures that define reusable workflow rules across modules. Procedures describe interaction order and verification expectations, not coding instructions.

---

## Interface

### Input

| Parameter                | Type     | Required | Description                                                          |
| ------------------------ | -------- | -------- | -------------------------------------------------------------------- |
| `procedure_intent`       | `string` | Yes      | Normalized description of the procedure scenario and purpose         |
| `architecture_artifacts` | `object` | Yes      | Relevant component diagrams, interface contracts, and ADR references |
| `existing_procedure`     | `object` | No       | Existing PROC artifact for update flows                              |
| `mode`                   | `string` | Yes      | `"initialize"` or `"update"`                                         |

### Output

```json
{
  "status": "ok" | "error",
  "procedure_id": "PROC-<id>",
  "artifact": "<markdown string>",
  "error": "<error message or null>"
}
```

---

## Artifact Structure

```markdown
# PROC-<id>: <Name>

**Version:** <semver>
**Status:** Active | Deprecated
**Applies to:** <service-id or "cross-service">

## When to Use

<Scenario and preconditions>

## Modules and Architecture References

<Components in interaction order with links to architecture artifacts>

## Steps

<Ordered high-level operations describing module interactions and state transitions>

## Verification

<How to verify correct execution: test categories, pipeline checks, observable outcomes>

## Related Procedures

<Related PROC IDs>
```

---

## Process

1. Parse the procedure intent and determine whether this is a new procedure or an update.
2. If updating, apply Chesterton context by preserving the existing rationale and dependencies.
3. Select applicable architectural components and contracts that constrain this procedure.
4. Draft `When to Use` with explicit preconditions and triggers.
5. Draft `Modules and Architecture References` in interaction order.
6. Draft `Steps` as module-level operations and expected state transitions.
7. Draft `Verification` with non-framework-specific test categories and observable outcomes.
8. Add related procedure references when applicable.

---

## Rules

- Steps must describe interactions and outcomes, not code-level implementation.
- Avoid language/framework/package specifics.
- Every procedure must reference at least one architecture artifact.
- Verification section is mandatory.

---

## Validation

| Check                          | Rule                                                       |
| ------------------------------ | ---------------------------------------------------------- |
| Architecture reference present | At least one architecture artifact is referenced           |
| Steps present                  | `## Steps` contains at least one ordered item              |
| Verification present           | `## Verification` is non-empty                             |
| No implementation detail       | No code snippets, framework APIs, or package/tool commands |

---

## Error Codes

| Code                        | Meaning                                     |
| --------------------------- | ------------------------------------------- |
| `ERR_NO_INTENT`             | `procedure_intent` is null or empty         |
| `ERR_NO_ARCHITECTURE_INPUT` | Required architecture artifacts are missing |
| `ERR_MODE_INVALID`          | `mode` is not `"initialize"` or `"update"`  |
