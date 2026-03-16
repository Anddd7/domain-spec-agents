---
name: draft-system-architecture
description: Draft system-level architecture artifacts from domain contexts and requirements. Use when defining service decomposition, communication topology, and system context.
---

# Skill: draft-system-architecture

**Version:** 0.1.0

---

## Purpose

Translate domain Bounded Contexts and NFR inputs into a system-level architecture specification. Produces three artifacts: System Context Diagram, Service Decomposition, and Inter-Service Communication Map.

This skill operates at the **inter-process level only**. It defines what services exist, how they communicate, and what cross-cutting infrastructure is required. It does not define intra-process structure.

---

## Interface

### Input

| Parameter                   | Type     | Required | Description                                                                                        |
| --------------------------- | -------- | -------- | -------------------------------------------------------------------------------------------------- |
| `domain_artifacts`          | `object` | Yes      | Parsed content of `spec/domain/` — glossary, four-color model, context map, event map              |
| `nfr_analysis`              | `object` | No       | Output of `draft-nfr-analysis`, if available — used to inform scaling and infrastructure decisions |
| `existing_system_artifacts` | `object` | No       | Existing `spec/architecture/system/` artifacts — required when updating, null on initialization    |
| `mode`                      | `string` | Yes      | `"initialize"` or `"update"`                                                                       |

### Output

```json
{
  "status": "ok" | "error",
  "artifacts": {
    "system_context": "<markdown string>",
    "service_decomposition": "<markdown string>",
    "communication_map": "<markdown string>"
  },
  "adrs": ["<markdown string>", "..."],
  "low_confidence_flags": ["<description>", "..."],
  "error": "<error message or null>"
}
```

---

## Process

### Step 1 — Map Bounded Contexts to Candidate Services

For each Bounded Context in the domain Context Map:

1. Create a candidate service with the same name and responsibility.
2. Apply **Occam's Razor**: evaluate whether two or more contexts can be merged into one service.

Merge criteria (all must apply):

- No independent scaling requirement between them
- No independent deployment lifecycle between them
- No team ownership boundary between them

If merged: record the merge decision as an ADR with rationale.
If kept separate: record the separation rationale in the service decomposition table.

### Step 2 — Define Inter-Service Communication

For each inter-context relationship in the domain Context Map:

1. Determine communication style:
   - **Synchronous (RESTful HTTP):** when the caller requires an immediate response to proceed
   - **Asynchronous (event/message):** when the caller can proceed without waiting, or when multiple consumers exist
2. Identify the direction: producer → consumer.
3. Flag any circular dependencies as `⚠️` — these require architectural review.

Default rules:

- No direct database sharing across services. Ever.
- Read-heavy cross-service data access: prefer async replication or a dedicated query service over synchronous calls.

### Step 3 — Identify Cross-Cutting Infrastructure

Evaluate the need for each of the following. Include only what is justified by a functional or NFR requirement:

| Infrastructure      | Include when                                           |
| ------------------- | ------------------------------------------------------ |
| API Gateway         | Any service is externally accessible                   |
| Auth Service        | AuthN is centralized; multiple services share identity |
| Message Broker      | Any async communication exists between services        |
| Observability Stack | Any service has latency, error rate, or audit NFR      |
| CDN / Static Host   | Any service serves static assets to external clients   |

For each included infrastructure component, record the justifying requirement.

### Step 4 — Draft Artifacts

**system-context.md**

- Mermaid `graph TD` diagram showing:
  - System boundary (rectangle)
  - External actors (users, external systems)
  - Internal services (nodes inside boundary)
  - External dependencies (nodes outside boundary)
- Brief text description of each external actor and dependency.

**service-decomposition.md**

- Markdown table:

| Service | Bounded Context | Responsibility | Scaling Rationale | Notes |
| ------- | --------------- | -------------- | ----------------- | ----- |

- One row per service.
- `Scaling Rationale`: why this service has an independent deployment unit (or note if merged).

**communication-map.md**

- Mermaid `graph LR` diagram showing:
  - Services as nodes
  - Directed edges labeled with communication style (`REST` / `Event` / `gRPC`)
  - Infrastructure nodes (gateway, broker) where applicable
- Text table below diagram:

| From | To  | Style | Trigger | Notes |
| ---- | --- | ----- | ------- | ----- |

### Step 5 — Create ADRs

For each significant decision made in Steps 1–3, create one ADR entry using the format defined in the ADR standard (see below).

Decisions that always require an ADR:

- Service merge or split decision
- Choice of async over sync communication (or vice versa) where non-obvious
- Inclusion of any cross-cutting infrastructure component

### Step 6 — Flag Low-Confidence Decisions

Any decision where:

- The domain model is ambiguous about context boundaries
- NFR input is absent but scaling assumptions were made
- A circular dependency was detected

Flag these with `⚠️` in the output and include a plain-language explanation.

---

## ADR Format

```markdown
# ADR-<id>: <title>

**Date:** <YYYY-MM-DD>
**Status:** Proposed | Accepted | Superseded
**Deciders:** Architecture Designer

## Context

<What situation or requirement drove this decision?>

## Decision

<What was decided?>

## Consequences

<What are the trade-offs? What becomes easier or harder?>

## Requirement Traceability

<Which functional requirement or NFR justifies this decision?>
```

---

## Architectural Standards Enforced by This Skill

| Standard                    | Rule                                                                         |
| --------------------------- | ---------------------------------------------------------------------------- |
| Inter-service communication | Sync: RESTful HTTP. Async: event/message-based.                              |
| Database isolation          | No direct database sharing across services.                                  |
| External traffic            | All external traffic routes through API gateway.                             |
| Auth boundary               | AuthN at gateway. AuthZ delegated to services where domain rules require it. |

---

## Validation

| Check                               | Rule                                                                                        |
| ----------------------------------- | ------------------------------------------------------------------------------------------- |
| Every Bounded Context accounted for | Each context maps to at least one service or is explicitly merged with documented rationale |
| No undefined infrastructure         | Every infrastructure component has a justifying requirement                                 |
| No circular dependencies            | Flag as `⚠️` if detected; do not block output                                               |
| Domain term alignment               | Every service name must appear in `spec/domain/glossary.md`                                 |

---

## Error Codes

| Code                          | Meaning                                                         |
| ----------------------------- | --------------------------------------------------------------- |
| `ERR_NO_DOMAIN_ARTIFACTS`     | `domain_artifacts` is null or empty                             |
| `ERR_MISSING_CONTEXT_MAP`     | Domain context map is absent — cannot derive service candidates |
| `ERR_MODE_INVALID`            | `mode` is not `"initialize"` or `"update"`                      |
| `ERR_UPDATE_WITHOUT_EXISTING` | `mode` is `"update"` but `existing_system_artifacts` is null    |
