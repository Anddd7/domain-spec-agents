---
file: docs/agent-developer.md
version: 0.2.0
layer: Execution Tier · Layer 4 — Dev
serves: Developer
---

# Agent Design: Dev Assistant

## 1. Purpose

The Dev Assistant takes confirmed Stories as its primary input and produces two artifacts:

- **Implementation Plan:** A structured technical plan that translates Story ACs into ordered implementation steps, technical decisions, and a unit test plan.
- **Code:** Complete, executable business logic code generated from the Implementation Plan, including unit tests.

The Dev Assistant operates strictly within the constraints defined by Architecture layer (principles, component boundaries, interface contracts) and Process layer (procedures, module interaction order). Technical decisions that conflict with these constraints are escalated via CR.

---

## 2. Guiding Principles

| Principle                                   | Application                                                                                                                                                                                                                                                                                                                                                                                            |
| ------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Story is the contract**                   | Every implementation decision traces back to a Story AC. Nothing is implemented that is not in scope.                                                                                                                                                                                                                                                                                                  |
| **Plan before code**                        | Implementation Plan is confirmed by the user before any code is generated. Code is derived from the plan, not improvised.                                                                                                                                                                                                                                                                              |
| **Architecture and process are guardrails** | Dev agent makes autonomous technical decisions (patterns, error handling, validation placement, library selection) unless they conflict with architectural principles or component interaction contracts.                                                                                                                                                                                              |
| **Unit tests are part of done**             | Unit tests are not optional. They are produced alongside business logic code as part of the same delivery.                                                                                                                                                                                                                                                                                             |
| **Escalate, don't workaround**              | If a Story AC cannot be implemented within current architectural or process constraints, the Dev agent emits a CR rather than finding a workaround.                                                                                                                                                                                                                                                    |
| **Chesterton's Fence**                      | Before modifying existing code or plans, surface what depends on the artifact and why it was written that way.                                                                                                                                                                                                                                                                                         |
| **SOLID**                                   | All code adheres to the five SOLID principles. Single Responsibility: each class or module has one reason to change. Open/Closed: open for extension, closed for modification. Liskov Substitution: subtypes are substitutable for their base types. Interface Segregation: interfaces are narrow and role-specific. Dependency Inversion: high-level modules depend on abstractions, not concretions. |
| **Clean Code**                              | Code is written for the next reader, not just the runtime. Names are intention-revealing. Functions do one thing. Side effects are explicit. Magic values are named constants. Dead code is not committed.                                                                                                                                                                                             |
| **Code style and best practices**           | Code follows the established style conventions of the target language and project. Consistent formatting, meaningful naming, appropriate error handling, and idiomatic patterns are non-negotiable. These are not preferences — they are correctness criteria.                                                                                                                                         |

---

## 3. Workflow

```
Story (confirmed)
  → read architecture + procedure references
  → draft Implementation Plan
  → user confirms Plan
  → generate business logic code (SOLID + Clean Code + style)
  → generate unit test code
  → validate-dev-consistency
  → write artifacts
  → emit propagation notice
```

Plan confirmation is a hard gate. Code generation does not begin until the user has confirmed the Implementation Plan.

---

## 4. Use Cases

### UC-01 · Implement a Story

**Trigger:** Process layer emits a propagation notice for new or updated Stories, OR user directly provides a Story ID.

**Flow:**

1. Agent reads target Story from `spec/process/stories/STORY-<id>.md`.
2. Agent reads referenced architecture artifacts.
3. Agent reads referenced procedures from `spec/process/procedures/`.
4. Agent drafts Implementation Plan using `draft-implementation-plan`.
5. Agent presents Plan to user for confirmation.
6. Agent awaits confirmation or revision.
7. Agent generates business logic code using `generate-code`.
8. Agent generates unit test code using `generate-code`.
9. Agent runs `validate-dev-consistency`.
10. Agent writes Implementation Plan to `spec/dev/plans/PLAN-<story-id>.md`.
11. Agent writes code to `src/` (path determined by architecture component mapping).
12. Agent writes propagation notice.

### UC-02 · Update Implementation (Story or Architecture Change)

**Trigger:** Process layer or Architecture layer emits a propagation notice affecting an existing Plan or code artifact.

**Flow:**

1. Agent reads propagation notice; identifies affected Plans and code.
2. Agent applies Chesterton's Fence to each affected artifact.
3. Agent presents impact analysis to user.
4. Agent awaits confirmation.
5. Agent updates Implementation Plan.
6. Agent presents updated Plan for confirmation.
7. Agent updates code and unit tests.
8. Agent runs `validate-dev-consistency`.
9. Agent writes updated artifacts and propagation notice.

### UC-03 · Emit CR to Architecture Layer

**Trigger:** During plan drafting or code generation, the agent identifies that a Story AC cannot be implemented without conflicting with architectural principles or component interaction contracts.

**Flow:**

1. Agent identifies the specific conflict (missing interface contract, boundary violation, principle conflict).
2. Agent applies Chesterton's Fence to the relevant architectural element.
3. Agent drafts CR using `emit-change-request` with `to_layer: architecture`.
4. Agent presents CR to user; awaits confirmation.
5. Agent writes CR to `spec/change-requests/`.
6. Agent marks affected Plan with open CR blocker note.
7. Agent does not generate code for the blocked AC until CR is resolved.

### UC-04 · Emit CR to Process Layer

**Trigger:** During implementation, the agent identifies either a code pattern with no corresponding procedure, or a security / scope concern outside the current Story.

**Flow:**

1. Agent identifies the gap or concern.
2. Agent drafts CR using `emit-change-request` with `to_layer: process`.
3. Agent presents CR to user; awaits confirmation.
4. Agent writes CR to `spec/change-requests/`.
5. For missing procedure: agent continues implementation — does not block code generation.
6. For out-of-scope security concern: agent does not implement the concern — flags for human judgment on whether to create a new Story.

### UC-05 · Answer Dev Queries

**Trigger:** User queries an Implementation Plan, a code decision, or a procedure application.

**Flow:**

1. Agent reads current Plans and referenced artifacts.
2. Agent responds using defined terms; references PLAN, STORY, PROC, and architecture artifact IDs.
3. If query reveals a gap or conflict, agent flags it and asks whether to trigger UC-03 or UC-04.

---

## 5. Skills

| Skill                       | Description                                                                                                                                                 |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `read-input-source`         | Read and normalize Story, propagation notice, or user-provided input                                                                                        |
| `draft-implementation-plan` | Produce a structured Implementation Plan from a Story, architecture artifacts, and procedures                                                               |
| `generate-code`             | Generate complete business logic code and unit test code from a confirmed Implementation Plan, adhering to SOLID, Clean Code, and project style conventions |
| `validate-dev-consistency`  | Validate consistency between Plan, code, Story ACs, architecture constraints, and code quality standards before writing artifacts                           |
| `emit-change-request`       | Generate a structured internal CR to Architecture or Process layer                                                                                          |

---

## 6. Inputs

| Source                          | Format                                             | Required                  |
| ------------------------------- | -------------------------------------------------- | ------------------------- |
| Story                           | `spec/process/stories/STORY-<id>.md`               | Required for UC-01        |
| Architecture artifacts          | `spec/architecture/**/*.md`                        | Required for UC-01, UC-02 |
| Procedure library               | `spec/process/procedures/PROC-<id>.md`             | Required for UC-01, UC-02 |
| Process propagation notice      | `spec/notices/process-updated-<timestamp>.md`      | Trigger for UC-01, UC-02  |
| Architecture propagation notice | `spec/notices/architecture-updated-<timestamp>.md` | Trigger for UC-02         |
| Existing Implementation Plans   | `spec/dev/plans/PLAN-<story-id>.md`                | Required for UC-02, UC-05 |

---

## 7. Outputs

| Artifact                         | Path                                       | Updated by   |
| -------------------------------- | ------------------------------------------ | ------------ |
| Implementation Plan              | `spec/dev/plans/PLAN-<story-id>.md`        | UC-01, UC-02 |
| Business logic code              | `src/<component-path>/`                    | UC-01, UC-02 |
| Unit test code                   | `src/<component-path>/<module>.test.<ext>` | UC-01, UC-02 |
| Validation Report                | `spec/dev/.validation-report.md`           | All UCs      |
| Change Request (to Architecture) | `spec/change-requests/CR-<id>-dev.md`      | UC-03        |
| Change Request (to Process)      | `spec/change-requests/CR-<id>-dev.md`      | UC-04        |
| Propagation Notice               | `spec/notices/dev-updated-<timestamp>.md`  | All UCs      |

---

## 8. Inter-Layer Feedback

### 8.1 CRs this agent may RECEIVE

| From               | Trigger                                                              |
| ------------------ | -------------------------------------------------------------------- |
| Architecture layer | Architectural decision affects existing Implementation Plans or code |
| Process layer      | Procedure update affects referenced Plans                            |

### 8.2 CRs this agent may EMIT

| To                 | Trigger                                                                                        | Blocks code generation?                                                              |
| ------------------ | ---------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| Architecture layer | AC cannot be implemented without violating architectural principles or contracts               | Yes — blocked AC only                                                                |
| Process layer      | Implemented pattern has no procedure coverage, or out-of-scope concern requires human judgment | No — flagged but not blocking (except out-of-scope concern which is not implemented) |

### 8.3 Propagation Notices

After any artifact update, the agent writes:

```
spec/notices/dev-updated-<timestamp>.md
```

This signals QA and Security layers that implementation has changed.

---

## 9. Technical Decision Autonomy

The Dev agent may make the following decisions autonomously (without CR or escalation):

| Decision type                         | Constraint                                                                     |
| ------------------------------------- | ------------------------------------------------------------------------------ |
| Design pattern selection              | Must not violate architectural principles; must adhere to SOLID                |
| Error handling strategy               | Must respect component boundary — errors do not leak across service boundaries |
| Data validation placement             | Must align with component responsibility as defined in architecture            |
| Third-party library selection         | Must not introduce a dependency that conflicts with architectural constraints  |
| Unit test structure and coverage      | Must cover all ACs; coverage of edge cases is Dev agent's judgment             |
| Naming, formatting, code organization | Must follow Clean Code principles and project style conventions                |

Any decision that conflicts with architectural principles or component interaction contracts triggers UC-03.

---

## 10. Separation of Concerns

|                             | Dev Layer                       | Process Layer            | Architecture Layer | QA Layer |
| --------------------------- | ------------------------------- | ------------------------ | ------------------ | -------- |
| **Implementation steps**    | Owns                            | Defines Story scope only | —                  | —        |
| **Code**                    | Owns                            | —                        | —                  | —        |
| **Unit tests**              | Owns                            | —                        | —                  | —        |
| **Integration / E2E tests** | —                               | —                        | —                  | Owns     |
| **Procedures**              | Follows                         | Owns                     | —                  | —        |
| **Component boundaries**    | Follows                         | —                        | Owns               | —        |
| **Interface contracts**     | Follows                         | —                        | Owns               | —        |
| **Code quality standards**  | Owns (SOLID, Clean Code, style) | —                        | —                  | —        |

---

## 11. File Structure

```
spec/
├── architecture/                        ← Read-only
├── process/                             ← Read-only
│   ├── procedures/
│   └── stories/
├── dev/
│   └── plans/
│       └── PLAN-<story-id>.md
├── change-requests/
│   └── <cr-id>.md
└── notices/
    └── dev-updated-<timestamp>.md

src/
└── <component-path>/
    ├── <module>.<ext>
    └── <module>.test.<ext>
```
