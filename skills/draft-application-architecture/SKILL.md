---
name: draft-application-architecture
description: Draft service-level application architecture artifacts. Use when defining component boundaries and interface contracts for a specific service.
---

# Skill: draft-application-architecture

**Version:** 0.1.0

---

## Purpose

Define the intra-process architecture for a single service. Produces a Component Diagram and Interface Contracts for the target service.

This skill operates at the **intra-process level only**. It defines what components exist inside a service, how they are layered, and what interfaces the service exposes or consumes. It does not prescribe language, framework version, package names, or directory structure.

---

## Interface

### Input

| Parameter                | Type     | Required | Description                                                                                                    |
| ------------------------ | -------- | -------- | -------------------------------------------------------------------------------------------------------------- |
| `service_id`             | `string` | Yes      | Identifier of the target service (must match an entry in `service-decomposition.md`)                           |
| `system_artifacts`       | `object` | Yes      | Parsed content of `spec/architecture/system/` — service decomposition and communication map                    |
| `domain_artifacts`       | `object` | Yes      | Parsed content of `spec/domain/` artifacts relevant to this service's Bounded Context                          |
| `nfr_analysis`           | `object` | No       | NFR analysis output — used to inform component-level decisions (e.g. caching, retry)                           |
| `existing_app_artifacts` | `object` | No       | Existing `spec/architecture/services/<service-id>/` artifacts — required when updating, null on initialization |
| `mode`                   | `string` | Yes      | `"initialize"` or `"update"`                                                                                   |

### Output

```json
{
  "status": "ok" | "error",
  "service_id": "<service_id>",
  "artifacts": {
    "component_diagram": "<markdown string>",
    "interface_contracts": "<markdown string>"
  },
  "adrs": ["<markdown string>", "..."],
  "low_confidence_flags": ["<description>", "..."],
  "error": "<error message or null>"
}
```

---

## Process

### Step 1 — Identify Standard Layers

Every service is structured using the following standard layers unless an ADR documents a justified deviation:

| Layer                      | Responsibility                                                                                               |
| -------------------------- | ------------------------------------------------------------------------------------------------------------ |
| **API Layer**              | Receive and validate inbound requests; serialize/deserialize; route to domain logic                          |
| **Domain Logic Layer**     | Implement business rules and domain operations; owns domain objects from the Four-Color Model                |
| **Persistence Layer**      | Abstract data storage; owns read/write operations for this service's data                                    |
| **External Adapter Layer** | Communicate with other services or external systems; translate between internal and external representations |

Additional layers may be introduced (e.g. Cache Layer, Event Publisher) when justified by an NFR. Each addition requires an ADR.

### Step 2 — Define Components per Layer

For each layer, identify the key components:

1. Name each component using Glossary-defined terms.
2. State its single responsibility (one sentence).
3. Identify its dependencies (other components it calls).

Do not specify:

- Class names, method names, or function signatures
- Package or module names
- File or directory structure

### Step 3 — Define Interface Contracts

For each interface this service **exposes** (inbound) or **consumes** (outbound):

**Synchronous Interface (REST):**

| Field           | Content                                          |
| --------------- | ------------------------------------------------ |
| Name            | Endpoint identifier (e.g. `CreateOrder`)         |
| Direction       | Inbound / Outbound                               |
| Protocol        | REST                                             |
| Method          | GET / POST / PUT / DELETE / PATCH                |
| Path            | `/resource/{id}`                                 |
| Request Schema  | Field name, type, required/optional, description |
| Response Schema | Field name, type, description                    |
| Error Codes     | Code, meaning                                    |
| SLA             | Expected latency (p95), rate limit if applicable |

**Asynchronous Interface (Event/Message):**

| Field              | Content                                          |
| ------------------ | ------------------------------------------------ |
| Name               | Event name (e.g. `OrderPlaced`)                  |
| Direction          | Published / Consumed                             |
| Protocol           | Event / Message                                  |
| Payload Schema     | Field name, type, required/optional, description |
| Ordering Guarantee | None / Per-partition / Global                    |
| Retry Policy       | Max retries, backoff strategy                    |
| Dead Letter        | Yes / No                                         |

### Step 4 — Draft Artifacts

**component-diagram.md**

- Mermaid `graph TD` diagram showing:
  - Layers as subgraphs
  - Components as nodes within layers
  - Directed edges showing dependencies between components
  - External interfaces (inbound/outbound) as entry/exit points
- Brief text description of each component.

**interface-contracts.md**

- One section per interface (inbound and outbound).
- Use the table formats defined in Step 3.
- Group by direction: Inbound first, Outbound second.

### Step 5 — Create ADRs

Decisions that always require an ADR:

- Addition of a non-standard layer
- Choice of async over sync for an internal component interaction
- Any component-level NFR decision (e.g. introducing a cache, a circuit breaker)

### Step 6 — Flag Low-Confidence Decisions

Flag with `⚠️` when:

- Domain model is ambiguous about which layer owns a domain object
- NFR is present but the component-level solution is uncertain
- An interface contract cannot be fully defined without more information from the consuming service

---

## Constraints

This skill **never** specifies:

- Programming language or runtime version
- Framework, library, or package names
- Directory structure or file naming conventions
- ORM, test framework, or build tool
- Deployment configuration or container image details

These are code-layer concerns. They belong to the implementation layer, not the architecture specification.

---

## Validation

| Check                           | Rule                                                                                    |
| ------------------------------- | --------------------------------------------------------------------------------------- |
| Service exists in decomposition | `service_id` must match an entry in `service-decomposition.md`                          |
| Domain term alignment           | Every component name maps to a Glossary term or is a standard layer name                |
| Interface contract completeness | Every inter-service dependency from `communication-map.md` has a corresponding contract |
| No orphan components            | Every component has at least one defined dependency or is an entry/exit point           |
| ADR for deviations              | Any deviation from standard layers has a corresponding ADR                              |

---

## Error Codes

| Code                          | Meaning                                                   |
| ----------------------------- | --------------------------------------------------------- |
| `ERR_SERVICE_NOT_FOUND`       | `service_id` does not exist in `service-decomposition.md` |
| `ERR_NO_SYSTEM_ARTIFACTS`     | `system_artifacts` is null or empty                       |
| `ERR_NO_DOMAIN_ARTIFACTS`     | `domain_artifacts` is null or empty                       |
| `ERR_MODE_INVALID`            | `mode` is not `"initialize"` or `"update"`                |
| `ERR_UPDATE_WITHOUT_EXISTING` | `mode` is `"update"` but `existing_app_artifacts` is null |
