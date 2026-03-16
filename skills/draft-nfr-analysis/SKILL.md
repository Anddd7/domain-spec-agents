---
name: draft-nfr-analysis
description: Analyze and map non-functional requirements to architectural decisions and affected artifacts. Use when handling performance, security, reliability, or compliance inputs.
---

# Skill: draft-nfr-analysis

**Version:** 0.1.0

---

## Purpose

Classify, map, and resolve Non-Functional Requirements (NFRs). Produces a traceable NFR Analysis artifact that links each NFR to one or more architectural decisions.

NFRs may originate from:

- Explicit human input (e.g. "support 500 concurrent file uploads")
- Implicit signals in feature descriptions (e.g. "real-time dashboard" implies latency NFR)
- Internal Change Requests from downstream layers (Dev, QA, Security)

---

## Interface

### Input

| Parameter               | Type     | Required    | Description                                                                                               |
| ----------------------- | -------- | ----------- | --------------------------------------------------------------------------------------------------------- |
| `raw_nfr_input`         | `string` | Conditional | Plain-text NFR description from human input — explicit or inferred from feature description               |
| `cr_input`              | `object` | Conditional | Parsed CR file from a downstream layer containing NFR-driven conflict                                     |
| `existing_nfr_analysis` | `object` | No          | Existing `nfr-analysis.md` content — required when updating, null on initialization                       |
| `existing_architecture` | `object` | No          | Current system and application architecture artifacts — used to evaluate whether NFR is already satisfied |

At least one of `raw_nfr_input` or `cr_input` must be provided.

### Output

```json
{
  "status": "ok" | "error",
  "nfr_entries": [
    {
      "id": "NFR-<NNN>",
      "category": "<category>",
      "source": "explicit" | "implicit" | "cr",
      "statement": "<normalized NFR statement>",
      "affected_elements": ["<service or component id>", "..."],
      "satisfied_by_existing": true | false,
      "resolution": "<architectural decision or null>",
      "adr_required": true | false,
      "status": "open" | "resolved"
    }
  ],
  "adrs": ["<markdown string>", "..."],
  "low_confidence_flags": ["<description>", "..."],
  "artifact": "<markdown string>",
  "error": "<error message or null>"
}
```

---

## Process

### Step 1 — Extract NFR Statements

**From explicit input:**

- Parse `raw_nfr_input` for measurable NFR statements.
- Normalize each to the form: `<System / Component> must <behavior> <measurable threshold> under <condition>`.
- Example: `"support 500 concurrent file uploads"` → `"The File Upload Service must support 500 concurrent upload sessions under normal operating load."`

**From implicit signals:**

- Scan feature descriptions for NFR-implying language:

| Signal phrase                         | Implied NFR category  |
| ------------------------------------- | --------------------- |
| "real-time", "instant", "live"        | Performance / Latency |
| "millions of users", "high traffic"   | Scalability           |
| "audit log", "compliance", "GDPR"     | Compliance / Security |
| "never lose data", "always available" | Reliability           |
| "monitor", "alert", "dashboard"       | Observability         |
| "zero downtime", "rolling update"     | Operability           |

- For each detected signal, generate a candidate NFR statement and mark `source: implicit`.
- Flag all implicit NFRs with `⚠️` — they require human confirmation before being treated as binding.

**From CR input:**

- Extract the NFR statement from the CR's `conflict` field.
- Mark `source: cr`.
- Link to the originating CR ID.

### Step 2 — Classify Each NFR

Assign one primary category per NFR:

| Category          | Definition                                                 |
| ----------------- | ---------------------------------------------------------- |
| **Performance**   | Throughput, latency, response time SLA                     |
| **Scalability**   | Concurrent users/requests, horizontal scaling, data volume |
| **Security**      | AuthN/AuthZ, encryption, vulnerability surface             |
| **Observability** | Logging, metrics, tracing, alerting                        |
| **Reliability**   | Fault tolerance, data consistency, recovery time           |
| **Operability**   | Deployment strategy, rollback, configuration management    |
| **Compliance**    | Data residency, retention, regulatory audit                |

### Step 3 — Map to Architectural Elements

For each NFR:

1. Identify which services and/or components are affected.
2. Identify which infrastructure components are implicated (e.g. message broker for reliability, observability stack for observability).

### Step 4 — Evaluate Against Existing Architecture

For each NFR:

- If existing architecture already satisfies it: set `satisfied_by_existing: true`, document the traceability link, set `status: resolved`.
- If not satisfied: set `satisfied_by_existing: false`, proceed to Step 5.

### Step 5 — Propose Resolution

For each unsatisfied NFR:

1. Propose the **minimal architectural change** that addresses the NFR.
2. Apply Occam's Razor: do not introduce new infrastructure unless simpler solutions are insufficient.
3. Set `adr_required: true` for any resolution that modifies existing architecture.
4. Set `status: open` — resolution is not applied until the user confirms.

Resolution examples by category:

| Category      | Common resolutions                                                         |
| ------------- | -------------------------------------------------------------------------- |
| Performance   | Async processing, caching layer, CDN                                       |
| Scalability   | Stateless service design, horizontal scaling policy, database read replica |
| Security      | Centralized AuthN at gateway, field-level encryption, rate limiting        |
| Observability | Structured logging standard, distributed tracing, metrics endpoint         |
| Reliability   | Circuit breaker pattern, idempotent event handling, dead letter queue      |
| Operability   | Blue/green deployment, feature flags, health check endpoints               |
| Compliance    | Data residency constraint on service deployment, audit event emission      |

### Step 6 — Draft NFR Analysis Artifact

Output `nfr-analysis.md` with the following structure:

```markdown
# NFR Analysis

**Last Updated:** <YYYY-MM-DD>

## Summary

| ID      | Category    | Source      | Statement | Status   |
| ------- | ----------- | ----------- | --------- | -------- |
| NFR-001 | Performance | explicit    | ...       | Resolved |
| NFR-002 | Scalability | implicit ⚠️ | ...       | Open     |

## Detail

### NFR-001 — <Category>

**Source:** Explicit / Implicit / CR-<id>
**Statement:** <normalized statement>
**Affected Elements:** <service/component list>
**Satisfied by Existing Architecture:** Yes / No
**Resolution:** <decision or "Pending human confirmation">
**ADR:** ADR-<id> / None
**Status:** Resolved / Open
```

---

## Validation

| Check                                                | Rule                                                                                         |
| ---------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| Every NFR has a normalized statement                 | No vague or unmeasurable NFR statements accepted without flagging                            |
| Every NFR maps to at least one architectural element | Unmapped NFRs are flagged as `⚠️`                                                            |
| Implicit NFRs are flagged                            | All `source: implicit` entries carry `⚠️` pending human confirmation                         |
| Open NFRs do not block output                        | Output is returned with `status: open` entries; agent presents them to user for confirmation |

---

## Error Codes

| Code                   | Meaning                                                                                   |
| ---------------------- | ----------------------------------------------------------------------------------------- |
| `ERR_NO_INPUT`         | Both `raw_nfr_input` and `cr_input` are null                                              |
| `ERR_UNMEASURABLE_NFR` | NFR statement cannot be normalized to a measurable form — flagged for human clarification |
