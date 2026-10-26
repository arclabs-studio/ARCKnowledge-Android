---
name: android-verify
description: >
  ARC's Android verification loop — the external verifier for Kotlin/Compose work: zero-warning
  Gradle build, unit tests, Compose preview screenshot validation, then install and walk the app
  on a headless emulator with the android CLI. Use after implementing anything on Android, and
  when the user says "verify the Android build", "does it work on Android", "run the Android
  tests", "check the emulator", "screenshot the app", "gradlew build", "testDebugUnitTest",
  "validateDebugScreenshotTest", "headless emulator", "install the APK", "android run",
  "android layout", "android screen capture". The procedure arc-verify-android runs; the main
  agent iterates until it is green.
---

# Android verification loop (ARC Labs)

Your own reading of the diff is never the exit condition — a green step below is. Run from the
project root (where `settings.gradle.kts` lives). Flags below were verified against
`android --help` for CLI 1.0.16625370 on 2026-10-21; if `android --version` differs, re-check
with `android <cmd> --help` before relying on them (see `arc-cli-android`).

## Preconditions

- `JAVA_HOME` (Android Studio JBR) and `ANDROID_HOME` set; `./gradlew` present. If not, stop and
  report — do not install a JDK/SDK on your own.
- Flavored projects prefix the variant: `testDevDebugUnitTest`, `validateDevDebugScreenshotTest`.
  List real task names with `./gradlew :app:tasks --all` rather than guessing.

## The loop

1. **Build, zero warnings.**
   `./gradlew build` — assemble + check. The gates are configured in the build
   (`allWarningsAsErrors`, Lint `warningsAsErrors`, detekt `maxIssues: 0`), so warnings fail it.
   Still scan the output: any line starting `w: ` (Kotlin) or `Warning:` means a gate is
   misconfigured — report it as a failure, do not call the build clean. Run the project's
   detekt and ktlint tasks/commands if `build` does not include them.
2. **Unit tests.** `./gradlew testDebugUnitTest` (or the module-scoped
   `./gradlew :feature:x:testDebugUnitTest` while iterating, the full task before reporting).
   Failures: read `build/reports/tests/` and `build/test-results/`.
3. **Screenshot validation.** `./gradlew validateDebugScreenshotTest` — the visual oracle.
   Report path `build/reports/screenshotTest/debug/` (standalone plugin). On AGP test suites
   the task is `test{Suite}{Target}{Variant}TestSuite` — see `android-skills:testing-setup`.
   **Never run `updateDebugScreenshotTest` to make validation pass**; updating references is a
   deliberate, user-approved act for an intended visual change.
4. **Emulator.** `android emulator list` → `android emulator start --headless <avd>`
   (returns when booted; `--cold` to skip the snapshot). No AVD? `android emulator create
   --list-profiles`, then ask before `android emulator create <profile>`.
5. **Install and launch.** Locate the APK: `./gradlew assembleDebug`, then
   `android describe --project_dir=.` prints the JSON files that map build targets to output
   artifacts. Then `android run --apks=<path/to/app-debug.apk>` (add `--activity=<name>` when
   the launcher activity is ambiguous). `android install --apks=…` installs without launching.
   With more than one device attached, pass `--device=<serial>` (serials from
   `$ANDROID_HOME/platform-tools/adb devices`; `android info` does not list devices).
6. **Inspect.** `android layout --pretty` (JSON tree: `text`, `content-desc`, `interactions`,
   `bounds`, `center`; `--full` for non-interactive nodes, `-o <file>` to save) — the primary
   oracle; it also shows missing `content-desc` for TalkBack. Then
   `android screen capture -o <scratchpad>/screen.png` and **look at the PNG** before going on.
   Drive flows with `android input tap "<selector>"`, `android input text "<text>" "<selector>"`,
   `android input scroll "<selector>" --dir=down --until="<selector>"`, `android input key back`
   (selectors: `text=`, `id=`, `class=`, `desc=` plus booleans like `clickable=true`).
   Re-run `layout` after every action. Elements `layout` cannot find:
   `android screen capture --annotate -o <file>` + `android screen resolve --screenshot <file>
   --string "#N"`.
7. **Clean up.** `android emulator stop` (pass `<avd-or-serial>` when several run). Screenshots
   and layout dumps go to the scratchpad, never the repo.

Re-run from step 1 after every fix. Never report on stale output.

## Judging the result

- **Fail:** build error or any warning, failing test, screenshot diff, crash/ANR on launch,
  element missing from `layout`, interactive element without a label, clipped/overlapping text,
  content under system bars (edge-to-edge broken).
- **Not a failure:** transient loading states, intended empty states, disabled controls on an
  incomplete form.
- **Cannot verify here — say so:** physical-device-only behavior, Play Billing purchases,
  real-account sign-in without provided credentials, Play Console state. Return
  `⚠️ NEEDS DEVICE VERIFICATION` naming exactly what to check by hand. A build success is
  never evidence that something works.
