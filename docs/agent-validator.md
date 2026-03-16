---
file: docs/agent-validator.md
version: 0.1.0
layer: Execution Tier · Layer 5 — QA
serves: QA Engineer / Tech Lead
---

# Agent Design: Validator

## 1. Purpose

The Validator is the delivery quality gate for the SpecStrata framework. It operates as a **black-box validator**: the code produced by the Dev layer is treated as an opaque artifact. The Validator verifies that all Story ACs are satisfied at the integration level, identifies and tests potentially affected areas beyond the immediate Story scope, and determines whether a Story — or a release — is ready to ship.

The Validator owns:

- Test Plans (per Story and per Release)
- Test Code (integration, E2E, contract, regression, and any performance or security tests required by Story scope)
- Test Reports
- CRs to Dev layer (bug reports)

---

## 2. Guiding Principles

| Principle                            | Application                                                                                                                                                                                                                |
| ------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Black-box validation**             | The Validator does not read implementation code or Implementation Plans. Validation is based on Story ACs, QA-defined supplementary conditions, and observable system behavior.                                            |
| **Story AC is the baseline**         | Every Story AC must be covered by at least one test case. QA may add supplementary acceptance conditions at the integration level; it may not remove or weaken Story ACs.                                                  |
| **Impact-aware scope**               | For every Story under test, QA identifies potentially affected components and flows using architecture dependency graphs and Process layer Story relationships. Regression coverage is determined by this impact analysis. |
| **Completeness over speed**          | A Story is not ready for release until all test cases pass and all CRs issued by QA are resolved. Partial passes do not constitute delivery.                                                                               |
| **Direct feedback to Dev**           | When a bug is found, QA issues a CR directly to Dev. QA does not route bug reports through Process layer.                                                                                                                  |
| **Dev owns bug triage**              | Validator reports bugs to Dev. If bug root cause appears to be ambiguous Story/procedure or missing constraints, Dev performs upstream feedback after triage.                                                              |
| **Release is a first-class trigger** | QA supports both Story-driven testing and release-driven testing. A release test run covers all Stories changed since the last release plus full regression on impacted areas.                                             |

---

## 3. Workflow

### Story-Driven (UC-01)

```
Dev propagation notice (or manual trigger)
  → read Story ACs + architecture dependency graph + Story relationships
  → draft Test Plan
  → user confirms Test Plan
  → generate test code
  → [test execution by user / CI]
  → receive execution results
  → generate Test Report
  → if failures: emit CR(s) to Dev
  → if pass: mark Story as QA-approved
  → write artifacts
  → emit propagation notice
```

### Release-Driven (UC-04)

```
User triggers release test (specifies release scope or "since last release")
  → identify changed Stories since last release
  → identify regression scope via impact analysis
  → draft Release Test Plan
  → user confirms Release Test Plan
  → generate / update test code
  → [test execution by user / CI]
  → receive execution results
  → generate Release Test Report
  → if failures: emit CR(s) to Dev
  → if pass: mark release as QA-approved
  → write artifacts
  → emit propagation notice
```

Test Plan confirmation is a hard gate. Test code generation does not begin until the user has confirmed the Test Plan.

---

## 4. Use Cases

### UC-01 · Validate a Story

**Trigger:** Dev layer emits a propagation notice for a completed Story, OR user manually provides a Story ID.

**Flow:**

1. Agent reads Story from `spec/process/stories/STORY-<id>.md`.
2. Agent reads architecture dependency graph from `spec/architecture/`.
3. Agent reads Story relationships from `spec/process/`.
4. Agent performs impact analysis: identifies components and flows potentially affected by this Story.
5. Agent drafts Test Plan using `draft-test-plan`.
6. Agent presents Test Plan to user for confirmation.
7. Agent awaits confirmation or revision.
8. Agent generates test code using `generate-test-code`.
9. Agent writes Test Plan to `spec/qa/plans/TESTPLAN-<story-id>.md`.
10. Agent writes test code to `tests/<scope>/`.
11. Agent notifies user that tests are ready for execution.
12. User or CI executes tests and provides results.
13. Agent receives results and generates Test Report using `generate-test-report`.
14. Agent writes Test Report to `spec/qa/reports/TESTREPORT-<story-id>-<timestamp>.md`.
15. If failures: agent emits CR(s) to Dev using `emit-bug-change-request`.
16. If all pass: agent marks Story as QA-approved in Test Report.
17. Agent writes propagation notice.

### UC-02 · Re-validate After Dev CR Resolution

**Trigger:** Dev layer emits a propagation notice resolving a QA-issued CR.

**Flow:**

1. Agent reads CR resolution notice; identifies affected test cases.
2. Agent updates Test Plan if supplementary conditions are affected.
3. Agent updates test code for affected cases.
4. Agent notifies user that updated tests are ready for execution.
5. User or CI executes affected tests and provides results.
6. Agent generates updated Test Report.
7. If failures: agent emits new CR(s) to Dev.
8. If all pass: agent marks Story as QA-approved.
9. Agent writes updated artifacts and propagation notice.

### UC-03 · Update Test Plan (Story or Architecture Change)

**Trigger:** Process layer or Architecture layer emits a propagation notice affecting an existing Test Plan.

**Flow:**

1. Agent reads propagation notice; identifies affected Test Plans.
2. Agent applies Chesterton's Fence to each affected Test Plan.
3. Agent presents impact analysis to user; awaits confirmation.
4. Agent updates Test Plan; presents for confirmation.
5. Agent updates test code.
6. Agent writes updated artifacts and propagation notice.

### UC-04 · Release Validation

**Trigger:** User manually triggers a release test run, specifying release scope or requesting "since last release."

**Flow:**

1. Agent identifies Stories changed since last release (or within specified scope).
2. Agent performs impact analysis across all changed Stories combined.
3. Agent identifies regression scope: components and flows affected by any changed Story.
4. Agent drafts Release Test Plan using `draft-test-plan` with `mode: release`.
5. Agent presents Release Test Plan to user for confirmation.
6. Agent awaits confirmation or revision.
7. Agent generates / updates test code for all in-scope cases.
8. Agent writes Release Test Plan to `spec/qa/plans/TESTPLAN-release-<version>.md`.
9. Agent notifies user that tests are ready for execution.
10. User or CI executes tests and provides results.
11. Agent generates Release Test Report using `generate-test-report`.
12. Agent writes Release Test Report to `spec/qa/reports/TESTREPORT-release-<version>-<timestamp>.md`.
13. If failures: agent emits CR(s) to Dev.
14. If all pass: agent marks release as QA-approved.
15. Agent writes propagation notice.

### UC-05 · Answer QA Queries

**Trigger:** User queries a Test Plan, Test Report, CR, or test coverage question.

**Flow:**

1. Agent reads relevant Test Plans, Test Reports, and referenced artifacts.
2. Agent responds using defined terms; references TESTPLAN, TESTREPORT, CR, STORY, and architecture IDs.
3. If query reveals a coverage gap or impact analysis gap, agent flags it and asks whether to update the Test Plan.

---

## 5. Skills

| Skill                     | Description                                                                                                                                          |
| ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| `read-input-source`       | Read and normalize Story, propagation notice, release scope, or user-provided input                                                                  |
| `perform-impact-analysis` | Identify components and flows potentially affected by a Story or set of Stories, using architecture dependency graph and Process Story relationships |
| `draft-test-plan`         | Produce a structured Test Plan from Story ACs, supplementary conditions, and impact analysis                                                         |
| `generate-test-code`      | Generate complete test code from a confirmed Test Plan                                                                                               |
| `generate-test-report`    | Produce a structured Test Report from execution results                                                                                              |
| `emit-bug-change-request` | Generate a structured bug CR to Dev layer                                                                                                            |

---

## 6. Inputs

| Source                                    | Format                                    | Required                            |
| ----------------------------------------- | ----------------------------------------- | ----------------------------------- |
| Story                                     | `spec/process/stories/STORY-<id>.md`      | Required for UC-01, UC-02, UC-03    |
| Architecture dependency graph             | `spec/architecture/`                      | Required for all UCs                |
| Story relationships                       | `spec/process/`                           | Required for all UCs                |
| Dev propagation notice                    | `spec/notices/dev-updated-<timestamp>.md` | Trigger for UC-01, UC-02            |
| Process / Architecture propagation notice | `spec/notices/`                           | Trigger for UC-03                   |
| Existing Test Plans                       | `spec/qa/plans/`                          | Required for UC-02, UC-03, UC-05    |
| Test execution results                    | User / CI provided                        | Required for Test Report generation |
| Last release marker                       | `spec/qa/releases/last-release.md`        | Required for UC-04                  |

---

## 7. Outputs

| Artifact              | Path                                                          | Updated by                     |
| --------------------- | ------------------------------------------------------------- | ------------------------------ |
| Test Plan (Story)     | `spec/qa/plans/TESTPLAN-<story-id>.md`                        | UC-01, UC-02, UC-03            |
| Test Plan (Release)   | `spec/qa/plans/TESTPLAN-release-<version>.md`                 | UC-04                          |
| Test Code             | `tests/<scope>/`                                              | UC-01, UC-02, UC-03, UC-04     |
| Test Report (Story)   | `spec/qa/reports/TESTREPORT-<story-id>-<timestamp>.md`        | UC-01, UC-02                   |
| Test Report (Release) | `spec/qa/reports/TESTREPORT-release-<version>-<timestamp>.md` | UC-04                          |
| CR (to Dev)           | `spec/change-requests/CR-<id>-qa.md`                          | UC-01, UC-02, UC-04            |
| Propagation Notice    | `spec/notices/qa-updated-<timestamp>.md`                      | All UCs                        |
| Last release marker   | `spec/qa/releases/last-release.md`                            | UC-04 (on QA-approved release) |

---

## 8. Inter-Layer Feedback

### 8.1 CRs this agent may RECEIVE

| From               | Trigger                                                                        |
| ------------------ | ------------------------------------------------------------------------------ |
| Dev layer          | Dev resolves a QA-issued CR; triggers UC-02                                    |
| Process layer      | Story or procedure update affects existing Test Plans; triggers UC-03          |
| Architecture layer | Architectural change affects component boundaries or contracts; triggers UC-03 |

### 8.2 CRs this agent may EMIT

| To        | Trigger                       | Blocks release?                            |
| --------- | ----------------------------- | ------------------------------------------ |
| Dev layer | Test case failure (bug found) | Yes — until CR resolved and re-test passes |

### 8.3 Propagation Notices

After any artifact update, the agent writes:

```
spec/notices/qa-updated-<timestamp>.md
```

---

## 9. Test Scope by Story Content

QA determines the applicable test types based on what the Story requires:

| Story contains                              | Test types applied                    |
| ------------------------------------------- | ------------------------------------- |
| Functional requirement                      | Functional integration test, E2E test |
| API / service interaction                   | Contract test                         |
| Shared module or cross-Story dependency     | Regression test                       |
| Performance requirement (explicit in Story) | Performance test                      |
| Security requirement (explicit in Story)    | Security test                         |
| All Stories in release scope                | Full regression on impacted areas     |

QA does not apply performance or security tests speculatively. If a Story does not mention a performance or security requirement, those test types are not applied for that Story. They are applied at release scope if any Story in the release includes such requirements.

---

## 10. Impact Analysis Methodology

For each Story under test, QA performs impact analysis using two sources:

| Source                                               | What it provides                                                        |
| ---------------------------------------------------- | ----------------------------------------------------------------------- |
| Architecture dependency graph (`spec/architecture/`) | Which components depend on the components modified by this Story        |
| Process Story relationships (`spec/process/`)        | Which other Stories share modules, interfaces, or flows with this Story |

The union of these two sources defines the **impact boundary**: the set of components and flows that must be included in regression coverage for this Story or release.

---

## 11. Separation of Concerns

|                          | QA Layer                 | Dev Layer | Process Layer | Architecture Layer |
| ------------------------ | ------------------------ | --------- | ------------- | ------------------ | --- |
| **Unit tests**           | —                        | Owns      | —             | —                  |
| **Integration tests**    | Owns                     | —         | —             | —                  |
| **E2E tests**            | Owns                     | —         | —             | —                  |
| **Contract tests**       | Owns                     | —         | —             | —                  |
| **Regression tests**     | Owns                     | —         | —             | —                  |
| **Performance tests**    | Owns (if in Story scope) | —         | —             | —                  |
| **Security tests**       | Owns (if in Story scope) | —         | —             | —                  |
| **Story AC baseline**    | Follows                  | —         | Owns          | —                  |
| **Component boundaries** | Follows                  | Follows   | —             | Owns               | —   |
| **Bug reports**          | Emits to Dev             | Receives  | —             | —                  |

---

## 12. File Structure

```
spec/
├── architecture/                          ← Read-only
├── process/                               ← Read-only
│   └── stories/
├── qa/
│   ├── plans/
│   │   ├── TESTPLAN-<story-id>.md
│   │   └── TESTPLAN-release-<version>.md
│   ├── reports/
│   │   ├── TESTREPORT-<story-id>-<timestamp>.md
│   │   └── TESTREPORT-release-<version>-<timestamp>.md
│   └── releases/
│       └── last-release.md
├── change-requests/
│   └── CR-<id>-qa.md
└── notices/
    └── qa-updated-<timestamp>.md

tests/
├── integration/
├── e2e/
├── contract/
├── regression/
├── performance/
└── security/
```
