# Audit readiness checklist — what to have before requesting an external audit

Derived from the Scope sections of 23 FYEO reports plus the recurring Informational findings that signal an unprepared codebase. Every box you skip becomes a finding or wasted billable review time.

## Scope & artifacts

- [ ] **Pinned commit hash** — the exact `git rev-parse HEAD` under review, written down. *Why:* every FYEO report binds scope to a hash; "the code changed since we looked" wastes the engagement.
- [ ] **Scoped file tree** — an explicit list of files/programs in scope (and what is out). *Why:* FYEO publishes this as Table 1; ambiguity here silently shrinks the review.
- [ ] **Written spec / design doc** — `docs/design.md` (from `/solana-architect`): numbered atomic requirements, actors, account map, PDA seeds, CPI plan. *Why:* FYEO runs "technical specification matching" — a doc claim not in code is scored **High**; no doc means they can't check the most productive class.
- [ ] **Threat model + actor/signer table** — who signs, who is trusted, what an attacker controls (PRINCIPLES.md Part B questions answered in writing). *Why:* access-control and account-validation are ~20% of all findings.
- [ ] **Known-issues list** — every TODO, half-implemented path, and deliberate deviation, disclosed up front. *Why:* FYEO-BIO-05 (TODO code in scope) and unannounced deviations burn review time.

## Code hygiene (the Informational-class eliminators)

- [ ] **No dead code / unused constants / unused error variants** — *Why:* dead code and unused constants appear in ~10 reports (FYEO-VOLTR-04, AX-SOL-12, BANGER-10, SF-03).
- [ ] **No manual account-size arithmetic** — `INIT_SPACE` or one sizing helper, with room for strings/Vecs. *Why:* FYEO-BANGER-12, FYEO-SAMO-08, FYEO-SF-08.
- [ ] **Custom errors on user-facing constraints** — *Why:* FYEO-SAMO-12.
- [ ] **Events on every privileged mutation, matching the real transition** — *Why:* FYEO-SPREE-03/05, FYEO-4CAST-07, FYEO-1INTRO-08, FYEO-GOV-ongoing-01.
- [ ] **`cargo clippy` clean** — *Why:* FYEO-SF-02 and general clarity findings in nearly every report.

## Tests & verification

- [ ] **Test suite covering every instruction, including negative cases per failure path** — plus a coverage statement. *Why:* missing tests are findings (FYEO-AX-SOL-09/14, VOLTR-05, BANGER-11); boundary/concurrent cases are specifically called out.
- [ ] **Fuzz / invariant plan if funds are at risk** — Trident (or equivalent) with shadow-state invariants, bounded parameter ranges, a master seed, and an honest "not tested" table. *Why:* see `audit-methodology.md` fuzzing section; arithmetic-critical programs get fuzzed.
- [ ] **Devnet deployment receipts** — program ID + transaction signatures. *Why:* proves the build deploys and the tested code is what runs; required by our own definition of done.

## Authority & operations

- [ ] **Documented admin keys + rotation path + upgrade-authority plan** — every privileged key listed, its rotation instruction or multisig/timelock plan, and who can pause vs resume. *Why:* FYEO-WAT-12, AMULET-ID-05/08, CRY-04, SAMO-03 — missing rotation/co-signer is a recurring class.
- [ ] **Config values bounded and justified** — fees, weights, caps, durations clamped at set-time. *Why:* input-bounds is ~11% of all findings.
- [ ] **Account lifecycle mapped** — every account type has a close path and rent-recovery story (or a documented reason not to). *Why:* FYEO-AMULET-ID-15, BANGER-14, SF-04.
- [ ] **Deployment plan** — program ID strategy, upgrade authority destination post-deploy (multisig), monitoring via emitted events. *Why:* upgrade-authority gating and observability findings assume this exists.
- [ ] **Remediation delivery plan** — how fix commits will be delivered (branch, hashes) within the agreed Verify window. *Why:* FYEO's Verify phase transitions findings to Remediated only against submitted remediation commits.
