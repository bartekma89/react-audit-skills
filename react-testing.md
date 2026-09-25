# React Testing Methodology

This document contains only the React-specific rules needed for auditing the `ui` package.

## 1. Behavior over implementation

Prefer verifying application behavior and outcomes over implementation details.

Potential implementation-detail coupling includes:
- internal React state;
- internal callback wiring when the outcome can be observed;
- internal component calls;
- unstable DOM/CSS structure;
- internals of dependency libraries.

Flag these only when they create meaningful fragility or false confidence.

## 2. Rendering

Use the project convention.

### No Router/Query dependency

```tsx
render(<Component />);
```

### Router/Query dependency

Use the application's existing:

```text
mockContext
```

Do not recreate equivalent providers manually when `mockContext` already provides the required context.

Do not use `mockContext` unnecessarily.

## 3. Selectors

Project preference:

```text
data-testid available
    ↓
getByTestId is acceptable/preferred

no suitable data-testid
    ↓
use the appropriate RTL query
```

Do not report `getByTestId` itself as a defect. Look for unstable, ambiguous, or implementation-specific selectors.

## 4. User interactions

Prefer `userEvent` for normal user interactions.

Review `fireEvent` rather than treating it as automatically wrong. A finding is justified when a normal user interaction is simulated at a lower level without a clear reason.

## 5. Internal UI libraries

Treat:

```text
@react-express/ui
@react-express/ui-styleguide
```

as black boxes.

Application tests should verify application behavior around them, not their internal DOM, state, or event-handler implementation.

## 6. Forms

`@react-express/form` wraps React Hook Form and Zod.

Test the application's public behavior:

```text
user input
→ validation
→ submit
→ loading/success/error
→ resulting behavior
```

Do not test the internals of `@react-express/form`, React Hook Form, or Zod.

## 7. React Query

When relevant, inspect:
- loading state;
- success state;
- empty result where relevant;
- error state where relevant;
- mutation behavior;
- invalidation/refetch where it affects the feature;
- QueryClient isolation.

Do not require tests for every library mechanism used internally.

## 8. Zustand

When Zustand affects behavior, check:
- important state transitions;
- reset/isolation;
- meaningful user interaction;
- leakage between tests.

Avoid direct store manipulation when normal application interaction can exercise the behavior. Direct store tests can be appropriate for store-specific logic.

## 9. React Router

Prefer observable navigation behavior:

```text
user action
→ navigation
→ expected route/screen
```

Use `mockContext` when the application test needs the project's Router context.

## 10. MSW

Use the existing MSW setup:

```text
handlers
node/server
server.use(...)
```

`server.use(...)` is appropriate for targeted scenario overrides such as API failure, empty result, or alternative response.

Audit for unrealistic responses, missing important response variants, leaked overrides, and unnecessary bypassing of the HTTP boundary.

## 11. EventBus

When a component dispatches EventBus actions, use:

```ts
mockEventBusDispatch()
```

from `@react-express/utils/testing` when appropriate.

Prefer verifying the dispatched event/action contract rather than replacing the whole EventBus with an unrelated module mock.

## 12. Important UI scenarios

For interactive/data-driven components, identify scenarios that materially affect behavior:

```text
initial
loading
success
empty
validation error
server error
user interaction
recovery/retry
navigation
```

Do not require all states for every component. Select scenarios based on actual behavior and risk.

## 13. Async tests

Synchronize through observable outcomes.

Prefer:

```ts
await user.click(button);
expect(await screen.findByText('Saved')).toBeInTheDocument();
```

over arbitrary sleeps.

Be suspicious of `setTimeout`, `sleep`, and unnecessarily broad `waitFor` blocks. Do not flag `waitFor` merely because it exists; assess whether it is needed and correct.

## 14. Assertions

A useful assertion proves intended behavior.

Potentially weak patterns:

```ts
expect(result).toBeDefined();
expect(mockFn).toHaveBeenCalled();
```

Flag only when they fail to prove an important expected outcome.

## 15. What not to flag

Do not report these by themselves:
- use of `getByTestId`;
- use of `fireEvent`;
- absence of `mockContext` in a simple component;
- absence of direct React Hook Form tests;
- absence of E2E;
- lack of 100% coverage;
- use of internal UI components without testing their internals.

A finding needs a concrete quality or coverage reason.

## 16. Review priority

When time is limited, prioritize:
1. false-confidence tests;
2. missing important behavior;
3. isolation/flakiness;
4. incorrect mocking boundaries;
5. weak assertions;
6. maintainability issues.

Do not spend significant audit time on style-only differences.
