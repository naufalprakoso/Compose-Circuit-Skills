# Clean Code and Review

## Clean Code

- Use clear `Screen`, `State`, `Event`, `Presenter`, and `Ui` names.
- Keep files, modules, and public APIs cohesive.
- Prefer constructor injection.
- Expose immutable data.
- Handle errors explicitly.
- Avoid hidden global mutable state.
- Avoid duplicated business rules in multiple presenters.
- Avoid repository calls from composables.
- Avoid network response types in UI state.
- Avoid generic `Any`, untyped maps, and stringly typed navigation when typed alternatives exist.
- Write comments that explain why, not what the code already says.
- Avoid premature abstraction.
- Avoid speculative use cases or interfaces with one trivial implementation unless they create a meaningful boundary.
- Avoid broad rewrites unrelated to the user request.

## Circuit-Specific Smells

- Presenter emits UI.
- Ui accesses repository, database, or network client.
- Screen carries excessive mutable data.
- Presenter is giant and unrelated concerns are interleaved.
- State uses unrelated booleans for mutually exclusive modes.
- Event sink mixes unrelated domains.
- Retention choice is incorrect.
- `Navigator`, `Context`, or views are stored in retained state.
- Coroutine scope is unbounded.
- Request repeats on recomposition.
- `SharedFlow` is used unnecessarily for UI-to-presenter events.
- State contains mutable collections.
- Raw exception is shown to users.
- Navigation triggers more than once.
- Cancellation is missing.
- Pagination can issue duplicate requests.
- Experimental API is used without acknowledgement.
- Android type leaks into `commonMain`.

## Coroutine-Specific Smells

- `CoroutineScope(...)` is constructed in a feature class without an injected owner.
- `GlobalScope`, production `runBlocking`, or thread sleeps appear in UI, presenter, repository, or tests.
- `Dispatchers.IO`, `Dispatchers.Default`, or `Dispatchers.Main` are hardcoded in logic that should be testable through injected dispatchers.
- A suspend API is not main-safe and pushes dispatcher choice to its caller.
- A broad `catch (Exception)` or `runCatching` maps `CancellationException` into an error state.
- `launch` starts work with no error path, no duplicate-event policy, and no test evidence.
- `async` is used without awaiting all children.
- `supervisorScope` is used without a documented partial-failure reason.
- A cold Flow is collected multiple times for expensive network/database work.
- `stateIn` or `shareIn` uses a scope whose lifetime is shorter or longer than the data it owns.
- `SharingStarted.WhileSubscribed` behavior is untested where upstream startup matters.
- `LaunchedEffect(Unit)` or `LaunchedEffect(true)` starts work that should instead be keyed, event-driven, or lower-layer owned.
- `rememberCoroutineScope` is used for work that must complete after the screen leaves composition.
- Retry, timeout, debounce, or polling logic uses real time in tests.
- A CPU-heavy loop has no suspension point, `isActive`, `ensureActive`, or `yield`.

## Review Approach

Start with evidence. Prioritize correctness, lifecycle, cancellation, security, data flow, accessibility, and user-visible impact over style. Trace data from source to repository/use case to presenter to state to UI. Provide concrete remediation and suggested verification.

Classify each item as defect, risk, opportunity, or style preference. Do not report style preferences as correctness defects.

## Circuit Data-Flow Trace

For every meaningful finding, trace the behavior through the Circuit loop:

```text
user action -> Ui event -> Presenter branch -> repository/use case -> state emission -> Ui rendering -> test evidence
```

For navigation and overlays, extend the trace:

```text
Ui event -> Presenter branch -> Navigator/overlay/result action -> stack or modal state -> restored state -> test evidence
```

For coroutine and networking issues, extend the trace:

```text
trigger -> coroutine owner -> dispatcher or upstream flow -> cancellation/error path -> state emission -> retry or recovery behavior
```

If a finding cannot be connected to one of these traces, report it as a lower-priority suggestion rather than a defect.
