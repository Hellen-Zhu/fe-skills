---
name: frontend-integration-test-generator
description: Generate or update frontend integration tests for page-level and feature-level behavior in React, Vue, Angular, or similar frontend apps. Use when testing pages, routes, state management, API mocks, multi-component interaction, data loading, submission flows, permissions, and error handling without relying on full browser E2E unless required.
---

# Frontend Integration Test Generator

## Goal

Create page-level or feature-level tests that verify multiple frontend parts working together: route, page component, state management, API mock, forms, tables, permissions, and user interactions.

## First Inspect

Before writing tests, inspect:

- `package.json`
- app routing setup
- page or feature entry point
- state management setup
- API client, generated client, service layer, or data-fetching hooks
- mock server setup such as MSW, MirageJS, WireMock, or local mock handlers
- existing integration tests and test utilities
- environment and config conventions

Follow the existing project conventions before adding new structure.

## Layer Boundary Principle

If behavior can be verified with a component test, do not promote it to an integration test. Write integration tests only for page-level collaboration, routing, state management, or API interaction.

Use integration tests for:

- page initialization with mocked API responses
- route params and query params affecting UI state
- search, filter, sort, pagination, and tab behavior that changes data or URL state
- form submit success, server validation failure, and API failure handling
- multi-component interaction inside a feature or page
- global store, cache, or context interaction
- permission, role, or feature-flag behavior across a page
- API empty data, error, timeout, retry, and optimistic update states

Do not use integration tests for:

- pure utility functions
- isolated component prop rendering
- local component behavior that can be covered by component tests
- real production backend validation
- full cross-system journeys
- browser-specific rendering bugs
- pixel-level visual regression

## Test Design Rules

Use realistic user flows while keeping the system boundary inside the frontend app:

- Prefer mock API at the network boundary instead of mocking every internal hook.
- Use stable fixtures with meaningful business data.
- Test request parameters when user actions should change API calls.
- Assert final UI state, URL state, store-visible behavior, and user-visible feedback.
- Include success and failure paths for critical flows.
- Keep data setup explicit and easy to understand.
- Avoid depending on test execution order.
- Control timers, dates, random IDs, animations, and external widgets.

## Case Selection Standard

Prioritize tests in this order:

1. Core page load and successful data rendering
2. Critical user action that mutates data
3. API failure and server validation error handling
4. Search, filter, sort, pagination, tabs, and URL-state combinations
5. Permission, role, or feature-flag variants
6. Regression cases from known bugs

A good integration test proves the page behavior is correct while staying faster and more stable than E2E.

## Implementation Workflow

1. Identify the feature boundary: page, route, store, API calls, child components, and user outcomes.
2. Reuse the project's render-with-providers helper.
3. Set up API mocks at the network layer when possible.
4. Add or reuse fixtures near the feature or shared test fixtures directory.
5. Write Given/When/Then-shaped tests using user-visible assertions.
6. Run the relevant integration test command.
7. If tests require environment variables, use the existing test environment pattern.

## Mocking Guidance

Prefer:

- MSW or equivalent handlers for HTTP APIs
- fake router with realistic route params
- real reducers, stores, caches, and providers with test initial state
- real child components unless they are heavy external widgets

Avoid:

- mocking the page or feature under test
- mocking every child component by default
- mocking business logic that the test is supposed to verify
- calling real backend services in CI integration tests

## Output Expectations

When finished, report:

- test file path
- scenarios covered
- mocks or fixtures added
- command run and result
- remaining risk that belongs in E2E or visual regression
