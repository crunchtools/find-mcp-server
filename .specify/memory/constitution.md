# find-mcp-server Constitution

> **Version:** 1.0.0
> **Ratified:** 2026-03-03
> **Status:** Active
> **Inherits:** [crunchtools/constitution](https://github.com/crunchtools/constitution) v1.17.0
> **Profile:** Claude Skill

## Overview

The `/find-mcp-server` skill discovers, evaluates, and installs third-party MCP servers. It combines traditional security vetting (dangerous code patterns, dependency analysis) with a quality scorecard derived from the CrunchTools MCP Server profile constitution.

## License

AGPL-3.0-or-later

## Versioning

Follow Semantic Versioning 2.0.0. MAJOR/MINOR/PATCH.

## SKILL.md Standards

- YAML frontmatter with `name`, `description`, `argument-hint`, `allowed-tools`
- Organized into numbered Phases with numbered Steps
- Phase gates require user decisions before proceeding

## Memory Integration

- Phase 1 Step 1: Searches memory for prior evaluations of the target service
- Phase 7: Stores evaluation results (grade, risk score, findings, installation decision)

## User Confirmation Gates

- Phase 1 → Phase 2: User selects a repo to evaluate
- Phase 5 → Phase 6: User decides whether to install after reviewing the combined report
- Phase 6: User confirms adding to Claude Code config

## Constitution Scorecard

The skill uses the CrunchTools MCP Server profile as a quality benchmark for evaluating third-party MCP servers. This is not about enforcing CrunchTools conventions on external projects — it uses the five-layer security model and engineering standards as a rubric to measure maturity across 8 dimensions:

1. Credential Protection (Layer 1)
2. Input Validation (Layer 2)
3. API Hardening (Layer 3)
4. Dangerous Operation Prevention (Layer 4)
5. Supply Chain Security (Layer 5)
6. Testing
7. Architecture
8. Documentation

Each dimension scores 0-3, producing a total out of 24 with letter grades A-F. The scorecard complements (not replaces) the traditional security analysis.

## Relationship to mcp-scout

This skill replaces the earlier `/mcp-scout` skill. Key differences:
- Renamed to follow the `<verb>-<noun>` naming convention (`find-mcp-server`)
- Added constitution-based quality scorecard (Phase 4)
- Combined report merges security findings with quality assessment
- Memory integration for storing and retrieving evaluations
- References constitution rather than embedding security patterns inline
