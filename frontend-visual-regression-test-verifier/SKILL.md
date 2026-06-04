---
name: frontend-visual-regression-test-verifier
description: Generate, update, or verify frontend visual regression tests for React, Vue, Angular, Storybook, Playwright, Percy, Chromatic, Loki, or similar UI projects. Use when validating layout stability, responsive views, screenshots, baseline diffs, design system states, modal/table/form visuals, text overflow, spacing, themes, or release visual quality gates.
---

# Frontend Visual Regression Test Verifier

## Goal

Create or verify visual regression coverage that proves important UI states still look correct. Prefer the project's existing screenshot, Storybook, browser, CI artifact, and baseline approval workflow.

## First Inspect

Before adding or changing visual tests, inspect:

- `package.json`
- existing visual, screenshot, Storybook, Playwright, Cypress, Percy, Chromatic, or Loki configuration
- existing stories, fixtures, screenshot baselines, and CI visual artifacts
- target component, page, route, viewport, theme, and UI states
- dynamic regions such as dates, random IDs, async counters, charts, maps, ads, animations, and remote images
- font loading, asset loading, mock data, and test environment setup

Do not introduce a new visual testing tool if the project already has one.

## Layer Boundary Principle

Use visual regression tests only when visual correctness is the thing being verified. If the behavior can be proven with DOM assertions, component tests, or integration tests, do that there and keep visual tests focused on appearance.

Use visual regression tests for:

- layout breakage, spacing, alignment, overflow, clipping, and wrapping
- responsive desktop, tablet, and mobile states
- modals, drawers, popovers, sticky headers, tables, forms, and dashboards
- loading, empty, error, disabled, selected, hover, focus, and validation states when their appearance matters
- design system components and themed variants
- high-risk pages where visual diff review is a release gate

Do not use visual regression tests for:

- business logic correctness
- API field or schema compatibility
- form validation logic unless the rendered error state is the visual target
- every minor component variant
- unstable content that cannot be controlled or masked
- full cross-page business journeys

## Test Design Rules

Make screenshots deterministic:

- Use fixed fixtures and meaningful business data.
- Freeze dates, timers, random IDs, and locale-sensitive output when possible.
- Disable animations, transitions, blinking cursors, skeleton shimmer, and auto-rotating content.
- Use fixed viewport sizes and device scale factors.
- Wait for fonts, images, route idle state, and critical async rendering before capture.
- Mock or mask dynamic regions such as charts, maps, ads, timestamps, avatars, and external widgets.
- Capture the smallest useful region when a full-page screenshot is noisy.
- Use full-page screenshots only when page-level layout is the risk.
- Keep baseline updates explicit and reviewable.

Do not hide real regressions with broad masks, loose thresholds, or automatic baseline updates.

## Case Selection Standard

Prioritize visual coverage in this order:

1. Design system primitives that many screens depend on
2. Core product pages and dashboards
3. Complex forms, tables, filters, modals, drawers, and empty/error states
4. Responsive states that are easy to break
5. Theme, density, locale, or permission variants that materially change layout
6. Regression cases from known visual bugs

A good visual test fails when the UI looks meaningfully wrong and stays stable when content or behavior changes harmlessly.

## Verification Workflow

1. Identify the visual risk: layout, overflow, responsive behavior, theme, state, or component style.
2. Choose the existing visual surface: Storybook story, component harness, preview URL, or routed page.
3. Stabilize data, time, animation, fonts, viewport, network, and dynamic regions.
4. Add or update screenshots for the smallest set of meaningful states.
5. Run the visual test command locally when possible.
6. Inspect the diff artifact, not only the pass/fail status.
7. Classify diffs as expected change, regression, or test noise.
8. Update baselines only for intentional UI changes after review.
9. Record any remaining visual risk that needs manual QA or broader viewport coverage.

## Diff Review Standard

Treat these as likely regressions:

- text overflow, clipping, overlap, or missing content
- shifted layout that hides controls or changes hierarchy
- broken responsive wrapping
- unexpected color, theme, typography, spacing, or density changes
- modal, drawer, dropdown, tooltip, or sticky element misplacement
- empty, loading, error, or validation states no longer visible

Treat these as possible test noise:

- timestamp, generated ID, or randomized avatar changed
- chart, map, ad, or external widget changed
- animation captured mid-transition
- font fallback or image loading race
- data order changed without product meaning

Do not approve visual diffs just because the numeric threshold is small. Human review is required for meaningful UI changes.

## Pipeline Guidance

For PR pipelines:

- Run changed stories/pages and critical shared components.
- Upload baseline, actual, diff, trace, and screenshot artifacts.
- Block on large diffs in critical pages.
- Require review for intentional visual changes.

For nightly pipelines:

- Run broader visual regression across core pages, responsive viewports, themes, and browser targets.

For release pipelines:

- Verify approved baselines.
- Re-run critical page visual checks against staging.
- Block release on unapproved critical diffs.

## Output Expectations

When finished, report:

- visual test file, story, route, or screenshot target
- states and viewports covered
- stabilization choices such as fixed data, frozen time, disabled animation, or masks
- command run and result
- diff interpretation: expected change, regression, or noise
- whether baselines were updated or require approval
