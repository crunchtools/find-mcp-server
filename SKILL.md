---
name: find-mcp-server
description: Discover, evaluate, and securely install MCP servers — with security vetting and constitution-based quality scoring
argument-hint: "[search-query OR repo-url]"
allowed-tools: Read, Write, Grep, Glob, Bash, AskUserQuestion, WebSearch, WebFetch, Task, mcp__memory__memory_search, mcp__memory__memory_store
---

# Find MCP Server

Discover and evaluate MCP servers before installation. Combines security vetting with a quality scorecard derived from the [CrunchTools MCP Server profile](https://github.com/crunchtools/constitution/blob/main/profiles/mcp-server.md).

## Usage

```
/find-mcp-server github actions
/find-mcp-server https://github.com/modelcontextprotocol/servers
/find-mcp-server filesystem server
```

---

## Phase 1: Discovery

### Step 1: Search Memory

Search memory for any prior evaluations of the target service or MCP server:
- `memory_search` for the service or query
- `memory_search` for "mcp server security report"

### Step 2: Find Candidates

**If user provided a direct repo URL:**
- Extract the repo owner and name
- Skip to Phase 2

**If user provided a search query:**

Search using multiple sources:
- `gh search repos "$query mcp server" --limit 10` for GitHub search
- WebFetch `https://github.com/wong2/awesome-mcp-servers` for curated lists
- WebFetch `https://registry.modelcontextprotocol.io` for official registry

Present results:
```
| # | Repository | Stars | Description | Last Updated |
|---|------------|-------|-------------|--------------|
| 1 | owner/repo | 1.2k  | Description | 2 days ago   |
```

### Step 3: Select

Ask user to select a repository to evaluate using `AskUserQuestion`.

**Do NOT proceed to Phase 2 until the user selects a repo.**

---

## Phase 2: Fork and Clone

### Step 1: Check Configuration

Check for existing config at `~/.config/find-mcp-server/config.json`:
```json
{
  "github_fork_org": "username",
  "clone_directory": "~/Projects",
  "auto_fork": true
}
```

If no config exists, ask user for preferences with `AskUserQuestion` and save.

### Step 2: Fork and Clone

```bash
gh repo fork $REPO_URL --clone=false --org=$FORK_ORG  # if auto_fork
cd $CLONE_DIR && git clone $FORK_URL
```

If fork fails, clone the original directly.

**Do NOT proceed to Phase 3 until the repo is cloned locally.**

---

## Phase 3: Security Analysis

Change to the cloned repo directory. This phase checks for dangerous code patterns.

### Step 1: Dependency Check

Identify the package manager (`package.json`, `pyproject.toml`, `requirements.txt`, `Cargo.toml`, `go.mod`) and review dependencies for:
- Suspicious package names (typosquatting)
- Very low download counts or no recent updates
- Known vulnerabilities (run `vet scan --report-summary` if SafeDep is installed)

### Step 2: Dangerous Code Patterns

Search for critical patterns using Grep:

**Command injection:**
- `shell=True`, `subprocess.*shell`, `os.system(`, `eval(`, `exec(`
- `child_process` (JS/TS)

**Network exposure:**
- `0.0.0.0`, `INADDR_ANY`

**Path traversal:**
- `../`, `path.join.*..`, `os.path.join.*..`

**Hardcoded secrets:**
- `api_key\s*=`, `password\s*=`, `secret\s*=` (in source files, not test fixtures)

### Step 3: Capability Review

Find tool declarations (`@mcp.tool`, `server.tool(`, `@tool`) and list all exposed tools with:
- Name, description, input parameters
- What resources they access (filesystem, network, shell)
- Whether paths are restricted or arbitrary

Flag broad permissions: full filesystem access, network access, shell execution, environment variable access.

**Do NOT proceed to Phase 4 until security analysis is complete.**

---

## Phase 4: Constitution Scorecard

Score the server against the [CrunchTools MCP Server profile](https://github.com/crunchtools/constitution/blob/main/profiles/mcp-server.md). This is NOT about whether external servers follow CrunchTools conventions — it uses the constitution as a **quality benchmark** to measure engineering maturity.

### Scoring Rubric

Score each dimension 0-3:
- **0** = Not present
- **1** = Minimal/partial
- **2** = Solid implementation
- **3** = Exceeds expectations

#### Layer 1: Credential Protection (0-3)

Check for:
- Credentials loaded from environment variables (not hardcoded)
- Credential types use SecretStr, opaque wrappers, or equivalent (not plain strings)
- Error messages scrub credentials before display
- `__repr__`/`__str__` on config objects don't expose secrets

**How to check:**
```
Grep: SecretStr|secret_str|opaque|redact
Grep: os.environ|process.env|env\(
Grep: sanitize|scrub|redact in error handling code
```

#### Layer 2: Input Validation (0-3)

Check for:
- Typed input parameters (Pydantic, Zod, JSON Schema, or equivalent)
- Field length limits, allowlists, or format validation
- Extra/unexpected fields rejected
- Injection-safe identifier handling

**How to check:**
```
Grep: Pydantic|BaseModel|zod|z\.object|JsonSchema|validate
Grep: max_length|maxLength|min_length|minLength
Grep: extra.*forbid|additionalProperties.*false
```

#### Layer 3: API Hardening (0-3)

Check for:
- Auth via headers (not URL query parameters)
- TLS/SSL certificate validation enabled
- Request timeouts configured
- Response size limits
- URL encoding on path parameters

**How to check:**
```
Grep: timeout|Timeout
Grep: ssl|tls|verify|certificate
Grep: max.*size|maxContentLength|response.*limit
Grep: Authorization|Bearer|PRIVATE-TOKEN|X-API-Key (in headers)
```

#### Layer 4: Dangerous Operation Prevention (0-3)

Check for ABSENCE of:
- `eval()`, `exec()`, `Function()` calls on user input
- Shell execution (`subprocess`, `child_process`, `os.system`)
- Filesystem writes outside designated paths
- Dynamic code loading

Scoring:
- **3** = Pure API wrapper, no dangerous operations
- **2** = Minimal shell use with proper escaping
- **1** = Some dangerous patterns but contained
- **0** = Unrestricted shell/eval/filesystem access

#### Layer 5: Supply Chain Security (0-3)

Check for:
- CI/CD pipeline (GitHub Actions, GitLab CI, etc.)
- Automated security scanning (Dependabot, Snyk, Trivy, pip-audit)
- Lockfile present (package-lock.json, uv.lock, Cargo.lock)
- Container image from reputable base (not `latest` without pinning)

**How to check:**
```
Check: .github/workflows/ or .gitlab-ci.yml
Grep: dependabot|snyk|trivy|pip-audit|CodeQL
Check: lockfile exists
Check: Dockerfile/Containerfile base image
```

#### Testing (0-3)

Check for:
- Test files exist
- Tests are mocked (no live API calls in CI)
- Coverage of tool functions (not just imports)
- CI runs tests automatically

**How to check:**
```
Check: tests/ or __tests__/ or *_test.go directory
Grep: mock|Mock|stub|Stub|fake|Fake
Grep: pytest|jest|vitest|go test in CI config
```

#### Architecture (0-3)

Check for:
- Separation of concerns (server registration vs. business logic)
- Config management separate from tools
- Error hierarchy (not just generic exceptions)
- Clean module structure (not everything in one file)

**How to check:**
```
Count: number of source files (1 file = 0, 2-3 = 1, 4-6 = 2, 7+ = 3)
Check: separate config/client/errors modules
Check: tool functions separate from server registration
```

#### Documentation (0-3)

Check for:
- README with installation instructions
- Environment variable documentation
- Tool listing with descriptions
- Security policy (SECURITY.md)

### Scorecard Output

```
## Constitution Scorecard: [repo-name]

| Dimension | Score | Evidence |
|-----------|-------|----------|
| Layer 1: Credential Protection | X/3 | [what was found] |
| Layer 2: Input Validation | X/3 | [what was found] |
| Layer 3: API Hardening | X/3 | [what was found] |
| Layer 4: Dangerous Op Prevention | X/3 | [what was found] |
| Layer 5: Supply Chain Security | X/3 | [what was found] |
| Testing | X/3 | [what was found] |
| Architecture | X/3 | [what was found] |
| Documentation | X/3 | [what was found] |
| **Total** | **X/24** | |

### Grade
- 20-24: **A** — Production-grade, would pass CrunchTools standards
- 15-19: **B** — Solid engineering, minor gaps
- 10-14: **C** — Functional but missing key practices
- 5-9:  **D** — Significant concerns
- 0-4:  **F** — Not recommended for production use
```

---

## Phase 5: Combined Report

Generate a single report combining security findings (Phase 3) and quality scorecard (Phase 4):

```markdown
# MCP Evaluation Report: [repo-name]

## Risk Score: [Low | Medium | High | Critical]
## Quality Grade: [A | B | C | D | F] (X/24)

## Summary
[2-3 sentence overview combining security posture and engineering quality]

## Capabilities
| Tool Name | Description | Access Level |
|-----------|-------------|--------------|
| tool1     | Does X      | Network only |

## Security Findings

### Critical Issues
- [List any critical findings]

### High Severity
- [List]

### Medium Severity
- [List]

### Low / Informational
- [List]

## Constitution Scorecard
[Full scorecard table from Phase 4]

## Dependency Analysis
- Package manager: [npm/pip/cargo/etc]
- Total dependencies: [count]
- Lockfile: [yes/no]
- Known vulnerabilities: [count]

## Recommendations
- [ ] [Specific recommendation 1]
- [ ] [Specific recommendation 2]

## Verdict
[SAFE TO INSTALL | REVIEW FURTHER | DO NOT INSTALL]
[Quality assessment: what would need to change for a higher grade]
```

**Risk Score Calculation:**
- **Critical**: Any critical security finding OR multiple high findings
- **High**: 1+ high severity findings OR 3+ medium findings
- **Medium**: 1-2 medium findings OR quality grade D/F
- **Low**: Only low/informational findings AND quality grade B or better

Present the report and ask the user what they want to do with `AskUserQuestion`:
- Install the server
- Pass on it
- Review the code further

**Do NOT proceed to Phase 6 unless the user chooses to install.**

---

## Phase 6: Installation

### Step 1: Generate Config

Create the MCP configuration snippet appropriate to the server's transport:

```json
{
  "mcpServers": {
    "server-name": {
      "command": "npx",
      "args": ["-y", "@package/name"],
      "env": {}
    }
  }
}
```

Or for HTTP transport:
```json
{
  "server-name": {
    "type": "http",
    "url": "http://127.0.0.1:PORT/mcp"
  }
}
```

### Step 2: Add to Claude Code

Ask the user if they want to add to `~/.claude.json` or `~/.claude/settings.json`. If yes, read existing settings, merge, and write back.

### Step 3: Monitoring

If the server runs as an HTTP container on lotor, hand off to `/deploy-mcp-server` Phase 6 so it gets a port from, and checks in, Nagios — Nagios is the registry of MCP ports. Note the scorecard grade (and **BETA** for third-party/experimental servers) in the memory stored for the install.

---

## Phase 7: Store in Memory

Store the evaluation using `memory_store`:
- Server name, repo URL, quality grade, risk score
- Key security findings
- Installation decision and config
- Any gotchas discovered during evaluation

---

## Error Handling

- `gh` CLI not installed: Provide installation instructions
- Fork fails: Clone directly without forking
- SafeDep not available: Proceed with manual dependency analysis
- Repo is private: Ask for authentication or skip
