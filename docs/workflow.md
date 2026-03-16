# Workflow

## Phase 0: Initialization (One-time)

1. Domain Expert works with Domain Analysis Assistant to establish domain artifacts.
2. Architect works with Architecture Design Assistant to establish architecture artifacts.
3. Dev Lead works with Process Design Assistant to establish process artifacts.
4. These outputs form the Spec Infrastructure.

## Phase N: Iteration (Per requirement)

1. A new requirement enters through the Process Design Assistant.
2. Domain layer evaluates whether domain artifacts must change.
3. Architecture layer evaluates whether architecture artifacts must change.
4. Process layer evaluates whether process artifacts must change.
5. Process Design Assistant decomposes work into Task Specs with Given / When / Then acceptance criteria.
6. Developer implements and writes unit tests.
7. Validator executes black-box validation.
8. If validation fails, a Change Request is raised to the correct upstream layer.

## Inter-Layer Feedback Protocol

- Downstream layers validate upstream outputs through execution.
- Conflicts are handled by explicit Change Requests.
- No silent assumptions or implicit corrections.
