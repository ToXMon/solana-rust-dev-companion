---
title: "Leash — Capstone Deliverable 1: Letter of Intent & On-Chain Use Cases"
subtitle: "Turbin3 Builders Cohort Q3 2026"
author: "Tolulope Shekoni (solo)"
date: "September 2026"
---

# Leash

**On-chain spending mandates for AI agents on Solana.**

*Category: Payments (agentic / machine-to-machine settlement)*

---

## How to read this document

**Part 1 — Final Project Proposal & LOI** is the polished, post-red-team version: overview, value proposition and PMF, target markets and user profiles, competitor landscape, founder-market fit, actors, use cases (one atomic state transition per Anchor instruction), the granularity check, and the consolidated on-chain requirements matrix.

**Part 2 — Process Appendix (Red Team Log)** is the proof of work: the original Phase 1 draft preserved verbatim, every adversarial attack with an Accept / Partial / Reject verdict and the resulting change, and the stress test that killed the first candidate direction (a crypto lottery) before this one was chosen.

**Sources** closes the document with every URL and number used.

---

# PART 1 — FINAL PROJECT PROPOSAL & LOI


## High-Level Overview

Leash is an open, permissionless Solana program that puts a hard, on-chain spending mandate around any AI agent's money. A principal — a developer, a team, or eventually a treasury — deposits USDC into a Leash vault, binds it to a registered agent identity, and sets the rules: per-payment cap, rolling-window cap, lifetime cap, allowlisted payee wallets, expiry, and an optional escalation threshold that requires the principal's co-signature. The agent then pays x402/MPP services (or any Solana payee) with its own key, but every payment is a Leash instruction that the program rejects if it breaks the mandate. A prompt-injected, looping, or stolen agent can never spend past its leash. Leash is the "spending limit for agents" that today exists only inside closed APIs (Squads Grid) or inside SDKs that a compromised agent can simply bypass.

**Category:** Payments (agentic / machine-to-machine settlement). Directly aligned with BlackRock's *Machine-Native Economy* thesis (Sept 2026) that x402/MPP-style rails, stablecoins, and know-your-agent (KYA) controls are the settlement layer for agentic commerce.

## Core Value Proposition & Product-Market Fit

Autonomous agents are already losing real money in two ways: **runaway execution** (a Google Mandiant case study, Sept 2026: an accounting agent looped into 15,000 high-cost API calls and ~$50k of charges in under an hour; a LangChain retry loop ran 11 days and cost $47k) and **hijacked authorization** (the Grok/Bankr prompt-injection drain of ~$155k in May 2026, followed by a second ~$440k Bankr breach two weeks later). In every case the root cause was the same: the agent held *standing spending power* with the policy living in the same trust domain as the agent. Leash moves the policy into the program that holds the funds, where reasoning bugs and injected instructions cannot reach it. Principals get a provable ceiling on loss and a receipt trail; agent developers get a drop-in "pay within policy" primitive instead of hand-rolling one; x402 sellers and facilitators get a counterparty-verifiable mandate they can read on-chain before releasing a resource. The incumbent, Squads, proves the demand for on-chain spending limits but ships them as a permissioned, general-purpose smart-account API; Leash's wedge is **open + agent-native**: permissionless CPI, binding to Solana Agent Registry / MPL Agent Registry identities, x402 payment-intent hashes in receipts, and an escalation path modeled on AP2 mandates.

**Value drivers (post coverage check):**
1. **Bounded blast radius** — worst-case loss is arithmetic, not hope: `min(lifetime_cap, 2 × window_cap)`.
2. **Counterparty-verifiable** — any facilitator, seller, or protocol can read a mandate account or CPI `verify_mandate`; off-chain policies structurally cannot offer this.
3. **Auditability** — every spend emits a receipt event (payee, amount, agent id, intent hash, slot) tied to an on-chain agent identity.
4. **Composability / speed** — one deposit + one `set_mandate`, then a `payWithLeash()` client; no vendor account, no keeper.
5. **Escalation, not approval** — optional co-sign above a threshold keeps sub-cent autonomy while giving humans a tap on big spends.
6. *(v2, out of capstone scope)* mandate over a yield-bearing position to remove idle-capital cost; confidential mandate balances via Token-2022.

**Honest market sizing:** x402 is small today — roughly $56M cumulative settled across Base and Solana, ~$1.2M/month organic in Sept 2026, with heavy wash volume and ~78 organic sellers. Solana carries ~18% of dollar volume but ~74% of transaction count, i.e., the high-frequency, low-value agent traffic Leash is built for. The loss-bounding problem, however, exists for *every* agent that holds a key, on any rail; x402/MPP is the first rail Leash speaks, not the only one.

**Business model:** the program is free and open-source. Revenue paths: (1) optional protocol fee in basis points on `spend`, configurable per mandate (default 0 during capstone); (2) a hosted, read-only **Mandate Manager** (dashboard, alerts, receipt export) for teams — the same free-protocol / paid-API pattern Squads uses.

## Target Markets & User Profiles

**Segments (ranked by time-to-revenue)**
1. **Solo and small-team agent builders on Solana** running 24/7 agents against paid APIs (OpenClaw, Solana Agent Kit, MCP tool authors, Colosseum hackathon alumni). *Now.*
2. **x402 / MPP sellers and facilitators** who want fewer failed or disputed payments and a way to gate premium endpoints to mandated agents. *Now.*
3. **AI-product teams** shipping one agent to many end users, each needing an isolated, capped envelope and a receipt trail. *6–12 months.*
4. **Enterprise ops / finance pilots** of agentic procurement or data purchasing that require hard caps, escalation, and reconciliation exports. *12–24 months; deliberately demoted after red-team.*

Agent frameworks and wallet platforms (SendAI, Crossmint, OpenClaw) are treated as **distribution channels**, not customers.

**Top user profiles**
- **Priya, indie agent developer.** Runs three research/trading agents on Solana from a $2k hot wallet and knows one bad prompt could zero it. Wants: "give it $50/day, allowlist these four payees, expire Friday, done."
- **Marcus, CTO of a small AI product.** Ships a research agent to ~2,000 users; needs each user's agent to have its own capped envelope, receipts for support/disputes, and no custody of user funds.
- **Lena, x402 API seller.** Runs a paid data endpoint; loses revenue to failed settlements and abusive loops. Wants to require a valid Leash mandate (and read remaining budget) before serving a premium route.

**Coverage check against ecosystem demand:** Colosseum's Agent Hackathon (Feb 2026) drew 454 submissions, with AI + infrastructure the top categories; three independent teams built devnet "PDA vault + policy + x402" projects (x402Guard, Sage, agent_fuel) in a single hackathon window — validation that builders feel this pain and that the primitive is buildable in capstone scope.

## Competitor Landscape (post coverage check)

**Bucket 1 — On-chain custody with limits (direct incumbents)**

| Player | What they do | Gap Leash exploits |
|---|---|---|
| **Squads Protocol / Grid** | Formally verified smart account; ONE_TIME/DAILY/WEEKLY/MONTHLY limits, destination allowlists; mainnet; Crossmint integrated | Permissioned early-partner API; general-purpose, not agent-native; no identity binding, no x402 intent/receipt semantics, not a permissionless CPI target |
| **Crossmint agent wallets** | Non-custodial delegation (limit, counterparties, time window); cards via Visa VIC | Vendor-locked; policy semantics owned by Crossmint; built on Squads underneath |
| **x402Guard, Sage, agent_fuel** | Devnet Anchor vaults with caps/allowlists for x402 | Single-dev hackathon maturity; no identity, receipts, escalation, or open CPI; validate demand |

**Bucket 2 — Off-chain policy or signing (the "good-enough" threat)**

| Player | What they do | Gap Leash exploits |
|---|---|---|
| **Turnkey / Privy** | Audited policy-enforced signing infrastructure | Off-chain, vendor dependency, not verifiable by the counterparty; defeated when the authorization path itself is compromised (Bankr class) |
| **@otomat/agent-wallet-kit, t402 Agent Policy Engine, v402pay** | Declarative budgets/whitelists wrapping `fetch()` / 402 flows | Enforced in the SDK the agent controls; bypassable |
| **Skyfire** ($9.5M; Coinbase Ventures, a16z CSX) | KYA + wallet + USDC payments network | Custodial, closed network, Base-first |
| **Locus** (YC F25) | Prepaid credits across 48 APIs; Visa BYOC | Off-chain billing layer; not Solana |
| **Payman** | Agents paying humans | Adjacent problem |

**Bucket 3 — Complements Leash integrates with, not competitors**

| Player | Role for Leash |
|---|---|
| **Solana Agent Registry** (ERC-8004 port, live Mar 2026) and **MPL Agent Registry** (Metaplex, mainnet) | Source of the agent identity a mandate binds to; reputation feedback target for receipts |
| **Coinbase x402 facilitator (~52% share), PayAI, solana-mpp** | Settlement rails whose 402 challenges supply the payee pubkey + intent hash |
| **Laso Finance, Swish** | Private payment rails; potential v2 pairing for confidential mandates |

**PMF adjustment after coverage check:** Leash does not compete on custody security or on convenience alone. It competes on being the **open, counterparty-verifiable, agent-identity-bound** policy layer — the thing every bucket above lacks and the thing the ecosystem's own standards (Agent Registry, did:aip, AP2 mandates) are converging toward.

## Founder-Market Fit (refined)

**Background.** Nine years in operational excellence and continuous improvement inside large enterprises — a career spent designing limits, allowlists, escalation paths, and audit trails around processes that people wanted to run unsupervised. That is precisely what Leash is: a control system for a new kind of worker. Data-science training; full-stack AI development; Rust/Anchor through Turbin3 (vault, escrow, AMM, Token-2022 extensions, Metaplex Core, confidential transfers).

**Motivation.** I run fleets of coding agents every day and refuse to be their circuit breaker. I am the first user, and — per red-team — I will validate beyond myself with at least five external builder interviews (Colosseum / SendAI communities) before the architecture diagram is finalized.

**Network.** Turbin3 cohort and instructors; Colosseum and Solana agent-builder communities; ideabrowser founder network.

**Known weaknesses and mitigations (from red-team).**
- *No prior fintech/security shipping record; solo.* → Minimal attack surface (a dozen small single-purpose instructions, USDC only, no protocol admin in the spend path, upgrade authority burned after audit); LiteSVM negative tests for every guard; Solana Audit Arena / Trail-of-Bits-style review before any mainnet; capstone target is audited open-source on devnet, not a custodial service.
- *Evenings/weekends time budget.* → Architecture has **no keeper and no off-chain service in the critical path**; the only hosted component is optional and read-only.
- *Self-as-user bias.* → External interview commitment above; GTM leads with third-party incident narratives (Bankr, Mandiant), not my own anecdotes.


Rule applied throughout: **one use case = one atomic state transition = one Anchor instruction handler.**

---

## Part A — Actors

| Class | Actor | Role | Signs Leash instructions? |
|---|---|---|---|
| **Direct** | **Principal** | Human or treasury key that funds a Leash and owns its policy | Yes — every configuration, funding, escalation-approval, and withdrawal instruction |
| **Direct** | **Agent** | The autonomous process; holds `agent_key`, a hot key with *no* authority except `spend` and `request_escalation` inside the mandate | Yes — `spend`, `request_escalation` |
| **Direct** | **Anyone (permissionless)** | Any key that closes an expired escalation to reclaim rent | Yes — `cancel_escalation` after expiry only |
| **Beneficiary** | **Payee / x402 seller** | Receives USDC in its ATA when `spend` succeeds; never signs a Leash instruction | No |
| **Beneficiary** | **End users of an agent product** (Marcus's customers) | Their funds are protected by the mandate; they interact only with the product's UI | No |
| **Administrator** | **Protocol upgrade authority** (temporary) | Deploys/upgrades the program until authority is burned post-audit; *never* in the spend path | Not for any user-facing instruction |
| **Administrator** | **Protocol config authority** (optional, capstone default: none) | Sets protocol fee bps and fee recipient if a fee is ever enabled | Only `init_protocol_config` / `update_protocol_config` |
| **Stakeholder** | **x402/MPP facilitators** (Coinbase, PayAI, solana-mpp) | Read mandate accounts / receipts to gate settlement or reduce disputes | No |
| **Stakeholder** | **Agent identity registries** (Solana Agent Registry, MPL Agent Registry) | Supply the `agent_identity` a Leash binds to; may consume receipts as reputation feedback | No |
| **Stakeholder** | **Auditors / security reviewers** | Verify invariants before mainnet | No |

---

## Part B — Use Cases

Template fields: **Actor (signer)** · **Precondition** · **Inputs** · **Accounts** (R = read, W = write, I = init, C = close) · **State transition** · **Postcondition / invariant** · **Failure modes** · **On-chain vs client-side**.

### UC-01 `create_leash`
- **Actor:** Principal
- **Precondition:** Leash PDA for `(principal, agent_id)` does not exist.
- **Inputs:** `agent_id: u64`, `agent_key: Pubkey`, `agent_identity: Option<Pubkey>` (Agent Registry record)
- **Accounts:** `principal` (signer, W payer) · `leash` (I) · `usdc_mint` (R) · `vault_ata` (I, ATA of `leash`) · token, ATA, system programs (R)
- **State transition:** Leash account created with `status = Active`, empty mandate (all caps = 0 → nothing spendable), counters zeroed, canonical bump stored; vault ATA created with `leash` as authority.
- **Postcondition:** No spend is possible until `set_mandate` runs (default-deny).
- **Failure modes:** duplicate `(principal, agent_id)`; wrong mint (must be the configured USDC mint).
- **Split:** on-chain — everything above. Client — generating `agent_key`, looking up `agent_identity`.

### UC-02 `set_mandate`
- **Actor:** Principal
- **Precondition:** Leash exists; `has_one = principal`.
- **Inputs:** `per_tx_cap`, `window_cap`, `window_seconds`, `lifetime_cap`, `expires_at: i64`, `escalation_threshold` (0 = disabled ⇒ set equal to `per_tx_cap`), `fee_bps` (must be ≤ protocol max)
- **Accounts:** `principal` (signer) · `leash` (W)
- **State transition:** Mandate fields overwritten. Counters (`spent_in_window`, `spent_lifetime`) are **not** reset (prevents "reset-by-edit" loophole).
- **Postcondition:** `per_tx_cap ≤ window_cap ≤ lifetime_cap`; `escalation_threshold ≤ per_tx_cap`; `expires_at > now`.
- **Failure modes:** inverted caps; expiry in the past; `window_seconds == 0`.
- **Split:** on-chain validation of arithmetic relations. Client — human-readable policy editor.

### UC-03 `deposit`
- **Actor:** Principal (or any funder)
- **Precondition:** Leash exists and is not `Closed`.
- **Inputs:** `amount: u64`
- **Accounts:** `funder` (signer) · `funder_ata` (W) · `leash` (R) · `vault_ata` (W) · `usdc_mint` (R) · token program (R)
- **State transition:** `transfer_checked(funder_ata → vault_ata, amount)`.
- **Postcondition:** vault balance increased by `amount`; mandate unchanged (funding ≠ authorization).
- **Failure modes:** `amount == 0`; insufficient funder balance; mint mismatch.
- **Split:** entirely on-chain.

### UC-04 `add_payee`
- **Actor:** Principal
- **Precondition:** Leash exists; Payee PDA for `(leash, payee)` does not exist.
- **Inputs:** `payee: Pubkey` (the `pay-to` wallet advertised in the 402 challenge), `label_hash: [u8;32]` (optional hash of a human label / domain, for client display only)
- **Accounts:** `principal` (signer, payer) · `leash` (R, `has_one = principal`) · `payee_entry` (I) · system program (R)
- **State transition:** Payee PDA created; **existence = allowed**.
- **Postcondition:** `spend` to this payee's ATA can now pass the allowlist check.
- **Failure modes:** duplicate payee; `payee == leash` (self-pay) rejected.
- **Split:** on-chain — pubkey allowlist. **Client — domain→pubkey resolution.** The program never sees a URL (red-team A4).

### UC-05 `remove_payee`
- **Actor:** Principal
- **Precondition:** Payee PDA exists.
- **Inputs:** none beyond accounts
- **Accounts:** `principal` (signer, rent recipient) · `leash` (R) · `payee_entry` (C)
- **State transition:** Payee PDA closed; rent returned.
- **Postcondition:** future `spend` to that payee fails the allowlist check.
- **Split:** on-chain.

### UC-06 `spend`  ← the core instruction
- **Actor:** Agent (`agent_key` signer)
- **Precondition:** `leash.status == Active`; `now < expires_at`; Payee PDA exists for `payee`; `amount ≤ per_tx_cap`; `amount ≤ escalation_threshold`; window and lifetime math below passes.
- **Inputs:** `amount: u64`, `intent_hash: [u8;32]` (SHA-256 of the canonical 402 challenge; opaque to the program)
- **Accounts:** `agent` (signer, must equal `leash.agent_key`) · `leash` (W) · `payee_entry` (R, seeds `["payee", leash, payee]`) · `vault_ata` (W) · `payee_ata` (W, ATA of `payee` for USDC; `init_if_needed` payer = agent's SOL) · `usdc_mint` (R) · optional `fee_ata` (W) · token + ATA programs (R) · `clock` sysvar (R)
- **State transition:**
  1. If `now ≥ window_start + window_seconds` → `window_start = now`, `spent_in_window = 0`.
  2. `require!(spent_in_window + amount ≤ window_cap)`; `require!(spent_lifetime + amount ≤ lifetime_cap)` (checked math).
  3. `transfer_checked(vault_ata → payee_ata, amount − fee)` signed by `leash` PDA; if `fee_bps > 0`, `transfer_checked(vault_ata → fee_ata, fee)`.
  4. `spent_in_window += amount`; `spent_lifetime += amount`; `nonce += 1`.
  5. `emit_cpi!(Receipt { leash, agent_identity, payee, amount, intent_hash, nonce, slot })`.
- **Postcondition (invariants):** `spent_in_window ≤ window_cap`; `spent_lifetime ≤ lifetime_cap`; vault balance decreased by exactly `amount`; worst-case loss bound `min(lifetime_cap, 2 × window_cap)` holds across any sequence of `spend`s.
- **Failure modes:** wrong signer; paused/expired; payee not allowlisted; any cap exceeded; overflow; insufficient vault balance; `amount == 0`.
- **Split:** on-chain — all enforcement and transfer. **Client / SDK** — receiving the 402 challenge, hashing it to `intent_hash`, retrying with the payment proof, deciding *whether* to pay. **Indexer** — turning `Receipt` events into dashboards and reputation feedback.

### UC-07 `request_escalation`
- **Actor:** Agent
- **Precondition:** `escalation_threshold < amount ≤ lifetime_cap − spent_lifetime`; Payee PDA exists; no Escalation PDA for this `intent_hash`.
- **Inputs:** `amount`, `intent_hash`, `ttl_seconds`
- **Accounts:** `agent` (signer, payer) · `leash` (R) · `payee_entry` (R) · `escalation` (I, seeds `["escalation", leash, intent_hash]`) · system program
- **State transition:** Escalation PDA created with `{payee, amount, intent_hash, expires_at = now + ttl}`. **No funds move.**
- **Postcondition:** principal can approve or cancel; agent cannot bypass.
- **Failure modes:** amount within normal cap (should use `spend`); duplicate intent; ttl too long.
- **Split:** on-chain record. Client — notification to principal (Mandate Manager / webhook).

### UC-08 `approve_escalation`
- **Actor:** Principal
- **Precondition:** Escalation exists and `now < escalation.expires_at`; leash `Active`.
- **Accounts:** `principal` (signer, rent recipient) · `leash` (W) · `escalation` (C) · `payee_entry` (R) · `vault_ata` (W) · `payee_ata` (W) · `usdc_mint` (R) · token program
- **State transition:** `transfer_checked(vault_ata → payee_ata, amount)`; `spent_lifetime += amount` (window counter **not** touched — escalations are explicitly human-approved); emit `Receipt{ escalated: true }`; close Escalation PDA.
- **Postcondition:** lifetime invariant preserved; escalation cannot be replayed (PDA closed).
- **Failure modes:** expired escalation; insufficient vault balance; lifetime cap exceeded.
- **Split:** on-chain.

### UC-09 `cancel_escalation`
- **Actor:** Principal (any time) **or** Anyone (only after `expires_at`)
- **Accounts:** `signer` · `leash` (R) · `escalation` (C, rent → agent who paid for it)
- **State transition:** Escalation PDA closed; no funds move.
- **Failure modes:** non-principal before expiry.
- **Split:** on-chain.

### UC-10 `pause`
- **Actor:** Principal
- **Precondition:** `status == Active`
- **Accounts:** `principal` (signer) · `leash` (W)
- **State transition:** `status = Paused`. Kill switch; `spend` and `approve_escalation` now fail.
- **Split:** on-chain.

### UC-11 `resume`
- **Actor:** Principal
- **Precondition:** `status == Paused`
- **State transition:** `status = Active`. Counters unchanged.
- **Split:** on-chain.

### UC-12 `rotate_agent_key`
- **Actor:** Principal
- **Inputs:** `new_agent_key: Pubkey`
- **Accounts:** `principal` (signer) · `leash` (W)
- **State transition:** `agent_key = new_agent_key`. Immediate revocation of the old hot key without moving funds.
- **Failure modes:** `new_agent_key == principal` (principal should not be the agent).
- **Split:** on-chain.

### UC-13 `withdraw`
- **Actor:** Principal
- **Inputs:** `amount`
- **Accounts:** `principal` (signer) · `principal_ata` (W) · `leash` (R) · `vault_ata` (W) · `usdc_mint` · token program
- **State transition:** `transfer_checked(vault_ata → principal_ata, amount)` signed by `leash` PDA.
- **Postcondition:** mandate counters unchanged (withdrawal is not a spend).
- **Failure modes:** `amount == 0`; insufficient balance.
- **Split:** on-chain.

### UC-14 `close_leash`
- **Actor:** Principal
- **Precondition:** `vault_ata.amount == 0`; no open Escalation PDAs (client must cancel first); Payee PDAs may remain but are orphaned — client should `remove_payee` first to reclaim rent.
- **Accounts:** `principal` (signer, rent recipient) · `leash` (C) · `vault_ata` (C via `close_account` CPI) · token program
- **State transition:** vault ATA closed, Leash PDA closed, rent returned.
- **Failure modes:** non-zero balance.
- **Split:** on-chain.

### UC-00 `init_protocol_config` *(optional; capstone default is "no fee, no config")*
- **Actor:** Protocol config authority
- **Inputs:** `max_fee_bps`, `fee_recipient`
- **Accounts:** `authority` (signer) · `protocol_config` (I, seeds `["config"]`)
- **State transition:** single global config created. Never read in `spend` unless `fee_bps > 0`.

---

## Part C — Adversarial Analysis & Granularity Check

| Rule | Check | Result / fix |
|---|---|---|
| **Atomicity** — one handler per use case | 15 use cases → 15 handlers. `spend` does two transfers (payee + optional fee) but it is one state transition ("a mandated payment"); splitting it would let a fee transfer succeed without the payment. | Pass. |
| **State ownership** — every state change in a program-owned account or explicit indexer | Mandate, counters, allowlist, escalations, status: program-owned PDAs. Receipts: `emit_cpi!` events consumed by an **explicit indexer** (Mandate Manager / Helius webhooks) — not program state, by design (rent). Domain↔pubkey mapping: **explicit client-side**, never trusted on-chain. | Pass. |
| **Real signers** — no hidden backend | Principal and Agent are the only signers of value-moving instructions; both are keys the user controls. No keeper, no facilitator signature is required by Leash. The optional Mandate Manager is read-only. | Pass. |
| **On-chain vs client-side** | On-chain: caps, windows, allowlist by pubkey, escalation state machine, transfers, receipts. Client: 402 challenge handling, `intent_hash` computation, domain resolution, notifications, dashboards, reputation aggregation. | Pass — documented per use case. |

**Red-team notes on the state machine (accepted fixes already applied above):**
- *"Edit the mandate to reset the counters."* → `set_mandate` never touches counters (UC-02).
- *"Approve an escalation twice."* → Escalation PDA is closed on approval (UC-08); PDA seed includes `intent_hash` so the same intent cannot be re-requested while open.
- *"Agent creates thousands of escalations to grief the principal's rent."* → Agent pays escalation rent; anyone can close after expiry and return it (UC-07/09). Client-side: Mandate Manager rate-limits notifications.
- *"Pause after the agent has already signed."* → Transactions are atomic; a paused leash rejects `spend` in the same slot forward. Bounded loss holds.
- *"Payee ATA doesn't exist; agent's SOL pays rent."* → Accepted trade-off; document that agents need a small SOL balance for fees/rent, or principal pre-creates payee ATAs in `add_payee` (v2 option).
- *"Facilitator can't tell a Leash payment from a plain transfer."* → `Receipt` event carries `leash`, `agent_identity`, `intent_hash`; the transfer's signer is the Leash PDA, which is itself the proof of mandate. Optional `verify_mandate` read-only CPI for the architecture-diagram challenge.

---

## Part D — Consolidated On-Chain Requirements Matrix

### Accounts & PDAs

| Account | Seeds | Owner / authority | Key fields | Closeable |
|---|---|---|---|---|
| `Leash` | `["leash", principal, agent_id.to_le_bytes()]` | Program | `principal`, `agent_key`, `agent_identity: Option<Pubkey>`, `usdc_mint`, `status: {Active, Paused}`, `per_tx_cap`, `window_cap`, `window_seconds`, `window_start`, `spent_in_window`, `lifetime_cap`, `spent_lifetime`, `expires_at`, `escalation_threshold`, `fee_bps`, `nonce`, `bump` | Yes (UC-14) |
| `vault_ata` | ATA(`leash`, `usdc_mint`) | Token program; authority = `leash` PDA | USDC balance | Yes (UC-14) |
| `PayeeEntry` | `["payee", leash, payee]` | Program | `payee`, `label_hash`, `bump` | Yes (UC-05) |
| `Escalation` | `["escalation", leash, intent_hash]` | Program | `payee`, `amount`, `intent_hash`, `requested_by`, `expires_at`, `bump` | Yes (UC-08/09) |
| `ProtocolConfig` (optional) | `["config"]` | Program | `authority`, `max_fee_bps`, `fee_recipient` | No |

### Instruction handlers (15)

`init_protocol_config`* · `create_leash` · `set_mandate` · `deposit` · `add_payee` · `remove_payee` · `spend` · `request_escalation` · `approve_escalation` · `cancel_escalation` · `pause` · `resume` · `rotate_agent_key` · `withdraw` · `close_leash`  (*optional)

### CPI dependencies

| Program | Used in | Purpose |
|---|---|---|
| SPL Token / Token-2022 via `TokenInterface` | `deposit`, `spend`, `approve_escalation`, `withdraw`, `close_leash` | `transfer_checked`, `close_account` |
| Associated Token Program | `create_leash`, `spend` (`init_if_needed` payee ATA) | ATA creation |
| System Program | all `init` / `close` | account creation, rent |
| Clock sysvar | `spend`, `request_escalation`, `approve_escalation`, `cancel_escalation` | window + expiry math |
| Solana Agent Registry / MPL Agent Registry (read-only, optional) | `create_leash` | validate `agent_identity` exists (stretch goal) |

### Custom constraints & invariants

- `has_one = principal` on every principal-only handler; `constraint = leash.agent_key == agent.key()` on agent handlers.
- Stored canonical bumps re-verified via `seeds`/`bump` on every access.
- All arithmetic `checked_*`; caps ordered `per_tx ≤ window ≤ lifetime`; `escalation_threshold ≤ per_tx_cap`.
- `set_mandate` cannot reduce counters; `withdraw` cannot alter counters.
- `status == Active` required for `spend` and `approve_escalation`.
- USDC mint pinned at `create_leash`; every token account constrained `token::mint = leash.usdc_mint`.
- No admin key in any value-moving path; `ProtocolConfig` is read only when `fee_bps > 0`.
- Events: `Receipt`, `MandateUpdated`, `EscalationRequested`, `EscalationResolved`, `StatusChanged`.

### Test obligations (feed into build phase)

Positive lifecycle: create → set_mandate → deposit → add_payee → spend ×N → withdraw → close.
Negative (one LiteSVM test each): wrong agent signer; paused; expired; non-allowlisted payee; per-tx cap; window cap; window rollover; lifetime cap; overflow; zero amount; counter-reset-by-edit; double approve; cancel-before-expiry by stranger; close with balance; mint mismatch.

---

# PART 2 — PROCESS APPENDIX (RED TEAM LOG)

This appendix preserves the original drafts and the adversarial critique that produced Part 1. Nothing here has been edited after the fact; refinements live only in Part 1.

## Appendix A — Original Phase 1 Draft (v1, pre-red-team)


> Status: ORIGINAL DRAFT. Preserved verbatim for the Process Appendix. Do not edit; refinements go in `03-phase1-refined.md`.

## High-Level Overview

Leash is an open, permissionless Solana program that lets a human (or a company treasury) fund an AI agent with USDC and bind that money to an on-chain spending mandate: per-payment cap, rolling daily cap, an allowlist of payee endpoints, an expiry, and a threshold above which the human must co-sign. The agent pays x402 / MPP services with its own key, but every payment is a program instruction that fails if it violates the mandate — so a prompt-injected or looping agent can never spend more than the mandate allows. Leash is the "spending limit for agents" that today only exists inside closed APIs (Squads Grid) or in SDKs the agent itself can bypass.

## Core Value Proposition & PMF

Every team shipping autonomous agents on Solana faces the same fork: give the agent a hot wallet (and pray), or bolt a spend-policy into the SDK (which a compromised or buggy agent simply ignores). Leash moves the policy into the program that holds the funds, where it cannot be ignored. Principals get a hard, verifiable ceiling on loss; agent developers get a drop-in "pay within policy" primitive instead of rolling their own; x402 sellers/facilitators get a receipt they can verify on-chain. The value drivers are (1) bounded blast radius for agent compromise, (2) auditability — every spend is an on-chain event tied to an agent identity, (3) composability — any facilitator or protocol can check a Leash mandate without integrating a vendor API, and (4) developer speed.

**Coverage check (self-assessed):** possible missing drivers — cost-of-capital (idle USDC in a vault earns nothing), UX for non-technical principals, and multi-agent/team hierarchies (an org funding many agents).

## Target Markets & User Profiles

**Segments**
1. Solo/indie agent builders on Solana (OpenClaw, Solana Agent Kit, MCP tool authors) who run agents 24/7 against paid APIs.
2. Small AI-product teams that expose or consume x402/MPP endpoints and need a budget envelope per customer agent.
3. Enterprise ops / finance teams piloting agentic procurement or data purchasing who need audit trails and hard caps.
4. Agent platforms/frameworks (SendAI, Crossmint-style embedded wallets) who want an open on-chain policy primitive instead of building one.

**User profiles**
- **Priya, indie agent dev**: runs 3 trading/research agents on Solana; lost $400 to a retry loop last month; wants "give it $50/day, allowlist these 4 APIs, done."
- **Marcus, AI-product CTO**: ships a research agent to 2,000 customers; needs each customer's agent to have its own capped envelope and a receipt trail for support/disputes.
- **Dana, enterprise ops lead**: piloting an agent that buys market data; finance will not approve a hot wallet; needs a co-sign threshold and monthly export for reconciliation.

## Competitor Landscape (initial list)

| Player | What they do | Enforced where | Gap Leash exploits |
|---|---|---|---|
| Squads Grid spending limits | Formally verified smart account with daily/weekly/monthly limits, destination allowlist | On-chain | Permissioned API access; not x402-aware; no agent-identity binding |
| Crossmint agent wallets | Non-custodial delegation with spend limit, counterparties, time window; cards via Visa VIC | Mixed (relies on Squads underneath) | Vendor-locked, closed |
| x402Guard (AceDataCloud) | Solana PDA vault with daily/per-call caps + endpoint allowlist, MCP URL | On-chain (devnet) | Single-vendor, hackathon maturity, no identity/receipts |
| Sage (ogazboiz) | Per-task USDC cap; owner approves task, agent key releases steps | On-chain (devnet) | Task-scoped only; no rolling windows, no allowlist |
| agent_fuel | Vault + reputation for x402 agents | On-chain (devnet) | Hackathon maturity |
| @otomat/agent-wallet-kit, t402 Agent Policy Engine | Declarative spend policy wrapping fetch() | Off-chain (SDK) | Bypassable by compromised agent |
| Skyfire ($9.5M, Coinbase Ventures) | KYA + wallet + USDC payments network | Off-chain custodial | Base-first, closed network |
| Locus (YC F25) | Prepaid credits for agents across 48 APIs | Off-chain | Not on-chain, not Solana |
| Payman | Agents paying humans | Off-chain | Different problem |

## Founder-Market Fit

Solo founder. 9 years in operational excellence / continuous improvement in large enterprises — I have spent a career putting controls, limits, and audit trails around processes that people wanted to run unsupervised. I build and operate fleets of coding agents daily, so I am my own first user and have felt the runaway-spend problem directly. Technical stack: full-stack AI development, data science, Rust/Anchor via Turbin3 (vault, escrow, AMM, Token-2022, Metaplex Core, confidential transfers). Motivation: I want agents to run my businesses without me being the circuit breaker. Network: Turbin3 cohort, Solana agent-builder communities (Colosseum, SendAI). Weaknesses: no prior fintech/compliance shipping experience; solo; evenings/weekends time budget.

## Appendix B — Red Team Log


Method: adversarial critique run against `01-phase1-draft-v1.md`. Each attack is scored **Accept / Partial / Reject** with the reasoning and the concrete change made in `03-phase1-refined.md`. Evidence links are to sources gathered during research (see `06-sources.md`).

---

## A. Attacks on Value Proposition & PMF

### A1. "You're selling a seatbelt for a car nobody drives yet."
**Attack:** x402 is tiny. Sept-2026 organic volume is ~$1.2M/month with only 78 organic sellers; 54% of the 133k listed x402 endpoints are unreachable; top-10 wallets hold 81% of reported volume. BlackRock's own paper calls agentic payment activity "nascent." A spending cap on a market this small has no customers.
**Verdict: ACCEPT (reframe).**
**Why:** The numbers are real and I will not pretend otherwise. But the pain Leash addresses is *not* x402-specific — it is "an autonomous process holds standing spending power." That pain is already producing five- and six-figure losses off-chain (Mandiant/Google case: agent loop → 15,000 API calls → ~$50k in <1 hour, Sept 2026; LangChain retry loop → $47k over 11 days) and on-chain (Grok/Bankr prompt-injection drain, ~$155k, May 2026; second Bankr breach ~$440k, May 19–20 2026).
**Change:** Lead the value prop with the loss-bounding problem, position x402/MPP as the *first* rail Leash speaks, not the only one. Add "any SPL/Token-2022 payee" as the general case.

### A2. "Squads Grid already ships formally verified spending limits on mainnet. You lose on day one."
**Attack:** Squads Smart Account Program has ONE_TIME/DAILY/WEEKLY/MONTHLY limits, destination allowlists, signer sets, on mainnet, formally verified, with Crossmint already integrated.
**Verdict: PARTIAL.**
**Why:** True, and Squads is the credible incumbent. But (a) Grid API is permissioned early-partner access, (b) it is a general smart-account product, not agent-native — no binding to an agent identity, no x402/MPP payment-intent awareness, no receipt emission designed for facilitators, and (c) it is not a permissionless CPI target a hackathon builder can compose against tonight. Leash's defensibility is *openness + agent-native semantics*, not cryptographic novelty.
**Change:** Name Squads explicitly as the incumbent in the value prop; state the wedge as "open, permissionless, agent-identity-bound mandate that any program or facilitator can verify via CPI/account read." Add a stated fallback: if Squads opens fully, Leash becomes a policy layer that can *delegate custody to* a Squads account — the mandate semantics remain the product.

### A3. "Off-chain policy SDKs (otomat, t402, Turnkey, Privy) are good enough for 95% of developers."
**Attack:** Most devs will wrap `fetch()` with a budget policy and move on. Turnkey/Privy signing policies are mature, audited, and chain-agnostic.
**Verdict: PARTIAL.**
**Why:** For a well-behaved agent, yes. The Grok/Bankr incident is the counter-case: the *authorization path itself* was compromised — the agent emitted a valid signature for an attacker's instruction. Any policy that lives in the same trust domain as the agent's reasoning or signing service is defeated by that class of attack. Turnkey/Privy policies are strong but are (a) off-chain, (b) a vendor dependency, (c) not verifiable by the counterparty. That said, the attack is right that *convenience wins by default*.
**Change:** Add Turnkey/Privy to the competitor table as the real "good-enough" threat. Make the GTM claim explicit: Leash must be *as easy* as an SDK wrapper (one `deposit + set_mandate`, then a drop-in `payWithLeash()` client). Add "counterparty-verifiable" as a value driver that off-chain policies structurally cannot offer.

### A4. "You cannot allowlist an *endpoint* on-chain. x402 pays a pubkey, not a URL."
**Attack:** The draft says "allowlist of payee endpoints." A Solana program cannot see a domain name. Either you trust an off-chain resolver (back to square one) or the feature is fake.
**Verdict: ACCEPT.**
**Why:** Correct and important — this is a Phase-2 granularity error caught early.
**Change:** The mandate allowlists **payee pubkeys** (the `pay-to` wallet advertised in the 402 challenge). Optionally the mandate can require that each spend carry a **payment-intent hash** (SHA-256 of the canonical 402 challenge) recorded in the receipt, so an indexer/facilitator can later prove which URL the payment was for. Domain→pubkey mapping is explicitly **client-side**; the program enforces only what it can verify. Documented in the On-Chain vs Client-Side split.

### A5. "Rolling 24h windows are gameable around the boundary."
**Attack:** An agent can spend the full daily cap at 23:59 and again at 00:01.
**Verdict: ACCEPT (minor).**
**Why:** True of every daily-limit product including bank cards. Bounded loss is still 2× daily cap, which is the design intent.
**Change:** Specify window semantics precisely in Phase 2: `window_start` + `window_seconds` bucket stored on the mandate; reset on first spend after expiry of the bucket. Optionally offer a **per-mandate lifetime cap** so the worst case is bounded regardless of windows.

### A6. "Human co-sign above a threshold destroys the autonomy that makes agents valuable."
**Attack:** If the human must approve, why have an agent?
**Verdict: PARTIAL.**
**Why:** For sub-cent API calls the threshold never triggers. For a $500 compute purchase, a human tap is exactly what AP2-style mandates (Google) and Visa TAP are converging on. It is a feature enterprises will demand.
**Change:** Keep it, but make it **optional and default-off** (threshold = per-tx cap). Frame it as "escalation," not "approval."

### A7. "Idle USDC in a PDA earns nothing; treasuries hate that."
**Verdict: ACCEPT (defer).**
**Change:** Note as a v2 driver ("mandate over a yield-bearing position"); explicitly out of capstone scope.

### A8. "Who pays you? Developers don't pay for an open-source program."
**Attack:** No monetization = no PMF, just a nice primitive.
**Verdict: ACCEPT.**
**Why:** Fair. The cohort asks for PMF, not just utility.
**Change:** State the model honestly: the **program is free and open**; revenue is (1) an optional protocol fee in basis points on each `spend` (configurable per mandate, default 0 for the capstone), and (2) a hosted **Mandate Manager** (dashboard, alerts, receipt export for finance) for teams. Comparable: Squads protocol is free, Squads Grid API is the business.

---

## B. Attacks on Target Markets & User Profiles

### B1. "Enterprise ops will not put USDC in a Solana PDA in 2026."
**Verdict: ACCEPT.**
**Change:** Demote enterprise to a **later** segment (12–24 months) and remove "Dana, enterprise ops lead" as a top-3 profile. Replace with an **x402 seller/facilitator** profile, who has a *today* incentive: fewer failed payments and disputes if buyers' agents carry verifiable mandates.

### B2. "Agent platforms (SendAI, OpenClaw) will build this themselves."
**Verdict: PARTIAL.**
**Why:** They might — which is exactly why the primitive must be open, small, and CPI-friendly so building on it is cheaper than building it. Platforms are a **channel**, not a customer.
**Change:** Reclassify frameworks/platforms as distribution partners in GTM, not as a target market segment.

### B3. "Your 'Priya' persona lost $400 — that's not enough pain to change tooling."
**Verdict: PARTIAL.**
**Why:** For a solo dev, $400 is annoying, not existential. But the *fear* of the Grok/Bankr class of loss is what drives adoption; the persona should reflect the asymmetric downside, not the realized loss.
**Change:** Rewrite Priya's pain as "runs agents with a $2k hot wallet and knows one bad prompt could zero it."

---

## C. Attacks on Competitor Landscape

### C1. "You missed the players who actually matter."
**Attack:** Missing: Turnkey and Privy (policy-enforced signing, audited, widely used), Coinbase AgentKit + Coinbase x402 facilitator (52% of x402 facilitator volume), Circle programmable wallets, MPL Agent Registry (Metaplex, mainnet — executive delegation), Solana Agent Registry (ERC-8004 port, live March 2026), Laso Finance (private rails for agents, Colosseum-backed), solana-mpp.
**Verdict: ACCEPT.**
**Change:** Add all of the above. Re-sort the table into three buckets: **incumbent on-chain custody** (Squads, Crossmint), **off-chain policy/signing** (Turnkey, Privy, otomat, t402, Skyfire, Locus), **complements to integrate with, not compete against** (Solana Agent Registry, MPL Agent Registry, Coinbase/PayAI facilitators, solana-mpp).

### C2. "Three hackathon projects already built your exact program on devnet."
**Attack:** x402Guard, Sage, agent_fuel are literally "PDA vault + policy + x402." You are late.
**Verdict: PARTIAL.**
**Why:** They validate the idea and prove it's buildable in a hackathon window — which is a *point in favor* of capstone feasibility. None has identity binding, receipts, escalation, or an open CPI interface; none is on mainnet; all are single-developer.
**Change:** Cite them as **market validation** and design Leash so it could absorb their users: publish the mandate account layout and a `verify_mandate` CPI so their SDKs could target Leash.

---

## D. Attacks on Founder-Market Fit

### D1. "You have never shipped a financial security primitive and you're doing it solo."
**Verdict: ACCEPT.**
**Change:** Add explicit mitigations: (1) tiny attack surface — ~7 instructions, one token (USDC), no admin upgrade authority after audit; (2) test discipline from the cohort (LiteSVM negative tests for every guard); (3) submit to Solana Audit Arena / Trail of Bits skill review before any mainnet; (4) capstone target is **devnet + audited open-source**, not custodial mainnet service.

### D2. "Lean/continuous-improvement is not fintech. This FMF is a stretch."
**Verdict: REJECT.**
**Why:** The product *is* a control: a limit, an allowlist, an escalation path, and an audit trail wrapped around a process that wants to run unsupervised. That is nine years of my job description applied to a new kind of worker. The framing stays; the wording gets tighter.

### D3. "Evenings/weekends can't sustain a protocol with keepers, facilitators, uptime."
**Verdict: ACCEPT (by design).**
**Change:** State as a design constraint: Leash has **no keeper, no off-chain service in the critical path**. The program enforces; the agent pays; the facilitator settles. The only hosted piece (Mandate Manager) is optional and read-only.

### D4. "You are your own first user — that's a bias, not a fit."
**Verdict: PARTIAL.**
**Why:** Building for yourself is a classic false-positive. But "I run a fleet of agents and refuse to be their circuit breaker" is exactly the persona the market is converging on (BlackRock's paper, the Bankr post-mortems).
**Change:** Keep, but commit to **5 external builder interviews** from the Colosseum/SendAI communities before the architecture diagram is finalized, and log them.

---

## E. Attacks on the Rejected Direction (Cryptoball) — recorded for proof-of-work

Recorded so the appendix shows *why* the pivot happened, not just that it did.

| Attack | Verdict | Evidence |
|---|---|---|
| Not in any of the four Turbin3 categories (gambling ≠ RWA/Tokenization/Payments/Collectibles) | ACCEPT | Assignment text |
| Private lotteries are illegal without a license in every US state; federal 18 U.S.C. §1301–1307 | ACCEPT | Kentucky AG v. VGW (June 2026); Michigan MGCB cease-and-desist (Aug 2024); Texas SSB emergency order vs. Slotie NFT (2022) |
| "Confidential winnings" + "refund anytime" are AML anti-patterns for a gambling operator | ACCEPT | Same enforcement pattern; operators are required to KYC winners |
| Megapot has already shipped every PRD differentiator with $5M from Dragonfly/Coinbase Ventures and $200M+ in drawings | ACCEPT | megapot.io, docs.megapot.io |
| Solo, <$5k, evenings/weekends cannot fund jackpot liquidity, licensing counsel, or paid acquisition | ACCEPT | Founder profile red flags: team_required, long_sales_cycles |
| VRF + commit/reveal + confidential-transfer skills are reusable | ACCEPT (salvage) | Carried into Leash: verifiable receipts; optional confidential mandate balances (v2) |

**Outcome:** Direction killed for the capstone. Mechanics salvaged.

## Appendix C — Sources


All fetched 2026-09-22/23 during LOI research. Numbers are as reported by the source on that date.

## Industry signal (assigned reading)
- BlackRock Digital Assets Research, *The Machine-Native Economy* (Sept 2026). Local text extract: `.devin/knowledge/raw/blackrock-machine-native-economy.txt`. Key claims used: agentic AI needs machine-native payment rails; x402 / ACP / MPP / AP2 / TAP as emerging standards; KYA checks off-chain with results passed on-chain; stablecoins >$300B cap, >$11T adjusted 2025 volume; agentic payments "nascent."

## x402 / agent-payment market data
- x402stats.io — facilitator share (Coinbase 52.6%, PayAI 23.7%); Sept-2026 organic volume ~$1.2M, 78 organic sellers; top-10 wallets 81% of volume. https://x402stats.io/facilitators · https://x402stats.io/llms-full.txt
- x402.fuchss.app — $56.01M cumulative settled; Solana $10.04M (17.9%); 54% of 133,581 listed endpoints unreachable. https://x402.fuchss.app/trust/report
- dev.to/pennyforgehq — Solana 74% of measured x402 transaction count (2026-09-09 pull). https://dev.to/pennyforgehq/i-pulled-all-680-x402-services-and-counted-the-payments-solana-does-74-of-the-measured-1d4h

## Incidents (customer pain)
- Grok / Bankrbot prompt-injection drain, ~$155k, 4 May 2026. https://riskatlas.principle.sg/cases/grok-bankrbot-morse-code-heist · https://beyondmachines.net/event_details/prompt-injection-attack-drains-155000-from-grok-linked-bankr-crypto-wallet-x-q-p-c-p
- Second Bankr breach ~$440k, 19–20 May 2026. https://blog.onesavie.com/how-attackers-drained-440k-from-bankr-through-prompt-injection-and-ai-trust-abuse-65408216c098
- Google Mandiant enterprise AI security report: runaway agent, 15,000 calls, ~$50k in <1 hour (16 Sept 2026). https://www.helpnetsecurity.com/2026/09/16/google-mandiant-enterprise-ai-security-risks-report/
- LangChain retry loop, 11 days, $47k. https://www.kognita.co/blog/ai-agent-runaway-cost-no-kill-switch

## Competitors
- Squads Grid spending limits API. https://squads.mintlify.app/grid/v1/api-reference/endpoint/spending-limits/post · Smart Account Program mainnet post: https://squads.xyz/blog/squads-smart-account-program-live-on-mainnet · Crossmint integration: https://squads.xyz/blog/crossmint-integration
- Crossmint, *How Agents Pay*. https://docs.crossmint.com/agents/how-agents-pay
- x402Guard (AceDataCloud). https://github.com/AceDataCloud/x402Guard
- Sage (ogazboiz). https://github.com/ogazboiz/sage
- agent_fuel (frankolien). https://github.com/frankolien/agent_fuel
- @otomat/agent-wallet-kit. https://www.npmjs.com/package/@otomat/agent-wallet-kit
- t402 Agent Policy Engine. https://docs.t402.io/advanced/agent-policy
- v402pay spec. https://github.com/valeo-cash/v402/blob/main/docs/spec-v2.md
- Skyfire — $8.5M seed (TechCrunch, Aug 2024); $9.5M total incl. Coinbase Ventures / a16z CSX (crypto.news). https://techcrunch.com/2024/08/21/skyfire-lets-ai-agents-spend-your-money/ · https://crypto.news/skyfire-raises-9-5m-to-build-ai-payment-network/
- Locus (YC F25) — $625k total. https://linkedin.com/company/paywithlocus

## Complements / standards
- Solana Agent Registry (ERC-8004 port, launched Mar 2026). https://solana.com/agent-registry · https://quantulabs.github.io/8004-solana/
- SIMD-0520 On-Chain Agent Identity Standard / did:aip. https://github.com/solana-foundation/solana-improvement-documents/pull/526
- MPL Agent Registry (Metaplex, mainnet). https://www.metaplex.com/docs/smart-contracts/mpl-agent
- KYA manifest standard (draft). https://github.com/open-kya/kya-standard
- Colosseum Agent Hackathon results (454 submissions). https://blog.colosseum.com/agent-hackathoin-ptoken-seeker-grants/ · Frontier winners incl. Laso Finance, Flovia. https://blog.colosseum.com/announcing-the-winners-of-the-solana-frontier-hackathon/

## Rejected direction (Cryptoball) evidence
- Megapot — $5M pre-seed (Dragonfly, Coinbase Ventures), $200M+ drawings, 19 jackpot winners, Pyth randomness. https://megapot.io/ · https://docs.megapot.io/
- RugPot Solana lottery architecture (Switchboard VRF). https://dev.to/wrench/how-we-built-a-bonding-curve-lottery-on-solana-3abk
- open-lotto (Solana, Switchboard). https://github.com/arussel/open-lotto
- Kentucky AG v. VGW complaint (17 June 2026). https://www.ag.ky.gov/Documents/2026.06.17%20Accepted%20KYOAG%20v.%20VGW%20Complaint%20-%20Final.pdf
- Michigan MGCB cease-and-desist, unlicensed online lottery (27 Aug 2024). https://www.michigan.gov/mgcb/news/2024/08/27/mgcb-issues-cease-and-desist-letter-to-one-country
- Texas SSB emergency C&D vs. Slotie NFT (2022). https://www.ssb.state.tx.us/sites/default/files/2022-10/ENF_22_CDO_1865_0.pdf
- Federal anti-lottery statutes 18 U.S.C. §§1301–1307 (PowerPick case). https://law.resource.org/pub/us/case/reporter/F3/200/200.F3d.647.98-15589.html

## Cohort guidance applied
- `knowledge/cohort/capstone-loi-and-architecture.md` — LOI Part 1 contents, human-first then red-team, submit as Google Doc/PDF.
