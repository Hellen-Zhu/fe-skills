---
name: frontend-test-layer-advisor
description: Decide the correct frontend test layer for a behavior, requirement, bug, or code change. Use before generating tests when it is unclear whether coverage belongs in unit, component, integration, E2E, visual regression, or contract tests. Supports both pre-implementation candidate layering and post-implementation final layering.
---

# Frontend Test Layer Advisor

## Goal

Classify frontend validation points into the lowest reliable test layer. Prevent important scenarios from being over-tested at E2E level when faster lower layers can prove the same behavior.

## First Inspect

Inspect available context:

- requirement, acceptance criteria, screenshots, or bug report
- current implementation, if it exists
- changed files, components, pages, hooks, routes, stores, API clients, and tests
- whether API, store, router, feature flags, permissions, or backend schema are involved
- whether the risk is behavior, workflow, visual stability, contract compatibility, or release confidence

## Phase-Aware Rule

Pre-implementation:

- recommend candidate layers
- design only business-level E2E
- do not invent component names, selectors, or exact test files

Post-implementation:

- inspect real code boundaries
- choose final layers
- route to the correct test generator skill

## Core Rule

Use the lowest layer that can prove the behavior:

```text
unit -> component -> integration -> E2E
```

More specifically:

```text
Can be proven as pure logic? Use unit.
Can be proven inside one component? Use component.
Needs page, routing, store, or API mock? Use integration.
Needs real deployed cross-page/cross-role behavior? Use E2E.
Needs visual correctness? Use visual regression.
Needs API compatibility? Use contract.
```

Do not escalate just because a scenario is important. Importance changes priority; dependency boundaries decide layer.

## Decision Matrix

Choose unit test when:

- pure functions, mappers, reducers, validation schemas, permission helpers, payload builders, formatters, or status transition logic are being verified

Choose component test when:

- one component owns the behavior through props, local state, events, callbacks, slots, or children
- validation is local to the form component
- a button, menu, dialog, badge, upload control, or action bar has role/status-specific behavior

Choose integration test when:

- route params, query params, global store, cache, providers, or API mock responses affect behavior
- a page composes multiple components
- a mutation changes list/detail state
- backend validation or API error mapping must be rendered

Choose E2E when:

- login, role switching, deployed environment, persisted backend state, or cross-page workflow is the value being tested

Choose visual regression when:

- layout, overflow, spacing, badge/button alignment, responsive behavior, theme, or screenshot diff is the risk

Choose contract test when:

- field names, types, enums, error shapes, or generated client compatibility are the risk

## Output Format

Always produce:

```text
Recommended layer:
Reason:
Why lower layers are insufficient:
Why higher layers are not needed:
Suggested cases:
Quality gate:
Next skill:
```

If multiple layers are needed, separate them by validation point.

## Escalation Rules

Escalate component to integration only when:

- routing changes behavior
- global state changes behavior
- API response changes behavior
- multiple components must collaborate
- server validation or backend error mapping is part of the behavior

Escalate integration to E2E only when:

- real auth, deployed backend, persisted data, or cross-role workflow is required
- mocks would hide the actual risk

## Output Expectations

When finished, report:

- final layer decision
- candidate vs final status if implementation is not yet available
- next generation skill to invoke
- any checks that should be explicitly kept out of E2E
