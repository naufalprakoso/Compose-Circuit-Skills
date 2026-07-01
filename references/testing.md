# Testing

## Test Layers

1. Domain and use-case tests
2. Repository and data tests
3. Presenter tests
4. UI tests
5. Navigation and overlay tests
6. Networking tests
7. Integration tests
8. Snapshot tests when already supported
9. Accessibility tests

## Presenter Tests

Use current Circuit testing APIs supported by the installed version, such as `Presenter.test()`, `presenterTestOf()`, `FakeNavigator`, `TestEventSink`, and SubPresenter utilities when available.

Test initial state, loading to content, loading to error, empty state, event handling, retry, navigation, pop or result behavior, duplicate events, cancellation, refresh, mutation success/failure, and retained-state behavior where important.

Prefer fakes over mocking Circuit core components. Do not compare whole state objects blindly when event-sink lambda equality makes that misleading. Assert meaningful properties and behavior. Invoke events from the current emitted state when the architecture requires it.

## UI Tests

Construct `UiState` directly and test content rendering, loading, empty, error, event emission, semantics, enabled/disabled states, long text, scroll behavior, accessibility labels, touch actions, and responsive variants. Use `TestEventSink` when appropriate.

## Networking Tests

Use the project's established tool: MockWebServer, Ktor MockEngine, fake transport, contract fixture, or equivalent.

Cover success, serialization failure, timeout, offline behavior, authentication refresh, rate limiting, retry boundaries, cancellation, redaction, and pagination.

## Coroutine Tests

- Wrap coroutine tests in `runTest`.
- Inject `TestDispatcher` instances instead of using real dispatchers.
- Make all test dispatchers share the same `testScheduler`.
- Prefer `StandardTestDispatcher` for deterministic ordering, `advanceUntilIdle`, and virtual-time assertions.
- Use `UnconfinedTestDispatcher` only when eager execution is intentional and ordering is not the behavior under test.
- Use virtual time for debounce, delay, retry, polling, and timeout.
- Avoid real delays and sleeps.
- Verify cancellation is propagated, especially through broad catches and `runCatching`-like helpers.
- Verify timeout paths and cleanup paths.
- Verify latest-query behavior with `flatMapLatest`, `collectLatest`, debounce, or manual job replacement.
- Verify retries terminate and do not retry non-transient failures.
- Verify duplicate operations do not race, double-submit, or navigate twice.
- Verify parallel work succeeds, cancels siblings on failure, or isolates sibling failure when `supervisorScope` is intentional.
- Replace Flow producers with fakes that can emit controlled values.
- Use `first()` for single-emission Flow assertions and `toList()` only for finite flows.
- Keep an active collector when asserting a `StateFlow` created with `stateIn(SharingStarted.WhileSubscribed(...))`.
- Assert behavior and meaningful state fields rather than whole state equality when event sink lambdas are present.

## Snapshot Tests

Use snapshot tests for visual regression when the project already supports them. Do not introduce snapshot infrastructure for a small change without justification.

## Validation Commands

Discover commands from the repository. Prefer narrow module tasks first, such as a single presenter test or module compile, then run broader checks if the change touches shared contracts. Do not invent Gradle paths; inspect `settings.gradle`, included builds, convention plugins, and CI scripts.
