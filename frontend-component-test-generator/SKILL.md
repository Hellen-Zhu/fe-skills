---
name: frontend-component-test-generator
description: Generate or update frontend component tests for React, Vue, Angular, or similar UI projects. Use when testing isolated components, props, events, user interactions, loading/empty/error states, forms, permissions, accessibility-visible behavior, and component-level UI logic without full page flows or real backend dependencies.
---

# Frontend Component Test Generator

## Goal

Create maintainable component tests that verify user-visible behavior of isolated UI components. Prefer the project's existing test stack, helpers, naming, fixtures, and conventions.

## First Inspect

Before writing tests, inspect:

- `package.json`
- existing test files near the target component
- test setup files such as `setupTests`, `vitest.setup`, `jest.setup`, or `test-utils`
- component source, types, props, emitted events, hooks, stores, and child dependencies
- existing mock patterns

Do not introduce a new test framework unless the project has none.

## Layer Boundary Principle

If behavior can be verified with a component test, do not promote it to an integration test. Write integration tests only for page-level collaboration, routing, state management, or API interaction.

Use component tests for:

- props-driven rendering
- user click, input, select, keyboard, and focus behavior
- form validation and submit states inside the component boundary
- loading, empty, error, disabled, and readonly states
- permission-based visibility inside the component
- callbacks, emitted events, slots, children, and local state transitions
- accessible names, roles, labels, and visible text

Do not use component tests for:

- full user journeys across pages
- real backend integration
- routing-heavy workflows
- page-level store orchestration
- visual pixel comparison
- implementation details such as private state, CSS class names, or internal function calls

## Test Design Rules

Write tests from the user's perspective:

- Prefer Testing Library queries such as `getByRole`, `getByLabelText`, and `getByText` when available.
- Use `userEvent` or the framework equivalent instead of low-level event dispatch when available.
- Assert visible outcomes, callback payloads, emitted events, disabled states, and validation messages.
- Mock only the component boundary: API hooks, stores, router, feature flags, permissions, browser APIs, or heavy child widgets.
- Keep each test focused on one behavior.
- Cover important states, not every prop combination.
- Avoid snapshots unless the project already uses them intentionally.

## Case Selection Standard

Prioritize tests in this order:

1. Critical business behavior inside the component
2. Permission or role differences
3. Form validation and submit behavior
4. Loading, empty, error, disabled, and readonly states
5. Accessibility-visible behavior such as labels, roles, focus, and keyboard interaction
6. Regression cases from known bugs

A good component test fails when the component's user-facing behavior breaks and stays stable through harmless refactors.

## Implementation Workflow

1. Identify the component's public API: props, events, slots, children, callbacks, and visible states.
2. Reuse existing render helpers, providers, factories, fixtures, and mocks.
3. Add the test beside the component or in the existing component test directory.
4. Keep mocks local unless shared setup already exists.
5. Prefer semantic user interactions and visible assertions.
6. Run the smallest relevant test command.
7. If the command is unknown, infer it from `package.json`.

## Output Expectations

When finished, report:

- test file path
- behaviors covered
- command run and result
- any untested risk that belongs in integration, E2E, or visual regression
