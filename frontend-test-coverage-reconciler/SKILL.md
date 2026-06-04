---
name: frontend-test-coverage-reconciler
description: Reconcile frontend requirements, acceptance criteria, test plans, and implemented tests across unit, component, integration, E2E, visual, and contract layers. Use after implementation or partial test creation to identify missing coverage, over-layered E2E cases, duplicate tests, quality gate gaps, and the next tests to generate.
---

# Frontend Test Coverage Reconciler

## Goal

Compare requirements and acceptance criteria against actual or planned tests. Confirm every important behavior is covered at the right layer and that E2E is not carrying checks better handled by unit, component, or integration tests.

## First Inspect

Inspect available:

- PRD, story, acceptance criteria, screenshots, and test matrix
- existing test files and naming conventions
- CI pipeline or quality gate configuration
- changed files, routes, pages, components, API mocks, and generated clients
- known gaps, flaky tests, bug history, or release blockers

Prefer local repo evidence over assumptions.

## Reconciliation Rules

For each acceptance criterion, map coverage across layers:

```text
AC -> unit/component/integration/E2E/visual/contract coverage -> status
```

Mark status as:

- Covered
- Partially covered
- Missing
- Over-layered
- Duplicate
- Blocked by missing implementation

## Layer Quality Rules

Flag a test as over-layered when:

- E2E checks local validation that a component test can prove
- E2E checks field mapping that integration test can prove
- integration checks pure mapper or permission logic that unit test can prove
- visual test is being used to prove business logic

Flag a coverage gap when:

- P0 acceptance has no automated test
- permission guard is only tested through hidden UI but not backend/contract or integration handling
- create/update/delete flow has no failure-path coverage
- contract-sensitive API field has no schema or mapper test
- release smoke does not cover the core business path

## Workflow

1. List acceptance criteria and business risks.
2. Inventory existing tests by layer and case intent.
3. Map each AC to existing coverage.
4. Identify missing, duplicated, flaky, or over-layered tests.
5. Recommend the lowest layer for each missing check.
6. Identify PR, release, nightly, and production quality gates.
7. Recommend next generation skills to invoke.

## Output Format

Use this table:

```text
AC ID | Requirement / Risk | Current Coverage | Status | Correct Layer | Quality Gate | Recommended Action | Next Skill
```

Also include:

- E2E cases to keep
- E2E cases to downshift
- missing PR gates
- missing release gates
- assumptions or blockers

## Example

For maker-checker Create Trade:

```text
AC-001 maker creates trade -> Pending Approval
Coverage:
- E2E-CT-001
- IT-CT-001
- IT-CT-007
- CT-CT-001/002
- UT-CT-008/011
Status: Covered
```

```text
AC-005 maker cannot approve own trade
Coverage:
- UT permission helper
- component details action bar
- integration trade details page
- E2E smoke
Status: Covered
```

## Output Expectations

When finished, report:

- coverage map
- missing tests with recommended layer
- over-layered tests to move down
- quality gate changes
- next skills to run
