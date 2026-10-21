---
name: android-testing-doctrine
description: >
  ARC's binding Android testing doctrine — what makes a test real, and the gate every test must
  pass. Read BEFORE creating any "*Test.kt" file or discussing Android "unit test", "JUnit 5",
  "JUnit5", "@Test", "runTest", "kotlinx-coroutines-test", "Turbine", "fake", "mock", "MockK",
  "Robolectric", "ComposeTestRule", "createComposeRule", "screenshot test",
  "Compose Preview Screenshot Testing", "validateDebugScreenshotTest", "reference images",
  "coverage", "JaCoCo", "TDD", "golden test vectors", "audit the tests", "tautological test".
  A test proves observable behavior of production code against an independent oracle.
  Kotlin/Gradle counterpart of swift-testing-doctrine.
---

# Tests That Actually Test — Android (ARC Labs)

**Precedence.** This skill decides whether a test is worth writing and how ARC structures it.
`android-skills:testing-setup` covers infrastructure mechanics (dependencies, runners, Gradle
setup, screenshot-test plugin vs. AGP test suites). Where it defaults differently — it suggests
JUnit 4 and MockK — **ARC's choices below win**; otherwise follow it for setup detail.

## The single principle

A test verifies **observable behavior of production code** against an **oracle independent of
that code** (fixture content, external requirement, separate calculation). If the assertion
derives from the same line you wrote in production, it is not a test.

**Mnemonic: if the test would pass whenever the code compiles, delete it.**

Kotlin's compiler already guarantees types, `when` exhaustiveness over sealed types, nullability
and data-class structure. Blacklist (never write): `entries.size == N` · `data class` field
round-trips (`Foo(name = "Ana").name == "Ana"`) · `copy()`/`equals()` of data classes ·
mapper round-trips that repeat the mapper · DTO deserialize-and-check-every-field with no
behavior · a `UiState` constructed in the test and asserted back · Hilt module wiring.

## Gate — 5 questions before writing ANY test (all must be yes)

1. Does it execute production code (not just construct or read a value)?
2. Is the oracle independent of the code under test?
3. Could it fail from a behavior change that still compiles?
4. Does it assert observable behavior (result/effect/emitted state), not structure?
5. Does the seam exist in production by design (interface + constructor injection)?

## The ARC stack

| Need | Use |
|---|---|
| Local unit tests (domain, data, ViewModels) | **JUnit 5** (mannodermaus Android JUnit 5 plugin) |
| Coroutines | `kotlinx-coroutines-test` — `runTest`, `StandardTestDispatcher` injected through the production dispatcher seam |
| Flows / `StateFlow<UiState>` | **Turbine** — assert the emitted sequence, not internal fields |
| Test doubles | **Fakes over mocks.** Hand-written fakes of repository/data-source interfaces in `core:testing`. MockK only when no seam can be designed — ask first (not on the allowlist) |
| Persistence | Room in-memory database through the real DAO (Room integration on-device per `testing-setup` when SQLite behavior matters) |
| UI behavior | Robolectric + Compose test (`createComposeRule`) in `test/`; semantic matchers first, `testTag` only when matchers get unwieldy. These run on JUnit 4 rules — keep them on the JUnit 4 runner as `testing-setup` describes |
| Visual oracle | **Compose Preview Screenshot Testing** — the same centralized-sample-data `@Preview`s, validated with `validate<Variant>ScreenshotTest`; reference images updated only on an intended visual change, and the diff reviewed |
| Coverage | JaCoCo (AGP built-in) |

Never hit a real backend (Firebase included) from a test: Firebase sits behind data-source
interfaces; tests use fakes. Use an emulator suite only for an explicitly integration-tagged test.

## Shared golden vectors with iOS

Pure-logic behavior shared with FavRes-iOS (duplicate detection, statistics, normalization) is
pinned by **golden JSON test vectors** from the shared contract: the same fixture files run in
Swift Testing (iOS) and JUnit 5 (Android). The vector is the oracle — never regenerate a vector
from the Kotlin implementation's output, and never edit one to make a test pass; a mismatch is
a parity bug to raise with the iOS side.

## Rules of engagement

- Tests derive from acceptance criteria and are written first (red). A test that passes before
  the feature exists is wrong — fix it so it fails for the right reason.
- **Never modify a test to make it pass** — fix the implementation. A change that breaks a
  previously green test is invalid.
- Nothing is "green" until `./gradlew build` is clean with zero warnings (Kotlin, Lint, detekt,
  ktlint) — see `android-verify`.
- Auditing an existing suite: apply the Gate to each test; failures get deleted (ask before
  deleting committed tests); fixtures are kept.

## ARC house rules on top

Given-When-Then comments in every test; a `makeSut()` factory instead of inline construction;
descriptive backtick names (`` `emits Error when repository throws` ``); JUnit 5 `@Tag`s
`unit` / `integration` / `ui` / `critical`; `@ParameterizedTest` for repeated logic. Coverage
floor: apps **80%** (packages 100%). Coverage is a floor, not a goal — a tautological test
that lifts the number still fails the Gate and still gets deleted.
