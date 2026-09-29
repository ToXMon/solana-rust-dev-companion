# Security review — agent instructions

How to perform a security review of a Solana program using this directory. Fresh-context, adversarial framing: you are checking the diff against intent, not defending the code.

## Load order

1. `PRINCIPLES.md` Part B — threat-model questions + non-negotiable checklist.
2. This file.
3. `fyeo-audit-findings-catalog.md` Section A — where Solana programs empirically fail; use it to allocate review time.
4. Section B classes relevant to the target (mapping below) — read every entry in those classes.
5. `audit-methodology.md` — write findings in the FYEO template; respect the severity definitions.
6. `skills/safe-solana-builder/references/shared-base.md` + the framework-specific reference — the rule-level detail.

## Feature → class mapping

| Program feature | Review these classes first |
|-----------------|---------------------------|
| Anything with token accounts | `account-validation` (always), `token-program` |
| Vault / deposit / withdraw / yield | `accounting-state`, `input-bounds`, `token-program` |
| Admin / config / pause / authorities | `access-control`, `initialization`, `upgradability-ops` |
| Staking / rewards / fee accrual | `arithmetic`, `accounting-state` |
| Launchpad / sale / raffle / airdrop | `randomness-ordering`, `input-bounds`, `accounting-state` |
| Signature verification / instruction introspection | `signature-introspection` (all entries — small class, all High+) |
| Governance / voting | `access-control`, `dos-resource`, `cpi-external` |
| Oracles / VRF / CPI to other programs | `cpi-external` |
| Public `initialize` / `init_if_needed` present | `initialization` |
| Any instruction taking a list of accounts/keys | `dos-resource`, `input-bounds` |

Then: `code-quality`, `testing`, `observability`, `spec-mismatch` apply to *every* program — sweep them last; they are the plurality of findings.

## Grep the tells

Each catalog entry carries an "Anchor/Rust tell" — the code smell. Fast first pass:

```bash
# unchecked accounts without justification
grep -rn 'UncheckedAccount' programs/ | grep -v 'CHECK'
# mutation without mut / missing signer
grep -rn '#\[account(' programs/ | grep -v 'mut'
# raw arithmetic in value paths
grep -rnE '[^_](\+|-|\*|/)[^=]' programs/*/src | grep -v checked_
# unbounded growth
grep -rn '\.push(' programs/ ; grep -rn 'init_if_needed' programs/
# introspection without header checks
grep -rn 'load_instruction_at\|ed25519\|instructions sysvar' programs/
# custody token accounts without field checks
grep -rn 'TokenAccount\|Account<' programs/ | grep -v 'mint\|authority'
```

Every match is a *candidate*, not a finding — validate reachability and impact before writing it up.

## Writing findings

- Use the FYEO template from `audit-methodology.md` (ID `SEC-<PROJ>-NN`, severity, status, description, proof of issue, severity & impact, recommendation).
- Severity uses FYEO definitions: spec-claimed-but-absent security = **High**; arithmetic/overflow = **High**; uptime/DoS = **Medium**.
- Prioritize by severity × exposure (TVL, permissionless?, upgradeable?) per `vulnerability-abundance.md` §3 — report ordering matters more than count.
- Output goes to `docs/security-review.md` in the user's working repo, with a severity histogram at the top.
- **Never write findings into this companion repo.**

## Verify phase

After fixes: re-review only the diff, move `Open → Remediated`, keep `Acknowledged` with the team's stated rationale. Findings are findings — the human decides.

## Feeding back

If you find a bug class not in the catalog, add it: new `####` entry under the right `###` class (or propose a new class with justification), update Section A counts, and add a cross-cutting bullet if it recurs. The distribution is meant to drift as the corpus grows.
