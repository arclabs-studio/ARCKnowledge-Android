# ARC Labs Android Subagents

Subagents are **autonomous executors** — they take a task, use tools, and return a result. This is distinct from Skills, which are **knowledge containers** that load patterns for Claude to apply.

| Concept | Role | How to trigger |
|---------|------|----------------|
| **Skills** | Load reference patterns — Claude applies them | `/arc-android-tdd-patterns`, `/arc-android-architecture`, etc. |
| **Agents** | Execute autonomously — Claude delegates the task | "Implement X with TDD", "Review before commit", "Debug this error" |

---

## The 12 ARC Labs Android Agents

### `arc-kotlin-tdd`
**Implements features using strict TDD** — writes JUnit 5 + MockK test suites before any production code.

| | |
|--|--|
| **Model** | claude-sonnet-4-6 |
| **Read-only** | No |
| **Triggers** | "Implement a feature", "write tests first", "create a UseCase", "create a ViewModel", "add a Repository", "start TDD" |

Skills invoked dynamically: `arc-android-tdd-patterns`, `arc-android-architecture`, `arc-android-presentation-layer`, `arc-android-data-layer`

---

### `arc-kotlin-reviewer`
**Delegated code reviewer** — evaluates code against 9 ARC Labs domains and produces a structured 🔴🟡🔵 report.

| | |
|--|--|
| **Model** | claude-sonnet-4-6 |
| **Read-only** | Yes |
| **Triggers** | "Review this code", "pre-merge review", "check for architecture violations", "audit this file", "review before I commit" |

Skills invoked dynamically: `arc-android-quality-standards`, `arc-android-architecture`, `arc-android-presentation-layer`, `arc-android-data-layer`

> **Distinction**: For guided review where you conduct the review with Claude's help, use the `/arc-android-final-review` skill instead.

---

### `arc-kotlin-debugger`
**Diagnoses and fixes build/test failures** — environment-first diagnostics, then code fixes.

| | |
|--|--|
| **Model** | claude-sonnet-4-6 |
| **Read-only** | No |
| **Triggers** | "Build failed", "BUILD FAILED", "Unresolved reference", "MissingBinding", "Hilt compilation error", coroutine deadlocks, "FAILED" in test output, pasted Gradle error logs |

Skills invoked dynamically: `arc-android-project-setup`, `arc-android-tdd-patterns`, `arc-android-architecture`

---

### `arc-gradle-manager`
**Manages Gradle dependencies and build configuration** — adds/removes/updates dependencies and verifies builds.

| | |
|--|--|
| **Model** | claude-haiku-4-5-20251001 |
| **Read-only** | No |
| **Triggers** | "Add a dependency", "update a library", "add a Gradle module", "resolve dependency conflict", "fix libs.versions.toml", "configure build.gradle.kts" |

Skills invoked dynamically: `arc-android-project-setup`

---

### `arc-project-explorer`
**Read-only codebase navigator** — maps architecture, traces data flows, finds symbols. No skill invocation for speed.

| | |
|--|--|
| **Model** | claude-haiku-4-5-20251001 |
| **Read-only** | Yes |
| **Triggers** | "Where is X implemented?", "find all ViewModels", "map the architecture", "trace the data flow for", "what calls this function?", "show me the dependency graph" |

No skills invoked — lightweight by design for speed.

---

### `arc-linear-bridge`
**Linear-to-Kotlin scaffolding** — reads tickets via MCP, creates branch, test skeleton, and feature memory file.

| | |
|--|--|
| **Model** | claude-haiku-4-5-20251001 |
| **Read-only** | No |
| **Triggers** | "Start working on ARC-[N]", "scaffold ticket ARC-[N]", "set up tests for [issue ID]" |

Skills invoked dynamically: `arc-android-tdd-patterns`, `arc-android-memory`

---

### `arc-pr-publisher`
**Publishes a feature branch as a PR** — validates Android-specific checklist, creates PR on GitHub, links Linear ticket, updates issue to "In Review".

| | |
|--|--|
| **Model** | claude-sonnet-4-6 |
| **Read-only** | No (creates PR via MCP) |
| **Triggers** | "Create a PR", "open a pull request", "publish my branch", "I'm ready to merge", "submit for review" |

Skills invoked dynamically: `arc-android-quality-standards`, `arc-android-tdd-patterns`

---

### `arc-release-orchestrator`
**Orchestrates the full release cycle** — bumps `versionName`/`versionCode` in build.gradle.kts, updates CHANGELOG, creates release branch and PR.

| | |
|--|--|
| **Model** | claude-sonnet-4-6 |
| **Read-only** | No |
| **Triggers** | "Prepare a release", "bump version to X.Y.Z", "create release branch", "ship vX.Y.Z" |

Skills invoked dynamically: `arc-android-workflow`

---

### `arc-play-distribution`
**Orchestrates beta distribution** — builds AAB, generates release notes, triggers Firebase App Distribution. Distinct from `arc-release-orchestrator` which handles code/PR.

| | |
|--|--|
| **Model** | claude-haiku-4-5-20251001 |
| **Read-only** | No (generates release notes, triggers GitHub Actions) |
| **Triggers** | "Send to Firebase App Distribution", "create a beta build", "distribute beta", "internal testing build" |

Skills invoked dynamically: `arc-android-github-actions-ci`

---

### `arc-play-store-listing`
**Executes Play Store Optimization** — researches competitors, generates ready-to-upload metadata files (title, description, keywords, release notes) in `aso/[app-id]/`.

| | |
|--|--|
| **Model** | claude-sonnet-4-6 |
| **Read-only** | No (writes `aso/[app-id]/` output files) |
| **Triggers** | "Optimize my Play Store listing", "ASO audit", "improve my keywords", "update screenshots strategy", "write release notes", "prepare Play Store metadata" |

Skills invoked dynamically: `app-marketing-context`, `keyword-research`, `metadata-optimization`, `screenshot-optimization`

---

### `arc-room-migration`
**Manages Room schema migrations** — the most conservative agent. Classifies changes as @AutoMigration vs manual, always confirms before any breaking change, writes migration tests before migration code.

| | |
|--|--|
| **Model** | claude-sonnet-4-6 |
| **Read-only** | No |
| **Triggers** | "Add a Room column", "rename a database field", "add a relationship", "Room migration crash", "schema version", "@AutoMigration" |

Skills invoked dynamically: `arc-android-data-layer`, `arc-android-tdd-patterns`

---

### `arc-dependency-auditor`
**Audits Gradle dependencies** — read-only analysis of `libs.versions.toml` across the project. Detects outdated versions, version inconsistencies across modules. Produces a prioritized report; apply updates with `arc-gradle-manager`.

| | |
|--|--|
| **Model** | claude-haiku-4-5-20251001 |
| **Read-only** | Yes |
| **Triggers** | "Audit my dependencies", "check for outdated packages", "are my dependencies up to date?", "check version consistency", "dependency health check" |

Skills invoked dynamically: `arc-android-project-setup`

---

## Master Table — Skills and MCPs per Agent

| Agent | ARC Labs Skills | MCPs |
|-------|----------------|------|
| `arc-kotlin-tdd` | arc-android-tdd-patterns, arc-android-architecture, arc-android-presentation-layer, arc-android-data-layer | linear_get_issue |
| `arc-kotlin-reviewer` | arc-android-quality-standards, arc-android-architecture, arc-android-presentation-layer, arc-android-data-layer | — |
| `arc-kotlin-debugger` | arc-android-project-setup, arc-android-tdd-patterns, arc-android-architecture | — |
| `arc-gradle-manager` | arc-android-project-setup | — |
| `arc-project-explorer` | — (inlined) | — |
| `arc-linear-bridge` | arc-android-tdd-patterns, arc-android-memory | linear_get_issue, linear_list_issues, github_create_branch, workflow_generate_branch_name |
| `arc-pr-publisher` | arc-android-quality-standards, arc-android-tdd-patterns | github_create_pr, linear_get_issue, linear_update_issue, workflow_get_conventions, github_list_prs |
| `arc-release-orchestrator` | arc-android-workflow | github_create_branch, github_create_pr, linear_list_issues, workflow_get_conventions |
| `arc-play-distribution` | arc-android-github-actions-ci | — |
| `arc-play-store-listing` | app-marketing-context, keyword-research, metadata-optimization | WebSearch, WebFetch |
| `arc-room-migration` | arc-android-data-layer, arc-android-tdd-patterns | — |
| `arc-dependency-auditor` | arc-android-project-setup | — |

---

## Design Principles

- **Minimum privilege**: read-only agents (`arc-kotlin-reviewer`, `arc-project-explorer`, `arc-dependency-auditor`) have no Edit/Write tools
- **Dynamic skill invocation**: agents invoke skills relevant to the task, not all skills by default
- **Environment first**: `arc-kotlin-debugger` always checks Gradle daemon and cache before touching code
- **Model selection**: Haiku for deterministic tasks (Gradle, exploration, scaffolding); Sonnet for complex reasoning (TDD, review, debugging, releases, migrations)
- **No commit/push**: all agents with write access are constrained from committing or pushing

---

## Integration with arcdevtools-setup

Symlinks for agents follow the same pattern as skills. To install in a project:

```bash
./ARCDevTools/arcdevtools-setup
```

The setup script symlinks `.claude/agents/arc-*.md` into the project's `.claude/agents/` directory.

Add to `.gitignore`:
```
.claude/agents/arc-*.md
```

---

## Adding a New Agent

1. Create `.claude/agents/arc-[name].md` with frontmatter (`name`, `description`, `model`, `tools`)
2. List the skills it will invoke dynamically in its instructions
3. Add entry to this `AGENTS.md`
4. Add entry to `Skills/skills-index.md` under the Agents section
