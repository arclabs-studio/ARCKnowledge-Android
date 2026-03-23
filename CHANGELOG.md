# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.2.0] - 2026-03-23

### Added
- Claude Code AI tooling infrastructure: skills, agents, and MCP server configuration
- AGENTS.md with full agent documentation, triggers, and model assignments
- Skills index at `Skills/skills-index.md` with ARC Labs Android skills routing
- MCP setup guide at `Tools/mcp-setup.md` (Context7, Linear, GitHub, Android Source Explorer, Android Docs MCP, Mobile MCP)
- Context7 usage guide at `Tools/context7-usage.md` with query patterns

### Changed
- Enriched all 30 documents with official Android documentation references
- Updated CLAUDE.md with MCP servers table and skills quick-decision guide

### Fixed
- `singletons.md`: replaced `MoshiConverterFactory` with Kotlinx Serialization (correct default)
- `apps.md`: replaced `kapt` with `ksp` for Hilt compiler
- `compose-performance.md`: fixed broken internal cross-references
- `.gitattributes`: added for consistent line endings across platforms

## [1.0.0] - 2026-02-25

### Added
- Initial ARCKnowledge-Android knowledge base
- Architecture documentation: Clean Architecture, MVVM, SOLID, DI, Navigation, Singletons
- Layer documentation: Presentation, Domain, Data
- Quality documentation: Testing, Code Style, Code Review, Documentation, Module Structure, README Standards, UI Guidelines, Compose Performance
- Tools documentation: Gradle, Android Studio, ARCDevTools-Android
- Workflow documentation: Git Commits, Git Branches, Plan Mode
- Skills index with ARC Labs Android skills
- CLAUDE.md entry point for AI agents
