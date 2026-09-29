# Solana Mental Model Primer

A 5-minute read for anyone who is about to write or review Solana code. These ideas come directly from the cohort transcripts and are the foundation of the build system.

## 1. Accounts are the unit of state; programs are the unit of code

On Solana, data lives in accounts, not in programs. A program is a stateless binary with an ID. When you want to store something, you create an account and make the program its owner.

> "Everything on Solana is an account."

**Implication:** Every instruction must explicitly list every account it reads or writes. You cannot reach for state that was not passed in.

## 2. PDAs give programs a deterministic identity

A Program Derived Address (PDA) is an address that lives off the normal key curve, so no private key exists. Only the program that knows the seeds can sign for it.

> "PDAs are program-controlled accounts without a private key."

**Pattern:** Use a PDA as a per-user vault, a config account, or an escrow state account. Store the canonical bump inside the account so you can re-verify the address later.

## 3. All ATAs are PDAs, but not all PDAs are ATAs

Associated Token Accounts (ATAs) are a special PDA derived from `(wallet, mint, token_program)`. They are just token vaults owned by the Token Program, with the wallet as authority.

## 4. Atomicity is your safety net

A transaction either fully succeeds or fully fails. This is why escrow works: the maker deposit, taker payment, and vault release happen in one transaction, or none of them happen.

> "We want all of this to happen in the program… no trusted parties."

## 5. Reason from actions → state → accounts

Before writing any instruction, ask:

1. What action is the user performing?
2. What state must change?
3. Which accounts are read? Which are written?
4. Who pays for the account creation/rent?
5. What seeds identify the state?

## 6. Anchor is macros over Rust, not magic

`#[account]`, `#[derive(Accounts)]`, and `#[program]` are Rust macros that generate validation and dispatch code at compile time. If something feels magical, run `anchor expand`.

## 7. Account types and constraints are different

- **Account type:** what kind of account you expect (e.g., `Account<'info, Escrow>`, `Signer<'info>`).
- **Constraint:** extra validation (e.g., `mut`, `has_one = maker`, `seeds`, `bump`).

You usually need both. Using only an account type can leave validation holes.

## 8. The vault has two conceptual buckets

A vault needs:
- A **state account** (who owns it, bumps, metadata)
- A **token/lamport holder** (the actual balance)

Whether these live in one or two on-chain accounts is a design choice, but separating them is the clearer default.

## 9. Use `transfer_checked`, not `transfer`

`transfer_checked` validates the mint and decimals. It is required for Token-2022 compatibility and avoids decimal mismatches.

## 10. Store bumps, re-derive seeds

When you create a PDA, capture `ctx.bumps` and store it. Every later instruction must re-derive the PDA with the same seeds and verify the stored bump.

## 11. Checked arithmetic is non-negotiable

```rust
vault.balance = vault.balance.checked_add(amount).ok_or(ErrorCode::Overflow)?;
```

## 12. Test after every instruction

Do not batch-implement four instructions and then test once. Write the test immediately after the instruction.

## 13. Devnet is the real classroom

Local tests tell you the code compiles and the logic is right. Devnet tells you the program actually lands, fees are paid, and accounts exist. Always capture the program ID and a transaction signature.

## 14. Solana is moving fast

Finality is getting faster (Alpenglow), ordering is getting fairer (Constellation), tokens are getting cheaper (p-token/Pinocchio), and the chain is becoming the settlement layer for agent payments (x402) and real-world assets (RWAs). When you pick a stack, check whether the latest version is already live.

## 15. Security is a habit, not an audit

Every instruction must answer: who is signing, what are they authorized to do, and what could go wrong if an attacker passes the wrong accounts? Build the answer into the constraints, not into comments.
