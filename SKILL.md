---
name: compose-circuit-skills
description: Build, review, refactor, debug, and test Kotlin Android, Jetpack Compose, Compose Multiplatform, Kotlin Multiplatform, and Slack Circuit app features. Use for Android UI, Compose UI, Kotlin app architecture, Circuit Screen, Presenter, Ui, CircuitUiState, CircuitUiEvent, navigation, overlays, retained state, SubCircuit, Compose performance and accessibility, repositories, networking, coroutines, cross-platform boundaries, and UI/presenter tests. Also use when migrating an existing Android Compose or ViewModel feature to Circuit. Do not use for generic Kotlin backend, server-side Kotlin, CLI tools, or non-app library work unless the user asks for Android, Compose, KMP, or Circuit guidance.
---

# Compose Circuit Skills

## Overview

Use this skill to implement, review, debug, test, and refactor production-grade Kotlin Android, Jetpack Compose, Compose Multiplatform, Kotlin Multiplatform app, and Slack Circuit features. For plain Kotlin backend, server-side Kotlin, CLI tools, or non-app library work, use it only when the user asks for Android, Compose, KMP, or Circuit guidance.

## Source Priority

Prefer repository evidence first, then official Slack Circuit docs/source/changelog, then official Android and Kotlin Multiplatform docs. Use third-party material only when the user explicitly supplies it or official sources are insufficient. When a Circuit API is version-sensitive, inspect the target project's installed version and, if needed, check the matching official docs, source tag, or changelog entry before recommending code.

## Version Detection

Before recommending APIs or changing architecture, inspect:

- Gradle settings, module build files, convention plugins, and version catalogs.
- Kotlin, Compose, Compose Multiplatform, Android Gradle Plugin, Slack Circuit, coroutines, DI, networking, serialization, persistence, and test library versions.
- Target platforms, source sets, and Android-only versus KMP boundaries.
- Existing Circuit factory/codegen mode, navigation stack APIs, retained-state APIs, overlays, SubCircuit usage, and test helpers.

Do not silently upgrade dependencies. Use the project's resolved APIs unless the user asks for an upgrade or the change is necessary and explained.

## Task Mode

Classify the request as one or more modes:

- New Circuit feature
- Circuit code review
- Refactor or migration to Circuit
- Bug fix
- Performance investigation
- Accessibility improvement
- Networking or coroutine investigation
- Testing task
- Kotlin Multiplatform adaptation

For review-only requests, do not make code changes unless the user explicitly asks for fixes.

## Load References

Load only the references needed for the task:

| Task | References |
| --- | --- |
| New screen or feature | `architecture-state.md`, `navigation-retention.md`, `compose-ui-performance.md`, `coroutines-lifecycle.md`, `testing.md`, `examples.md` |
| Remote-data feature | `architecture-state.md`, `data-networking.md`, `network-resilience-security.md`, `coroutines-lifecycle.md`, `testing.md` |
| Code review or refactor | `clean-code-review.md`, `architecture-state.md`, plus affected-domain references |
| Navigation, results, overlays, retained state | `navigation-retention.md`, `architecture-state.md`, `testing.md`, `examples.md` |
| Circuit factory, codegen, or DI wiring | `codegen-di.md`, `architecture-state.md`, `source-map.md` if APIs are uncertain |
| Compose UI implementation | `compose-ui-performance.md`, `accessibility-quality.md`, `testing.md`, `examples.md` |
| Performance | `compose-ui-performance.md`, `testing.md`, `source-map.md` if measurement tooling or stability APIs are uncertain |
| Accessibility | `accessibility-quality.md`, `compose-ui-performance.md`, `testing.md`, `examples.md` |
| Coroutine, Flow, lifecycle, or async event work | `coroutines-lifecycle.md`, `testing.md`, `examples.md`, `source-map.md` if official guidance or API behavior is uncertain |
| Kotlin Multiplatform | `cross-platform.md`, plus relevant architecture/data/UI references |
| Circuit API uncertainty | `source-map.md`, then current official Circuit docs or matching source tag |

Do not load every reference by default.

## Repository Inspection

Inspect before coding:

1. Read relevant `AGENTS.md`, module guidance, and local conventions.
2. Map module boundaries and source sets.
3. Inspect Gradle and version catalog files.
4. Find existing `Screen`, `Presenter`, `Ui`, state, event, factory, and test patterns.
5. Detect manual factories versus `@CircuitInject` or KSP code generation.
6. Detect DI, navigation, overlays, retained state, record lifecycle, repository, use-case, networking, persistence, and coroutine conventions.
7. Discover test frameworks and runnable Gradle tasks from the repository.
8. Search for similar existing features and prefer consistent patterns when they are sound.

## Establish Invariants

Before implementation or review, identify expected user-visible behavior, state ownership, event ownership, navigation ownership, data source of truth, cancellation and lifecycle behavior, error and retry behavior, platform boundaries, accessibility requirements, and required test coverage. For refactors, preserve behavior unless the request explicitly changes it.

## Implementation Workflow

Make the narrowest coherent change. Follow existing module and package conventions. Add or update tests with production changes. Avoid unrelated cleanup, speculative abstractions, incidental dependency or framework migrations, dummy production data, fake production implementations, and unimplemented behavior.

## Review Workflow

Anchor every finding to repository evidence. Include file path and line number when possible. Explain impact and give a concrete correction. Distinguish verified defects from risks, opportunities, and style preferences. Do not output a generic checklist disconnected from the code.

Use this finding format:

```text
[Severity] Concise finding title

Evidence:
`path/to/File.kt:line`

Problem:
Explain the concrete issue.

Impact:
Explain the correctness, lifecycle, performance, accessibility, security, or maintainability impact.

Recommended change:
Give a specific correction.

Test:
Describe the test that should prove the fix.
```

Severity: Critical for likely security compromise, data loss, or severe production failure; High for correctness, crash, concurrency, lifecycle, or major navigation defects; Medium for meaningful maintainability, performance, resilience, or accessibility issues; Low for localized quality or consistency improvements.

## Non-Negotiable Circuit Rules

- Ui does not access repositories, databases, or network clients.
- Presenter does not emit Compose UI and remains presentation logic, not a data source.
- UI-to-presenter interactions use typed Circuit events.
- Mutually exclusive screen modes prefer sealed state over unrelated booleans.
- Navigation is typed and follows the project's Circuit convention.
- Context, Navigator, views, and other leak-prone objects are not stored in retained or saveable state.
- `GlobalScope` is prohibited, `CancellationException` is not swallowed, and blocking work does not run on the main thread.
- Suspend APIs are main-safe, dispatchers are injectable in testable logic, and Flow sharing uses an owner with the right lifetime.
- Long-running durable work is not owned solely by a screen presenter.
- Raw DTOs, HTTP responses, database cursors, transport exceptions, secrets, and tokens do not leak into UI state.
- Retry policies are bounded and error-aware; authentication secrets are never logged.
- Composable APIs follow modifier, state hoisting, slot, stability, lazy layout, semantics, and adaptive-layout conventions.
- Compose performance changes require evidence.
- `@Stable` and `@Immutable` are contracts, not warning suppressors.
- Accessibility is implemented and tested for new user-facing behavior.
- New production behavior includes appropriate tests.
- Existing project conventions are followed when sound.
- Experimental Circuit APIs are identified explicitly.
- Validation claims must correspond to commands actually run.

## Validation

Discover and run the smallest applicable checks first, then broader checks if practical:

- Compilation
- Kotlin or Android lint
- Unit tests, common tests, presenter tests, UI tests, navigation or overlay tests
- Android instrumentation tests or Compose UI tests
- Snapshot tests when already supported
- Accessibility checks
- Static analysis or relevant Gradle verification tasks

If a check cannot run, state the exact blocker and the command that should be rerun.

## Output Contracts

For implementation tasks, report summary, architecture decisions, files changed, behavior covered, tests and commands run, and remaining risks or assumptions.

For review tasks, report findings first, ordered by severity, then open questions, then brief supporting context.

## Definition of Done

Finish only when the requested mode is complete, the relevant references were applied, version-sensitive APIs were checked against the target project, tests or review evidence were collected, no known errors remain unreported, and the final response includes concrete validation evidence.
