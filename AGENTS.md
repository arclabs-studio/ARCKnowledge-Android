# ARC Labs Android Subagents

Subagents are **autonomous executors** — they take a task, use tools, and return a result. This is distinct from Skills, which are **knowledge containers** that load patterns for Claude to apply.

| Concept | Role | How to trigger |
|---------|------|----------------|
| **Skills** | Load reference patterns — Claude applies them | `android-doctrine`, `android-testing-doctrine`, etc. (see [Skills/skills-index.md](Skills/skills-index.md)) |
| **Agents** | Execute autonomously — Claude delegates the task | "Implement X with TDD", "Review before commit", "Debug this error" |

---

## Active Agents

These agents exist (definitions in ARCAgentStack `config/agents/`, symlinked into `~/.claude/agents`) and are the ones [CLAUDE.md](CLAUDE.md) routes to.

| Agent | Model | Read-only | Role |
|-------|-------|-----------|------|
| `arc-verify-android` | Sonnet | Yes (Bash, Read, Grep, Glob) | External verifier — zero-warning Gradle build, unit tests, Compose preview screenshot validation, install-and-walk on a headless emulator via the `android` CLI. Iterate until it returns a pass. |
| `arc-audit-material` | Sonnet | Yes (Bash, Read, Grep, Glob) | Audits UI work against Material 3 Expressive, the Navigation 3 shell, edge-to-edge, adaptive layout and TalkBack. One blocker and the UI work unit does not close. |

---

## Design Principles

- **Minimum privilege**: read-only agents (`arc-verify-android`, `arc-audit-material`) have no Edit/Write tools
- **Dynamic skill invocation**: agents invoke skills relevant to the task, not all skills by default
- **Model selection**: the agent's frontmatter pins its model; both active agents run on Sonnet
- **No commit/push**: all agents with write access are constrained from committing or pushing

---

## Installation

ARC agents and skills come from **ARCAgentStack** (`config/agents/`, `skills/android/`), symlinked into `~/.claude` — nothing to install per project. ARCDevTools-Android's `arc-setup` does **not** install agents or skills.

---

## Adding a New Agent

1. Create `config/agents/arc-[name].md` in ARCAgentStack with frontmatter (`name`, `description`, `model`, `tools`), using the `arc-agent-authoring` skill
2. List the skills it will invoke dynamically in its instructions
3. Add entry to this `AGENTS.md` (Active Agents) and to the agents table in [CLAUDE.md](CLAUDE.md)
4. Add entry to `Skills/skills-index.md` under the Agents section
