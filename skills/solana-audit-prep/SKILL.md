---
name: solana-audit-prep
description: Prepare a Solana program for an external security audit — scope document, readiness checklist, known issues
argument-hint: "<program directory or repo root>"
allowed-tools:
  - read
  - edit
  - grep
  - exec
permissions:
  allow:
    - Read(**)
    - Write(**)
    - Exec(git *)
    - Exec(anchor *)
    - Exec(cargo *)
    - Exec(solana *)
---

You are preparing a Solana codebase for an external security audit. The goal: the auditor's Kickoff and Ramp-up phases start with complete artifacts, so billable review time goes to real findings — not to discovering your TODOs.

## Process

1. Load `knowledge/security/audit-readiness-checklist.md` and walk it item by item against the repo.
2. Gather the scope facts:
   - `git rev-parse HEAD` → the pinned commit hash
   - the scoped file tree (`programs/`, shared libs, client if in scope) — and what is explicitly **out** of scope
   - pointer to the spec: `docs/design.md` (run `/solana-architect` first if it doesn't exist — spec-matching is half the review)
   - the threat model + actor/signer table (PRINCIPLES.md Part B questions, answered in writing)
   - the known-issues list: every TODO, half-implemented path, deliberate deviation
3. Check the readiness items that are pass/fail: clippy clean, no dead code/unused constants, events on privileged mutations, negative-test coverage statement, admin keys + rotation + upgrade-authority plan, devnet receipts, fuzz/invariant plan if funds at risk.
4. Write the results to `docs/audit-scope.md` **in the user's working repo** (never in this companion repo).
5. Report readiness gaps as a checkbox list — each gap is a finding the auditor would otherwise write.

## `docs/audit-scope.md` template

```markdown
# Audit Scope — <project> — <date>

- **Commit:** <full hash>
- **Remediation delivery:** <branch/PR flow for the Verify window>
- **In scope:** <file tree>
- **Out of scope:** <files/deps excluded>
- **Spec:** docs/design.md (+ any supplementary docs)
- **Threat model:** <actor/signer table; trusted parties; attacker capabilities>
- **Admin keys & authorities:** <list + rotation path + upgrade authority plan>
- **Known issues:** <disclosed TODOs/deviations>
- **Test coverage:** <suite summary incl. negative cases; fuzz/invariant plan>
- **Deployment:** devnet program ID + tx signatures; mainnet plan
```

## Output

- `docs/audit-scope.md` in the user's working repo
- Readiness checklist result: pass/gap per item, with the fix for each gap
- Recommendation on whether the codebase is actually ready to hand over
