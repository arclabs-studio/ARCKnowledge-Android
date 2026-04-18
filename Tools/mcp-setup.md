# 🔌 MCP Setup

**Model Context Protocol (MCP) servers extend AI agents with real-time access to documentation, project management, and version control. This guide configures the MCP servers recommended for ARC Labs Android projects.**

---

## 🎯 Why MCP?

AI agents work best when they have access to **current, authoritative information** rather than relying solely on training data. MCP servers provide:

- **Real-time documentation** -- Official library docs that stay current across versions
- **Project management integration** -- Linear issues, status updates, and planning directly from the agent
- **Version control context** -- GitHub branches, PRs, and code review without leaving the conversation
- **Reduced hallucination** -- Agents cite actual documentation instead of guessing API signatures

---

## 📋 Recommended MCP Servers

### For Android Development

| Server | Purpose | Priority |
|--------|---------|----------|
| **Context7** | Library documentation (Compose, Hilt, Room, Retrofit, Coroutines) | Required |
| **Linear** | Issue tracking, sprint management, project planning | Required |
| **GitHub** | Repository management, PR creation, branch operations | Required |
| **Android Source Explorer** | AOSP + Jetpack source code exploration (API level 36) | Recommended |
| **Android Docs MCP** | developer.android.com real-time access | Recommended |
| **Mobile MCP** | ADB-based emulator automation and UI testing | Recommended |
| **Android Skills (Google)** | 6 official Google skills on-demand (AGP 9, XML→Compose, Nav3, R8, Play Billing, edge-to-edge) | Recommended |
| **Play Store MCP** | Google Play Console releases and track management | Optional |

---

## 🚀 Context7 Setup

[Context7](https://context7.com) provides real-time library documentation for AI agents. It resolves library names to IDs and fetches up-to-date docs, code examples, and API references.

### Installation

Add to your Claude Code MCP configuration (`.claude/mcp.json` or global `~/.claude/mcp.json`):

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"]
    }
  }
}
```

### Available Tools

| Tool | Description |
|------|-------------|
| `resolve-library-id` | Resolve a library name to its Context7 library ID |
| `get-library-docs` | Fetch documentation for a specific library by ID |

### Verification

After configuration, verify the server is working:

```
> Use Context7 to look up Jetpack Compose documentation
```

The agent should resolve the library ID and return current Compose API documentation.

---

## 🚀 Linear MCP Setup

Linear integration enables AI agents to create, update, and query issues directly.

### Installation

```json
{
  "mcpServers": {
    "linear": {
      "command": "npx",
      "args": ["-y", "@anthropic/linear-mcp@latest"],
      "env": {
        "LINEAR_API_KEY": "<your-linear-api-key>"
      }
    }
  }
}
```

### Available Operations

| Operation | Description |
|-----------|-------------|
| `list_issues` | Query issues by team, status, or label |
| `get_issue` | Get full issue details by ID |
| `create_issue` | Create a new issue with title, description, labels |
| `update_issue` | Update issue status, assignee, or description |
| `list_teams` | List available teams |
| `list_labels` | List available labels |

### Usage Example

```
> Create a Linear issue for implementing restaurant search
> Move ARC-123 to "In Progress"
> What are the open issues in the Android team?
```

---

## 🚀 GitHub MCP Setup

GitHub integration enables branch management, PR creation, and repository operations.

### Installation

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@anthropic/github-mcp@latest"],
      "env": {
        "GITHUB_TOKEN": "<your-github-token>"
      }
    }
  }
}
```

### Available Operations

| Operation | Description |
|-----------|-------------|
| `list_branches` | List repository branches |
| `create_branch` | Create a new branch |
| `list_prs` | List pull requests |
| `create_pr` | Create a pull request with title and body |
| `get_pr` | Get PR details including status and reviews |

---

## 📐 Project-Level CLAUDE.md Configuration

Every ARC Labs Android project should include MCP availability in its local `CLAUDE.md`. Use this template section:

```markdown
## MCP Servers Available

The following MCP servers are configured and available for this project:

| Server | Status | Usage |
|--------|--------|-------|
| Context7 | Active | Use for library documentation lookups (Compose, Hilt, Room, etc.) |
| Linear | Active | Use for issue management (team: Android, project: <project-name>) |
| GitHub | Active | Use for branch/PR operations (repo: ARC-Labs-Studio/<repo-name>) |

### Context7 Usage

When implementing features or debugging issues, use Context7 to look up:
- Official API documentation for Jetpack libraries
- Code examples from library maintainers
- Migration guides for version upgrades
- Configuration options and best practices

Example: Before implementing a Room migration, query Context7 for the latest
Room migration documentation to ensure you're using current APIs.
```

---

## 🚀 Android Source Explorer Setup

Explores AOSP and Jetpack source code on-demand using Tree-sitter + LSP. Supports API level 36 (Android 16) and AndroidX Compose/Lifecycle/Activity.

### Installation

```bash
# Install via uv
pip install uv
uv tool install android-source-explorer-mcp

# Sync Android sources (one-time, ~2GB)
uv run android-source-explorer sync --api-level 36 --androidx "compose,lifecycle,activity,navigation"
```

### MCP Configuration

```json
{
  "mcpServers": {
    "android-source-explorer": {
      "command": "uv",
      "args": ["run", "android-source-explorer", "serve"]
    }
  }
}
```

### When to Use

Use when debugging unexpected framework behavior, understanding internal Compose mechanics, or verifying how AndroidX APIs are implemented.

---

## 🚀 Android Docs MCP Setup

Provides real-time access to developer.android.com — search docs, look up API references, browse packages, and check release notes.

### Installation

```bash
# Requires Bun
brew install bun
git clone https://github.com/ankit-verma-209171/android-docs-mcp
cd android-docs-mcp && bun install
```

### MCP Configuration

```json
{
  "mcpServers": {
    "android-docs": {
      "command": "bun",
      "args": ["run", "/path/to/android-docs-mcp/src/index.ts"]
    }
  }
}
```

### When to Use

Use when Context7 doesn't have the specific library, or when you need the official API reference for a specific Android version.

---

## 🚀 Mobile MCP Setup

Cross-platform ADB + simctl automation for Android and iOS. Enables UI testing, screenshot capture, location simulation, and push notification testing from Claude.

### Installation

```bash
npx -y @mobilenext/mobile-mcp@latest
```

### MCP Configuration

```json
{
  "mcpServers": {
    "mobile-mcp": {
      "command": "npx",
      "args": ["-y", "@mobilenext/mobile-mcp@latest"]
    }
  }
}
```

### Prerequisites

- Android: ADB installed and device/emulator connected
- iOS: Xcode + Simulator (cross-platform support)

### When to Use

Use for visual UI verification, automated testing flows, screenshot capture to validate layout changes, and testing push notifications.

---

## 🚀 Android Skills MCP Setup (Recommended)

Exposes Google's 6 official Android skills to Claude Code agents on-demand, without copying files. Uses BM25 search and loads in <200ms.

### Installation

```bash
claude mcp add android-skills -- npx -y android-skills-mcp
```

### MCP Configuration

```json
{
  "mcpServers": {
    "android-skills": {
      "command": "npx",
      "args": ["-y", "android-skills-mcp"]
    }
  }
}
```

### Available Skills

| Skill | Use When |
|-------|---------|
| `agp-9-upgrade` | Migrating to Android Gradle Plugin 9 |
| `migrate-xml-views-to-jetpack-compose` | Converting XML layouts to Compose |
| `navigation-3` | Setting up or migrating to Navigation 3 |
| `r8-analyzer` | Diagnosing R8/ProGuard shrinking issues |
| `play-billing-library-version-upgrade` | Upgrading Play Billing Library |
| `edge-to-edge` | Implementing edge-to-edge display support |

### When to Use

- AGP upgrade triggered a build failure → skill covers all breaking changes
- Migrating XML layouts to Compose → skill provides step-by-step patterns
- R8 stripping needed classes in release build → skill diagnoses ProGuard rules
- Updating Play Billing to latest → skill covers API migration

> **Source:** `github.com/android/skills` (Apache-2.0) — maintained by Google. See [Tools/android-cli.md](android-cli.md) for the full Android CLI binary.

---

## 🚀 Play Store MCP Setup (Optional)

Manages Google Play Console releases: deploy versions, promote between tracks (internal → alpha → beta → production), view release status.

### Installation

```bash
# Build from source (Kotlin JAR)
git clone https://github.com/antoniolg/play-store-mcp
cd play-store-mcp && ./gradlew shadowJar
```

### MCP Configuration

```json
{
  "mcpServers": {
    "play-store": {
      "command": "java",
      "args": ["-jar", "/path/to/play-store-mcp-all.jar"],
      "env": {
        "PLAY_STORE_SERVICE_ACCOUNT_KEY_PATH": "$PLAY_STORE_KEY_PATH"
      }
    }
  }
}
```

### Prerequisites

- Google Play Console service account with Editor permissions
- Service account JSON key file
- App already published (can't create new apps via API)

### When to Use

Use for promoting builds between tracks without opening Play Console, checking release status, and scripting deployment workflows.

---

## 🔧 Full MCP Configuration Template

Copy this complete configuration for a new ARC Labs Android project:

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"]
    },
    "linear": {
      "command": "npx",
      "args": ["-y", "@anthropic/linear-mcp@latest"],
      "env": {
        "LINEAR_API_KEY": "$LINEAR_API_KEY"
      }
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@anthropic/github-mcp@latest"],
      "env": {
        "GITHUB_TOKEN": "$GITHUB_TOKEN"
      }
    },
    "android-source-explorer": {
      "command": "uv",
      "args": ["run", "android-source-explorer", "serve"]
    },
    "mobile-mcp": {
      "command": "npx",
      "args": ["-y", "@mobilenext/mobile-mcp@latest"]
    },
    "android-skills": {
      "command": "npx",
      "args": ["-y", "android-skills-mcp"]
    }
  }
}
```

### Environment Variables

Store secrets in your shell profile, not in committed files:

```bash
# ~/.zshrc or ~/.bashrc
export LINEAR_API_KEY="lin_api_..."
export GITHUB_TOKEN="ghp_..."
```

Then reference them in the MCP config:

```json
{
  "env": {
    "LINEAR_API_KEY": "$LINEAR_API_KEY"
  }
}
```

---

## ✅ MCP Setup Checklist

### Required
- [ ] Context7 MCP installed and resolving library IDs
- [ ] Linear MCP installed with valid API key
- [ ] GitHub MCP installed with valid token
- [ ] Project CLAUDE.md documents MCP availability
- [ ] API keys stored in environment variables (not committed)

### Recommended
- [ ] Android Source Explorer installed and synced (API 36 + AndroidX)
- [ ] Mobile MCP installed and device/emulator accessible via ADB
- [ ] Android Docs MCP installed (if Context7 coverage is insufficient)
- [ ] Android Skills MCP configured (`claude mcp add android-skills -- npx -y android-skills-mcp`)

### Verification
- [ ] Agent verified: can fetch Compose docs via Context7
- [ ] Agent verified: can list Linear issues
- [ ] Agent verified: can list GitHub branches
- [ ] Agent verified: can take emulator screenshot via Mobile MCP

---

## ❌ Common Mistakes

### 1. Committing API Keys

```json
// ❌ API key hardcoded in committed config
{
  "env": {
    "LINEAR_API_KEY": "lin_api_abc123def456"
  }
}
```

```json
// ✅ Reference environment variable
{
  "env": {
    "LINEAR_API_KEY": "$LINEAR_API_KEY"
  }
}
```

### 2. Not Documenting MCP Availability

If the project CLAUDE.md doesn't mention MCP servers, agents won't know to use them. Always include the MCP section.

### 3. Using Stale Training Data Instead of Context7

```
// ❌ Agent guesses API based on training data
"I believe the Room migration API works like this..."

// ✅ Agent uses Context7 for current documentation
"Let me check the Room documentation via Context7..."
[resolves library ID, fetches current docs]
"According to the Room 2.6.1 documentation..."
```

---

## 📚 Further Reading

- [Context7 Documentation](https://context7.com)
- [Model Context Protocol Specification](https://spec.modelcontextprotocol.io)
- [Claude Code MCP Configuration](https://docs.anthropic.com/en/docs/claude-code/mcp)
- [Context7 Usage Guide](./context7-usage.md)
- [ARCDevTools-Android](./arcdevtools-android.md)
- [Skills Index](../Skills/skills-index.md)

---

**Remember**: An AI agent with real-time documentation access makes fewer mistakes and produces more accurate code. Configure MCP once, benefit on every task.
