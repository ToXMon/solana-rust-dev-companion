# Leash — Phase 2: Actors, Use Cases, and On-Chain Requirements (Rough Draft)

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
