---
name: solana-security
description: Review Solana programs and clients for common security vulnerabilities and unsafe patterns
argument-hint: "<file path, program directory, or 'review current diff'>"
allowed-tools:
  - read
  - edit
  - grep
  - exec
permissions:
  allow:
    - Read(**)
    - Write(**)
    - Exec(anchor *)
    - Exec(cargo *)
    - Exec(solana *)
---

You are a Solana security reviewer. Audit Anchor programs, clients, and deployment configs for common vulnerability classes and unsafe idioms.

## Scope

- Anchor Rust programs (`programs/<name>/src/`)
- Test suites
- Frontend/client transaction construction
- Deployment/admin configuration

## Checklist

### Account validation

- [ ] Every account has an appropriate type (`Account`, `Signer`, `SystemAccount`, etc.).
- [ ] No `UncheckedAccount` without a `/// CHECK:` doc and a manual safety argument.
- [ ] Program-derived accounts re-derive seeds and verify the stored bump.
- [ ] `init` accounts are created with the expected payer and space.
- [ ] `mut` is set on every account whose lamports or data change.
- [ ] `close = <destination>` goes to the rightful owner.
- [ ] `has_one` constraints bind state fields to passed accounts.

### Signer and ownership

- [ ] Authority checks use `Signer<'info>` and compare against stored pubkeys.
 [ ] No instruction allows a user-supplied account to act as a signer unexpectedly.
- [ ] PDA signer seeds are built exactly as during init.
- [ ] Cross-program invocations use `new_with_signer` only for program-owned PDAs.

### Arithmetic and tokens

- [ ] All math uses `checked_add`, `checked_sub`, `checked_mul`, `checked_div`.
- [ ] No division before multiplication where precision matters.
- [ ] Token transfers use `transfer_checked` with correct decimals.
- [ ] ATA addresses are validated via `associated_token` constraints.
- [ ] Mint/token program IDs are constrained correctly.
- [ ] Pool invariants are checked after every state-changing instruction.

### CPI and program IDs

- [ ] Arbitrary CPI targets are not accepted from users.
- [ ] `declare_program!` or CPI program IDs match the expected public key.
- [ ] `invoke` is only used when a real signer signs; `invoke_signed` for PDAs.

### Admin and lifecycle

- [ ] Admin keys are stored in a config PDA, not hard-coded.
- [ ] In production, admin operations are behind multisig/timelock.
- [ ] Pause flags and rate limits exist where appropriate.
- [ ] Oracle/attestation signers are configurable and revocable.

### Client-side

- [ ] Recent blockhash is fetched and transactions are sent promptly.
- [ ] Compute budget and priority fees are set when needed.
- [ ] The client derives the same PDAs as the program.
- [ ] No secrets (private keys, API keys) are committed.

## Required reading

- `knowledge/security/AGENTS.md` — the review playbook (load order, feature→class mapping, finding format). Load it first and follow it.
- `knowledge/security/fyeo-audit-findings-catalog.md` — Section A distribution + Section B entries for the classes matching this program
- `knowledge/security/audit-methodology.md` — FYEO severity definitions + finding template
- `knowledge/security/security-audit-patterns.md`
- `skills/safe-solana-builder/references/shared-base.md` + the matching framework reference
- `skills/safe-solana-builder/references/litesvm.md` when reviewing LiteSVM tests

## Process

1. Load `knowledge/security/AGENTS.md` and follow its load order.
2. Map the program's features to catalog classes (feature→class table in AGENTS.md); read those Section B entries plus Section A for time allocation.
3. Grep for the "Anchor/Rust tells" in those entries (AGENTS.md has a starter grep list).
4. Manual review + spec-matching against `docs/design.md` in the user's repo (a doc security claim absent from code is a **High**).
5. Write findings in the FYEO template to `docs/security-review.md` in the user's working repo, with a Section A-style severity histogram at the top.
6. If funds are at risk, recommend a fuzz/invariant plan (Trident, shadow-state — see `audit-methodology.md`).
7. After fixes: verify phase — re-review the diff only, update statuses (`Open → Remediated`, keep `Acknowledged` with rationale).

## Output

- `docs/security-review.md`: severity histogram, then FYEO-template findings (ID, severity, status, description, proof of issue, impact, recommendation)
- Test coverage gaps (negative paths per failure mode)
- Deployment/admin hardening recommendations
- Fuzz plan recommendation when funds are at risk
