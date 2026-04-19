# Changelog

All notable changes to The Architect Overlay will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2.0.0] - 2026-04-19

### Added
- **Phase 1: Separation of Concerns**
  - Extracted SDD orchestrator from AGENTS.md to `.gentle/orchestrator.md`
  - Created `.gentle/context-protocol.md` for contextual loading
  - Created `.gentle/sdd.yaml` for project-specific configuration
  - Created `.gentle/standards/` with 3 architectural patterns
  
- **Phase 2: Contextual Loading Protocol**
  - Dynamic skill injection based on SDD phase
  - Standards absorption (hexagonal, clean, testing)
  - Template maestro in `templates/.gentle/`
  
- **PROACTIVE CTO MODE**
  - Intelligent guidance when requirements are unclear
  - Tech stack recommendations with justification
  - Automatic standards file creation offer

### Changed
- AGENTS.md reduced from 224 to 65 lines (71% reduction)
- opencode.json wiring updated to read both identity and workflow files

### Fixed
- Context pollution from auto-loading 10+ skills globally
- Scope mixing between global identity and local workflow

## [1.0.0] - 2026-04-18

### Added
- Initial release based on Gentleman Programming's SDD methodology
- Basic SDD workflow implementation
- Engram integration for persistent memory
