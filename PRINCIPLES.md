# PRINCIPLES.md

Behavioral contract for any coding agent using this repo. These apply to every task — homework, capstone, LOI, architecture, or production code. Read this before ENTRY.md.

**Tradeoff:** these principles bias toward caution and simplicity over speed. For trivial tasks, use judgment.

---

## Part A — How to think and code

Adapted from [Andrej Karpathy's observations on LLM coding pitfalls](https://x.com/karpathy/status/2015883857489522876) via [multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills) (MIT), and from Kun Chen's [/kun](https://github.com/kunchenguid/kun) task routing.

### 1. Think before coding
**Don't assume. Don't hide confusion. Surface tradeoffs.**
- State assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop, name what's confusing, ask.

### 2. Simplicity first
**Minimum code that solves the problem. Nothing speculative.**
- No features beyond what was asked. No abstractions for single-use code.
- No "flexibility"/"configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you wrote 200 lines and it could be 50, rewrite it.
- On Solana specifically: fewer accounts, fewer instructions, fewer PDAs. Every extra account is attack surface and compute. A vault is a state PDA + a token account — not a framework.

### 3. Surgical changes
**Touch only what you must. Clean up only your own mess.**
- Don't "improve" adjacent code, comments, or formatting. Match existing style.
- Don't refactor what isn't broken. If you notice unrelated dead code, mention it — don't delete it.
- Remove imports/variables your change orphaned; leave pre-existing dead code alone.
- Test: every changed line traces directly to the user's request.

### 4. Goal-driven execution
**Define success criteria. Loop until verified.**
- "Add validation" → "write tests for invalid inputs, then make them pass."
- "Fix the bug" → "write a test that reproduces it, then make it pass."
- "Refactor X" → "tests pass before and after."
- For multi-step work, state a plan: `1. [step] → verify: [check]`.
- On Solana: a program is not "done" until it has (a) a negative test per failure path, (b) a passing localnet/LiteSVM run, (c) a devnet program ID + tx signature you can paste into Explorer.

### 5. Task routing (from /kun)
Pick the sequence by ask type, then follow it:
- **Ideation / LOI / brainstorm** → research → planning (artifact + open questions) → iterate with the human
- **Feature / new program** → research → requirements → architecture → implementation → validation
- **Bug fix** → research → reproduction (failing test) → implementation → validation
- **Refactor** → research → guardrails (tests first) → implementation → validation
- **Explain something** → research → explainer (diagrams > prose)

**Validation** means an *adversarial* review pass with a fresh context (subagent or separate session) that checks the diff against the stated intent and the security checklist — not the implementer re-reading its own work. Findings are findings, not suggestions; the human decides.

### 6. Human-first, then AI red-team
For LOIs, requirements, and architecture: the human does the initial thinking and framing. AI is for red-teaming a draft the human already produced. Never paste the assignment into the agent and accept the first output. Treat every AI finding as something to evaluate and log an override decision for — not something to adopt blindly.

---

## Part B — Security & threat-model first (Solana)

Security is a design input, not a review step. Before writing any instruction, answer:

1. **Who are the actors and who signs?** Direct users, beneficiaries, admins, third-party/trusted parties (oracles, multisigs, custodians), stakeholders (treasury/DAO). Every signer explicit.
2. **What state changes, and who owns it?** For each state transition: exactly one owning instruction handler, one PDA (with visible seeds) or one external account. If you can't say it in one sentence, the design is tangled.
3. **What does an attacker control?** Every account passed in is attacker-controlled until validated. Every `UncheckedAccount` needs a written reason.
4. **What is the blast radius of each privileged key?** Admin keys live in config accounts, never hardcoded; recommend multisig/timelock. Permanent-delegate mints are a reason to *reject* a mint in permissionless DeFi.

### Non-negotiable checklist (every program, every PR)
- [ ] Owner and signer checks explicit on every account that matters
- [ ] PDAs: canonical bump stored and re-verified; seeds re-derived, not trusted from input
- [ ] Token transfers use `transfer_checked` with the right decimals; mint verified against stored state
- [ ] Arithmetic uses `checked_*` / `u128` mul-div; rounding direction chosen deliberately
- [ ] No arbitrary CPI to user-supplied program IDs
- [ ] `init` vs `init_if_needed` chosen deliberately; no frontrunnable initialization
- [ ] State machine: terminal states absorbing, transitions enumerated
- [ ] Negative test per failure path
- [ ] Devnet deploy verified (program ID + tx signatures) before any mainnet talk
- [ ] Never commit keys, wallet JSONs, `.env`, or API keys

### Recurring real-audit findings (from ~20 FYEO Solana audits — check these explicitly)
- **Initialization is first-come-first-served / re-initializable** → gate `initialize` to a known authority or upgrade authority; make config non-deletable or re-init-safe
- **Token account mint/owner/delegate not checked** → every token account: verify `mint`, `owner`, and that `delegate` is `None` where it must be
- **Authority transfer without co-signer** → new admin/owner must sign; add an on-chain rotation path for every privileged key
- **No bounds on config values** (fees, weights, limits) → clamp at set-time, not just at use-time; reject zero amounts
- **Accounting drift** (fees not accumulated, withdraw ignores ledger, harvest never called) → one ledger, one place it mutates; invariant tests
- **Unpinned external program IDs in CPI** → hardcode or store-and-verify; never accept from instruction accounts unchecked
- **Signature/instruction-introspection bugs** (unchecked Ed25519 header fields, no domain separation → cross-type replay) → validate every header field, bind message to program + type + recipient
- **Panics on fixed-size copies / unbounded `Vec` growth** → length-check before copy; cap collections against `max_len`
- **Legacy-only token program** → accept Token-2022 via `TokenInterface` unless there's a reason not to
- **Missing events on state changes; dead code; incomplete tests** → emit on every privileged mutation; delete dead code; tests are audit scope

Full catalog with citations: `knowledge/security/fyeo-audit-findings-catalog.md`. How to use it in a review: `knowledge/security/AGENTS.md`.

### Mindset: vulnerabilities are infinite, exploits are few
No checklist or audit is complete (`knowledge/security/vulnerability-abundance.md`). Therefore: (1) prefer constructs that *eliminate a class* over checks that catch an instance; (2) weight review time by where Solana programs empirically fail (the catalog's distribution); (3) prioritize findings by exploit likelihood × exposure (TVL, permissionless?, upgradeable?), not by count; (4) design for the flaw you didn't find — multisig/timelock upgrade authority, separate pause/resume, caps and floors, events for anomaly detection; (5) raise your own discovery rate: fuzz with shadow-state invariants, negative tests per failure path, spec-matching review.

Detailed rules: `knowledge/security/` and `skills/safe-solana-builder/`.

---

## Part C — Requirements before diagrams, diagrams before code

From the Turbin3 architecture-diagram methodology (`knowledge/process/architecture-diagramming.md`):
- Write **numbered, atomic** `"The protocol shall …"` requirements first. One action / one state change / one CPI each.
- Then actors + signers. Then PDA/state ownership. Then the diagram.
- **Bidirectional traceability**: every requirement → a diagram element and every element → a requirement. Unmapped = fail.
- Cut to MVP. A capstone is a ~2-week PoC on devnet, not a production protocol.
- The spec is a living draft. Expect week-1 changes.

---

## Part D — Stack defaults (override only with a reason)
- Anchor 0.31+; Rust `stable` unless `rust-toolchain.toml` says otherwise
- Token-2022 for new token features; `TokenInterface` to support both programs
- Metaplex Core for new NFTs
- `@solana/kit` for new clients; `@solana/web3.js` only for legacy
- Tests: LiteSVM (fast Rust), Surfpool (real state/time travel), devnet (final proof). Test after *each* instruction, not in a batch.
- Newer standards may lack wallet/explorer support — check real adoption before committing.

---

**These principles are working if:** diffs are small and traceable, programs ship with negative tests and devnet receipts, clarifying questions come *before* implementation, and requirements docs exist before any diagram or code.
