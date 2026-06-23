# Fleet Architecture

## Philosophy

This fleet is built around three principles:

**Role specialization over generalism.** Each agent has a narrow, well-defined scope. The investigator diagnoses but never patches. The spec-reviewer checks compliance but never code quality. The crypto-security-reviewer stays in crypto and redirects everything else. Narrow scope means agents are auditable, their outputs predictable, and their failure modes understood.

**Anti-sycophancy as a structural property.** The most expensive failure mode in LLM-assisted engineering is an agent that agrees with the implementer instead of finding the bug. This fleet addresses that structurally:
- Reviewers are dispatched *after* implementers complete — separate context, no shared assumptions.
- Crypto-security-reviewer and code-quality-reviewer run in *parallel* on crypto diffs — neither sees the other's output before forming a verdict.
- CoVe (Chain-of-Verification, arxiv 2309.11495) is required for hypothesis confidence claims in investigator and infrastructure-auditor.
- Findings must have `file:line` + cited invariant before they can block a merge.

**Parallel dispatch over sequential handoffs.** Independent tasks run in parallel. Reviews of a crypto diff dispatch code-quality-reviewer AND crypto-security-reviewer simultaneously. Infrastructure primer runs while the plan is being drafted. This is not just efficiency — it prevents the second reviewer from being anchored by the first reviewer's framing.

---

## Two-Stage Review Chain

Every non-trivial code change passes through two reviewers in sequence:

```
implementer → spec-reviewer → code-quality-reviewer → ship
```

**spec-reviewer** (stage 1): Binary PASS/FAIL. Did the implementer follow the spec? Checks required files, required code blocks, acceptance criteria. Does not judge style or architecture.

**code-quality-reviewer** (stage 2): Multi-stage independent review with OWASP quick-scan. Assumes spec passed. Checks correctness, style, security, performance, maintainability. Has its own anti-sycophancy contract (hard quotas, no affirmation phrases, mandatory file:line citations).

The stages are separate because the same model in the same context cannot give both "did you do what you said?" and "is what you did good?" simultaneously without anchoring on the implementer's intent.

---

## Parallel Review for Crypto Diffs

When a diff touches cryptographic code (any match against: `crypto::|@noble/|libsodium|ring|rustls|jwt|jose|sign|verify|encrypt|decrypt|hkdf|hmac|aead|nonce|iv|kdf|prng|csprng|zeroize|sframe`), dispatch **both** reviewers in parallel:

```
crypto diff → code-quality-reviewer  (parallel)
           → crypto-security-reviewer (parallel)
```

Code-quality-reviewer catches: API misuse, refactor mistakes, build hygiene, general correctness.

Crypto-security-reviewer sweeps the 12-class threat catechism: nonce reuse, transcript binding, KCI/UKS, constant-time, side channels, key handling, downgrade paths, plus live probes (tls_probe, sframe_integrity_probe, relay_jwt_forge_probe) when externally-observable behavior changed.

Neither reviewer should see the other's output before forming their verdict. Parallel dispatch is the mechanism.

For critical-tier changes (new protocol introduction, sFrame ratchet, JWT middleware, TLS pinning), crypto-security-reviewer is **blocking** — the PR cannot merge until it APPROVE or APPROVE_WITH_RISKS.

---

## Debug Spectrum

Three debug agents cover a spectrum from diagnosis-only to full-arc execution:

```
investigator           — diagnose only, ranked hypotheses, never patches
atomic-engineer        — full multi-PR arc (empirical capture → fix → PR → verify)
deep-fix               — single-dispatch hard-bug (evidence rich, diagnose+fix in one shot)
```

**investigator**: Use when you need ranked hypotheses before committing to a fix direction. Returns CERTAIN/LIKELY/POSSIBLE/UNVERIFIED hypotheses with file:line anchors, molecular timelines, and discriminator tests. Never writes code.

**atomic-engineer**: Use when you want the full fix arc in one dispatch. Owns: empirical capture → Phase 2 hypothesis → Phase 3 targeted fix → Phase 4 TDD verify → Phase 5 observability close → Phase 6 independent review → Phase 7 PR. Stops at `gh pr create`.

**deep-fix**: Use when investigator already returned a LIKELY/CERTAIN hypothesis and you want execution without re-diagnosis, or when evidence is rich enough (stack trace + metric spike) to skip the full investigator pass.

All three: metrics-first discipline (pull live counters before reading code), CoVe verification on hypotheses, observability close (every fix must close the gap that let the bug be silent).

---

## infrastructure-auditor as Mandatory First Step

Before any meaningful work on an existing repo or service, dispatch infrastructure-auditor in **primer mode**. This produces a context dossier: deployed versions, in-house capabilities, invariants, existing workarounds (reported neutrally).

This is not optional overhead — it prevents:
- Reimplementing an in-house capability that already exists at a known file:line
- Violating a non-obvious constraint (deploy mechanism, resource limits, ownership rules)
- Planning a refactor without knowing an active refactor is already in flight

The primer takes one pass and runs in the background while the plan is being drafted. The overhead is low; the cost of missing a constraint in the plan is high.

---

## Phase-Boundary STOP Discipline

Every agent that implements or modifies code has explicit phase boundaries with STOP conditions. The most important:

**STOP at `gh pr create`** — implementers (go-elite-engineer, tdd-implementer, atomic-engineer, deep-fix) never merge. The controller (human or orchestrating agent) merges. Critical changes (crypto/auth/payments/migrations/deploy) require explicit per-PR acknowledgment from the operator.

**STOP at plan** — architect produces a plan and stops. It does not implement. The plan goes to tdd-implementer or go-elite-engineer.

**STOP at diagnosis** — investigator produces hypotheses and stops. It does not fix.

Phase discipline is what makes parallel dispatch safe. A reviewer reading a diff in parallel with an implementer's next task is safe because neither has merge access and both stop at their phase boundary.

---

## ASCII Agent Interaction Diagram

```
  ┌──────────────────────────────────────────────────────────────┐
  │                    BEFORE ANY CODE WORK                      │
  │                                                              │
  │  infrastructure-auditor (primer) ──→ context dossier        │
  └──────────────────────────────────────────────────────────────┘

  ┌──────────────────────────────────────────────────────────────┐
  │                      PLAN WORKFLOW                           │
  │                                                              │
  │  infrastructure-auditor ──→ architect ──→ tdd-implementer×N │
  │                                    └──→ go-elite-engineer×N  │
  └──────────────────────────────────────────────────────────────┘

  ┌──────────────────────────────────────────────────────────────┐
  │                      SHIP WORKFLOW                           │
  │                                                              │
  │  implement ──→ spec-reviewer ──→ code-quality-reviewer ──→ ship
  │                                         │                    │
  │                               [crypto diff]                  │
  │                               crypto-security-reviewer       │
  │                                         │                    │
  │                               [frontend diff]                │
  │                               design-quality-reviewer        │
  │                                                              │
  │  After deploy:                                               │
  │  empirical-reviewer ──→ VERIFIED                            │
  └──────────────────────────────────────────────────────────────┘

  ┌──────────────────────────────────────────────────────────────┐
  │                     DEBUG WORKFLOW                           │
  │                                                              │
  │  evidence ──→ investigator ──→ (hypothesis)                  │
  │                    │                │                        │
  │                    │          atomic-engineer                │
  │                    │                │                        │
  │                    └──→ deep-fix (evidence rich)            │
  │                              │                               │
  │                         PR ──→ empirical-reviewer ──→ close  │
  └──────────────────────────────────────────────────────────────┘

  ┌──────────────────────────────────────────────────────────────┐
  │                   BEFORE ANY PR OPEN                         │
  │                                                              │
  │  code-intelligence MCP:                                      │
  │    prepare_change → review_delta → suggest_reviewers         │
  └──────────────────────────────────────────────────────────────┘
```
