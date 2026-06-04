---
name: frontend-e2e-acceptance-case-designer
description: Design frontend E2E acceptance test cases and decide what belongs in E2E for business workflows, roles, permissions, statuses, deployed environments, production smoke, and cross-page user journeys. Use before or after implementation to define positive/negative, smoke/regression, quality-gate E2E scope without relying on brittle selectors or component internals.
---

# Frontend E2E Acceptance Case Designer

## Goal

Design E2E cases as business acceptance contracts. E2E cases should prove that a real user can complete a critical business journey in a real browser against a deployed environment, and that the final persisted business state is correct.

Do not treat E2E as "important test coverage." Treat E2E as "real workflow risk coverage."

## First Inspect

Before designing E2E cases, inspect:

- requirement, story, PRD, screenshots, or acceptance criteria
- user roles and permissions
- core state transitions
- entry points such as list, details, creation page, notifications, or deep links
- test environment constraints and data setup needs
- existing E2E conventions if a repo is available

Do not write selectors, locators, component names, or brittle UI steps before implementation.

## E2E Entry Standard

A case belongs in E2E only when it needs several of these:

- real user role or permission
- real browser execution
- deployed preview, staging, or production environment
- cross-page navigation or multi-step journey
- backend persistence or final system state
- list/detail consistency after mutation
- cross-role handoff such as maker/checker
- release or production smoke confidence

Use this rule:

```text
E2E case = business workflow + real role + real environment + persisted state + release risk
```

If the value of the test is only a local rule, component behavior, API mock rendering, visual layout, or schema compatibility, do not put it in E2E.

## E2E Scope Boundary

Use E2E for:

- cross-role workflows
- cross-page business journeys
- authentication or authorization smoke
- deployed preview, staging, or production smoke
- final persisted backend state
- list/details consistency after real actions
- status transitions that must be proven through the real backend
- core create/edit/submit/approve/reject flows

Do not use E2E for:

- every required-field validation
- every component state
- every API error mapping
- every row action variant
- pixel-level layout checks
- API schema compatibility
- payload field mapping
- all detail page fields
- local button disabled states
- all table filter/sort/pagination combinations

Those belong in component, integration, visual, or contract tests after implementation details are known.

## Downshift Rules

Explicitly move checks out of E2E when a lower layer can prove them:

```text
Required-field rules -> unit/component
Form error display -> component
Page with API mock -> integration
Create payload shape -> unit/contract
Detail field mapping -> integration/contract
Action menu visibility matrix -> component/integration
Badge color/layout -> visual regression
API schema compatibility -> contract
```

Do not escalate just because a scenario is important. Importance changes priority; dependency boundaries decide layer.

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

Write business-level steps:

```text
Good:
Given maker is logged in
When maker creates a valid trade
Then trade details shows Pending Approval
```

Avoid script-level design:

```text
Avoid:
Click the third combobox
Click data-testid=create-trade-save-btn
Assert class=blue-badge
Wait 500ms
```

Implementation may use locators later, but the case design should remain stable if the UI is refactored.

## Positive vs Negative

Positive E2E proves a core workflow succeeds.

Negative E2E proves a business-critical guard works, such as:

- unauthorized role cannot access an action
- user cannot approve their own work
- invalid status cannot transition
- failed mutation does not corrupt state

Do not use E2E negative cases for every local validation rule.

Use negative E2E only when failure protection is business-critical, such as permissions, status guards, production-safe read-only access, or failed persisted mutation behavior.

## Smoke vs Regression

Smoke E2E:

- very small set
- fast enough for release or deployment gate
- proves minimum core workflow availability
- protects P0 role/status flows
- usually 3-8 cases per major business module
- should fail the release when broken

Regression E2E:

- broader workflow coverage
- includes alternate entry points and critical failure guards
- can run in release or nightly pipelines
- can include secondary routes, alternate entry points, and non-P0 negative cases

Prefer:

```text
Smoke: one main happy path + one or two critical permission/status guards
Regression: alternate entry points + additional guards + less common status paths
```

## Quality Gate Standard

Mark an E2E case as a release gate only when:

- it is a P0 business journey or permission guard
- failure means the version should not ship
- it is stable and repeatable
- data can be isolated, created, or cleaned up
- lower layers cannot provide equivalent release confidence

Mark as nightly-only when:

- it is valuable but slower
- it covers alternate entry points
- it has higher data/environment cost
- it is more likely to be flaky but still useful for trend detection

Mark as production smoke when:

- it is safe for production
- it is read-only or uses controlled test data
- it confirms critical pages, roles, and core statuses load after deployment

## Validation Points

For each E2E case, prefer 3-6 high-signal checkpoints:

- final route or page opened
- generated business ID is visible
- final status is correct
- key business field or summary matches input
- role-appropriate action is visible or hidden
- list and details agree after mutation

Avoid asserting dozens of detail fields in E2E. Use integration and contract tests for broad field mapping.

## Output Format

Use a table with:

```text
Case ID | Scenario | Preconditions / Data | Steps | Check Points | Expected Result | Positive/Negative | Smoke/Regression | Quality Gate | Lower-Layer Coverage | Notes
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
Lower-layer coverage:
- unit: payload and permission rules
- component: create form validation
- integration: create API mock + detail field mapping
```

```text
E2E-CT-006
Negative Smoke
Given maker has created a Pending Approval trade
When the same maker opens the trade details
Then Approve and Reject are unavailable
Lower-layer coverage:
- unit: canApprove(maker, ownPendingTrade) returns false
- component: action bar hides Approve/Reject
- integration: details page hides actions for maker session
```

Good Create Trade E2E set:

```text
Smoke positive:
- maker creates trade -> Pending Approval
- checker approves pending trade from portal list -> Live
- checker rejects pending trade from details -> Draft

Smoke negative:
- checker cannot create trade
- maker cannot approve own pending trade

Regression:
- checker approves from details
- checker cannot approve/reject Live or Draft trade
- production read-only smoke for portal/details availability
```

## Output Expectations

When finished, report:

- final E2E case list
- why each case belongs in E2E
- which cases are smoke gates
- which cases are regression only
- which checks should be covered in lower layers instead
- which lower-layer tests are expected for support coverage
- data setup and cleanup risks
- assumptions about environment, users, and test data
