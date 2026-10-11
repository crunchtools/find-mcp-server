# find-mcp-server Constitution

> **Version:** 1.1.0
> **Ratified:** 2026-03-03
> **Amended:** 2026-10-02
> **Status:** Active
> **Inherits:** [crunchtools/constitution](https://github.com/crunchtools/constitution) v1.24.0
> **Profile:** Claude Skill

The `/find-mcp-server` skill discovers, evaluates and installs third-party
MCP servers, combining security vetting (dangerous code patterns, dependency
analysis) with a quality scorecard derived from the CrunchTools MCP Server
profile.

This file holds what is specific to this skill. The fleet rules and the
Claude Skill profile (frontmatter, phased workflow, gates, memory
integration) apply at the inherited version and are checked against this
repo's files by `constitution.yml`. They are not restated here.

## Constitution Scorecard

The MCP Server profile is used as a rubric to measure a third-party server's
maturity, not to impose CrunchTools conventions on it. Eight dimensions,
each scored 0 to 3, for a total out of 24 and a letter grade A to F:

1. Credential protection (Layer 1)
2. Input validation (Layer 2)
3. API hardening (Layer 3)
4. Dangerous operation prevention (Layer 4)
5. Supply chain security (Layer 5)
6. Testing
7. Architecture
8. Documentation

The scorecard complements the security analysis; it does not replace it.
The Autonomous Agent profile's allowlist threshold is expressed on this
scale.

## Phase Gates

- Phase 1 to Phase 2: the user selects a repo to evaluate.
- Phase 5 to Phase 6: the user decides whether to install after the combined
  report.
- Phase 6: the user confirms the Claude Code config change.

## Memory Records

Phase 1 Step 1 searches memory for prior evaluations of the target; Phase 7
stores the result (grade, risk score, findings, installation decision).

## Relationship to mcp-scout

Replaces the earlier `/mcp-scout` skill: renamed to the `<verb>-<noun>`
pattern, added the scorecard (Phase 4) and the combined report, added memory,
and references the constitution instead of embedding security patterns.

## History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-03-03 | Initial constitution |
| 1.1.0 | 2026-10-02 | Manifest under constitution v1.18.0: profile restatement removed, scorecard and skill specifics kept |
