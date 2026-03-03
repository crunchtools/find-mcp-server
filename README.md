# find-mcp-server

Claude Code skill for discovering, evaluating, and installing MCP servers with security vetting and constitution-based quality scoring.

## What It Does

`/find-mcp-server <query>` searches for MCP servers, forks and clones them for local review, runs security analysis, scores them against the [CrunchTools MCP Server profile](https://github.com/crunchtools/constitution/blob/main/profiles/mcp-server.md), and generates a combined evaluation report.

## Installation

```bash
git clone https://github.com/crunchtools/find-mcp-server ~/Projects/crunchtools/find-mcp-server
ln -sf ~/Projects/crunchtools/find-mcp-server ~/.claude/skills/find-mcp-server
```

## Usage

```
/find-mcp-server github actions
/find-mcp-server https://github.com/owner/mcp-server-name
/find-mcp-server filesystem server
```

## Phases

1. **Discovery** — search GitHub, awesome-mcp-servers, MCP Registry
2. **Fork and Clone** — fork to your org, clone for local review
3. **Security Analysis** — dangerous code patterns, dependency review, capability audit
4. **Constitution Scorecard** — 8-dimension quality assessment against MCP Server profile
5. **Combined Report** — risk score + quality grade + recommendations
6. **Installation** — generate config, add to Claude Code
7. **Store in Memory** — save evaluation for future reference

## Constitution Scorecard

Uses the CrunchTools five-layer security model as a quality benchmark:

| Dimension | What it measures |
|-----------|-----------------|
| Layer 1: Credential Protection | SecretStr/env vars, error scrubbing |
| Layer 2: Input Validation | Pydantic/Zod, field limits, extra rejection |
| Layer 3: API Hardening | Header auth, TLS, timeouts, response limits |
| Layer 4: Dangerous Op Prevention | No eval/exec/shell, pure API wrappers |
| Layer 5: Supply Chain Security | CI, scanning, lockfiles, base images |
| Testing | Mocked tests, coverage, CI integration |
| Architecture | Module separation, error hierarchy, clean structure |
| Documentation | README, env vars, tool listing, security policy |

Each scores 0-3 (total /24). Grades: A (20+), B (15-19), C (10-14), D (5-9), F (0-4).

## Replaces

This skill replaces the earlier `/mcp-scout` skill with constitution-based scoring, memory integration, and the `<verb>-<noun>` naming convention.

## License

AGPL-3.0-or-later
