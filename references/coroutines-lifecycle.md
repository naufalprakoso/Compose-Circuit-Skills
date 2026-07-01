# Coroutines and Lifecycle

## Contents

- [Structured Concurrency](#structured-concurrency)
- [Dispatcher and Main-Safety Policy](#dispatcher-and-main-safety-policy)
- [API Shape](#api-shape)
- [Circuit Presenter Work](#circuit-presenter-work)
- [Parallel Work](#parallel-work)
- [Cancellation and Cleanup](#cancellation-and-cleanup)
- [Flow](#flow)
- [Compose Side Effects](#compose-side-effects)
- [Event Handling](#event-handling)
- [Testing Coroutines](#testing-coroutines)

## Structured Concurrency

- Do not use `GlobalScope`.
- Do not create arbitrary unmanaged `CoroutineScope` instances.
- Do not use `runBlocking` in production UI or presenter code.
- Tie work to an explicit owner.
- Allow cancellation to propagate.
- Do not swallow `CancellationException`.
- Use supervisor behavior only when sibling failure isolation is intentional and documented.
- Do not use `async` for fire-and-forget work. Await every `Deferred` or use `launch` with an explicit error path.
- Avoid storing `Job` handles unless cancellation, replacement, or status inspection is part of the behavior.

## Dispatcher and Main-Safety Policy

- Make suspend functions main-safe.
- Move blocking operations to an appropriate dispatcher in the data layer or platform boundary.
- Inject dispatchers instead of hardcoding `Dispatchers.IO`, `Dispatchers.Default`, or `Dispatchers.Main` inside production logic.
- Avoid scattering `withContext(Dispatchers.IO)` throughout presentation code.
- Let repository/data APIs own blocking concerns.
- Keep dispatcher choice close to the code that performs blocking IO, CPU-heavy mapping, encryption, serialization, or platform calls.
- Prefer a project-local `DispatcherProvider`, named dispatcher bindings, or constructor-injected `CoroutineDispatcher` values that tests can replace.
- On KMP, hide platform dispatcher differences behind common interfaces or DI bindings rather than leaking platform dispatchers into presentation code.

Main-safe means callers can invoke the suspend API from the main thread without knowing whether it performs network, disk, CPU, or platform work.

```kotlin
class ProductRepository(
  private val service: ProductService,
  private val ioDispatcher: CoroutineDispatcher,
) {
  suspend fun product(id: ProductId): Product =
    withContext(ioDispatcher) {
      service.fetchProduct(id).toDomain()
    }
}
```

## API Shape

- Expose `suspend` functions for one-shot operations.
- Expose `Flow` for data that changes over time.
- Expose immutable `StateFlow` only from owners that truly maintain observable state.
- Do not expose mutable flow types outside their owner.
- Do not make UI or presenter callers choose dispatchers for data-layer work.
- Map infrastructure exceptions into domain/result types before state reaches UI.
- Keep long-lived background work behind repository, use-case, app-scope, or durable scheduler APIs.

## Circuit Presenter Work

Choose the owner based on lifetime:

- Use `LaunchedEffect` for composition-driven work keyed to screen inputs.
- Use a remembered composition scope for event-triggered suspend work that should cancel with the presenter.
- Use retained-state producers available in the installed Circuit version for observable screen state.
- Use repositories for shared long-running streams and caching.
- Use WorkManager or another durable scheduler for work that must survive UI lifetime.

Long-running or durable operations must not depend solely on a presenter remaining composed.

| Work lifetime | Preferred owner |
| --- | --- |
| Re-run when a screen key changes | `LaunchedEffect(key)` or retained producer keyed by that input |
| User event that may cancel when screen leaves composition | `rememberCoroutineScope()` from the presenter call site |
| Work that should complete after the user leaves the screen | Repository/use-case method backed by an injected external scope |
| App/session-wide stream or cache | Repository or app-scoped data owner |
| Work that must survive process/UI lifetime | Durable scheduler or platform background-work abstraction |

Avoid launching business work directly from composable rendering. UI emits events; the presenter or a lower layer owns the coroutine.

## Parallel Work

- Assume suspend calls are sequential unless parallelism is required.
- Use `coroutineScope { async { ... } }` for parallel subtasks whose failures should cancel siblings.
- Use `supervisorScope` only when one child may fail without cancelling the others.
- Keep parallel decomposition inside a suspend function so callers still see one main-safe API.
- Await all children before returning, and map errors at the boundary that owns the behavior.

```kotlin
suspend fun productDetail(id: ProductId): ProductDetail =
  coroutineScope {
    val product = async { productRepository.product(id) }
    val recommendations = async { recommendationRepository.forProduct(id) }
    ProductDetail(product.await(), recommendations.await())
  }
```

## Cancellation and Cleanup

- Treat cancellation as a normal control path, not an error to map into failed UI.
- Always rethrow `CancellationException` from broad catches.
- Avoid `runCatching` around suspend work unless cancellation is rethrown before failure mapping.
- For CPU-heavy loops, call `ensureActive()`, check `isActive`, yield, or reach suspending functions frequently.
- Use `withTimeout` or `withTimeoutOrNull` for bounded waits, but remember timeout is cooperative.
- Use `NonCancellable` only for short cleanup that must finish, such as closing or flushing a resource, and document why.

```kotlin
try {
  repository.submit()
} catch (e: CancellationException) {
  throw e
} catch (e: IOException) {
  state = SubmitState.Failed
}
```

## Flow

- Distinguish cold flows from hot shared state.
- Avoid duplicate collection of expensive upstream flows.
- Use `stateIn` or `shareIn` in the appropriate owner with an explicit `SharingStarted` policy and initial value when needed.
- Use `distinctUntilChanged` only where equality reflects meaningful UI changes.
- Use `debounce` for user input and `flatMapLatest` for replacing outdated searches.
- Use `collectLatest` when only the latest result should update UI.
- Use `combine` for derived presentation state.
- Place `catch` where it handles the intended upstream failures and preserves cancellation.
- Use retry operators with bounded and typed policies.
- Avoid turning every value into a global `StateFlow`.
- Do not collect the same cold network/database flow from multiple places when one shared owner should fan out values.
- Prefer `first()` for one-shot reads from a flow in suspend logic.
- Be explicit about backpressure: `conflate`, `buffer`, `debounce`, and `sample` change behavior and need a product reason.

## Compose Side Effects

- Use `LaunchedEffect` with stable keys for suspend work tied to a composition point.
- Treat `LaunchedEffect(Unit)` and `LaunchedEffect(true)` as suspicious; use them only for work that should start once for that call site.
- Use `rememberUpdatedState` when an effect must keep running while observing the latest lambda or value.
- Use `DisposableEffect` for subscriptions or callbacks that require cleanup.
- Use `snapshotFlow` when Compose state must become a Flow.
- Use `produceState` or retained Circuit producers when an async source should drive state.
- Use `rememberCoroutineScope` for event handlers such as snackbar display or presenter-owned event work, not for work that must outlive the screen.

## Event Handling

- Prevent duplicate submissions when operations are not idempotent.
- Disable, serialize, or conflate actions intentionally.
- Read latest state when launching event work.
- Handle races between refresh and mutation.
- For optimistic updates, define rollback and reconciliation.
- Navigate after successful completion when correctness requires it.
- Record whether a launched operation can be replaced, queued, ignored, or joined when the same event arrives again.
- Keep event handlers small enough that validation, state changes, business calls, navigation, and tracking are traceable.

## Testing Coroutines

- Use `runTest`.
- Replace real dispatchers with `TestDispatcher`.
- Make all `TestDispatcher` instances share the same `testScheduler`.
- Prefer `StandardTestDispatcher` when ordering and virtual-time control matter.
- Use `UnconfinedTestDispatcher` only when eager execution simplifies a test without hiding ordering bugs.
- Use virtual time for debounce, delay, retry, and timeout.
- Verify cancellation paths.
- Verify timeout paths.
- Verify latest-query and duplicate-submit behavior.
- Verify parallel success and partial-failure behavior when `async` or `supervisorScope` is used.
- Avoid real delays.
- Avoid reliance on `Dispatchers.Main` in local tests unless explicitly configured.
- For `stateIn(SharingStarted.WhileSubscribed(...))`, keep at least one collector active in tests or the upstream flow may not start.
