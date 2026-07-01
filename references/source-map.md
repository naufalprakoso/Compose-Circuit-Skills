# Source Map

Sources last checked: 2026-07-01.

Latest Circuit release observed during this update: `0.34.0`. The upstream changelog also had unreleased SubCircuit Metro-mode fixes. Always inspect the target project's resolved Circuit version and, for snapshots or unreleased commits, inspect the matching source revision directly.

## Official Source Hierarchy

1. Target repository code, Gradle files, lockfiles, and existing conventions.
2. Slack Circuit official repository, docs, API reference, and changelog.
3. Android Developers docs for Compose performance, accessibility, testing, and coroutine guidance.
4. Kotlin official docs for Kotlin Multiplatform and Compose Multiplatform platform behavior.
5. User-supplied project guidance such as `AGENTS.md`.

## Official Links

| Area | Source |
| --- | --- |
| Codex skill format | https://developers.openai.com/codex/skills |
| Circuit repository | https://github.com/slackhq/circuit |
| Circuit docs home | https://slackhq.github.io/circuit/ |
| States and events | https://slackhq.github.io/circuit/getting-started/states-and-events/ |
| Presenter docs | https://slackhq.github.io/circuit/docs/presenter/ |
| Presenter scaling patterns | https://slackhq.github.io/circuit/docs/presenter-patterns/ |
| Ui docs | https://slackhq.github.io/circuit/docs/ui/ |
| Navigation docs | https://slackhq.github.io/circuit/docs/navigation/ |
| Navigation migration | https://slackhq.github.io/circuit/docs/navigation-navstack-migration/ |
| Overlays | https://slackhq.github.io/circuit/docs/overlays/ |
| Testing docs | https://slackhq.github.io/circuit/docs/testing/ |
| Code generation docs | https://slackhq.github.io/circuit/docs/code-gen/ |
| SubCircuit docs | https://slackhq.github.io/circuit/circuitx/subcircuit/ |
| Circuit changelog | https://slackhq.github.io/circuit/changelog/ |
| Compose performance | https://developer.android.com/develop/ui/compose/performance |
| Compose performance best practices | https://developer.android.com/develop/ui/compose/performance/bestpractices |
| Compose stability | https://developer.android.com/develop/ui/compose/performance/stability |
| Compose side effects | https://developer.android.com/develop/ui/compose/side-effects |
| Compose accessibility | https://developer.android.com/develop/ui/compose/accessibility |
| Compose accessibility testing | https://developer.android.com/develop/ui/compose/accessibility/testing |
| Android coroutine best practices | https://developer.android.com/kotlin/coroutines/coroutines-best-practices |
| Android coroutine testing | https://developer.android.com/kotlin/coroutines/test |
| Android Flow guide | https://developer.android.com/kotlin/flow |
| Android Flow testing | https://developer.android.com/kotlin/flow/test |
| Android StateFlow and SharedFlow | https://developer.android.com/kotlin/flow/stateflow-and-sharedflow |
| Kotlin coroutine cancellation and timeouts | https://kotlinlang.org/docs/cancellation-and-timeouts.html |
| Kotlin composing suspending functions | https://kotlinlang.org/docs/composing-suspending-functions.html |
| KMP expect/actual | https://kotlinlang.org/docs/multiplatform/multiplatform-expect-actual.html |
| KMP platform APIs | https://kotlinlang.org/docs/multiplatform/multiplatform-connect-to-apis.html |
| Compose Multiplatform behavior | https://kotlinlang.org/docs/multiplatform/compose-platform-specifics.html |

## Checking the Target Project Version

Search Gradle files and version catalogs for `com.slack.circuit`, `circuit-foundation`, `circuit-runtime`, `circuit-codegen`, `circuit-test`, `circuitx`, and plugin aliases. Then inspect dependency locking, generated dependency reports, or `./gradlew :module:dependencies` when needed. Also inspect source imports because older projects may use packages from older artifact splits.

After detecting the installed version, compare against the Circuit changelog and matching Git tag:

```bash
git ls-remote --tags https://github.com/slackhq/circuit.git
git clone --depth 1 --branch "$RESOLVED_CIRCUIT_TAG" https://github.com/slackhq/circuit.git /tmp/circuit-source
```

Use the tag that matches the project. If the project uses a snapshot or unreleased commit, inspect that commit or branch directly.

## Version-Sensitive Areas

| Area | Check |
| --- | --- |
| Navigation and NavStack APIs | `SaveableBackStack` versus `SaveableNavStack`, `rememberCircuitNavigator`, forward/backward APIs, `Navigator.StateOptions`, root-pop handling |
| Retained-state APIs | `rememberRetained`, `rememberRetainedSaveable`, `produceRetainedState`, lifecycle of retained records |
| Lifecycle presentation behavior | record lifecycle, paused records, composition locals, back-stack retention |
| Code generation modes | Anvil, Hilt, kotlin-inject-anvil, Metro, KSP args, assisted injection requirements |
| SubCircuit | availability of `circuitx-subcircuit`, `SubPresenter`, `SubUi`, `SubCircuitContent`, `@SubCircuitInject`, Anvil or Metro mode, test artifact |
| Overlays | `circuit-overlay`, `OverlayEffect`, dialog and sheet APIs, overlay versus `PopResult` |
| Gesture navigation | `circuitx-gesture-navigation`, predictive back behavior, platform support |
| Testing helpers | `presenterTestOf`, `Presenter.test`, `FakeNavigator`, `TestEventSink`, SubPresenter tests |
| Compose Multiplatform support | supported targets, source-set availability, UI behavior differences, codegen target support |

Do not copy examples from docs into product code blindly. Treat docs as API direction, then match local versions and conventions.
