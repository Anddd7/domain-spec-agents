---
name: validate-architecture-consistency
description: Validate consistency and traceability across architecture artifacts before write operations. Use as the architecture-layer quality gate.
---

# Skill: validate-architecture-consistency

**Version:** 0.1.0

---

## Purpose

Validate consistency within and across all architecture artifacts before they are written to disk. This skill is a mandatory gate — no artifact is written if any blocking check fails.

---

## Interface

### Input

| Parameter          | Type     | Required | Description                                                                        |
| ------------------ | -------- | -------- | ---------------------------------------------------------------------------------- |
| `domain_artifacts` | `object` | Yes      | Parsed `spec/domain/` artifacts — glossary, four-color model, context map          |
| `system_artifacts` | `object` | Yes      | Parsed `spec/architecture/system/` artifacts                                       |
| `app_artifacts`    | `object` | No       | Parsed `spec/architecture/services/<service-id>/` artifacts — validated if present |
| `nfr_analysis`     | `object` | No       | Parsed NFR analysis — validated if present                                         |
| `adrs`             | `array`  | No       | ADR entries generated in the current session — validated if present                |

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
- `status: fail` — one or more blocking checks failed; artifacts must not be written.

---

## Checks

### Group 1 — Domain Alignment (Blocking)

| Check                          | Rule                                                                                                                                                                                  | Failure message                                                        |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| **Service name in glossary**   | Every service name in `service-decomposition.md` must appear as a term in `spec/domain/glossary.md`                                                                                   | `Service "<name>" has no Glossary entry`                               |
| **Component name in glossary** | Every component name in any `component-diagram.md` must appear in the Glossary or be a standard layer name (API Layer, Domain Logic Layer, Persistence Layer, External Adapter Layer) | `Component "<name>" has no Glossary entry and is not a standard layer` |
| **Bounded Context coverage**   | Every Bounded Context in `spec/domain/context-map.md` must map to at least one service in `service-decomposition.md`                                                                  | `Bounded Context "<name>" has no corresponding service`                |

### Group 2 — Structural Integrity (Blocking)

| Check                               | Rule                                                                                                                                                                                                       | Failure message                                                      |
| ----------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| **No orphan services**              | Every service in `service-decomposition.md` must appear in at least one edge in `communication-map.md` (as producer or consumer) or be explicitly marked as a standalone service with documented rationale | `Service "<name>" has no defined relationships in communication map` |
| **Interface contract completeness** | Every inter-service dependency edge in `communication-map.md` must have a corresponding Interface Contract in the consuming or producing service's `interface-contracts.md`                                | `Dependency "<from> → <to>" has no interface contract`               |
| **No database sharing**             | No two services may reference the same persistence component by name                                                                                                                                       | `Services "<A>" and "<B>" share persistence component "<name>"`      |

### Group 3 — NFR Traceability (Blocking)

| Check                        | Rule                                                                                                                                                               | Failure message                                                              |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------- |
| **NFR linked to decision**   | Every NFR with `status: resolved` must reference at least one ADR or an existing architectural element                                                             | `NFR-<id> is marked resolved but has no linked ADR or architectural element` |
| **Infrastructure justified** | Every infrastructure component (gateway, broker, observability stack, etc.) in `system-context.md` must be traceable to at least one NFR or functional requirement | `Infrastructure component "<name>" has no traceable requirement`             |

### Group 4 — ADR Completeness (Blocking)

| Check                   | Rule                                                                                         | Failure message                                            |
| ----------------------- | -------------------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| **ADR required fields** | Every ADR must contain: Status, Context, Decision, Consequences, Requirement Traceability    | `ADR-<id> is missing required field: <field>`              |
| **ADR for deviations**  | Any deviation from standard layers in application architecture must have a corresponding ADR | `Non-standard layer "<name>" in service "<id>" has no ADR` |

### Group 5 — Chesterton Check (Blocking)

| Check                             | Rule                                                                                                      | Failure message                                       |
| --------------------------------- | --------------------------------------------------------------------------------------------------------- | ----------------------------------------------------- |
| **Reason field on modifications** | Any element modified from a previous version must include a `reason` field explaining the prior rationale | `Modified element "<name>" is missing a reason field` |

### Group 6 — Warnings (Non-Blocking)

| Check                            | Condition                                          | Warning message                                           |
| -------------------------------- | -------------------------------------------------- | --------------------------------------------------------- |
| **Circular dependency**          | A cycle is detected in `communication-map.md`      | `⚠️ Circular dependency detected: <A> → <B> → <A>`        |
| **Implicit NFR unconfirmed**     | Any NFR with `source: implicit` and `status: open` | `⚠️ Implicit NFR-<id> has not been confirmed by user`     |
| **Low-confidence flags present** | Any `⚠️` flags emitted by upstream skills          | `⚠️ Low-confidence decision in <artifact>: <description>` |
| **Missing SLA on interface**     | An interface contract has no SLA defined           | `⚠️ Interface "<name>" has no SLA defined`                |

---

## Behavior

1. Run all blocking checks first. Collect all failures before returning — do not stop at the first failure.
2. Run all warning checks.
3. Return the output object.
4. If `status: fail`:
   - The agent must not write any artifact to disk.
   - The agent must present all `blocking_failures` to the user with plain-language explanations.
   - The agent must ask the user how to resolve each failure before retrying.
5. If `status: pass` with warnings:
   - The agent may write artifacts to disk.
   - The agent must present all warnings to the user after writing.

---

## Validation of This Skill's Own Inputs

| Check                      | Rule                                                                        |
| -------------------------- | --------------------------------------------------------------------------- |
| `domain_artifacts` present | Must contain at least `glossary.md` and `context-map.md`                    |
| `system_artifacts` present | Must contain at least `service-decomposition.md` and `communication-map.md` |

---

## Error Codes

| Code                      | Meaning                                              |
| ------------------------- | ---------------------------------------------------- |
| `ERR_NO_DOMAIN_ARTIFACTS` | `domain_artifacts` is null or missing required files |
| `ERR_NO_SYSTEM_ARTIFACTS` | `system_artifacts` is null or missing required files |
