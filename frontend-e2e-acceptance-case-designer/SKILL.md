---
name: frontend-e2e-acceptance-case-designer
description: Design frontend E2E acceptance test cases before implementation for business workflows, roles, permissions, statuses, and deployed user journeys. Use when defining smoke/regression E2E scope from requirements without relying on final selectors, component structure, or implementation details.
---

# Frontend E2E Acceptance Case Designer

## Goal

Design E2E cases as business acceptance contracts before implementation exists. E2E cases should describe who performs which business action, through which high-level entry point, and what final system state must be true.

## First Inspect

Before designing E2E cases, inspect:

- requirement, story, PRD, screenshots, or acceptance criteria
- user roles and permissions
- core state transitions
- entry points such as list, details, creation page, notifications, or deep links
- test environment constraints and data setup needs
- existing E2E conventions if a repo is available

Do not write selectors, locators, component names, or brittle UI steps before implementation.

## E2E Scope Boundary

Use E2E for:

- cross-role workflows
- cross-page business journeys
- authentication or authorization smoke
- deployed preview, staging, or production smoke
- final persisted backend state
- list/details consistency after real actions

Do not use E2E for:

- every required-field validation
- every component state
- every API error mapping
- every row action variant
- pixel-level layout checks
- API schema compatibility

Those belong in component, integration, visual, or contract tests after implementation details are known.

## Case Design Rules

Each case must include:

- Case ID
- title
- positive/negative
- smoke/regression
- quality gate recommendation
- preconditions and test data
- high-level steps
- checkpoints
- expected final result
- data cleanup or isolation notes

Use Given/When/Then language. Keep cases resilient to UI refactors.

## Positive vs Negative

Positive E2E proves a core workflow succeeds.

Negative E2E proves a business-critical guard works, such as:

- unauthorized role cannot access an action
- user cannot approve their own work
- invalid status cannot transition
- failed mutation does not corrupt state

Do not use E2E negative cases for every local validation rule.

## Smoke vs Regression

Smoke E2E:

- very small set
- fast enough for release or deployment gate
- proves minimum core workflow availability
- protects P0 role/status flows

Regression E2E:

- broader workflow coverage
- includes alternate entry points and critical failure guards
- can run in release or nightly pipelines

## Output Format

Use a table with:

```text
Case ID | Scenario | Preconditions / Data | Steps | Check Points | Expected Result | Positive/Negative | Smoke/Regression | Quality Gate | Notes
```

## Example

For maker-checker Create Trade:

```text
E2E-CT-001
Positive Smoke
Given maker is logged in
When maker creates a valid trade
Then trade details opens
And status is Pending Approval
And maker cannot see Approve or Reject
```

```text
E2E-CT-006
Negative Smoke
Given maker has created a Pending Approval trade
When the same maker opens the trade details
Then Approve and Reject are unavailable
```

## Output Expectations

When finished, report:

- final E2E case list
- which cases are smoke gates
- which cases are regression only
- which checks should be covered in lower layers instead
- data setup and cleanup risks
