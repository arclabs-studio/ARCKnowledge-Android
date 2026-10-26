---
name: arc-cli-android
description: Use before ANY Android tooling task and before running the `android` CLI. Sets ARC's policy — `./gradlew` + the official `android` CLI are the sanctioned Android tools (the equivalent of the Xcode MCP on iOS); Android Studio GUI is optional; mobile-mcp only when the CLI cannot do it. Triggers on "android CLI", "android command", "build the Android app", "run on the emulator", "start the emulator", "headless emulator", "install the APK", "android run", "android install", "android layout", "android screen capture", "tap on the emulator", "android input", "android sdk install", "android docs", "android skills", "android studio render-compose-preview", "version lookup", "create Android project", "android create", "android describe", "adb". Covers the default policy, when to use, key commands with real flags, workflow recipes, auth, gotchas, and when NOT to use.
license: MIT
metadata:
  author: ARC Labs Studio
  version: "1.0"
  cli: android
  cli-version-verified: "1.0.16625370"
  flags-verified: "2026-10-21"
---

# arc-cli-android

## What this CLI does

Google's `android` CLI (`brew install --cask android-cli`, binary `/opt/homebrew/bin/android`)
is the agent-oriented front end to the Android SDK: create projects from templates, manage SDK
packages and AVDs, install and launch APKs, dump the UI tree, capture (annotated) screenshots,
drive input by accessibility selector, search the official Android Knowledge Base, manage
Android agent skills, and talk to a running Android Studio (render Compose previews, look up
artifact versions). It does **not** build — Gradle does. Google's own reference skill is
`android-cli` (installed by `android init`); this skill adds ARC's policy on top.

## Default policy (read first)

- **Build, test, lint:** `./gradlew` — always the project's wrapper, never a global `gradle`.
- **Everything device/SDK/docs:** the `android` CLI.
- **Android Studio GUI:** optional. Needed only for `android studio …` subcommands.
- **`mobile-mcp`:** only when the CLI genuinely cannot do the job — ask the user first.
- **Raw `adb`/`emulator`/`sdkmanager`/`avdmanager`:** prefer the `android` equivalent. `adb` is
  acceptable for what the CLI does not cover (listing device serials, logcat). `adb` is not on
  `PATH` by default: `$ANDROID_HOME/platform-tools/adb`.
- **Never invent a flag.** Verify with `android <cmd> [<sub>] --help` before using anything not
  listed below, and re-verify when `android --version` changes.

## When to use

- Start/stop/list/create emulators, headless for agent loops
- Install or launch a built APK; inspect the running UI as JSON; capture screenshots
- Tap/type/swipe/scroll by accessibility selector instead of pixel guessing
- Install SDK packages (`platforms;…`, `cmdline-tools;latest`)
- Look up current Android docs (`android docs`) before writing against a young API
- Render a Compose preview or look up latest artifact versions through a running Studio

## When NOT to use

- Compiling, unit tests, Lint, screenshot tests → `./gradlew` tasks (`android-verify`).
- Firebase project/app setup → `arc-mcp-firebase`. Play Console → not covered; ask the user.
- The rules for the code itself → `android-doctrine` / `android-testing-doctrine`.

## When the CLI is missing or fails

`command -v android` empty → `brew install --cask android-cli` is in the ARCAgentStack
`Brewfile`; ask the user before installing. SDK path wrong → `android info sdk`; override per
call with `--sdk=<path>`. Never silently swap in a different tool to "make it work" — report
the blocker.

## Authentication

None for local work. `android auth login` (tokens in the OS keyring unless
`--no-use-keyring`) / `auth status` / `auth logout` exist for Google services — needed for
`android device remote …` (remote physical devices, takes `--project=<gcp-project>`). Do not log
in on the user's behalf.

## Key commands (verified against `--help`, CLI 1.0.16625370)

Global options: `--sdk=<path>`, `-v/--verbose`, `-V/--version`, `-h/--help`.

| Command | Key params | Purpose |
|---|---|---|
| `android info [<field>]` | e.g. `sdk` | SDK path and CLI version (does **not** list devices) |
| `android init` | — | One-time env setup; installs the `android-cli` skill for detected agents |
| `android create [<template>]` | `--name`, `--output/-o`, `--application-id`, `--namespace`, `--min-sdk`, `--list`, `--force` | New project from template (`empty-activity` default) |
| `android describe` | `--project_dir=<dir>` | Prints JSON files mapping build targets → output artifacts (APK paths) |
| `android sdk install <pkg>[@<ver>]…` | `--beta`, `--canary`, `--force`, `--no-downgrade` | Install SDK packages |
| `android sdk list [<pattern>]` / `update [<pkg>]` / `remove <pkg>` | `--all`, `--all-versions` | Inspect/maintain the SDK |
| `android emulator list` | `--long` | Available AVDs |
| `android emulator create [<profile>]` | `--list-profiles` | New AVD from a device profile |
| `android emulator start <avd>` | `--headless`, `--cold` | Boots; returns when ready |
| `android emulator stop [<avd-or-serial>]` / `remove` | — | Stop / delete |
| `android install` | `--apks=<a.apk,b.apk>`, `--device`, `--install-options`, `--use-delta-install` | Install without launching |
| `android run` | `--apks`, `--activity`, `--device`, `--debug`, `--type` (`ACTIVITY`, `TILE`, `WATCH_FACE`, …), `--install-options` | Deploy and launch |
| `android layout` | `--pretty/-p`, `--full`, `--flat`, `--no-idle`, `--output/-o`, `--device` | UI tree as JSON (primary oracle) |
| `android screen capture` | `--output/-o`, `--annotate/-a`, `--device` (+ annotate tuning flags) | PNG screenshot |
| `android screen resolve` | `--screenshot=<annotated.png>` `--string="tap #3"` | Replace `#N` labels with coordinates |
| `android input tap <selector>` | `--match=single\|first`, `--wait=<ms>`, `--device` | Tap by selector |
| `android input text <text> <selector>` | `--submit`, `--match`, `--wait` | Type (replaces existing text) |
| `android input swipe <selector>` | `--dir`, `--distance`, `--speed`, `--match`, `--wait` | Swipe from element |
| `android input scroll <selector>` | `--dir`, `--amount`, `--until=<selector>`, `--speed` | Scroll a container |
| `android input key <key>` | `--alt`, `--ctrl`, `--shift` | `back`, `home`, `recents`, `enter`, `tab`, `esc`, … or a keycode |
| `android docs search "<query>"` / `docs fetch kb://…` | — | Official Android Knowledge Base |
| `android skills list` / `find <kw>` / `add <skill>` / `update` / `remove` | `--agent`, `--all`, `--project`, `--long` | Google Android agent skills |
| `android studio check` | — | Running Studio instances (needs Studio **Quail 1+**) |
| `android studio render-compose-preview <file> <composable>` | `--output-image-file`, `--print-semantics`, `--project`, `--pid` | Render a preview via Studio |
| `android studio version-lookup <artifact…>` | `--project`, `--pid` | Latest stable/preview versions |
| `android studio analyze-file` / `find-declaration` / `find-usages` / `open-file` | `--project`, `--pid` | Studio code intelligence |
| `android update` | `--url` | Self-update the CLI |

Selectors: comma-separated `key=value` — `text`/`txt`, `id`/`res`, `class`/`clz`,
`desc`/`content`, plus booleans `clickable`, `scrollable`, `checkable`, `focusable`,
`longclickable`, `checked`, `focused`, `selected`. String fields match as case-insensitive
substrings.

## Workflow recipes

**Emulator verification loop** — the full procedure is `android-verify`; in short:
`./gradlew assembleDebug` → `android emulator start --headless Medium_Phone_API_36.1` →
`android run --apks=app/build/outputs/apk/debug/app-debug.apk` (confirm the path with
`android describe`) → `android layout --pretty` → `android screen capture -o <scratchpad>/s.png`
→ `android input tap "text=Save,clickable=true"` → `layout` again → `android emulator stop`.

**Docs before code:** `android docs search "navigation 3 bottom sheet scene"` →
`android docs fetch kb://…` from the result. Pair with the `android-skills:*` plugin skills.

**Version check before bumping the catalog:** with Studio open,
`android studio version-lookup androidx.navigation3:navigation3-runtime`; without Studio, use
Context7 / Maven metadata and say which source you used.

## Gotchas

- **`android init` writes a real directory, not a symlink:** `~/.claude/skills/android-cli`.
  `reconcile-symlinks.sh` only removes unexpected *symlinks*, so it survives reconciles; do not
  move it into this repo or delete it. Refresh it with `android skills update android-cli`.
- **`android run`/`install` do not build.** They need `--apks`; stale APK = stale verification.
  Rebuild after every change.
- **Device serials come from `adb devices`**, not `android info`. Pass `--device` whenever more
  than one device/emulator is attached.
- **`layout` first, screenshots second.** Always open the PNG and look at it before acting on it.
  Missing elements after an action: wait a moment and re-run `layout`.
- **`input text` replaces** the field's text; check the target has `FOCUSED` state.
- **`android studio …` needs a running Android Studio Quail 1 or newer** — otherwise it reports
  "No running Studio instances found". It is optional; never block the loop on it.
- **`android create --list` downloads the template bundle** on first use.
- **`--no-metrics`** is accepted by 1.0.16625370 but is not shown in `--help`; do not rely on
  undocumented flags in scripts.
- `layout --diff/-d` is deprecated and a no-op.

## See also

- `android-verify` / `arc-verify-android` — the verification loop and its agent
- `android-doctrine`, `android-testing-doctrine` — ARC's binding rules for the code
- `android-cli` — Google's reference skill (journeys, annotated screenshots, adb input detail)
- `android-skills:*` — Google's official plugin skills (API detail)
