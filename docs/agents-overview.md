# Agents Overview

## Specification Tier

| Agent                                                                   | Serves        | Responsibility                                                                            |
| ----------------------------------------------------------------------- | ------------- | ----------------------------------------------------------------------------------------- |
| [Domain Analysis Assistant](agent-domain-analysis-assistant.md)         | Domain Expert | Produces domain artifacts such as glossary, four-color model, event map, and context map. |
| [Architecture Design Assistant](agent-architecture-design-assistant.md) | Architect     | Produces C4 artifacts, architecture constraints, and ADRs.                                |
| [Process Design Assistant](agent-process-design-assistant.md)           | Dev Lead      | Produces process catalog, standards, layering rules, test strategy, and task specs.       |

## Execution Tier

| Agent                           | Serves        | Responsibility                                              |
| ------------------------------- | ------------- | ----------------------------------------------------------- |
| [Developer](agent-developer.md) | Delivery team | Agent-led implementation and unit testing.                  |
| [Validator](agent-validator.md) | Delivery team | Agent-led black-box validation against acceptance criteria. |
