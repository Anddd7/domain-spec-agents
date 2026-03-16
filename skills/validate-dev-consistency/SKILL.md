# Skill: validate-dev-consistency

**File:** `skills/validate-dev-consistency/SKILL.md`
**Version:** 0.2.0
**Used by:** Dev Assistant

---

## Purpose

Validate consistency between Implementation Plan, generated code, Story ACs, architecture constraints, and code quality standards (SOLID, Clean Code, style) before any artifact is written to disk. Mandatory gate — no artifact is written if any blocking check fails.

---

## Interface

### Input

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `story` | `object` | Yes | Parsed `STORY-<id>.md` — ACs, scope, references |
| `architecture_artifacts` | `object` | Yes | Parsed `spec/architecture/` — principles, component diagrams, interface contracts |
| `procedures` | `array` | Yes | Referenced `PROC-<id>.md` entries |
| `plan` | `object` | Yes | Implementation Plan being validated |
| `code_artifacts` | `array` | No | Generated code and test files (if validating post-generation) |

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

---

## Checks

### Group 1 — Plan Integrity (Blocking)

| Check | Rule | Failure message |
|-------|------|----------------|
| **Full AC coverage in steps** | Every Story AC must be addressed by at least one implementation step | `PLAN-<id> step coverage missing for AC-<n>` |
| **Full AC coverage in tests** | Every Story AC must be addressed by at least one unit test case | `PLAN-<id> test coverage missing for AC-<n>` |
| **No out-of-scope steps** | Every implementation step must map to at least one Story AC | `PLAN-<id> step "<name>" has no AC mapping — out of scope` |
| **Open Blockers declared** | Any AC that cannot be implemented due to an open CR must be listed in Open Blockers | `PLAN-<id> has unresolved conflict on AC-<n> with no blocker declared` |
| **Technical decisions documented** | Plan must contain at least one Technical Decision entry | `PLAN-<id> has no technical decisions documented` |
| **SOLID annotations present** | Each implementation step must note which SOLID principle(s) apply and how | `PLAN-<id> step "<name>" is missing SOLID annotation` |

### Group 2 — Architecture Compliance (Blocking)

| Check | Rule | Failure message |
|-------|------|----------------|
| **No boundary violations** | Implementation steps must not describe direct calls across component boundaries that bypass defined interface contracts | `PLAN-<id> step "<name>" crosses component boundary without defined contract` |
| **Principle compliance** | Implementation steps must not contradict architectural principles stated in `spec/architecture/` | `PLAN-<id> step "<name>" conflicts with architectural principle "<principle>"` |
| **Interface contracts respected** | Any interface used in implementation steps must exist in `spec/architecture/` | `PLAN-<id> references undefined interface contract "<name>"` |

### Group 3 — Code Integrity (Blocking, post-generation only)

| Check | Rule | Failure message |
|-------|------|----------------|
| **Plan-code alignment** | Every implementation step in the confirmed Plan must have a corresponding code artifact | `PLAN-<id> step "<name>" has no corresponding code artifact` |
| **Test co-location** | Unit test files must be co-located with the module they test | `Test file for "<module>" is not co-located` |
| **No blocked AC implemented** | Code must not exist for any AC marked as blocked by an open CR | `Code generated for blocked AC-<n> in PLAN-<id>` |
| **No cross-boundary direct calls** | Code must not contain direct calls across component boundaries that bypass defined interfaces | `"<file>" contains cross-boundary call to "<component>" without interface contract` |

### Group 4 — SOLID Compliance (Blocking, post-generation only)

| Check | Rule | Failure message |
|-------|------|----------------|
| **Single Responsibility** | No class or module implements more than one distinct responsibility | `"<file>" class "<name>" has multiple responsibilities — split required` |
| **Dependency Inversion** | Business logic classes must not instantiate concrete dependencies internally — dependencies must be injected | `"<file>" class "<name>" instantiates concrete dependency "<type>" internally` |
| **Interface Segregation** | Interfaces must not contain methods unused by any single implementor | `"<file>" interface "<name>" contains methods not used by implementor "<class>"` |
| **Open/Closed** | Existing stable classes must not be modified to add new behavior — extension points must be used | `"<file>" modification to stable class "<name>" violates Open/Closed — use extension` |

### Group 5 — Clean Code Compliance (Blocking, post-generation only)

| Check | Rule | Failure message |
|-------|------|----------------|
| **No magic values** | Numeric or string literals used in logic must be named constants | `"<file>" contains magic value "<value>" at line <n>` |
| **No dead code** | No unreachable code, unused variables, or commented-out code blocks | `"<file>" contains dead code at line <n>` |
| **No unexplained side effects** | Functions with side effects must declare them in their signature or docstring | `"<file>" function "<name>" has undeclared side effect` |
| **Intention-revealing names** | Names must not require inline comments to explain their meaning | `"<file>" identifier "<name>" is not intention-revealing — rename required` |

### Group 6 — Style Compliance (Blocking, post-generation only)

| Check | Rule | Failure message |
|-------|------|----------------|
| **Consistent formatting** | Code must follow consistent indentation, line length, and bracket style throughout the module | `"<file>" has inconsistent formatting at line <n>` |
| **Idiomatic error handling** | Error handling must follow the idiomatic pattern for the target language, applied consistently | `"<file>" uses non-idiomatic error handling at line <n>` |
| **No unnecessary abstractions** | No interfaces, base classes, or wrappers introduced without a concrete reason traceable to the Plan | `"<file>" introduces unnecessary abstraction "<name>"` |

### Group 7 — Chesterton Check (Blocking)

| Check | Rule | Failure message |
|-------|------|----------------|
| **Reason field on modifications** | Any Plan or code artifact modified from a previous version must include a `reason` field in the Plan header | `Modified artifact "PLAN-<id>" is missing a reason field` |

### Group 8 — Warnings (Non-Blocking)

| Check | Condition | Warning message |
|-------|-----------|----------------|
| **Missing procedure reference** | A Plan implements a pattern with no corresponding procedure | `⚠️ PLAN-<id> implements pattern "<name>" with no procedure coverage — consider emitting CR to Process layer` |
| **Single test case per AC** | An AC has only one unit test case | `⚠️ PLAN-<id> AC-<n> has only one test case — verify edge cases are covered` |
| **Open CR blocker present** | Plan has one or more open CR blockers | `⚠️ PLAN-<id> is partially blocked by CR-<id>` |
| **Deprecated procedure referenced** | Story references a deprecated procedure | `⚠️ PLAN-<id> references deprecated PROC-<id>` |
| **Long function** | A function exceeds 30 lines — may indicate a Single Responsibility violation | `⚠️ "<file>" function "<name>" is <n> lines — consider splitting` |
| **Deep nesting** | A function contains more than 3 levels of nesting — may indicate complex control flow | `⚠️ "<file>" function "<name>" has <n> nesting levels — consider extracting` |

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

| Code | Meaning |
|------|---------|
| `ERR_NO_STORY` | `story` parameter is null or missing ACs |
| `ERR_NO_ARCHITECTURE_ARTIFACTS` | `architecture_artifacts` is null or missing required files |
| `ERR_NO_PLAN` | `plan` parameter is null or empty |