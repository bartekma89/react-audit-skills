---
name: react-test-audit
description:
  Define and apply a React/TypeScript unit testing approach, testing
  methodology, test standards, and test strategy for auditing the `ui` package.
  Review Vitest, React Testing Library, MSW, React Query, Zustand, React Hook
  Form/Zod, and React Router for test quality, test architecture, mocking,
  isolation, coverage, and missing scenarios. Produce an evidence-based test
  audit report and improvement plan without modifying tests during the audit.
---

# React Test Audit

## Objective

Audit the **`ui` package only** and answer:

> Do the current tests provide meaningful confidence that the `ui` package
> behaves correctly?

The audit covers both:

- quality of existing tests;
- important missing test scenarios.

The output is a report followed by an actionable improvement plan.

## Scope

### In scope

- package: `ui` in the Lerna monorepo
- React + TypeScript
- Vitest
- React Testing Library
- MSW
- TanStack Query / React Query
- Zustand
- React Hook Form + Zod
- React Router
- package/test configuration used by `ui`
- colocated tests, typically:

```text
Foo.tsx
Foo.spec.tsx
```

### Out of scope

Do not audit the implementation or tests of internal dependencies such as:

- `@react-express/ui`
- `@react-express/ui-styleguide`
- `@react-express/form`
- other `@react-express/*` packages

Treat them as black-box dependencies from the perspective of `ui`.

Do not expand into E2E, performance, security, standalone accessibility
auditing, or framework migration.

## Project-Specific Rules

### Rendering

When a component does not require React Router or TanStack Query context:

```tsx
render(<Foo />);
```

When it requires Router and/or Query context, use the application's existing
`mockContext` helper.

Do not use `mockContext` unnecessarily and do not recreate equivalent
Router/Query wrappers in individual tests.

### `@react-express/utils/testing`

The relevant testing helper from this package is:

```ts
mockEventBusDispatch;
```

Use it when testing EventBus dispatch behavior. Do not invent a separate
EventBus mocking pattern when this helper is appropriate.

`mockContext` is application-level infrastructure; it does **not** come from
`@react-express/utils/testing`.

### Internal UI libraries

Treat `@react-express/ui` and `@react-express/ui-styleguide` as black boxes.
Their own tests are outside this audit.

Application tests should verify how `ui` uses these components and the resulting
application behavior, not their internal DOM/state/event-handler implementation.

### `@react-express/form`

This internal library wraps React Hook Form and Zod. Test the public behavior of
the application's form integration: user input, validation outcome, submit
behavior, relevant loading/success/error states, and resulting application
behavior.

Do not test RHF, Zod, or `@react-express/form` internals.

### Selectors

When a stable `data-testid` exists, `getByTestId` is an accepted project
convention and may be preferred.

When there is no suitable `data-testid`, use the appropriate React Testing
Library query.

Do not create a finding merely because a test uses `getByTestId`. Look for
brittle, ambiguous, or implementation-specific selectors instead.

### MSW

Use the existing MSW infrastructure:

```text
handlers
node/server
server.use(...)
```

`server.use(...)` is appropriate for targeted scenario overrides such as API
errors, empty results, or alternative responses.

Review whether handlers are realistic, isolated, reset correctly, and whether
important API behavior is unnecessarily hidden by module/service mocks.

## Audit Workflow

### 1. Discover the `ui` package

Inspect:

- `package.json` and relevant package metadata;
- Vitest configuration;
- test setup;
- scripts;
- MSW setup;
- shared test helpers;
- coverage configuration;
- local test documentation.

Use the repository's actual test scripts. Do not invent commands when a
canonical project command exists.

### 2. Map the tests

Identify:

- test files;
- tested components/hooks/utilities;
- integration-style tests;
- shared helpers;
- heavily tested areas;
- lightly or apparently untested areas.

Use source files to validate whether important behavior is actually covered.

### 3. Review what is tested

Check relevant behavior such as:

- normal/success path;
- loading state where relevant;
- empty state where relevant;
- validation;
- relevant API errors;
- user interactions;
- state transitions;
- navigation outcomes;
- EventBus behavior;
- meaningful edge cases.

Missing scenarios belong in the report. They are not automatically defects;
explain why they matter.

### 4. Review how it is tested

Load these references as needed:

- `references/react-testing.md`
- `references/typescript.md`
- `references/antipatterns.md`
- `references/test-types.md`

Focus on behavior vs implementation details, assertions, selectors,
rendering/context, mocking, MSW, isolation, async behavior, and appropriate test
level.

### 5. Check false confidence

For important tests ask:

> Could production behavior be broken while this test still passes?

Pay particular attention to:

- mock-only verification;
- weak assertions;
- mocking the code whose behavior should be tested;
- unrealistic API mocks;
- implementation-detail assertions.

Only create a finding when supported by concrete repository evidence.

### 6. Review isolation and determinism

Check:

- shared mutable fixtures;
- QueryClient leakage;
- router state leakage;
- EventBus mock leakage;
- MSW handler leakage;
- global mocks;
- timers;
- order-dependent tests.

When safe, validate suspicious cases by running the smallest relevant test
subset first.

### 7. Review configuration

Report:

- runner and environment;
- setup files;
- coverage configuration/thresholds;
- MSW configuration;
- test scripts;
- relevant global mocks.

Recommend configuration changes only when supported by evidence.

### 8. Review coverage

When available, report:

- statements;
- branches;
- functions;
- lines;
- thresholds.

Interpret coverage separately from test effectiveness. Identify important
untested behavior and weak coverage that may create false confidence.

### 9. Identify good practices

Explicitly preserve useful patterns such as sensible `render`/`mockContext`
usage, MSW, `mockEventBusDispatch`, colocated tests, and black-box use of
internal libraries.

## Findings

Use stable IDs:

```text
CONF-xxx
TEST-xxx
REACT-xxx
MOCK-xxx
ISO-xxx
ASYNC-xxx
COV-xxx
ARCH-xxx
```

Every substantive finding must include:

- Severity;
- Confidence;
- Status;
- File and line(s);
- Evidence;
- Problem;
- Why it matters;
- Recommendation;
- Reference when relevant.

### Severity

Use `Critical`, `High`, `Medium`, or `Low`.

Reserve `Critical` for issues that substantially undermine confidence in
important functionality or a large part of the suite. Do not inflate severity.

### Confidence

Use `High`, `Medium`, or `Low`.

For uncertain conclusions use:

```text
Status: Needs manual verification
```

rather than presenting an inference as fact.

## Missing Test Scenarios

Missing scenarios are part of the audit. Prioritize using:

1. business/user impact;
2. meaningful branching behavior;
3. error risk;
4. implementation complexity;
5. regression risk.

For each important gap document:

- feature;
- missing scenario;
- current coverage;
- why it matters;
- suggested test level.

Do not demand every theoretical edge case.

## Verification

Use the project's normal commands, preferably:

```text
targeted tests
    ↓
full test suite
    ↓
coverage
    ↓
lint/typecheck/build when part of normal verification
```

During the audit do not modify production code or tests. If a command cannot be
run, document the limitation.

## Report

Generate `TEST_AUDIT.md` using `references/report-template.md`.

The report must distinguish:

- observations;
- confirmed findings;
- uncertain/manual-review items;
- missing scenarios;
- recommendations.

End with an actionable improvement plan.

Do not give the repository an overall score, grade, ranking, or quality label.

## Completion Checklist

- [ ] Only `ui` package audited
- [ ] Test environment identified
- [ ] Test landscape mapped
- [ ] Configuration reviewed
- [ ] Existing test quality reviewed
- [ ] React-specific practices reviewed
- [ ] Assertions reviewed
- [ ] Mocking and MSW reviewed
- [ ] EventBus testing reviewed where relevant
- [ ] Isolation reviewed
- [ ] Async behavior reviewed
- [ ] Coverage reported/interpreted when available
- [ ] Missing scenarios identified
- [ ] Good practices documented
- [ ] Findings backed by evidence
- [ ] Improvement plan produced
- [ ] Verification plan produced
- [ ] Limitations documented
- [ ] `TEST_AUDIT.md` generated
