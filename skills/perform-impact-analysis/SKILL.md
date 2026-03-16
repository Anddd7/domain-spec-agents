---
name: perform-impact-analysis
description: Identify full impact boundaries for story or release validation by combining architecture dependencies and process relationships. Use before drafting validator test plans.
---

# Skill: perform-impact-analysis

**File:** `skills/perform-impact-analysis/SKILL.md`
**Version:** 0.1.0
**Used by:** Validator

---

## Purpose

Identify the full set of components and flows potentially affected by a Story or set of Stories. The output — the **impact boundary** — defines the regression coverage scope for a Test Plan. This skill is mandatory before drafting any Test Plan.

---

## Interface

### Input

| Parameter                | Type     | Required | Description                                                                             |
| ------------------------ | -------- | -------- | --------------------------------------------------------------------------------------- |
| `stories`                | `array`  | Yes      | One or more parsed `STORY-<id>.md` objects                                              |
| `architecture_artifacts` | `object` | Yes      | Parsed `spec/architecture/` — component diagrams, dependency graph, interface contracts |
| `process_artifacts`      | `object` | Yes      | Parsed `spec/process/` — Story relationships, shared module references                  |
| `mode`                   | `string` | Yes      | `"story"` or `"release"`                                                                |

### Output

```json
{
  "impact_boundary": [
    {
      "name": "<component or flow name>",
      "type": "component" | "flow" | "interface",
      "source": "architecture-dependency" | "story-relationship" | "both",
      "directly_modified": true | false,
      "regression_required": true | false,
      "regression_rationale": "<why regression is or is not required>"
    }
  ],
  "gaps": [
    {
      "type": "missing-dependency-graph" | "missing-story-relationship" | "ambiguous-boundary",
      "detail": "<what is missing or ambiguous>"
    }
  ],
  "analysis_summary": "<plain-language summary of what was found>"
}
```

---

## Analysis Steps

### Step 1 — Identify Directly Modified Components

From each Story's scope description and AC list, identify the components and interfaces that the Story explicitly modifies or introduces.

Mark each as `directly_modified: true`.

### Step 2 — Expand via Architecture Dependency Graph

For each directly modified component, traverse the architecture dependency graph in `spec/architecture/`:

- Identify all components that **depend on** the modified component (downstream dependents).
- Identify all components that the modified component **depends on** (upstream dependencies — included only if the Story changes the interface, not just the implementation).

Mark each discovered component as `source: "architecture-dependency"`.

### Step 3 — Expand via Story Relationships

From `spec/process/`, identify:

- Other Stories that reference the same modules, interfaces, or flows as the current Story.
- Shared flows that pass through the modified components.

Mark each discovered component or flow as `source: "story-relationship"`.

### Step 4 — Merge and Deduplicate

Take the union of Steps 1, 2, and 3. Deduplicate by component / flow name. If a component appears in both Step 2 and Step 3, mark it as `source: "both"`.

### Step 5 — Determine Regression Requirement

For each item in the impact boundary:

| Condition                                               | regression_required                          |
| ------------------------------------------------------- | -------------------------------------------- |
| Directly modified                                       | `true` — functional tests required           |
| Downstream dependent (architecture)                     | `true` — regression tests required           |
| Upstream dependency (interface unchanged)               | `false` — no regression required             |
| Shared flow (story-relationship)                        | `true` — regression tests required           |
| Shared module (story-relationship, no interface change) | Judgment call — flag as warning if uncertain |

### Step 6 — Identify Gaps

Flag any of the following as gaps:

- A component is referenced in a Story AC but does not appear in the architecture dependency graph.
- A Story references a shared module but no Story relationship is defined in `spec/process/`.
- A component boundary is ambiguous (multiple possible interpretations of scope).

Gaps are returned in the `gaps` array and must be presented to the user before the Test Plan is drafted.

---

## Release Mode Behavior

When `mode: "release"`:

1. Run Steps 1–6 for each Story in the release scope independently.
2. Take the union of all impact boundaries across all Stories.
3. Deduplicate. If a component appears in multiple Story analyses, it appears once with `source` reflecting all contributing sources.
4. A component marked `regression_required: true` by any Story in the release scope retains that marking in the merged boundary.
5. Return a single merged impact boundary covering the full release scope.

---

## Gap Handling

If the `gaps` array is non-empty:

1. Validator presents all gaps to the user in plain language.
2. Validator asks the user to resolve each gap before proceeding.
3. Validator does not draft the Test Plan until all gaps are resolved or explicitly accepted by the user.

If the user accepts a gap without resolving it, the Validator notes the accepted gap in the Test Plan under a "Known Coverage Gaps" section and flags it as a warning in the Test Report.
