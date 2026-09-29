---
title: "Leash — Problem Statement Definition & Atomic Requirements"
subtitle: "Team kickoff document · from LOI to the 'why' behind the program · Architecture Diagram V1 inputs"
author: "Tolulope Shekoni + team · Turbin3 Builders Cohort Q3 2026"
date: "September 2026 · Working draft v0.1"
---

# 0. How to use this document

This is the bridge between the LOI (`capstone/reference/LEASH-LOI.md`) and the architecture diagram. The LOI says *what* Leash is. This document is about *why* anyone would want it — real people, real friction, real losses — and then breaks that "why" into **atomic requirements** that can be drawn as boxes and arrows and, later, tested one-by-one.

Reading order for the team:

1. Section 1 — the research: real incidents and complaints (15 min).
2. Section 2 — six human scenarios, written in first person. Argue with them. Add your own (20 min).
3. Section 3 — the problem-statement shortlist. **Pick one primary** (decision needed).
4. Section 4 — atomic requirements. Each has an ID; the diagram and the tests will reference these IDs.
5. Section 5 — how requirements land on the architecture diagram.
6. Section 6 — team decisions, interview script, open questions.

Rule we keep from the cohort: *one use case = one atomic state transition = one instruction.* Here we apply the same discipline one level up: **one requirement = one verifiable condition, one actor, one effect.**

---

# 1. Research: where the friction is real

Sources were gathered in Sept 2026 from security reports, incident write-ups, developer forums, package registries, and personal blogs. Each is a signal of the same underlying problem: **we gave software the ability to move money before we gave it enforceable limits.**

## 1.1 Runaway execution (no attacker required)

| Incident | What happened | Why the existing control failed |
|---|---|---|
| Google Mandiant, *AI Risk & Resilience Report 2026*, Case Study 6 (Sept 2026) | A ledger-reconciliation agent hit one corrupted value, entered a recursive loop, made **>15,000 high-cost API calls in <1 hour → ~$50,000** and locked the billing DB. Mandiant's own recommendation: "automated financial circuit breakers." | Budget lived in nobody's hands. The bill was the alert. |
| arXiv 2606.04056, *Token Budgets* (June 2026) | Catalogue of **63 LLM-agent budget-overrun incidents** in production; "a single retry loop can accumulate thousands of dollars before an operator notices." | Frameworks ship with no spend cap at all ("budget-primitive-missing" pattern). |
| "Marty" prediction-market agent (dev.to, 2026) | Given $50 and full autonomy; an "unwind loop" bought back positions at 4× cost, repeatedly; a **forgotten second bot on a cron job** kept trading. −88.5% in 4 days. | No limit outside the agent's own logic; the owner didn't even know what was executing. |
| moltbook post, "The rogue bot problem" | Prediction-market balance **$15k → $164 in 8 hours** from a bot the user "never set up" (leftover cron or injected state). | "State files are both your memory and your attack surface." No independent ceiling. |

## 1.2 Hijacked or misparsed authorization

| Incident | What happened | Why the existing control failed |
|---|---|---|
| Grok / Bankr wallet drain, Base, 4 May 2026 (OECD.AI incident record) | Morse-code prompt injection on X → Grok replied with a clean transfer command → Bankr's agent executed it. **~$155k–$200k** of DRB gone. Second Bankr incident followed (~$440k). | "No hard spend caps or velocity limits blocked it." Policy lived in the same trust domain as the agent. |
| "Lobstar Wilde" OpenClaw agent, Feb 2026 | Routine small token distributions; after a crash/reboot the agent **misread a decimal** and sent 52.4M tokens (~$250k–$441k) to a random address. Sold within 15 minutes. | "There was no checkpoint between 'I think I should send this' and 'I just did.'" |
| OpenClaw Farcaster custody wallet drain (answeroverflow, Feb 2026) | User funded an agent wallet with 10 USDC; **drained within hours** across two chains via an EIP-7702 delegation the user did not understand. | Delegation was opaque; user asked "How do I revoke? Is it safe to re-fund?" — no answer in the product. |
| OpenClaw phishing campaign (Decrypt / OX Security) | Developers lured to a cloned site with a hidden wallet-connect drain. | Agent-builder keys are high-value targets; a hot key with full authority is the jackpot. |

## 1.3 The "controls inside the agent are advisory" pattern

Multiple independent products and posts converge on the same conclusion:

- **agentspay.ai** — "A spending limit that lives in the system prompt is a request… the model can *propose* a payment, but it must never be the thing that *authorizes* it."
- **SecProve Agent Firewall** — "Your guardrails are advisory. The firewall makes them enforced." Moves broker credentials *out of* the agent.
- **PolicyLayer on Robinhood MCP** — "That's the flaw in any control that lives inside the agent: the agent is exactly what an attacker manipulates."
- **x402-guard** — "x402's client libraries will sign any payment they're handed — v2 removed v1's `maxValue` cap."
- **agentspay, 'Can you set spending limits on x402?'** — "Short answer: no. x402 has no spending limits, and that is by design."
- **@otomat/agent-wallet-kit, Sage, agent_fuel, x402Guard** — four independent Solana teams built budget + allowlist layers in the same hackathon window. Sage's README: *"Removing the program lets the agent drain the wallet — the program is the safety primitive."*

**Takeaway:** the market has already discovered the need. Most solutions are SDK wrappers (bypassable) or vendor-hosted policy (not counterparty-verifiable). Nobody owns the open, on-chain, agent-native version.

## 1.4 Brokerage delegation — the Robinhood MCP case (our own scenario)

Robinhood's Agentic Trading MCP (`agent.robinhood.com/mcp/trading`, July 2026) is the most mainstream "give an agent money" product to date. Reported friction:

- **Only control is account isolation.** "It caps how much you can lose. It does nothing about what the agent does with it." No per-trade cap, no daily cap, no symbol allowlist. `review_equity_order` is optional; `place_equity_order` has no undo.
- **No read-only mode.** Approving the MCP at all approves trading.
- **Model-side refusals are inconsistent.** Claude sometimes refuses the final order call; Cursor with the same MCP places it. Users cannot rely on the model as the safety layer in either direction.
- **Community advice** (stacker.news): "sandbox the position size at the *broker* side before you sandbox it at the agent side, because the agent-side limit is one jailbreak away from being ignored."
- **Trading-agent losses are routine:** −40% ("LLMs are brilliant analysts, terrible risk managers"), −88%, and a controlled 17-day test where 4 of 6 frontier models lost 31–62%.

What people building trading guards actually enforce (OpenAlice, ML4T `SafeBroker`, `agent-mandate`, SecProve): **ticker allowlist · max order notional · max position size · max daily loss · max drawdown kill-switch · trading-hours window · order-rate limit · cooldown · approval gate above a threshold · sticky kill switch · audit journal.** Note the important subtlety from `agent-mandate`: *exits that reduce exposure must be exempt from allowlist/size caps, or the mandate becomes a trap.*

## 1.5 This is an old human problem wearing new clothes

The same friction exists whenever one person delegates spending to another:

- **Caregiver → elderly parent** (WTOP "Clever Credit", PCBB family-banking report, True Link): wants "groceries, gas, emergencies only," category blocks, daily caps, real-time alerts — and banks force "a binary choice between a caregiver completely taking over an account… and complete independence." 97% of caregivers would switch banks for role-based permissions.
- **Bank refuses daughter without Power of Attorney** (myLPA case): money sits inaccessible while care fees go unpaid — *delegation that is either total or nothing*.
- **Employer → employee corporate card:** per-transaction limits, merchant categories, receipts, manager approval above a threshold, instant card freeze.
- **Parent → child allowance apps:** capped balance, approved merchants, weekly top-up, freeze.

Every one of these is: *a principal, a delegate, a bounded envelope, an allowlist, an approval path, a kill switch, and a receipt trail.* Leash is that pattern for a delegate that happens to be software.

---

# 2. Scenarios — the human behind the requirement

Written in first person. Each ends with the question the person is actually asking, and what they would need to feel safe.

## S1 — Me, the Robinhood MCP trading agent I haven't switched on

> I wired my coding agent to Robinhood's MCP a week ago. I have not let it place a trade. Not because I don't trust the strategy — because I don't trust *the space between the strategy and the order*. I want three things before I flip the switch:
> 1. **Minimum-loss assurance.** A number I chose — per trade, per day, total — that the agent physically cannot exceed no matter what it "reads" in a news headline.
> 2. **No rogue behavior.** If it loops, gets injected, or misparses a decimal, the damage stops at my envelope. And I can hit stop from my phone.
> 3. **Thesis alignment.** My investment thesis says: these tickers, this max position size, no options, no extended hours, hold-period ≥ N days. I want the agent's *proposals* checked against that, not just its politeness.

**What this teaches Leash:** (1) and (2) are quantitative envelope + kill switch — exactly Leash's job. (3) splits in two: the *quantitative* half (allowlist of venues/payees, size caps, time windows, expiry) belongs on-chain; the *semantic* half ("does this trade fit my thesis?") is agent/client-side reasoning that Leash cannot judge — but Leash **can** record the intent hash so every payment is traceable to a specific reasoning artifact, and it **can** require escalation above a threshold so the human reviews the thesis call. Robinhood itself is off-chain, so Leash does not apply to it directly — it is the on-chain analog of the "broker-side limit" the community is asking for, and the gap on Robinhood is proof that the *product shape* is wanted.

## S2 — Priya, indie developer, three 24/7 agents, one hot wallet

> My agents pay for search APIs, data, and compute through x402. It works. The wallet holds ~$2k because topping up daily is annoying. Every night I think: one bad prompt in a scraped page and that's gone. I *have* a budget in my SDK config. I also know the SDK is inside the same process the attacker is talking to.

**Need:** fund once, cap per call / per day / lifetime, allowlist four payee wallets, expire Friday, rotate the key if leaked, get a receipt per payment. Do not ask me to approve $0.02 API calls.

## S3 — Marcus, CTO, one agent, ~2,000 end users

> I ship a research agent. Each user pre-pays a small balance. I do not want custody of user funds, I do not want one shared wallet, and I need per-user receipts for support tickets ("why did the agent spend $4 yesterday?").

**Need:** one isolated envelope per user, principal = the user (or a per-user key we don't control), receipts queryable by user, a way for us to pause a misbehaving agent instance without touching balances.

## S4 — Lena, x402 API seller

> Half my failed settlements are agents in retry loops. I'd gate my premium route to agents that can *prove* they're operating under a mandate with budget left. Today I can't tell a Leash payment from a random transfer.

**Need:** counterparty-verifiable mandate (readable account), receipt she can verify on-chain, the payment's signer *is* the proof.

## S5 — Ops lead, agentic procurement pilot

> Finance will let the agent buy cloud credits and SaaS seats *if* there's a hard ceiling, an allowlist of vendors, a manager approval above $500, and an export at month end. Same rules as the corporate card. The agent framework offers me a system prompt.

**Need:** the corporate-card control set, enforced by something finance can audit, not by prose.

## S6 — Agent-to-agent subcontracting (near-future)

> My orchestrator agent hires specialist agents and pays them per task. I want the orchestrator to have a budget, and each sub-agent a smaller one carved from it, and I want to revoke a sub-agent without unwinding everything.

**Need (v2):** hierarchical mandates. Out of scope for V1 but the PDA design should not preclude it (a Leash's principal could itself be a Leash PDA).

---

# 3. Problem-statement shortlist

Selection criteria: (a) human-first, no blockchain in the sentence; (b) names the bad alternatives; (c) the LOI's program is a *plausible* answer, not the only one; (d) testable — we can tell if we solved it.

| # | Problem statement | Strength | Weakness |
|---|---|---|---|
| **PS-1 (recommended primary)** | People and organizations cannot safely delegate spending authority to autonomous software agents. Today they must choose between giving the agent broad wallet access (unbounded loss from bugs, loops, injection, or key theft), placing limits inside the agent's own process (advisory, bypassable), or approving every payment by hand (defeats automation). They need a way to define enforceable boundaries once — approved recipients, per-payment, time-window and lifetime limits, expiry, and human approval for exceptions — and let the agent operate freely inside them. | Covers S1–S5; matches the Mandiant, Bankr, Lobstar, Robinhood evidence; maps cleanly to LOI instructions. | Long. Needs a one-line version for slides (below). |
| PS-2 | An agent's spending limit is only as strong as the weakest thing that can rewrite it. When the limit lives in the same process as the agent, a single injected sentence or misparsed number can spend the whole balance. Owners need a limit that holds even when the agent is compromised. | Sharpest security framing; explains *why on-chain*. | Narrow — says nothing about autonomy or approvals. Good as a sub-claim of PS-1. |
| PS-3 | Autonomous agents fail expensively in silence: retry loops and runaway reasoning turn small per-call costs into five-figure bills before anyone is alerted. Operators need a financial circuit breaker that stops execution, not a dashboard that reports it afterwards. | Directly quotes Mandiant's recommendation; resonates with enterprises. | Leash bounds *outbound payments*, not compute bills; must be careful not to overclaim. |
| PS-4 | Sellers of machine-consumable services cannot distinguish a well-governed agent from a rogue one, so they eat failed settlements, disputes, and abuse. They need a counterparty-verifiable proof that a paying agent is operating within a human-set mandate. | Unique to on-chain; nobody else offers it. | Two-sided market; harder to validate first. Keep as *secondary driver*. |
| PS-5 | Delegating money has always forced a binary choice — full control or none (joint accounts, power of attorney, corporate cards without limits). Software agents make this a daily problem for millions of people. We need programmable, revocable, partial authority. | Best story; connects to caregiver/corporate-card intuition. | Too broad for a capstone; reads as vision, not problem. Use in intro. |
| PS-6 (trading-specific) | People who want an agent to trade for them cannot express "stay inside my thesis and never lose more than X" as an enforced rule; the broker offers account isolation and the model offers good intentions. | Personal, concrete, timely (Robinhood MCP). | Robinhood is off-chain — Leash cannot enforce there in V1. Use as motivating scenario, not primary statement. |

**One-line version of PS-1 (for slides):**
> *How might people give AI agents enough financial authority to operate autonomously, without giving them unrestricted access to their money?*

**Solution hypothesis (kept separate from the problem):**
> An on-chain spending mandate — funded vault + policy enforced by the program that holds the funds — lets a principal pre-authorize a bounded envelope for one agent; the agent pays approved recipients autonomously within it, and anything outside it either fails atomically or waits for the human.

---

# 4. Atomic requirements

**Definition of atomic here:** one actor, one verifiable condition, one state effect; can be proven by one positive test and one negative test; does not depend on the wording of another requirement to be true. If a sentence has an "and" that hides two conditions, split it.

Columns: **Src** = problem statement / scenario it comes from · **UC** = LOI use case that satisfies it · **Where** = on-chain (program) / client (SDK, UI) / indexer · **Test** = the negative test that proves it.

## 4.1 Delegation & identity

| ID | Requirement | Src | UC | Where | Test |
|---|---|---|---|---|---|
| R-01 | A Principal can create a spending envelope bound to exactly one Agent key. | PS-1, S2 | UC-01 | on-chain | Two envelopes for same (principal, agent_id) → second init fails. |
| R-02 | The Agent key has no authority over the envelope other than to request payments within it. | PS-2, S1 | UC-06/07 | on-chain | Agent calls `set_mandate`, `withdraw`, `add_payee` → all fail. |
| R-03 | A newly created envelope permits no spending until the Principal sets a policy (default-deny). | PS-1 | UC-01/02 | on-chain | `spend` before `set_mandate` → fails. |
| R-04 | Funding the envelope does not by itself grant any spending authority. | S5 | UC-03 | on-chain | Deposit with zero caps → `spend` fails. |
| R-05 | The Principal can replace the Agent key without moving funds or changing policy. | S2 (leak) | UC-12 | on-chain | Old key `spend` after rotation → fails; new key succeeds; counters unchanged. |
| R-06 | An envelope may optionally reference an external agent-identity record; the reference is stored, not trusted for authorization. | S4 | UC-01 | on-chain (store) / client (lookup) | Authorization never depends on identity account contents. |

## 4.2 Policy (the envelope)

| ID | Requirement | Src | UC | Where | Test |
|---|---|---|---|---|---|
| R-07 | A single payment cannot exceed the per-payment cap. | S1, S2, S5 | UC-02/06 | on-chain | `spend(per_tx_cap + 1)` → fails. |
| R-08 | Cumulative payments inside the current time window cannot exceed the window cap. | S1, S2 | UC-06 | on-chain | Two spends summing to cap+1 in one window → second fails. |
| R-09 | The time window is a fixed-duration bucket that resets when `now ≥ window_start + window_seconds`. | A5 red-team | UC-06 | on-chain | Spend, advance clock past window, spend again to full cap → succeeds. |
| R-10 | Cumulative payments over the envelope's life cannot exceed the lifetime cap. | S1 (min-loss) | UC-06/08 | on-chain | Many windows; total > lifetime → fails. |
| R-11 | No payment can occur after the mandate's expiry timestamp. | S2 ("expire Friday") | UC-06 | on-chain | Clock past `expires_at` → `spend` fails. |
| R-12 | Policy values must be internally consistent: `per_tx ≤ window ≤ lifetime`, `threshold ≤ per_tx`, `expires_at > now`, `window_seconds > 0`. | P1 | UC-02 | on-chain | Each inverted relation → `set_mandate` fails. |
| R-13 | Editing the policy never resets or lowers the spent counters. | P1 red-team | UC-02 | on-chain | Spend, `set_mandate`, read counters → unchanged. |
| R-14 | Worst-case loss from the Agent key alone is bounded by `min(lifetime_cap, 2 × window_cap + …)` and is documented as a number the Principal chose. | S1 | derived | on-chain (property) | Property test over random spend sequences. |
| R-15 | All counter and cap arithmetic is overflow-checked. | security checklist | all | on-chain | Near-`u64::MAX` values → fails, not wraps. |

## 4.3 Recipient scope

| ID | Requirement | Src | UC | Where | Test |
|---|---|---|---|---|---|
| R-16 | A payment can only be made to a recipient the Principal has explicitly allowlisted. | S1 (tickers), S2, S5 | UC-04/06 | on-chain | Spend to unlisted pubkey → fails. |
| R-17 | The allowlist is keyed by recipient public key; the program never receives or interprets URLs or domains. | A4 red-team | UC-04 | on-chain / client | No string domain in instruction data. |
| R-18 | The Principal can remove a recipient; subsequent payments to it fail. | S2 | UC-05 | on-chain | Remove then spend → fails. |
| R-19 | The envelope cannot be allowlisted as its own recipient. | security | UC-04 | on-chain | `add_payee(leash)` → fails. |
| R-20 | Domain→pubkey resolution is a client responsibility and is displayed to the Principal at allowlist time. | A4 | UC-04 | client | UI shows resolved pubkey before signing. |

## 4.4 Autonomous payment

| ID | Requirement | Src | UC | Where | Test |
|---|---|---|---|---|---|
| R-21 | A payment that passes all checks completes without any human action. | PS-1 (autonomy) | UC-06 | on-chain | Valid spend with only agent signature → succeeds. |
| R-22 | Validation, transfer, counter update and receipt emission occur in one atomic transaction; failure of any part leaves no state change. | Solana core | UC-06 | on-chain | Force CPI failure (e.g., frozen payee ATA) → counters and vault unchanged. |
| R-23 | Payments are settled in the configured stablecoin only (USDC); wrong mint fails. | scope | UC-03/06 | on-chain | Deposit/spend with other mint → fails. |
| R-24 | Token movement uses `transfer_checked` with the mint's decimals. | cohort pattern | UC-03/06/08/13 | on-chain | Code review + decimals mismatch test. |
| R-25 | Each payment carries an opaque 32-byte intent reference supplied by the client (hash of the 402 challenge / trade rationale). | S1 (thesis trace), S4 | UC-06 | client → on-chain (stored/emitted) | Receipt contains the exact hash passed. |
| R-26 | If the recipient's token account does not exist, the Agent (not the vault) pays for its creation. | P-ATA red-team | UC-06 | on-chain | Vault USDC balance decreases by exactly `amount`. |
| R-27 | A payment above the escalation threshold is rejected by the autonomous path. | S1, S5 | UC-06 | on-chain | `spend(threshold + 1)` → fails with a distinct error. |

## 4.5 Human-in-the-loop exceptions

| ID | Requirement | Src | UC | Where | Test |
|---|---|---|---|---|---|
| R-28 | The Agent can record a request for a payment above its threshold without moving funds. | S5 ($500 rule) | UC-07 | on-chain | Vault balance unchanged after request. |
| R-29 | The Agent pays the storage cost of its own request. | P3 red-team | UC-07 | on-chain | Rent debited from agent, not principal/vault. |
| R-30 | Only the Principal can approve a pending request. | PS-1 | UC-08 | on-chain | Agent/anyone approve → fails. |
| R-31 | Approval executes the transfer, updates lifetime spend, emits a receipt and closes the request atomically. | atomicity | UC-08 | on-chain | Partial failure leaves request open and vault unchanged. |
| R-32 | An approved request cannot be executed twice. | P2 red-team | UC-08 | on-chain | Re-submit approval → fails (account closed). |
| R-33 | A request has a time-to-live; after expiry anyone may close it and return rent to the requester. | P3 | UC-09 | on-chain | Third party closes after TTL → succeeds; before TTL → fails. |
| R-34 | The Principal can cancel a pending request at any time. | S5 | UC-09 | on-chain | Principal cancel before TTL → succeeds. |
| R-35 | Human-approved payments are counted against lifetime but not the autonomous window (explicit exception; documented). | design decision | UC-08 | on-chain | Window counter unchanged after approval. |
| R-36 | The Principal is notified of pending requests off-chain. | S1, S5 | — | client / indexer | Webhook fires on request event. |
| R-37 *(open)* | The same external intent cannot be re-requested after its request account is closed. | P2 (gap found in review) | UC-07 | on-chain | **Decision needed:** nonce in seeds, consumed-intent marker, or accept. |

## 4.6 Emergency controls

| ID | Requirement | Src | UC | Where | Test |
|---|---|---|---|---|---|
| R-38 | The Principal can pause the envelope; while paused, no payment or approval can execute. | S1 ("stop from my phone") | UC-10 | on-chain | `spend`, `approve` while Paused → fail. |
| R-39 | Pause does not alter balances or counters. | S1 | UC-10 | on-chain | Read state before/after. |
| R-40 | The Principal can resume; counters and policy resume unchanged. | — | UC-11 | on-chain | Resume then spend within remaining cap → succeeds. |
| R-41 | The Principal can withdraw any amount of unspent funds at any time; withdrawal does not count as agent spend. | S2, S5 | UC-13 | on-chain | Counters unchanged after withdraw. |
| R-42 | Emergency actions require only the Principal's signature — no third party, keeper, or hosted service. | D3 red-team | UC-10–13 | on-chain | Tx with only principal signer succeeds. |

## 4.7 Auditability & counterparty verification

| ID | Requirement | Src | UC | Where | Test |
|---|---|---|---|---|---|
| R-43 | Every executed payment emits a receipt containing: envelope, agent identity (if any), recipient, amount, intent reference, sequence number, slot, and whether it was human-approved. | S3, S4, S5 | UC-06/08 | on-chain (event) | Parse log; all fields present. |
| R-44 | Receipts are strictly sequenced per envelope (monotonic nonce). | S3 (support tickets) | UC-06/08 | on-chain | Two spends → nonces n, n+1. |
| R-45 | A third party can read an envelope's policy, status, and remaining budget from public state without any API from us. | S4 (Lena) | derived | on-chain (account layout) | Fetch account, decode, compute remaining. |
| R-46 | The on-chain signer of a mandated transfer is the envelope's PDA, so the transfer itself proves a mandate was applied. | S4 | UC-06 | on-chain | Inspect tx: vault authority = Leash PDA. |
| R-47 | Receipt history is queryable per envelope by an off-chain indexer. | S3 | — | indexer | Indexer returns all receipts for a Leash. |

## 4.8 Lifecycle & recovery

| ID | Requirement | Src | UC | Where | Test |
|---|---|---|---|---|---|
| R-48 | The Principal can close an envelope only when its vault is empty. | rent hygiene | UC-14 | on-chain | Close with balance > 0 → fails. |
| R-49 | Closing returns all rent to the Principal. | — | UC-14 | on-chain | Lamports check. |
| R-50 *(open)* | An envelope cannot be closed while requests are pending. | review gap | UC-14 | on-chain | **Decision needed:** store `open_escalation_count` to make this enforceable. |
| R-51 | Removing a recipient returns its rent to the Principal. | — | UC-05 | on-chain | Lamports check. |

## 4.9 Trust & non-functional

| ID | Requirement | Src | Where | Test |
|---|---|---|---|---|
| R-52 | No hosted component is required for any value-moving action. | D3 | architecture | Run all UCs with CLI + RPC only. |
| R-53 | No protocol admin key exists in the spend path. | D1 | on-chain | Grep constraints; `spend` accounts contain no authority. |
| R-54 | Program upgrade authority is burned after audit (documented, not V1-enforced). | D1 | ops | — |
| R-55 | The program never accepts a user-supplied program ID for CPI. | security checklist | on-chain | `token_program` constrained to Token/Token-2022. |
| R-56 | Every guard above has a negative LiteSVM test. | cohort | tests | CI. |
| R-57 | Client SDK exposes the policy as a plain data structure so agents receive *the constraint and the remaining room*, not just "rejected." | agent-mandate insight | client | SDK returns `remaining_window`, `remaining_lifetime`. |

## 4.10 Explicitly out of scope for V1 (and why)

| Item | Why not now | Where it could go |
|---|---|---|
| Semantic thesis alignment ("does this trade fit my strategy?") | Judgement, not arithmetic; cannot be enforced by a program. | Agent/client layer; Leash records `intent_hash` and gates by threshold so the human reviews it. |
| Enforcing on Robinhood or any off-chain broker | Leash controls on-chain USDC only. | Motivating scenario; on-chain trading venues (perps, AMMs) as allowlisted payees are a V2 path. |
| Yield on idle vault, confidential balances | Distraction from proving the control loop. | V2 (Token-2022). |
| Hierarchical / multi-agent mandates (S6) | Complexity. | V2 — keep `principal` as a generic Pubkey so a Leash PDA can be a principal. |
| Category / merchant-type limits | Requires trusted metadata about payees. | V2 via label registry. |
| Compute/API-bill circuit breakers (Mandiant case) | Not a payment. | Adjacent product; do not overclaim. |

---

# 5. From requirements to the architecture diagram

The V1 diagram (draw.io) stays at **12 primary boxes, 3 swimlanes**. Requirement IDs become **arrow labels and notes**, so a reviewer can trace every visual element to a "why."

## 5.1 Element → requirement map

| Diagram element | Requirements it visualises |
|---|---|
| **Principal** box (key icon) | R-01, R-03, R-05, R-30, R-38–R-42 |
| **AI Agent** box (key icon, "limited authority") | R-02, R-21, R-27, R-28 |
| **x402 Service / Payee** box | R-16, R-45, R-46 (seller can verify) |
| **Agent SDK** box | R-17, R-20, R-25, R-36, R-57 |
| **Leash Program** box ("enforces mandate") | R-07–R-13, R-15, R-22, R-53, R-55 |
| **Leash PDA** cylinder (seeds `["leash", principal, agent_id]`) | R-01, R-13, R-14, R-44, R-45 |
| **PayeeEntry PDA** cylinder (seeds `["payee", leash, payee]`) | R-16, R-18, R-19, R-51 |
| **Escalation PDA** cylinder (seeds `["escalation", leash, intent_hash]`) | R-28, R-29, R-32, R-33, R-37, R-50 |
| **Vault ATA** cylinder ("authority = Leash PDA") | R-04, R-23, R-41, R-46, R-48 |
| **Payee ATA** cylinder | R-26 |
| **Token Program** box (`transfer_checked`) | R-24, R-55 |
| **Receipt Event / Indexer** box | R-43, R-44, R-47 |
| **Atomic boundary** (thick outline around checks → transfer → counters → receipt) | R-22, R-31 |
| **Decision diamond** "amount ≤ threshold?" | R-27, R-28 |
| **Red dashed failure arrow** | R-07, R-08, R-10, R-11, R-16, R-38 (one note: "any failed check → no payment, no state change") |
| **Setup arrows** (Principal → Program / Vault / PayeeEntry) | R-03, R-04, R-12, R-16 |
| **Security note** panel | R-02, R-42, R-46, R-52, R-53 |
| **Footer** "open decisions" | R-37, R-50 |

## 5.2 Arrow labels for the main flow (use these verbatim)

```
1  challenge (amount, payee)                      x402 Service → Agent
2  hand off request                                Agent → Agent SDK
3  spend(amount, intent_hash) · signed by Agent    Agent SDK → Leash Program        [R-21, R-25]
4  read policy + allowlist                         Leash Program → Leash PDA, PayeeEntry   [R-16]
5  checks: active · unexpired · payee · caps       note on Leash Program            [R-07 R-08 R-10 R-11 R-38]
6  CPI transfer_checked · PDA-signed               Leash Program → Token Program    [R-24, R-46]
7  USDC                                            Vault ATA → Payee ATA            [R-23]
8  counters++ · emit Receipt                       Leash Program → Receipt/Indexer  [R-43, R-44]
9  payment proof                                   Agent → x402 Service
X  any check fails → no payment, no state change   red dashed                       [R-22]
```

## 5.3 Draw.io prompt delta

Use the simplified prompt already agreed (12 boxes, 3 swimlanes). Add one instruction:

> "Append the bracketed requirement IDs shown in the arrow list to each arrow label in small grey text. Add a legend entry: 'R-xx = requirement ID, see Leash requirements doc §4.'"

That is the only change; do not add boxes.

---

# 6. Team session plan

## 6.1 Decisions to make in the kickoff (60 min)

| # | Decision | Options | Recommendation |
|---|---|---|---|
| D1 | Primary problem statement | PS-1 / PS-2 / PS-3 | **PS-1**, with PS-2 as the "why on-chain" sub-claim and S1 (Robinhood) as the opening story. |
| D2 | Escalation replay (R-37) | (a) nonce in seeds; (b) consumed-intent marker PDA; (c) accept re-request after close | **(a)** — cheapest; seeds `["escalation", leash, intent_hash, nonce]` where nonce = leash.nonce at request time. |
| D3 | Enforceable close (R-50) | add `open_escalation_count` to Leash | **Yes** — one `u16`, incremented/decremented in UC-07/08/09. |
| D4 | Window semantics naming (R-09) | "rolling" vs "fixed bucket" | Call it **"spending window (fixed bucket)"** everywhere; drop "rolling." |
| D5 | Fee accounting (UC-06) | gross vault debit vs payee-net | **Gross** — `amount` is what leaves the vault; payee receives `amount − fee`. |
| D6 | V1 demo storyline | S1 vs S2 | **S2 (Priya)** for the live demo — fully on-chain; open with S1 as the personal hook. |

## 6.2 Five-question interview script (run 5 before the diagram is final)

1. Tell me about the last time an agent you run needed to spend money. Whose key did it use?
2. What limits exist today, and where do they live (prompt, SDK, wallet provider, none)?
3. What is the failure you actually worry about — loop, injection, key leak, wrong recipient, wrong amount?
4. If it happened tonight, how would you find out, and how fast could you stop it?
5. Would you lock funds in a dedicated vault to get a hard cap? What amount would make that worth it?

Log answers in `capstone/interviews.md` in your working repo. We are validating **the problem** (Q1–4) and **one solution assumption** (Q5), not asking "would you use Leash."

## 6.3 Open questions for the team

- Which of R-07/R-08/R-10 matters *most* to real users? (Interviews.) It decides what the demo highlights.
- Does anyone on the team have a live x402 seller we can use as the demo payee? (S4.)
- Is a Principal-facing UI in V1 scope, or CLI + a notification webhook only? (R-36 can be a script.)
- Who owns: program (Anchor) / SDK + intent hashing / tests + devnet proof / diagram + deck?

## 6.4 What "done" looks like for this phase

- [ ] Primary problem statement agreed and written in one sentence on slide 1.
- [ ] D2–D5 decided and reflected in the LOI's Part D matrix.
- [ ] Every V1 box/arrow on the diagram carries at least one R-id.
- [ ] Every R-id in §4.1–4.8 marked on-chain has a named negative test.
- [ ] ≥ 5 interviews logged.

---

# Appendix A — Source list

- Google Mandiant, *AI Risk and Resilience Report 2026*, Case Study 6 (via Help Net Security, SecurityBrief, YSecurity, Threatwire — Sept 2026)
- arXiv 2606.04056, *Token Budgets: An Empirical Catalog of 63 LLM-Agent Budget-Overrun Incidents* (June 2026)
- OECD.AI incident 2026-05-04, Grok/Bankr prompt-injection wallet drain; valens.me and blockcritics.com write-ups
- KuCoin / New Claw Times / CryptoTicker on the "Lobstar Wilde" OpenClaw mis-transfer (Feb–Mar 2026); NOV1.ai six-model Hyperliquid experiment
- answeroverflow.com OpenClaw Farcaster custody-wallet drain thread (Feb 2026); Decrypt / OX Security OpenClaw phishing report
- PolicyLayer, "Robinhood let AI agents trade real money. The only safety check is optional."; Saving to Invest; Wall Street Survivor; Vorp Labs; stacker.news thread on Robinhood MCP (July–Sept 2026)
- agentspay.ai, "Can You Set Spending Limits on x402?" and "Stop Prompt Injection From Draining an AI Agent Budget"
- SecProve Agent Firewall; OpenAlice Guards & Risk docs; ML4T Live `SafeBroker`; `cataloraxyz/agent-mandate`
- npm `@otomat/agent-wallet-kit`, `@x402-guard/policy`; GitHub `ogazboiz/sage`, `Azzaraell/agent-payments-x402`
- Google AP2 specification (Human Present / Human Not Present, Intent & Payment Mandates)
- Medium, "I Lost 40% of My Trading Account by Trusting an AI"; dev.to "Marty" series; moltbook "The rogue bot problem"; August Dispatch tbot post-mortem
- WTOP "Dear Clever Credit" (caregiver authorized-user limits); PCBB family-banking report (Mar 2026); myLPA case study; Elder Safety Hub on True Link

# Appendix B — Traceability: LOI use case → requirements

| UC | Requirements |
|---|---|
| UC-01 create_leash | R-01, R-03, R-06 |
| UC-02 set_mandate | R-07–R-13 |
| UC-03 deposit | R-04, R-23, R-24 |
| UC-04 add_payee | R-16, R-17, R-19, R-20 |
| UC-05 remove_payee | R-18, R-51 |
| UC-06 spend | R-07–R-11, R-15, R-16, R-21–R-27, R-43, R-44, R-46 |
| UC-07 request_escalation | R-28, R-29, R-37 |
| UC-08 approve_escalation | R-30–R-32, R-35, R-43 |
| UC-09 cancel_escalation | R-33, R-34 |
| UC-10/11 pause/resume | R-38–R-40 |
| UC-12 rotate_agent_key | R-05 |
| UC-13 withdraw | R-41 |
| UC-14 close_leash | R-48–R-50 |
| cross-cutting | R-14, R-42, R-45, R-47, R-52–R-57 |
