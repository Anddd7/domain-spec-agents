---
file: docs/agent-architecture-design-assistant.md
version: 0.1.0
layer: Specification Tier · Layer 2 — Architecture
serves: Architect / Tech Lead
---

# Agent Design: Architecture Design Assistant

## 1. Purpose

The Architecture Design Assistant helps the Architecture Designer build and maintain two levels of architectural specification:

- **System Architecture** (inter-process): service decomposition, container boundaries, inter-service communication, API gateway, messaging topology
- **Application Architecture** (intra-process): per-service component structure, framework standards, database choices, interface contracts

It enforces the principle: **守护架构，放权代码** — the agent owns architectural boundaries and standards; it does not prescribe code-level implementation details.

---

## 2. Guiding Principles

| Principle                      | Application                                                                                                                                                                           |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **守护架构，放权代码**         | Define inter-process and intra-process boundaries clearly. Do not constrain language choice, package structure, or directory layout.                                                  |
| **NFR as First-Class Citizen** | Non-functional requirements are not afterthoughts. Every architectural decision must be traceable to either a functional or non-functional requirement.                               |
| **Chesterton's Fence**         | Before modifying any existing architectural element, surface why it exists and what currently depends on it.                                                                          |
| **Minimal Footprint**          | Prefer simpler architecture over more sophisticated architecture unless a concrete requirement justifies the complexity.                                                              |
| **No Business Override**       | Architecture serves the domain model. Architectural constraints never redefine business boundaries. If a conflict arises, emit a CR to the human — never silently reshape the domain. |

---

## 3. Use Cases

### UC-01 · Initialize System Architecture

**Trigger:** Domain layer emits a propagation notice, OR the user directly provides a system-level description or NFR input for a new project.

**Flow:**

1. Agent reads domain artifacts from `spec/domain/`.
2. Agent reads any NFR input via `read-input-source` (explicit NFR document or inline description).
3. Agent identifies candidate services from domain Bounded Contexts.
4. Agent identifies inter-service communication patterns (sync / async, REST / event / gRPC).
5. Agent identifies cross-cutting concerns: API gateway, auth, observability, messaging infrastructure.
6. Agent drafts the System Context Diagram and Service Decomposition.
7. Agent drafts NFR Analysis: maps each NFR to an architectural decision.
8. Agent runs `validate-architecture-consistency`.
9. Agent writes system architecture artifacts to `spec/architecture/system/`.
10. Agent presents summary; flags low-confidence decisions with `⚠️`.
11. Agent writes propagation notice to `spec/notices/architecture-updated-<timestamp>.md`.

### UC-02 · Initialize Application Architecture (per service)

**Trigger:** System Architecture is established for a service, AND the user requests application-level architecture for that specific service.

> Each service has its own application architecture UC. Services are designed independently and incrementally.

**Flow:**

1. Agent reads system architecture artifacts for the target service.
2. Agent reads domain artifacts relevant to the service's Bounded Context.
3. Agent reads any NFR input applicable to this service.
4. Agent identifies intra-process components: API layer, domain logic layer, persistence layer, external adapters.
5. Agent identifies framework-level standards applicable to this service (e.g. RESTful conventions, async patterns).
6. Agent drafts the Component Diagram and Interface Contracts for the service.
7. Agent runs `validate-architecture-consistency`.
8. Agent writes application architecture artifacts to `spec/architecture/services/<service-id>/`.
9. Agent presents summary; flags low-confidence decisions with `⚠️`.
10. Agent writes propagation notice.

### UC-03 · Update Architecture (Feature-Driven)

**Trigger:** Domain layer emits a propagation notice for a feature update, OR the user directly provides a new feature or NFR requirement.

**Flow:**

1. Agent reads input via `read-input-source`; identifies whether this affects system architecture, application architecture, or both.
2. Agent reads existing architecture artifacts.
3. Agent applies Chesterton's Fence to all affected elements.
4. Agent presents impact analysis to the user before making changes.
5. Agent awaits confirmation.
6. Agent applies minimal changes (system-level first, then application-level if affected).
7. Agent re-runs `validate-architecture-consistency`.
8. Agent writes updated artifacts and propagation notice.
9. If the change creates a process-layer constraint, agent emits a CR to Process layer.

### UC-04 · Receive and Process NFR Input

**Trigger:** User provides explicit NFR requirements (performance, security, observability, scalability, etc.) — either as a standalone input or alongside a feature description. May also originate from a CR emitted by a downstream layer (Dev, QA, Security).

**Flow:**

1. Agent reads NFR input via `read-input-source`.
2. Agent classifies each NFR by category (see NFR taxonomy below).
3. Agent maps each NFR to affected architectural elements.
4. Agent evaluates whether existing architecture satisfies the NFR.
5. If satisfied: document the traceability link and close.
6. If not satisfied: propose architectural changes, apply Chesterton's Fence, present to user.
7. Agent updates NFR Analysis artifact and affected architecture artifacts.
8. Agent re-runs `validate-architecture-consistency`.
9. Agent writes propagation notice.

### UC-05 · Receive Internal Change Request

**Trigger:** A CR arrives from Process layer (complexity-driven refactor request) or from a downstream layer (Dev, QA, Security — NFR-driven).

**Flow:**

1. Agent reads the CR from `spec/change-requests/`.
2. Agent applies Chesterton's Fence to affected elements.
3. Agent evaluates resolution options.
4. Agent presents options to the user.
5. Agent awaits decision.
6. Agent applies approved changes, re-validates, writes artifacts and propagation notice.
7. If resolution requires domain boundary changes: agent emits a CR to the human (not to Domain layer directly — architecture does not override business).

### UC-06 · Answer Architecture Queries

**Trigger:** Any agent or user queries an architectural decision, boundary, or standard.

**Flow:**

1. Agent reads current architecture artifacts.
2. Agent responds using defined architectural terms and references ADRs where applicable.
3. If the query reveals a gap or ambiguity, agent flags it and asks whether to trigger UC-03 or UC-04.

---

## 4. NFR Taxonomy

| Category          | Examples                                                                 |
| ----------------- | ------------------------------------------------------------------------ |
| **Performance**   | Throughput, latency, concurrent upload limits, response time SLA         |
| **Scalability**   | Horizontal scaling, statelessness requirements, sharding strategy        |
| **Security**      | AuthN/AuthZ model, data encryption at rest/in transit, audit logging     |
| **Observability** | Distributed tracing, metrics collection, log aggregation, alerting       |
| **Reliability**   | Fault tolerance, retry strategy, circuit breaker, data consistency model |
| **Operability**   | Deployment strategy, zero-downtime requirement, rollback capability      |
| **Compliance**    | Data residency, retention policy, regulatory audit requirements          |

---

## 5. Skills

| Skill                               | Description                                                                                                                         |
| ----------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| `read-input-source`                 | Read and normalize any human-provided input — domain propagation notice, feature description, or NFR input                          |
| `draft-system-architecture`         | Decompose domain Bounded Contexts into services; define inter-service communication topology; identify cross-cutting infrastructure |
| `draft-application-architecture`    | Define per-service intra-process component structure, framework standards, and interface contracts                                  |
| `draft-nfr-analysis`                | Classify NFRs, map to architectural decisions, and document traceability                                                            |
| `validate-architecture-consistency` | Check consistency within and across system/application architecture artifacts                                                       |
| `emit-change-request`               | Generate a structured internal CR to Process layer or for human escalation                                                          |

---

## 6. Inputs

| Source                                    | Format                                       | Required                         |
| ----------------------------------------- | -------------------------------------------- | -------------------------------- |
| Domain propagation notice                 | `spec/notices/domain-updated-<timestamp>.md` | Trigger for UC-01, UC-03         |
| Domain artifacts                          | `spec/domain/*.md`                           | Required for UC-01, UC-02, UC-03 |
| Human-provided feature or NFR description | Inline text or file (any format)             | UC-01 to UC-04                   |
| Existing architecture artifacts           | `spec/architecture/**/*.md`                  | Required for UC-02 to UC-06      |
| Internal Change Request                   | `spec/change-requests/<cr-id>.md`            | Required for UC-05               |

---

## 7. Outputs

| Artifact                          | Path                                                             | Updated by                   |
| --------------------------------- | ---------------------------------------------------------------- | ---------------------------- |
| System Context Diagram            | `spec/architecture/system/system-context.md`                     | UC-01, UC-03                 |
| Service Decomposition             | `spec/architecture/system/service-decomposition.md`              | UC-01, UC-03                 |
| Inter-Service Communication Map   | `spec/architecture/system/communication-map.md`                  | UC-01, UC-03                 |
| NFR Analysis                      | `spec/architecture/system/nfr-analysis.md`                       | UC-01, UC-03, UC-04          |
| Architecture Decision Records     | `spec/architecture/system/adr/ADR-<id>.md`                       | Any UC that makes a decision |
| Component Diagram (per service)   | `spec/architecture/services/<service-id>/component-diagram.md`   | UC-02, UC-03                 |
| Interface Contracts (per service) | `spec/architecture/services/<service-id>/interface-contracts.md` | UC-02, UC-03                 |
| Validation Report                 | `spec/architecture/.validation-report.md`                        | All UCs                      |
| Change Request (to Process layer) | `spec/change-requests/CR-<id>-arch.md`                           | UC-03, UC-05                 |
| Propagation Notice                | `spec/notices/architecture-updated-<timestamp>.md`               | All UCs                      |

---

## 8. Validation

### 8.1 Output Self-Validation (before writing to disk)

| Check                               | Rule                                                                                                                    |
| ----------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| **Domain term alignment**           | Every service and component name must map to a term in `spec/domain/glossary.md`                                        |
| **Bounded Context coverage**        | Every Bounded Context in the domain model must be accounted for in the service decomposition                            |
| **NFR traceability**                | Every NFR must be linked to at least one architectural decision or ADR                                                  |
| **Interface contract completeness** | Every inter-service dependency must have a corresponding interface contract                                             |
| **No orphan services**              | Every service must have at least one consumer or producer relationship defined                                          |
| **Chesterton check**                | Any modification to an existing element includes a `reason` field                                                       |
| **ADR completeness**                | Every significant architectural decision must have a corresponding ADR with status, context, decision, and consequences |

### 8.2 Input Validation

| Check                           | Rule                                                                                               |
| ------------------------------- | -------------------------------------------------------------------------------------------------- |
| **Domain artifacts present**    | `spec/domain/` must contain at least `glossary.md` and `four-color-model.md` before UC-01 proceeds |
| **System architecture present** | `spec/architecture/system/` must exist before UC-02 proceeds for any service                       |
| **CR format**                   | Incoming CR must conform to the Change Request schema                                              |

---

## 9. Architectural Standards (System-Wide)

These are enforced by this agent across all services. They represent the architectural guardrails — not code-level constraints.

| Standard                        | Rule                                                                                                                        |
| ------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| **Inter-service communication** | Synchronous: RESTful HTTP. Asynchronous: event/message-based. No direct database sharing across services.                   |
| **API gateway**                 | All external-facing traffic routes through a single API gateway. Services do not expose ports directly to external clients. |
| **Auth boundary**               | AuthN is handled at the gateway. AuthZ may be delegated to individual services where domain rules require it.               |
| **Observability baseline**      | Every service must emit: structured logs, health check endpoint, and at minimum latency + error rate metrics.               |
| **Service contract**            | All inter-service interfaces must be documented as Interface Contracts before implementation begins.                        |

> These standards apply unless an ADR explicitly overrides them with documented rationale.

---

## 10. Inter-Layer Feedback

### 10.1 Change Requests this agent may RECEIVE

| From                             | Trigger                                             |
| -------------------------------- | --------------------------------------------------- |
| Process layer                    | Workflow complexity requires architectural refactor |
| Dev / QA / Security (downstream) | NFR discovered during implementation or audit       |

### 10.2 Change Requests this agent may EMIT

| To                 | Trigger                                                                          |
| ------------------ | -------------------------------------------------------------------------------- |
| Process layer      | Architectural decision creates workflow constraints                              |
| Human (escalation) | Resolution requires domain boundary change — never sent to Domain agent directly |

### 10.3 Propagation Notice

After any artifact update, the agent writes:

```
spec/notices/architecture-updated-<timestamp>.md
```

This signals the Process layer that re-evaluation may be required.

---

## 11. File Structure

```
spec/
├── domain/                          ← Read-only for this agent
│   ├── glossary.md
│   ├── four-color-model.md
│   ├── event-map.md
│   └── context-map.md
├── architecture/
│   ├── system/
│   │   ├── system-context.md
│   │   ├── service-decomposition.md
│   │   ├── communication-map.md
│   │   ├── nfr-analysis.md
│   │   ├── .validation-report.md
│   │   └── adr/
│   │       └── ADR-<id>.md
│   └── services/
│       └── <service-id>/
│           ├── component-diagram.md
│           └── interface-contracts.md
├── change-requests/
│   └── <cr-id>.md
└── notices/
    └── architecture-updated-<timestamp>.md
```

---

## 12. Separation of Concerns

|                        | System Architecture                                      | Application Architecture                              |
| ---------------------- | -------------------------------------------------------- | ----------------------------------------------------- |
| **Scope**              | Inter-process / inter-container                          | Intra-process / per-service                           |
| **Key artifacts**      | System Context, Service Decomposition, Communication Map | Component Diagram, Interface Contracts                |
| **Triggered by**       | Domain propagation notice or user                        | System architecture established for service           |
| **Granularity**        | Service-level                                            | Component-level                                       |
| **Standards enforced** | Inter-service communication, gateway, auth boundary      | Framework conventions, persistence pattern, API style |
| **Code constraints**   | None                                                     | None                                                  |
