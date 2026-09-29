# FYEO audit methodology — how a professional Solana review is structured

Synthesized from the "Our Process / Methodology" sections of the FYEO public reports (`fyeo-io/public-audit-reports`) plus the Solana Foundation gov-contract fuzzing report. Read this to structure your own internal review the way an external auditor will — so the audit is a verification, not a discovery exercise.

## The pipeline

### 1. Kickoff

Engagement starts after scoping. The kickoff meeting establishes:

- Designated points of contact
- Communication methods and frequency
- Shared documentation (spec, design docs)
- Code and/or other artifacts necessary for project success
- Follow-up meeting schedule, such as a technical walkthrough
- Timeline and duration

→ Everything on this list is the team's homework *before* kickoff. See `audit-readiness-checklist.md`.

### 2. Ramp-up

Auditors gain proficiency on the project: reviewing prior work (including academic papers), refreshing language-specific constructs, and researching common flaws and recent advancements in the area.

### 3. Review

The bulk of the work — architecture analysis, code review, and spec matching, performed **manually** with reviewer experience; no dynamic testing in a standard code review, only custom-built scripts/tools to assist. Two tracks:

**Code safety** — the categories they check (non-comprehensive, per FYEO):

- General code safety and susceptibility to known issues
- Poor coding practices and unsafe behavior
- Leakage of secrets or sensitive data through memory mismanagement
- Susceptibility to misuse and system errors
- Error management and logging

**Technical specification matching** — code checked against the provided documentation:

- Proper implementation of the documented protocol phases
- Proper error handling
- Adherence to the protocol's logical description

→ This is why a written spec matters: "implementation ≠ documented functionality" is a *finding* on its own (the `spec-mismatch` class — and FYEO counts a security claim in the docs that isn't in the code as **High**).

### 4. Reporting

Draft report → executive summary (findings count + general risk statement) → technical details per finding (severity + recommendations) → optional public report. Non-security best-practice issues are called out as encountered (most `code-quality`/`testing`/`observability` entries in the catalog are this). Severity is weighed by **impact, ease of exploitability, and probability of attack** — exact definitions below.

### 5. Verify

Fixes are verified within an agreed window; statuses transition `Open → Remediated` in the final report. `Acknowledged` means the team saw the finding and chose not to fix (risk accepted) — it stays in the report. Remediation commits are referenced by hash in the report's scope section (e.g., Voltr: review commit + remediation commit both listed).

### The disclaimer

Every report carries it, verbatim: *"although we did our best in our analysis, no code audit or assessment is a guarantee of the absence of flaws. Our effort was constrained by resource and time limits along with the scope of the agreement."* An audit is a sample, not a proof — design for the miss (see `vulnerability-abundance.md` §4).

## Severity definitions (as stated in the reports)

| Severity | Definition (FYEO's criteria, non-exhaustive) |
|----------|--------------------------------------------|
| **Critical** | Vulnerability will lead to a loss of protected assets — immediate loss, low complexity to exploit, high probability of exploit |
| **High** | Vulnerability has potential to lead to a loss of protected assets — including: any security claim made in documentation not found in code; mismatches between stated and actual functionality; unprotected key material; weak encryption of keys; badly generated key material; transaction signatures not verified; spending of funds through logic errors; calculation errors, overflows and underflows |
| **Medium** | Vulnerability hampers the uptime of the system or can lead to other problems — e.g., insecure calls to third-party libraries; use of untested/nonstandard/non-peer-reviewed crypto functions; program crashes, core dumps, or sensitive data written to logs |
| **Low** | Vulnerability has a security impact but does not directly affect protected assets — e.g., overly complex functions; unchecked return values from third-party libraries that could alter execution flow |
| **Informational** | General recommendations |

Note the High tier: **spec-mismatch and arithmetic errors are rated High**, not Medium — matching the catalog's distribution.

## Ongoing / diff-review engagements

Several reports are "Ongoing" engagements (Banger 2024.12.20, Banger 2025.03.25, Spree 2025.08.11, Turbine 2025.12.04): short reports reviewing **the modifications since a previous review** — a commit range or version bump, not the full codebase. They follow the same finding format, typically produce 0–2 findings, and end with a continued-robustness conclusion. This is the post-audit model: every material change gets a diff review rather than a re-audit.

## Fuzzing (Trident methodology — Solana Foundation gov-contract report)

FYEO's fuzz campaign reports a different shape of evidence. Key elements:

- **Shadow / expected-state tracking:** the fuzzer maintains an `ExpectedState` mirroring what the contract *should* compute (per-proposal totals, per-validator and per-delegator trackers). After each successful operation both contract and fuzzer update state; they are compared at iteration end — mismatches panic as invariant violations.
- **Invariants checked:** vote-total correctness, no overflow/underflow (checked_add/sub confirmed panic-free), vote-count bookkeeping, override cache application, no double voting, epoch validation, finalized-proposal immutability, override authorization.
- **Scale:** 5,000+ iterations / 25,000+ flow executions; randomized parameters bounded to realistic ranges (delegator stake 1–1000 SOL, vote BPs summing to 10,000, 5 validators × 5 delegators), plus explicit edge cases (100% single category, override before/after validator vote, repeated modifies).
- **Reproducibility:** every run emits a `MASTER SEED`; any failure is replayed with `MASTER_SEED=<seed> cargo run`.
- **Explicit non-coverage:** a "What this does NOT prove" table — instructions needing Clock syscalls or external-program CPIs were not tested, Merkle cryptography was mocked, large-scale/value extremes untested. Confidence table then grades each aspect HIGH vs MEDIUM accordingly.

The lesson: a fuzz report is credible because it *enumerates what it did not cover*. Write yours the same way.

## FYEO-style finding template

Use this for `docs/security-review.md` in your working repo:

```markdown
# Security Review — <program> — <date>
Scope: <repo>@<commit> · <file tree summary> · Reviewer: <agent/human>
Findings: <n> (C:c H:h M:m L:l I:i)

## SEC-<PROJ>-NN — <title>
- **Severity:** Critical | High | Medium | Low | Informational
- **Status:** Open | Acknowledged | Remediated
- **Description:** what is wrong and why it matters (mechanism, affected accounts/instructions)
- **Proof of Issue:** file, line, code excerpt or reproduction trace
- **Severity and Impact Summary:** who can trigger it, what they gain/lose, blast radius
- **Recommendation:** the fix, or the accepted-risk rationale
```

Related: `fyeo-audit-findings-catalog.md` (the empirical corpus), `audit-readiness-checklist.md` (pre-audit requirements), `AGENTS.md` (how to run this review with an agent).
