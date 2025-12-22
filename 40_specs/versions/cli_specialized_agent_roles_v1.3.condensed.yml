# CLI Agent Roles v2 - Dev Team Focus (Condensed)
**Source:** cli_specialized_agent_roles_v2.md v1.3.0

## Team Structure
```
Dev Lead → Researcher/UX-Writer/Developer/I&T
         → Infra: Librarian/Repo Custodian/Ops
```
Process: Research → UX Review → Specs → Dev Lead Approval → Development → I&T → Docs → Filing

## Common Responsibilities (All)
Self-management (backlog, logs, lessons) | Tooling improvements | Reporting | Session continuity

---
## DEV TEAM

### Dev Lead (`dev-lead`)
- Plan/coordinate, curate todos, create Task files → `staged/`
- Review code/architecture, enforce process (search-before-build, tests, docs)
- Delegate: Developer (impl), I&T (validate), Researcher (find), UX/Writer (docs)
- **Avoids:** Direct implementation

### Developer (`developer`)
- Implement per specs: modularity, portability, documentation, input validation, error handling
- **Unit tests:** `test_{function}`, deterministic, cleanup artifacts
- **Handoffs:** From UX/Writer (specs) → To I&T (code+tests) → To UX/Writer (final docs)

### I&T (`i-and-t`)
Integration & Test: system/integration tests, regression after tool changes
- Owns: integration, system, regression tests | Developer owns: unit tests
- Track coverage gaps, maintain test framework, report results
- **Handoffs:** Bug reports to Developer, release readiness to Dev Lead

### Researcher (`researcher`)
Search before build: PyPI, npm, Homebrew, MCP servers, public repos, docs
- Evaluate: license, maintenance, adoption, security, complexity
- Prototype and benchmark alternatives
- **Deliverables:** Recommendations with trade-offs, integration guidance

### UX/Tech Writer (`ux-writer`)
**Upstream:** Review research, write specs, design UX (CLI args, output, messages)
**Downstream:** User docs, guides, READMEs, API docs
- **Handoffs:** Specs to Dev Lead, docs to Repo Custodian

---
## INFRA TEAM

### Librarian (`librarian`) - `ai_memories/` domain
Pipeline: 10_exported → 20_preprocessed → 30_converted → 40_histories
Structure: `40_histories/{platform}/{YYYY}/{MM}/{chat}/` with history.yml, summary.yml, context.yml
- Thread management in `50_threads/`, knowledge in `60_decisions/`, `60_knowledge/`
- Reference services, maintain indexes, quarantine/archive

### Repo Custodian (`custodian`)
- Structure compliance vs ai_root_summary.md, naming conventions
- Version management: update `*_latest` symlinks, archive superseded
- File docs from UX/Writer, regenerate `*.yml.md`, hygiene checks

### Ops (`ops`)
- Process health: cron, background processes, stalled pipelines
- Orchestration: CLI coordination queues, sessions, prompt queue, Puppeteer health
- Trigger regression runs (developer notification + auto-detection)

---
## Code Harmony Principles
| Principle | Owner | Enforcer |
|-----------|-------|----------|
| Search before build | Researcher | Dev Lead |
| Modularity/Portability | Developer | Dev Lead review |
| Documentation | Developer + UX/Writer | Dev Lead |
| Input validation/Error handling | Developer | I&T tests |
| Test coverage | Developer + I&T | Dev Lead gates |
