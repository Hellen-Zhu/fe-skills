---
name: frontend-unit-test-generator
description: Generate or update frontend unit tests for pure logic in JavaScript, TypeScript, React, Vue, Angular, or similar apps. Use after implementation exists for permission helpers, status transition rules, payload builders, validators, mappers, reducers, hooks without DOM-heavy behavior, formatters, and other deterministic logic that should not require rendering a component or page.
---

# Frontend Unit Test Generator

## Goal

Create fast, deterministic unit tests for pure frontend logic. Prefer the project's existing test framework, naming, factories, fixtures, and assertion style.

## First Inspect

Before writing tests, inspect:

- `package.json`
- existing unit tests near the target logic
- test setup files such as `vitest.setup`, `jest.setup`, or shared fixtures
- source files for helpers, validators, reducers, mappers, schemas, hooks, or payload builders
- existing naming and mock conventions

Do not introduce a new test framework unless the project has none.

## Layer Boundary Principle

Use unit tests for logic that can be proven without rendering UI or running a page.

Use unit tests for:

- permission helpers
- status transition rules
- validation schemas
- payload builders
- API response mappers
- reducers and pure state transitions
- formatters and parsers
- deterministic hooks/composables when they do not need real DOM behavior

Do not use unit tests for:

- visible UI behavior
- user interactions
- routing and provider composition
- API mock-driven page behavior
- deployed business workflows
- visual correctness

## Test Design Rules

- Use table-driven tests for role/status/operation matrices.
- Cover positive and negative rule outcomes.
- Assert exact returned objects for payload and mapper logic.
- Keep each test small and independent.
- Avoid mocking the logic under test.
- Prefer meaningful business fixtures over anonymous values.
- Include edge cases that are business-significant, not every possible primitive input.

## Case Selection Standard

Prioritize:

1. permission and security rules
2. status transition rules
3. payload and schema compatibility logic
4. validation rules that gate submission
5. data mappers that feed important UI fields
6. regression cases from known bugs

## Implementation Workflow

1. Identify the exported logic and its stable public contract.
2. Reuse existing fixtures and factories.
3. Add tests near the source file or in the existing unit test directory.
4. Prefer table-driven cases for matrices.
5. Run the smallest relevant test command.
6. If command is unknown, infer it from `package.json`.

## Example Targets

For maker-checker Create Trade:

```text
canCreateTrade(checker) -> false
canApprove(maker, ownPendingTrade) -> false
canApprove(checker, pendingTrade) -> true
transition(Pending Approval, Approve) -> Live
transition(Pending Approval, Reject) -> Draft
buildRejectPayload(emptyReason) -> validation error
```

## Output Expectations

When finished, report:

- test file path
- logic covered
- command run and result
- any behavior that belongs in component, integration, or E2E tests instead
