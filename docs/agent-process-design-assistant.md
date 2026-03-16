---
file: docs/agent-process-design-assistant.md
version: 0.1.0
layer: Specification Tier · Layer 3 — Process
serves: Dev Lead / Senior Developer
---

# Agent Design: Process Design Assistant

## 1. Purpose

The Process Design Assistant manages two distinct but related concerns:

- **Procedures (工序):** Reusable, named operation patterns that define _how development work is structured_ — which modules are touched, in what order, and how correctness is verified. Procedures are rules and constraints, not exhaustive task lists.
- **Stories:** Feature-level work items decomposed to Story granularity — with Acceptance Criteria, traceability to procedures and architecture, and explicit scope boundaries.

The agent does not prescribe code-level implementation. It defines the rules of the game; Dev layer plays it.

---

## 2. Guiding Principles

| Principle                              | Application                                                                                                                                                                              |
| -------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Rules over recipes**                 | Procedures define constraints and patterns, not step-by-step instructions. A procedure for "Add HTTP Endpoint" tells you what to touch and how to verify — not how to write the handler. |
| **Traceability in both directions**    | Every Story traces to architecture. Every Procedure traces to architecture. Nothing floats free.                                                                                         |
| **Minimal scope, explicit exclusions** | Stories must state what they do not include. Scope creep is a process failure.                                                                                                           |
| **Procedures evolve**                  | The procedure library is not frozen at project initialization. It grows through refactoring and developer feedback. New patterns are first-class contributions.                          |
| **Chesterton's Fence**                 | Before modifying an existing procedure or story, surface what depends on it.                                                                                                             |

---

## 3. Two-Phase Model

### Phase 1 — Procedure Library Initialization

Triggered after Architecture layer has produced system and application architecture for a service. A senior developer initializes the project structure, and the natural interaction patterns between modules become the seed procedures.

Procedures capture:

- What modules are involved in a given type of change
- The order of operations across those modules
- How to verify the change is correct (test, lint, pipeline)

Procedures do **not** capture:

- Language-specific implementation details
- Framework internals
- Code organization or directory structure

### Phase 2 — Story Decomposition

Triggered by a feature requirement (from Domain or Architecture propagation notice, or direct user input). The agent decomposes the feature into Stories at AC-level granularity, linking each Story to relevant procedures and architecture artifacts.

Stories capture:

- User value
- Acceptance Criteria (input/output, verifiable conditions)
- Referenced procedures and architecture
- Explicit scope exclusions
- Story dependencies

Stories do **not** capture:

- Implementation steps (that is Dev layer's responsibility)
- Technical decisions (that is Architecture layer's responsibility)

---

## 4. Use Cases

### UC-01 · Initialize Procedure Library (for a service)

**Trigger:** Architecture layer emits a propagation notice for a service, AND a senior developer has initialized the project structure for that service.

**Flow:**

1. Agent reads architecture artifacts for the target service from `spec/architecture/services/<service-id>/`.
2. Agent reads the senior developer's description of the initialized project structure via `read-input-source`.
3. Agent identifies interaction patterns between architectural components (e.g. API Layer → Domain Logic Layer → Persistence Layer for a new endpoint).
4. Agent drafts candidate procedures using `draft-procedure`.
5. Agent presents candidate procedures to user for review.
6. Agent awaits confirmation or revision.
7. Agent writes confirmed procedures to `spec/process/procedures/`.
8. Agent writes propagation notice.

### UC-02 · Add or Update Procedure (Developer-Initiated)

**Trigger:** A senior developer has designed a new pattern (not yet implemented) and wants to register it as a procedure, OR the user directly provides a new procedure description.

**Flow:**

1. Agent reads new procedure description via `read-input-source`.
2. Agent applies Chesterton's Fence if updating an existing procedure.
3. Agent drafts or updates the procedure using `draft-procedure`.
4. Agent presents to user for confirmation.
5. Agent writes to `spec/process/procedures/`.
6. Agent writes propagation notice.

### UC-03 · Receive CR from Dev Layer (Implementation-Discovered Pattern)

**Trigger:** Dev layer emits a CR indicating that implemented code follows a pattern not yet covered by any procedure.

**Flow:**

1. Agent reads CR from `spec/change-requests/`.
2. Agent reads the relevant implementation description from the CR.
3. Agent evaluates whether this pattern should become a new procedure or update an existing one.
4. Agent applies Chesterton's Fence if updating.
5. Agent presents proposed procedure to user for confirmation.
6. Agent writes confirmed procedure to `spec/process/procedures/`.
7. Agent closes CR with resolution note.
8. Agent writes propagation notice.

### UC-04 · Decompose Feature into Stories

**Trigger:** Domain or Architecture layer emits a propagation notice for a new feature, OR user directly provides a feature description.

**Flow:**

1. Agent reads feature input via `read-input-source`.
2. Agent reads relevant architecture artifacts (service decomposition, component diagrams, interface contracts).
3. Agent reads procedure library for relevant procedures.
4. Agent drafts Stories using `draft-stories`.
5. Agent presents Stories to user for review.
6. Agent awaits confirmation or revision.
7. Agent writes confirmed Stories to `spec/process/stories/`.
8. Agent writes propagation notice.

### UC-05 · Update Stories (Architecture or Procedure Change)

**Trigger:** Architecture layer emits a propagation notice that affects existing Stories, OR a procedure update affects referenced Stories.

**Flow:**

1. Agent reads propagation notice and identifies affected Stories.
2. Agent applies Chesterton's Fence to affected Stories.
3. Agent presents impact analysis to user.
4. Agent awaits confirmation.
5. Agent updates affected Stories.
6. Agent re-runs `validate-process-consistency`.
7. Agent writes updated artifacts and propagation notice.
8. If the update reveals that architecture needs to change, agent emits CR to Architecture layer.

### UC-06 · Emit CR to Architecture Layer

**Trigger:** During Story decomposition or procedure design, the agent identifies that the current architecture creates excessive workflow complexity (e.g. a dependency chain that cannot be parallelized, or a missing interface contract).

**Flow:**

1. Agent identifies the specific architectural element causing the constraint.
2. Agent applies Chesterton's Fence to that element.
3. Agent drafts a CR using `emit-change-request` with `to_layer: architecture`.
4. Agent presents the CR to user before emitting.
5. Agent awaits confirmation.
6. Agent writes CR to `spec/change-requests/`.
7. Agent notes the open CR in the affected Story or procedure as a blocker.

### UC-07 · Answer Process Queries

**Trigger:** Any agent or user queries a procedure, Story, or process constraint.

**Flow:**

1. Agent reads current procedure library and Story artifacts.
2. Agent responds using defined procedure and Story terms.
3. If query reveals a gap, agent flags it and asks whether to trigger UC-02 or UC-04.

---

## 5. Skills

| Skill                          | Description                                                                                                                                    |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `read-input-source`            | Read and normalize any human-provided input — propagation notice, feature description, procedure description, or project structure description |
| `draft-procedure`              | Draft or update a procedure entry based on architectural component interactions and developer-provided patterns                                |
| `draft-stories`                | Decompose a feature into Stories with AC, procedure references, architecture references, and scope exclusions                                  |
| `validate-process-consistency` | Check consistency between procedures, stories, and architecture artifacts                                                                      |
| `emit-change-request`          | Generate a structured internal CR to Architecture layer                                                                                        |

---

## 6. Inputs

| Source                                          | Format                                             | Required                         |
| ----------------------------------------------- | -------------------------------------------------- | -------------------------------- |
| Architecture propagation notice                 | `spec/notices/architecture-updated-<timestamp>.md` | Trigger for UC-01, UC-04, UC-05  |
| Architecture artifacts                          | `spec/architecture/**/*.md`                        | Required for UC-01, UC-04, UC-05 |
| Senior developer project structure description  | Inline text or file                                | Required for UC-01               |
| Human-provided procedure or feature description | Inline text or file                                | UC-02, UC-04                     |
| Internal Change Request (from Dev layer)        | `spec/change-requests/<cr-id>.md`                  | Required for UC-03               |
| Existing procedure library                      | `spec/process/procedures/*.md`                     | Required for UC-02 to UC-07      |
| Existing stories                                | `spec/process/stories/*.md`                        | Required for UC-05, UC-07        |

---

## 7. Outputs

| Artifact                               | Path                                          | Updated by          |
| -------------------------------------- | --------------------------------------------- | ------------------- |
| Procedure files                        | `spec/process/procedures/PROC-<id>.md`        | UC-01, UC-02, UC-03 |
| Story files                            | `spec/process/stories/STORY-<id>.md`          | UC-04, UC-05        |
| Validation Report                      | `spec/process/.validation-report.md`          | All UCs             |
| Change Request (to Architecture layer) | `spec/change-requests/CR-<id>-process.md`     | UC-06               |
| Propagation Notice                     | `spec/notices/process-updated-<timestamp>.md` | All UCs             |

---

## 8. Inter-Layer Feedback

### 8.1 CRs this agent may RECEIVE

| From               | Trigger                                             |
| ------------------ | --------------------------------------------------- |
| Architecture layer | Architectural decision creates workflow constraints |
| Dev layer          | Implemented pattern not covered by any procedure    |

### 8.2 CRs this agent may EMIT

| To                 | Trigger                                           |
| ------------------ | ------------------------------------------------- |
| Architecture layer | Workflow complexity requires architectural change |

### 8.3 Propagation Notices

After any artifact update, the agent writes:

```
spec/notices/process-updated-<timestamp>.md
```

This signals Dev layer that procedures or stories have changed.

---

## 9. File Structure

```
spec/
├── architecture/                        ← Read-only for this agent
│   ├── system/
│   └── services/
├── process/
│   ├── procedures/
│   │   └── PROC-<id>.md
│   └── stories/
│       └── STORY-<id>.md
├── change-requests/
│   └── <cr-id>.md
└── notices/
    └── process-updated-<timestamp>.md
```

---

## 10. Separation of Concerns

|                         | Process Layer                                                  | Dev Layer                            |
| ----------------------- | -------------------------------------------------------------- | ------------------------------------ |
| **Procedures**          | Define interaction patterns, module scope, verification method | Implement the actual steps           |
| **Stories**             | Define AC, scope, references                                   | Define implementation steps          |
| **Architecture**        | Read-only reference                                            | Read-only reference                  |
| **Code structure**      | Not specified                                                  | Owned by Dev                         |
| **Technical decisions** | Not made                                                       | Made within architectural guardrails |
