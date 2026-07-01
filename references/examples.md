# Examples

## Table of Contents

- [Basic Screen, State, Event, Presenter, and Ui](#basic-screen-state-event-presenter-and-ui)
- [Sealed Loading, Content, and Error State](#sealed-loading-content-and-error-state)
- [Repository-Backed Presenter](#repository-backed-presenter)
- [Retained Flow Observation](#retained-flow-observation)
- [Event Suspend Work](#event-suspend-work)
- [Main-Safe Repository](#main-safe-repository)
- [Work That Must Complete](#work-that-must-complete)
- [Parallel Suspend Composition](#parallel-suspend-composition)
- [Navigator Usage](#navigator-usage)
- [Pop Result](#pop-result)
- [Overlay Handling](#overlay-handling)
- [Presenter Decomposition](#presenter-decomposition)
- [State Producer](#state-producer)
- [SubCircuit](#subcircuit)
- [StaticScreen](#staticscreen)
- [Presenter Test](#presenter-test)
- [Ui Test](#ui-test)
- [Coroutine Retry Test](#coroutine-retry-test)
- [KMP Boundary](#kmp-boundary)
- [Before and After](#before-and-after)

Use the target project's resolved Circuit version and imports. The short examples show architecture shape. The compile-oriented slices include the local assumptions they rely on; adapt imports, annotations, and DI to the project.

## Compile-Oriented Slice Assumptions

Examples assume current Circuit concepts such as `Screen`, `CircuitUiState`, `CircuitUiEvent`, `Presenter`, `Navigator`, `FakeNavigator`, and `TestEventSink`. If a project uses older APIs, check `source-map.md` and the matching Circuit source tag before copying names.

## Basic Screen, State, Event, Presenter, and Ui

```kotlin
data class ProductDetailsScreen(val productId: String) : Screen {
  data class State(
    val title: String,
    val eventSink: (Event) -> Unit,
  ) : CircuitUiState

  sealed interface Event : CircuitUiEvent {
    data object BackClicked : Event
  }
}

class ProductDetailsPresenter(
  private val screen: ProductDetailsScreen,
  private val navigator: Navigator,
) : Presenter<ProductDetailsScreen.State> {
  @Composable
  override fun present(): ProductDetailsScreen.State {
    return ProductDetailsScreen.State(
      title = "Product ${screen.productId}",
      eventSink = { event ->
        when (event) {
          ProductDetailsScreen.Event.BackClicked -> navigator.pop()
        }
      },
    )
  }
}

@Composable
fun ProductDetails(state: ProductDetailsScreen.State, modifier: Modifier = Modifier) {
  Column(modifier) {
    Text(state.title)
    Button(onClick = { state.eventSink(ProductDetailsScreen.Event.BackClicked) }) {
      Text("Back")
    }
  }
}
```

## Sealed Loading, Content, and Error State

```kotlin
sealed interface ProductState : CircuitUiState {
  data object Loading : ProductState
  data class Content(val product: ProductUiModel, val eventSink: (ProductEvent) -> Unit) : ProductState
  data class Error(val message: String, val eventSink: (ProductEvent) -> Unit) : ProductState
}
```

## Repository-Backed Presenter

```kotlin
class ProductPresenter(
  private val screen: ProductDetailsScreen,
  private val repository: ProductRepository,
) : Presenter<ProductState> {
  @Composable
  override fun present(): ProductState {
    var reloadKey by rememberRetained { mutableIntStateOf(0) }
    val loadState by produceRetainedState<ProductLoadState>(
      ProductLoadState.Loading,
      screen.productId,
      reloadKey,
    ) {
      value = repository.loadProduct(screen.productId).toProductLoadState()
    }

    return when (val state = loadState) {
      ProductLoadState.Loading -> ProductState.Loading
      is ProductLoadState.Loaded -> ProductState.Content(state.product.toUiModel()) { event ->
        when (event) {
          ProductEvent.Retry -> Unit
        }
      }
      is ProductLoadState.Failed -> ProductState.Error(state.userMessage) { event ->
        when (event) {
          ProductEvent.Retry -> reloadKey++
        }
      }
    }
  }
}
```

## Retained Flow Observation

```kotlin
val order by produceRetainedState<Order?>(null, screen.orderId) {
  repository.observeOrder(screen.orderId).collect { value = it }
}
```

Check installed Circuit APIs before choosing `produceRetainedState`, `rememberRetained`, or project-specific retained helpers.

## Event Suspend Work

```kotlin
var submitState by rememberRetained { mutableStateOf<SubmitState>(SubmitState.Idle) }
val scope = rememberCoroutineScope()

return CheckoutState(submitState) { event ->
  when (event) {
    CheckoutEvent.Submit -> {
      if (submitState != SubmitState.Submitting) {
        scope.launch {
          submitState = SubmitState.Submitting
          submitState =
            try {
              submitOrder()
              SubmitState.Complete
            } catch (e: CancellationException) {
              throw e
            } catch (e: IOException) {
              SubmitState.Failed
            }
        }
      }
    }
  }
}
```

Never use `runCatching` around suspend work unless cancellation is rethrown before mapping failures.

## Main-Safe Repository

```kotlin
class ProductRepository(
  private val service: ProductService,
  private val ioDispatcher: CoroutineDispatcher,
) {
  suspend fun product(id: ProductId): Product =
    withContext(ioDispatcher) {
      service.fetchProduct(id).toDomain()
    }

  fun observeProduct(id: ProductId): Flow<Product> =
    service.observeProduct(id).map { it.toDomain() }
}
```

The caller should not choose `Dispatchers.IO`. Inject dispatchers so tests can replace them with `TestDispatcher` instances.

## Work That Must Complete

```kotlin
class BookmarkRepository(
  private val dataSource: BookmarkDataSource,
  private val externalScope: CoroutineScope,
) {
  suspend fun bookmark(id: ProductId) {
    externalScope.async {
      dataSource.bookmark(id)
    }.await()
  }
}
```

Use an app, session, or navigation-graph scope only when the operation should complete after the current presenter leaves composition. Keep ordinary screen work tied to the presenter. Use `async/await` when the caller needs failures propagated; reserve `launch/join` for explicit best-effort work with a separate error policy.

## Parallel Suspend Composition

```kotlin
suspend fun productDetail(id: ProductId): ProductDetail =
  coroutineScope {
    val product = async { productRepository.product(id) }
    val reviews = async { reviewRepository.reviewsFor(id) }
    ProductDetail(product.await(), reviews.await())
  }
```

Use `supervisorScope` only when one failed child should not cancel siblings, and test that partial-failure behavior explicitly.

## Navigator Usage

```kotlin
eventSink = { event ->
  when (event) {
    is ProductEvent.SellerClicked -> navigator.goTo(SellerScreen(event.sellerId))
    ProductEvent.BackClicked -> navigator.pop()
  }
}
```

At the root:

```kotlin
val navStack = rememberSaveableNavStack(root = HomeScreen)
val navigator = rememberCircuitNavigator(navStack, onRootPop = { finish() })
NavigableCircuitContent(navigator = navigator, backStack = navStack)
```

Use the target version's navigation API names.

## Pop Result

```kotlin
data object FilterScreen : Screen {
  data class Result(val query: String) : PopResult
}

val filterNavigator = rememberAnsweringNavigator<FilterScreen.Result>(navigator) { result ->
  query = result.query
}
```

## Overlay Handling

```kotlin
data class DeleteState(
  val itemName: String,
  val modal: DeleteModal?,
  val eventSink: (DeleteEvent) -> Unit,
) : CircuitUiState

sealed interface DeleteModal {
  data object ConfirmDelete : DeleteModal
  data object Deleting : DeleteModal
  data class Failed(val message: String) : DeleteModal
}

sealed interface DeleteEvent : CircuitUiEvent {
  data object DeleteClicked : DeleteEvent
  data object ConfirmDeleteClicked : DeleteEvent
  data object DismissModal : DeleteEvent
}
```

Use the installed overlay artifact and project conventions for dialog rendering. Prefer an overlay for transient confirmation over a navigated screen unless the flow is a destination that should return a typed `PopResult`. Avoid a lone boolean when async work, failure, result data, or multiple modal states are involved.

## Presenter Decomposition

```kotlin
@Composable
override fun present(): OrderState {
  val order = observeOrder()
  val payment = observePayment()
  return buildState(order, payment)
}
```

Extract observation and mapping before introducing child presenters.

## State Producer

```kotlin
class AvailabilityStateProducer(private val repository: InventoryRepository) {
  @Composable
  fun produce(productId: String): AvailabilityState {
    val inventory by produceRetainedState<Inventory?>(null, productId) {
      value = repository.loadInventory(productId)
    }
    return inventory.toAvailabilityState()
  }
}
```

## SubCircuit

```kotlin
sealed interface PriceCardEvent {
  data class OpenTerms(val productId: String) : PriceCardEvent
}

data class PriceCardScreen(val productId: String) : SubScreen<PriceCardEvent>

data class PriceCardState(
  val price: String,
  val eventSink: (PriceCardUiEvent) -> Unit,
) : SubCircuitUiState

sealed interface PriceCardUiEvent {
  data object TermsClicked : PriceCardUiEvent
}

class PriceCardPresenter(
  private val screen: PriceCardScreen,
) : SubPresenter<PriceCardEvent, PriceCardState> {
  @Composable
  override fun present(outerEventSink: (PriceCardEvent) -> Unit): PriceCardState {
    return PriceCardState(price = "...") {
      outerEventSink(PriceCardEvent.OpenTerms(screen.productId))
    }
  }
}
```

Use SubCircuit only when the installed version includes it and the child component has its own reusable presenter/UI contract while delegating navigation or cross-cutting events to the parent. Do not introduce SubCircuit just to split a small composable or avoid passing a callback.

## StaticScreen

```kotlin
data object AboutScreen : StaticScreen

@Composable
fun About(modifier: Modifier = Modifier) {
  Text("About", modifier)
}
```

Use only when no presenter-managed state is needed.

## Presenter Test

```kotlin
@Test
fun opensSeller() = runTest {
  val navigator = FakeNavigator(ProductDetailsScreen("p1"))
  val presenter = ProductPresenter(ProductDetailsScreen("p1"), navigator, repository = FakeProductRepository())

  presenter.test {
    val state = awaitItem() as ProductState.Content
    state.eventSink(ProductEvent.SellerClicked("s1"))
    assertThat(navigator.awaitNextScreen()).isEqualTo(SellerScreen("s1"))
  }
}
```

Use assertions supported by the project's test stack.

## Ui Test

```kotlin
@Test
fun retryClickEmitsEvent() {
  val sink = TestEventSink<ProductEvent>()
  composeRule.setContent {
    ProductError(message = "Offline", eventSink = sink)
  }
  composeRule.onNodeWithText("Retry").performClick()
  sink.assertEvent(ProductEvent.Retry)
}
```

## Coroutine Retry Test

```kotlin
@Test
fun retryStopsAfterBound() = runTest {
  val attempts = AtomicInteger(0)

  try {
    retryTransient(maxAttempts = 3) {
      attempts.incrementAndGet()
      throw IOException("timeout")
    }
    fail("Expected retry to fail")
  } catch (e: IOException) {
    assertThat(e.message).isEqualTo("timeout")
  }

  assertThat(attempts.get()).isEqualTo(3)
}
```

## Dispatcher Injection Test

```kotlin
@Test
fun repositoryUsesInjectedDispatcher() = runTest {
  val dispatcher = StandardTestDispatcher(testScheduler)
  val repository = ProductRepository(fakeService, dispatcher)

  val product = async { repository.product(ProductId("p1")) }

  assertThat(product.isCompleted).isFalse()
  advanceUntilIdle()
  assertThat(product.await().id).isEqualTo(ProductId("p1"))
}
```

All `TestDispatcher` instances in a test should share the `runTest` scheduler.

## KMP Boundary

```kotlin
interface SecureTokenStore {
  suspend fun readToken(): String?
  suspend fun writeToken(token: String)
}

class AuthRepository(private val tokenStore: SecureTokenStore) {
  suspend fun currentToken(): String? = tokenStore.readToken()
}
```

Provide Android and iOS implementations at platform entry points. Prefer this over `expect`/`actual` when tests need fakes or behavior may vary.

## Before and After

Boolean soup to sealed state:

```kotlin
// Before
data class State(val isLoading: Boolean, val error: Throwable?, val data: Product?)

// After
sealed interface State : CircuitUiState {
  data object Loading : State
  data class Content(val product: ProductUiModel) : State
  data class Error(val message: String, val eventSink: (Event) -> Unit) : State
}
```

Repository call in Ui to presenter flow:

```kotlin
// Before: a composable launches a repository call directly during rendering.
// After: a presenter observes the repository and emits render-ready state.
```

`GlobalScope` to structured concurrency:

```kotlin
val scope = rememberCoroutineScope()
scope.launch { repository.submit() }
```

For work that must complete after the screen leaves composition, move the lifetime down:

```kotlin
class OrderRepository(private val externalScope: CoroutineScope) {
  suspend fun submit(order: Order) {
    externalScope.async { submitRemote(order) }.await()
  }
}
```

Raw DTO state to presentation model:

```kotlin
data class ProductUiModel(val title: String, val priceText: String)
```
