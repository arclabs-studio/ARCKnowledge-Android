# Android CLI

**Google's official command-line interface for agent-first Android development.** Standardizes core development workflows for AI agents and humans: environment setup, project scaffolding, virtual device management, skill distribution, and UI automation.

> **Status:** Preview (April 2026). Not yet required — `./gradlew` remains the build system. Install opt-in.

---

## Why Use Android CLI

| Metric | Value |
|--------|-------|
| Token reduction (LLM context) | **>70%** vs standard toolsets |
| Task completion speed | **3x faster** (Google internal benchmarks) |
| Skill access | On-demand, without copying files |

AI agents (Claude Code, Gemini, Codex) use `android` commands as atomic tools — each command produces machine-readable JSON output, avoiding the need to parse Gradle output or Android Studio logs.

---

## Installation

```bash
# Download from Google's developer tools page
# https://developer.android.com/tools/agents

# After installation, verify
android --version

# Self-update when new versions drop
android update
```

> **ARC Labs default today:** Use `android-skills-mcp` (MCP server) to access the 6 official skills without installing the CLI binary. See [Tools/mcp-setup.md](mcp-setup.md).

---

## Subcommands Reference

### Project

| Command | Purpose |
|---------|---------|
| `android create [template]` | Scaffold new project (default: `empty-activity-agp-9`) |
| `android create list` | List available templates |
| `android describe` | Analyze project → outputs JSON (build targets, APK paths) |

```bash
# Create project from template
android create

# Describe current project structure as JSON
android describe
```

### SDK Management

| Command | Purpose |
|---------|---------|
| `android sdk install <pkg[@ver]>` | Install SDK package |
| `android sdk list <pattern>` | List packages (supports regex) |
| `android sdk remove <pkg>` | Remove package |
| `android sdk update` | Update all packages |
| `android info` | Show current SDK path |

```bash
# Install specific platform
android sdk install "platforms;android-36"

# List installed build tools
android sdk list "build-tools"
```

### Virtual Devices

| Command | Purpose |
|---------|---------|
| `android emulator create` | Create AVD (default profile: `medium_phone`) |
| `android emulator list` | List virtual devices |
| `android emulator start <name>` | Launch emulator |
| `android emulator stop <serial>` | Stop emulator |

> **Note:** `android emulator` is currently disabled on Windows.

```bash
# Create default phone emulator
android emulator create

# Start by name
android emulator start Pixel_9_API_36
```

### Skills

| Command | Purpose |
|---------|---------|
| `android skills add [--all]` | Install skills for detected agent |
| `android skills find <query>` | Search skills by description |
| `android skills list [--long]` | List available skills |
| `android skills remove --skill=<name>` | Remove skill |

```bash
# Install all official skills
android skills add --all

# Find skills for your task
android skills find "compose migration"
android skills find "r8 obfuscation"
```

### Documentation

| Command | Purpose |
|---------|---------|
| `android docs search <query>` | Search Android Knowledge Base |
| `android docs fetch <kb://url>` | Fetch doc content |

```bash
# Search official docs
android docs search "WorkManager constraints"

# Fetch specific page
android docs fetch kb://androidx/work/constraints
```

### Build and Deploy

| Command | Purpose |
|---------|---------|
| `android run --apks=<paths>` | Deploy pre-built APKs to device/emulator |

```bash
# Deploy debug APK
android run --apks=app/build/outputs/apk/debug/app-debug.apk
```

### UI Automation

| Command | Purpose |
|---------|---------|
| `android layout [--pretty] [--diff]` | Get UI layout tree as JSON |
| `android screen capture [--annotate]` | Screenshot with optional labeled bounding boxes |
| `android screen resolve` | Translate annotated labels to (x, y) coordinates |

```bash
# Capture annotated screenshot for agent navigation
android screen capture --annotate

# Get layout hierarchy
android layout --pretty
```

### Other

| Command | Purpose |
|---------|---------|
| `android init` | Install `android-cli` skill for agent support |
| `android update` | Self-update the CLI |

**Global flags:** `--sdk=<path>`, `-h/--help`, `-V/--version`

---

## The 6 Official Skills

Available at `github.com/android/skills` (Apache-2.0). These are what `android skills add --all` installs:

| Skill | Use When |
|-------|---------|
| `agp-9-upgrade` | Migrating to Android Gradle Plugin 9 |
| `migrate-xml-views-to-jetpack-compose` | Converting XML layouts to Compose |
| `navigation-3` | Setting up or migrating to Navigation 3 |
| `r8-analyzer` | Diagnosing R8/ProGuard shrinking and obfuscation issues |
| `play-billing-library-version-upgrade` | Upgrading Play Billing Library version |
| `edge-to-edge` | Implementing full edge-to-edge display support |

**ARC Labs usage:** Access these via `android-skills-mcp` (Claude Code MCP) rather than installing files locally. See [Tools/mcp-setup.md](mcp-setup.md) → Android Skills MCP section.

---

## Coexistence with `./gradlew`

Android CLI **does not replace Gradle**. It complements it:

| Task | Use |
|------|-----|
| Build APK/AAB | `./gradlew assembleDebug` / `bundleRelease` |
| Run tests | `./gradlew test` |
| Lint/detekt | `./gradlew ktlintCheck detekt` |
| SDK management | `android sdk install` |
| Create AVD | `android emulator create` |
| Deploy APK to emulator | `android run --apks=...` |
| UI layout inspection | `android layout` |
| Access official skills | `android skills find` / MCP |
| Search Android docs | `android docs search` |

---

## Agent Integration

When an AI agent has `android` available as a tool, it can:

1. Call `android describe` to understand project structure without parsing Gradle
2. Call `android skills find "compose"` to locate the right skill for the task
3. Call `android screen capture --annotate` to navigate the UI programmatically
4. Call `android docs search` for up-to-date API guidance

Skills trigger **automatically** when a prompt matches the skill's description metadata — agents load them on-demand to keep context windows lean.

---

## References

- [Android CLI docs](https://developer.android.com/tools/agents/android-cli)
- [Android Skills docs](https://developer.android.com/tools/agents/android-skills)
- [android/skills repo](https://github.com/android/skills) (Apache-2.0)
- [skydoves/android-skills-mcp](https://github.com/skydoves/android-skills-mcp) — MCP wrapper for Claude Code
- [Agent Skills open standard](https://agentskills.io)
