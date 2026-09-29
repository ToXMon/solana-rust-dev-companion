---
title: "Capstone Deliverable 1: Letter of Intent & On-Chain Use Cases"
subtitle: "Leash — on-chain spending mandates for AI agents on Solana"
author: "Tolulope Shekoni (solo) · Turbin3 Builders Cohort Q3 2026"
date: "September 2026"
---

# Part 1 — Final Project Proposal & LOI

## Project Overview

Leash is an open Solana program that lets a person or company fund an AI agent with USDC and bind that money to an on-chain spending mandate: a per-payment cap, a rolling-window cap, a lifetime cap, an allowlist of payee wallets, an expiry, and an optional threshold above which the human must co-sign. The agent pays x402/MPP services (or any Solana payee) with its own key, but every payment is a Leash instruction that the program rejects if it violates the mandate. A prompt-injected, looping, or stolen agent can never spend past its leash. **Category: Payments** (agentic / machine-to-machine settlement).

## Core Value Proposition & Product-Market Fit

Autonomous agents are already losing real money two ways: *runaway execution* (Google Mandiant, Sept 2026: an agent looped into 15,000 API calls and ~$50k in under an hour) and *hijacked authorization* (the Grok/Bankr prompt-injection drain, ~$155k, May 2026, followed by a ~$440k Bankr breach two weeks later). The root cause is the same in every case: the agent held standing spending power, and the policy meant to limit it lived in the same trust domain as the agent. Leash moves the policy into the program that holds the funds, where reasoning bugs and injected instructions cannot reach it. Principals get a provable ceiling on loss and a receipt trail; agent developers get a drop-in "pay within policy" primitive; x402 sellers and facilitators get a counterparty-verifiable mandate they can read on-chain. The incumbent, Squads, proves demand for on-chain spending limits but ships them as a permissioned, general-purpose smart-account API; Leash's wedge is **open and agent-native** — permissionless CPI, binding to Solana Agent Registry identities, x402 payment-intent hashes in receipts, and an escalation path modeled on AP2 mandates. This sits squarely inside BlackRock's *Machine-Native Economy* thesis (Sept 2026) that x402/MPP rails, stablecoins, and know-your-agent controls are the settlement layer for agentic commerce.

**Coverage check — value drivers found missing in v1 and added:** counterparty-verifiability (off-chain policies structurally cannot offer it); a lifetime cap so worst-case loss is `min(lifetime_cap, 2 × window_cap)`; an honest business model (free program; optional bps fee on spend; hosted read-only Mandate Manager for teams). Deferred to v2: yield on idle vault balances; confidential mandate balances via Token-2022.

**Honest market note:** x402 is small today — ~$56M cumulative settled, ~$1.2M/month organic in Sept 2026, with heavy wash volume. Solana carries ~18% of dollar volume but ~74% of transaction count, i.e., the high-frequency agent traffic Leash targets. The loss-bounding problem exists for every agent holding a key on any rail; x402/MPP is the first rail Leash speaks, not the only one.

## Target Markets & User Profiles

**Segments (ranked by time-to-revenue)**

1. Solo and small-team agent builders on Solana running 24/7 agents against paid APIs (OpenClaw, Solana Agent Kit, MCP tool authors, Colosseum alumni) — *now*.
2. x402/MPP sellers and facilitators wanting fewer failed or disputed settlements and a way to gate premium routes to mandated agents — *now*.
3. AI-product teams shipping one agent to many end users, each needing an isolated, capped envelope — *6–12 months*.
4. Enterprise ops/finance pilots of agentic procurement requiring hard caps, escalation, and reconciliation exports — *12–24 months*.

Agent frameworks and wallet platforms (SendAI, Crossmint, OpenClaw) are distribution channels, not customers.

**User profiles**

- **Priya, indie agent developer.** Runs three research/trading agents from a $2k hot wallet and knows one bad prompt could zero it. Wants: "give it $50/day, allowlist these four payees, expire Friday, done."
- **Marcus, CTO of a small AI product.** Ships one research agent to ~2,000 users; needs a capped envelope per user, receipts for support, and no custody of user funds.
- **Lena, x402 API seller.** Loses revenue to failed settlements and abusive loops; wants to require a valid mandate and read remaining budget before serving a premium route.

**Coverage check against ecosystem demand:** Colosseum's Agent Hackathon (Feb 2026) drew 454 submissions; three independent teams built devnet "PDA vault + policy + x402" projects (x402Guard, Sage, agent_fuel) in one hackathon window — builders feel this pain and the primitive is buildable in capstone scope.

## Competitor Landscape

| Player | What they do | Enforced | Gap Leash exploits |
|---|---|---|---|
| **Squads Protocol / Grid** | Formally verified smart account; daily/weekly/monthly limits, destination allowlists; mainnet | On-chain | Permissioned partner API; general-purpose, not agent-native; no identity binding, no x402 intent/receipts, not a permissionless CPI target |
| **Crossmint agent wallets** | Non-custodial delegation (limit, counterparties, window); cards via Visa VIC | On-chain (via Squads) | Vendor-locked; policy semantics owned by Crossmint |
| **x402Guard, Sage, agent_fuel** | Devnet Anchor vaults with caps/allowlists for x402 | On-chain | Single-dev hackathon maturity; no identity, receipts, escalation, or open CPI — validate demand |
| **Turnkey / Privy** | Audited policy-enforced signing | Off-chain | Vendor dependency; not counterparty-verifiable; defeated when the authorization path itself is compromised (Bankr class) |
| **otomat agent-wallet-kit, t402, v402pay** | Declarative budgets wrapping `fetch()`/402 flows | Off-chain (SDK) | Runs inside the agent's own process; bypassable |
| **Skyfire** ($9.5M; Coinbase Ventures) | KYA + wallet + USDC payment network | Off-chain, custodial | Closed network, Base-first |
| **Locus** (YC F25), **Payman** | Prepaid API credits; agents paying humans | Off-chain | Not on-chain / adjacent problem |
| *Complements:* Solana Agent Registry (ERC-8004 port), MPL Agent Registry, Coinbase & PayAI facilitators, solana-mpp | Identity and settlement layers | — | Leash binds to and emits for these rather than competing |

**Research method:** x402stats.io and x402.fuchss.app facilitator data; Colosseum Agent and Frontier hackathon results; GitHub; Squads/Crossmint/Skyfire docs and funding announcements. **PMF adjustment after coverage check:** Leash does not compete on custody security or convenience; it competes on being the open, counterparty-verifiable, agent-identity-bound policy layer that every bucket above lacks.

## Founder-Market Fit

Solo founder. Nine years in operational excellence and continuous improvement inside large enterprises — a career designing limits, allowlists, escalation paths, and audit trails around processes people wanted to run unsupervised; Leash is that control system applied to a new kind of worker. Data-science training, full-stack AI development, and Rust/Anchor through Turbin3 (vault, escrow, AMM, Token-2022, Metaplex Core, confidential transfers). I run fleets of coding agents daily and refuse to be their circuit breaker — I am the first user. Network: Turbin3 cohort, Colosseum and Solana agent-builder communities.

**Weaknesses and mitigations (from red-team):** no prior fintech/security shipping record and solo → minimal attack surface (a dozen single-purpose instructions, USDC only, no admin in the spend path), a LiteSVM negative test per guard, external security review before any mainnet, capstone target is audited open-source on devnet. Evenings/weekends time budget → no keeper and no off-chain service in the critical path by design. Self-as-user bias → five external builder interviews before the architecture diagram is finalized.

## Actors

| Class | Actor | Signs Leash instructions? |
|---|---|---|
| Direct | **Principal** — funds the Leash and owns its policy | Yes: configure, fund, approve escalations, withdraw, close |
| Direct | **Agent** — holds `agent_key`, a hot key with no authority beyond the mandate | Yes: `spend`, `request_escalation` |
| Direct | **Anyone** — closes an expired escalation to reclaim rent | Yes: `cancel_escalation` after expiry only |
| Beneficiary | **Payee / x402 seller** — receives USDC when `spend` succeeds | No |
| Beneficiary | **End users of an agent product** — funds protected by the mandate | No |
| Administrator | **Program upgrade authority** — burned after audit; never in the spend path | No user-facing instruction |
| Administrator | **Protocol config authority** (optional; capstone default none) | Only `init_protocol_config` |
| Stakeholder | **x402/MPP facilitators**, **agent identity registries**, **auditors** | No |

## Use Cases — one atomic state transition = one Anchor instruction

Accounts: **Leash** PDA `["leash", principal, agent_id]` (policy + counters + agent binding), its USDC **vault ATA**, **PayeeEntry** PDA `["payee", leash, payee]` (existence = allowed), **Escalation** PDA `["escalation", leash, intent_hash]`.

| # | Instruction | Signer | Precondition | State transition | Postcondition / invariant |
|---|---|---|---|---|---|
| 1 | `create_leash(agent_id, agent_key, agent_identity?)` | Principal | Leash for (principal, agent_id) absent | Init Leash (Active, all caps 0) + vault ATA | Default-deny: nothing spendable until a mandate is set |
| 2 | `set_mandate(per_tx, window, window_secs, lifetime, expires_at, escalation_threshold, fee_bps)` | Principal | `has_one = principal` | Overwrite policy fields; counters untouched | `per_tx ≤ window ≤ lifetime`; `threshold ≤ per_tx`; `expires_at > now`; editing cannot reset spend |
| 3 | `deposit(amount)` | Principal / any funder | Leash exists | `transfer_checked` funder ATA → vault ATA | Balance up; policy unchanged (funding ≠ authorization) |
| 4 | `add_payee(payee)` | Principal | PayeeEntry absent | Init PayeeEntry | Payee pubkey now passes allowlist |
| 5 | `remove_payee(payee)` | Principal | PayeeEntry exists | Close PayeeEntry, rent → principal | Future spends to payee fail |
| 6 | **`spend(amount, intent_hash)`** | **Agent** | Active; not expired; PayeeEntry exists; `amount ≤ per_tx` and `≤ threshold`; window + lifetime math passes | Roll window if elapsed; checked-add counters; PDA-signed `transfer_checked` vault → payee ATA (+ optional fee); emit `Receipt{leash, agent_identity, payee, amount, intent_hash, nonce, slot}` | `spent_in_window ≤ window_cap`; `spent_lifetime ≤ lifetime_cap`; worst-case loss `min(lifetime, 2×window)` |
| 7 | `request_escalation(amount, intent_hash, ttl)` | Agent | `threshold < amount ≤ lifetime remaining`; PayeeEntry exists | Init Escalation (agent pays rent); **no funds move** | Principal must act; agent cannot bypass |
| 8 | `approve_escalation` | Principal | Escalation exists, not expired; Active | PDA-signed transfer; `spent_lifetime += amount`; emit receipt; close Escalation | Lifetime invariant holds; no replay (PDA closed) |
| 9 | `cancel_escalation` | Principal any time / Anyone after expiry | Escalation exists | Close Escalation, rent → agent | No funds move |
| 10 | `pause` | Principal | Active | `status = Paused` | Kill switch: `spend`/`approve` fail |
| 11 | `resume` | Principal | Paused | `status = Active` | Counters unchanged |
| 12 | `rotate_agent_key(new_key)` | Principal | `new_key ≠ principal` | Replace `agent_key` | Old hot key revoked without moving funds |
| 13 | `withdraw(amount)` | Principal | Balance sufficient | PDA-signed transfer vault → principal ATA | Counters unchanged (withdrawal ≠ spend) |
| 14 | `close_leash` | Principal | Vault balance 0; no open escalations | Close vault ATA (`close_account`) and Leash | Rent returned |
| 0 | `init_protocol_config(max_fee_bps, fee_recipient)` *(optional)* | Config authority | Absent | Init `["config"]` | Read only when `fee_bps > 0` |

**On-chain vs client-side split.** On-chain: caps, windows, pubkey allowlist, escalation state machine, transfers, receipt events. Client/SDK: receiving the 402 challenge, hashing it to `intent_hash`, resolving domain → payee pubkey (the program never sees a URL), retry with payment proof, notifications. Indexer: turning `Receipt` events into dashboards and reputation feedback.

**Granularity check.** *Atomicity:* 15 use cases → 15 handlers (`spend`'s optional fee transfer is part of the same "mandated payment" transition). *State ownership:* all mutable state in program-owned PDAs; receipts are events consumed by an explicit indexer. *Real signers:* Principal and Agent are the only value-moving signers; no keeper or backend signature. *On-chain vs client:* documented above and per row.

**CPI dependencies:** SPL Token / Token-2022 via `TokenInterface` (`transfer_checked`, `close_account`), Associated Token Program (`init_if_needed` payee ATA), System Program, Clock sysvar; optional read of Solana Agent Registry / MPL Agent Registry at `create_leash`.

---

# Part 2 — Process Appendix (Red Team Log)

## A. Original Phase 1 draft (v1, verbatim, pre-red-team)

> **Overview (v1).** Leash is an open, permissionless Solana program that lets a human (or a company treasury) fund an AI agent with USDC and bind that money to an on-chain spending mandate: per-payment cap, rolling daily cap, an allowlist of payee endpoints, an expiry, and a threshold above which the human must co-sign. The agent pays x402/MPP services with its own key, but every payment is a program instruction that fails if it violates the mandate — so a prompt-injected or looping agent can never spend more than the mandate allows. Leash is the "spending limit for agents" that today only exists inside closed APIs (Squads Grid) or in SDKs the agent itself can bypass.
>
> **Value prop (v1).** Every team shipping autonomous agents on Solana faces the same fork: give the agent a hot wallet (and pray), or bolt a spend-policy into the SDK (which a compromised or buggy agent simply ignores). Leash moves the policy into the program that holds the funds, where it cannot be ignored. Principals get a hard, verifiable ceiling on loss; agent developers get a drop-in "pay within policy" primitive instead of rolling their own; x402 sellers/facilitators get a receipt they can verify on-chain. Value drivers: (1) bounded blast radius, (2) auditability, (3) composability, (4) developer speed. *Self-assessed gaps:* cost of idle capital, non-technical UX, multi-agent hierarchies.
>
> **Target markets (v1).** (1) Solo/indie agent builders; (2) small AI-product teams; (3) enterprise ops/finance teams piloting agentic procurement; (4) agent platforms/frameworks. Profiles: Priya (indie dev, lost $400 to a retry loop), Marcus (AI-product CTO), Dana (enterprise ops lead needing co-sign and monthly export).
>
> **Competitors (v1).** Squads Grid, Crossmint, x402Guard, Sage, agent_fuel, otomat/t402 SDKs, Skyfire, Locus, Payman.
>
> **FMF (v1).** Solo. 9 years operational excellence/continuous improvement; builds and operates coding-agent fleets daily; full-stack AI dev; Rust/Anchor via Turbin3. Weaknesses: no fintech/compliance shipping experience; solo; evenings/weekends.

## B. Red-team attacks, verdicts, and changes

| # | Attack (AI critique) | Verdict | Why / change made in Part 1 |
|---|---|---|---|
| A1 | "A seatbelt for a car nobody drives" — x402 organic volume is ~$1.2M/mo with 78 organic sellers; BlackRock calls it nascent | **Accept (reframe)** | Pain is "standing spending power," not x402-specific; led with Mandiant/LangChain/Bankr losses; x402 positioned as *first* rail. Added honest market note. |
| A2 | Squads already ships formally verified limits on mainnet | **Partial** | True incumbent, but permissioned API, not agent-native, not a permissionless CPI target. Named Squads explicitly; wedge = open + agent-native; fallback = Leash as policy layer over a Squads account. |
| A3 | Off-chain policy SDKs / Turnkey / Privy are good enough for 95% of devs | **Partial** | Bankr shows the authorization path itself gets compromised. Added Turnkey/Privy as the real "good-enough" threat; added counterparty-verifiability as a driver; GTM must match SDK convenience. |
| A4 | You cannot allowlist an *endpoint* on-chain; x402 pays a pubkey | **Accept** | Mandate allowlists **payee pubkeys**; `intent_hash` recorded in receipt; domain resolution explicitly client-side. |
| A5 | Rolling windows are gameable at the boundary | **Accept (minor)** | Precise bucket semantics; added **lifetime cap** so worst case is bounded regardless. |
| A6 | Human co-sign kills autonomy | **Partial** | Kept, optional, default-off; framed as escalation (AP2/TAP convergence). |
| A7 | Idle USDC earns nothing | **Accept (defer)** | v2 driver; out of capstone scope. |
| A8 | Who pays you? | **Accept** | Free program; optional bps fee; hosted Mandate Manager (Squads free-protocol/paid-API pattern). |
| B1 | Enterprise won't put USDC in a Solana PDA in 2026 | **Accept** | Enterprise demoted to 12–24 months; replaced "Dana" with "Lena, x402 seller." |
| B2 | Platforms (SendAI) will build it themselves | **Partial** | Reclassified as distribution channels; primitive kept small and CPI-friendly so adopting beats building. |
| B3 | Priya's $400 loss isn't enough pain | **Partial** | Rewrote as asymmetric fear: $2k hot wallet, one bad prompt. |
| C1 | Missed Turnkey, Privy, Coinbase facilitator, Circle, MPL Agent Registry, Solana Agent Registry, Laso, solana-mpp | **Accept** | Added; re-sorted into on-chain custody / off-chain policy / complements. |
| C2 | Three hackathon teams already built your program | **Partial** | Cited as validation and feasibility proof; design an open CPI so their SDKs could target Leash. |
| D1 | Never shipped a security primitive, and solo | **Accept** | Added mitigations: tiny surface, negative tests, external review, devnet + open-source target. |
| D2 | Lean/CI is not fintech | **Reject** | The product *is* a control (limit, allowlist, escalation, audit trail). Tightened wording only. |
| D3 | Evenings/weekends can't run a protocol | **Accept (by design)** | No keeper, no off-chain service in the critical path. |
| D4 | You're your own first user — bias | **Partial** | Kept; committed to 5 external builder interviews; GTM leads with third-party incidents. |
| P1 | "Edit the mandate to reset counters" | **Accept** | `set_mandate` never touches counters. |
| P2 | "Approve an escalation twice" | **Accept** | Escalation PDA closed on approval; seeded by `intent_hash`. |
| P3 | "Agent griefs principal with thousands of escalations" | **Accept** | Agent pays rent; anyone can close after expiry. |

## C. Rejected direction: Cryptoball (crypto lottery) — stress test that triggered the pivot

The first candidate was a Powerball-style on-chain lottery with NFT tickets, refund-anytime, VRF draws, and confidential winnings. It was killed before Phase 1 was written:

| Attack | Verdict | Evidence |
|---|---|---|
| Not in any of the four assignment categories (gambling ≠ RWA/Tokenization/Payments/Collectibles) | Accept | Assignment text |
| Private lotteries are illegal without a license in every US state; federal 18 U.S.C. §§1301–1307 | Accept | Kentucky AG v. VGW (June 2026); Michigan MGCB cease-and-desist (Aug 2024); Texas SSB order vs. Slotie NFT |
| "Confidential winnings" + "refund anytime" are AML anti-patterns for a gambling operator | Accept | Operators are required to KYC winners |
| Megapot already ships every differentiator: $5M pre-seed (Dragonfly, Coinbase Ventures), $200M+ in drawings, Pyth VRF, instant payout | Accept | megapot.io, docs.megapot.io |
| Solo, <$5k, evenings/weekends cannot fund jackpot liquidity, licensing counsel, or acquisition | Accept | Founder profile |
| VRF / commit-reveal / confidential-transfer skills are reusable | Accept (salvage) | Carried into Leash receipts and v2 confidential mandates |

## D. Key sources

BlackRock, *The Machine-Native Economy* (Sept 2026) · x402stats.io and x402.fuchss.app (facilitator and volume data) · Google Mandiant enterprise AI security report via Help Net Security (16 Sept 2026) · Grok/Bankr incident write-ups (AI RiskAtlas, BeyondMachines, OneSavie) · Squads Grid spending-limits API and mainnet announcement · Crossmint *How Agents Pay* · GitHub: x402Guard, Sage, agent_fuel, otomat, t402, v402 · Skyfire funding (TechCrunch, crypto.news) · Solana Agent Registry (solana.com), SIMD-0520, MPL Agent Registry (Metaplex) · Colosseum Agent and Frontier hackathon results · Megapot docs · Kentucky AG v. VGW complaint; Michigan MGCB and Texas SSB enforcement releases.
