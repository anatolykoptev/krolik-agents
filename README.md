# claude-agent-fleet

> A production-grade fleet of 24 Claude Code subagents — independent reviewers, debuggers, implementers, and infra auditors — built and battle-tested across a multi-repo production environment. Used in production, not toys.

<div align="center">

[![License MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Last commit](https://img.shields.io/github/last-commit/anatolykoptev/claude-agent-fleet)](https://github.com/anatolykoptev/claude-agent-fleet/commits/main)
[![Subagents](https://img.shields.io/badge/subagents-24-green)](agents/)
[![Skills](https://img.shields.io/badge/skills-2-purple)](skills/)
[![PRs welcome](https://img.shields.io/badge/PRs-welcome-brightgreen)](CONTRIBUTING.md)

</div>

---

## What this is

These are role-specialized Claude Code subagents wired into a real review/debug/ship workflow. Each agent has a precise scope, an explicit anti-sycophancy contract, and defined termination rules (most stop at `gh pr create` — the operator merges).

The fleet was built and is actively used against a multi-service production stack: Go, Rust, TypeScript/SvelteKit, SQL migrations, Docker/Compose. Nothing here is theoretical. Every design decision was driven by a real failure mode — an agent that agreed with the implementer instead of finding the bug, a reviewer that missed a nonce reuse, a plan that didn't account for an active refactor in flight.

**Key properties:**
- **Anti-sycophancy** — reviewers are dispatched after implementers complete, in separate context. Crypto reviewer and code-quality reviewer run in parallel on crypto diffs; neither sees the other's output.
- **Phase-boundary discipline** — every implementer stops at `gh pr create`. Every reviewer is read-only. No agent auto-merges.
- **Evidence-first** — findings require `file:line` + cited invariant. Hypotheses require molecular timelines. "Looks good" is not a verdict.
- **Observability close** — every fix arc must instrument the failure class so the next regression surfaces a metric, not a user complaint.

---

## Quick start

```bash
# Option A: Claude Code marketplace
/plugin marketplace add anatolykoptev/claude-agent-fleet

# Option B: manual install
cp -r agents/* ~/.claude/agents/
```

---

## What's inside

| Item | Count | Description |
|---|---|---|
| Subagents | 24 | Role-specialized Claude Code agents |
| Skills | 2 | llm-council, postgres |
| Plugins | 4 | go-tools, startup-pressure-pro, claude-code-memdb, local-marketplace |
| Full prompts | 5 | code-quality-reviewer, crypto-security-reviewer, investigator, atomic-engineer, infrastructure-auditor |

---

## Agent roster

### Review & Quality

| Agent | Description | Model |
|---|---|---|
| **code-quality-reviewer** | Independent, anti-sycophancy multi-stage review with OWASP quick-scan. CRITICAL/HIGH/MEDIUM/LOW findings with file:line citations. | `opus` |
| **crypto-security-reviewer** | 12-class adversarial crypto catechism: confidentiality, integrity, authenticity, transcript binding, forward secrecy, replay, downgrade, KCI/UKS, side channels, key handling, library misuse, language pitfalls. Live pentesting probes. | `opus` |
| **spec-reviewer** | Binary spec-compliance gate before code-quality review. PASS/FAIL/DEVIATION_ACCEPTABLE. | `sonnet` |
| **empirical-reviewer** | Post-deploy runtime verification: counters, endpoints, env gates. Distinguishes NO_TRAFFIC from WIRING_BROKEN. | `sonnet` |
| **observability-auditor** | Silent-failure coverage gap finder. Two modes: audit (find blind spots) and design (instrument a new path). Proposes exact counter + alert rule. | `sonnet` |
| **design-quality-reviewer** | Independent UX/visual review against DESIGN.md tokens. Nielsen heuristics, WCAG 2.1 AA, cognitive load. Hard gate on missing DESIGN.md. | `sonnet` |
| **design-empirical-reviewer** | Post-deploy Lighthouse + axe-core + screenshot visual verification at 3 viewports. Catches CSP-broken styles, FOIT, broken LCP elements. | `opus` |

### Debug & Diagnostics

| Agent | Description | Model |
|---|---|---|
| **investigator** | Live evidence → ranked hypotheses (CERTAIN/LIKELY/POSSIBLE/UNVERIFIED) → file:line fix proposals. Metrics-first discipline. CoVe verification. Never patches. | `opus` |
| **atomic-engineer** | Multi-PR fix arc: empirical capture → diagnose → targeted fix → TDD verify → observability close → independent review → PR. 12 inviolable axioms. | `opus` |
| **deep-fix** | Single-dispatch hard-bug fixer: abbreviated diagnosis + RED test + patch + pre-mortem + observability + PR. Use when evidence is rich and you want execution without serial handoffs. | `opus` |
| **build-error-resolver** | Minimal-diff build/type error resolution across Rust, Go, TypeScript. Classifies error type, applies minimal fix, verifies build, reports NEEDS_REFACTOR when scope exceeds its mandate. | `opus` |

### Implementation

| Agent | Description | Model |
|---|---|---|
| **architect** | System design with constraint capture, reversibility tags, pre-mortem, and phased plan with STOP conditions. Subsystem-tier+ decisions. | `opus` |
| **go-elite-engineer** | FAANG-grade implementation: Go, Rust, TypeScript, Svelte, Python, shell, SQL, Dockerfile. Correct idioms, existing style matched, full verify cycle, PR. | `sonnet` |
| **tdd-implementer** | Plan-task executor: RED → GREEN → REFACTOR → commit. One task at a time. STOPS at commit. | `sonnet` |
| **rebase-resolver** | Conflict-by-conflict rebase preserving intent from both sides. Per-hunk decisions with intent reasoning. Never wholesale "theirs" or "ours". | `sonnet` |
| **refactor-cleaner** | Evidence-based dead code and duplicate removal using code-intelligence tools. Blast-radius check via impact_analysis before every deletion. | `opus` |
| **wip-curator** | WIP archaeology: triage dirty checkouts, stash piles, orphaned worktrees. Ship/discard/split decisions. | `sonnet` |

### Research & Infrastructure

| Agent | Description | Model |
|---|---|---|
| **code-research** | Three modes: competitor landscape, solution search (build/borrow/wait), port cost-matrix. Combines web research with codebase analysis. | `opus` |
| **infrastructure-auditor** | Pre-implementation context primer + workaround audit. Default first step before any meaningful work on an existing repo. CoVe verification in audit mode. | `opus` |
| **doc-auditor** | Docs-vs-reality drift detection. Evidence-anchored stale claims ("doc says X, code shows Y at file:line since commit Z"). Two modes: audit and update. | `sonnet` |
| **doc-updater** | Codemap generation and README maintenance from live codebase state. | `opus` |
| **prompt-master** | 7-stage production prompt engineering pipeline: intake → research → expert panel → multi-level analysis → synthesis → adversarial critique → revision. Returns complete XML prompt. | `opus` |

### Frontend

| Agent | Description | Model |
|---|---|---|
| **design-orchestrator** | Frontend design router: build/polish/animate → impeccable skill; pre-merge review → design-quality-reviewer; post-deploy → design-empirical-reviewer. | `sonnet` |
| **e2e-runner** | Playwright E2E specialist: stable journey specs (data-testid + ARIA selectors), flaky test quarantine, artifact collection, suite maintenance. | `opus` |

---

## Skills

| Skill | Description |
|---|---|
| **llm-council** | Multi-model deliberation for high-stakes decisions. Queries multiple models independently, aggregates verdicts, surfaces disagreements. |
| **postgres** | Production Postgres operations: EXPLAIN ANALYZE, index strategy, safe large-table migrations, connection pool tuning. Reads live DB state before recommending. |

---

## Plugins

| Plugin | Description |
|---|---|
| **go-tools** | Code-intelligence MCP integration: semantic search, call traces, impact analysis, dead code, duplicate detection, repo analysis. |
| **startup-pressure-pro** | Startup idea validation: pressure-test with adversarial critique, market sizing, competitor analysis. |
| **claude-code-memdb** | Persistent memory across sessions using a local vector DB. Store and retrieve project-specific knowledge. |
| **local-marketplace** | Browse and install agents from the fleet without internet access. |

---

## Workflow

```
              ── SHIP WORKFLOW ──

 implement ──→ spec-reviewer ──→ code-quality-reviewer ──→ empirical-reviewer ──→ ship
                                          │
                                crypto-security-reviewer  (on crypto diff, parallel)
                                          │
                               design-quality-reviewer    (on frontend diff, parallel)

              ── DEBUG WORKFLOW ──

 evidence ──→ investigator ──→ atomic-engineer ──→ fix ──→ empirical-reviewer ──→ close
                    └──────→ deep-fix  (evidence rich, single dispatch)

              ── PLAN WORKFLOW ──

 idea ──→ infrastructure-auditor ──→ architect ──→ tdd-implementer × N ──→ spec-reviewer

              ── BEFORE ANY PR ──

 code-intelligence MCP: prepare_change → review_delta → suggest_reviewers
```

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for how to submit new agents.

## License

MIT — see [LICENSE](LICENSE)
