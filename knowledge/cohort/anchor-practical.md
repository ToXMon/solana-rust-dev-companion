# Anchor practical guidance — cohort notes

Hands-on Anchor mechanics from the cohort: account types, constraints, init/close, discriminators, token CPIs, vault/escrow design. Read this when writing or reviewing Anchor account structs and CPIs. For account-model intuition see `fundamentals-and-accounts.md`; for Token-2022 specifics see `token-2022-and-extensions.md`.

## Anchor account types and constraints

- **Account types** describe *what kind* of account is expected; **constraints** are validation rules applied on top. Both are usually needed together. (`2026_09_02`, ~00:15:32–00:21:17)
- `AccountInfo` is deprecated; prefer `UncheckedAccount` when you truly need an unchecked account and document why. (`2026_09_02`, ~00:05:04–00:05:45)
- `Box<T>` moves the account data to the heap and keeps only a reference on the stack. Use it when stack-space errors appear; do not box everything because heap access is slower. The compiler often tells you how many bytes you are over stack. (`2026_09_02`, ~00:06:29–00:10:44)
- `AccountLoader` is for zero-copy access; useful for large/complex state. (`2026_09_02`, ~00:05:45–00:06:15)
- `Interface` / `InterfaceAccount` lets one program accept either SPL Token or Token 2022 mints/accounts. The same pattern is useful for migration contracts (V1/V2 state). (`2026_09_02`, ~00:10:44–00:12:35)
- `Signer`, `SystemAccount`, `Sysvar` (clock/rent), and `Option<Account>` are standard building blocks. (`2026_09_02`, ~00:12:35–00:13:40)

## `init`, `init_if_needed`, `mut`, and `close`

- `init` fails if the account already exists; `init_if_needed` should only be used when repeat initialization is acceptable. (`2026_08_31`, ~01:10:38–01:15:04)
- Any account whose lamport/data field changes needs `mut` in the constraints. (`2026_08_31`, ~01:03:47–01:09:46)
- `close = destination` sends rent to `destination` and zeroes the account. For token accounts, call the token program's `close_account` CPI instead. (`2026_09_02`, ~01:21:03–01:26:03)

## Custom discriminators

- Anchor normally hashes `global:<instruction_name>` or `account:<AccountName>` and takes the first 8 bytes. For simple programs you can supply a custom discriminator (even a single byte) to save compute and instruction data. (`2026_09_02`, ~00:57:09–01:02:39)
- Store the discriminator in the account struct so it is checked on deserialization.

## Token CPIs

- Prefer `transfer_checked` over `transfer`; it validates decimals/mint early and is required for Token 2022 compatibility. (`2026_09_02`, ~01:17:46–01:19:00)
- For token mints/burns/ATA creation, build the CPI context with the right token-program ID.

## One-account vs. two-account vault

- A program-owned PDA can mutate its own `lamports` field directly without a System Program CPI, so you can store data + SOL in a single PDA. However, using `system_program::transfer` on a data-carrying account will fail. (`2026_09_02`, ~00:28:55–00:31:40)
- Mixing deposited user lamports with rent can confuse developers; separate state and vault accounts is the clearer default, but one-account is technically valid.

## Example programs

- **Vault:** PDA state account + token account; instructions: initialize, deposit, withdraw, close. Used to introduce Anchor constraints, stored bumps, and System/Token CPIs. (`2026_08_31`; `2026_09_02`)
- **Escrow:** Trustless two-party swap. Maker deposits token A into a program-owned vault and publishes desired token B amount. Taker sends token B to maker and receives token A atomically. Refund returns funds to maker. (`2026_09_02`, ~00:33:51–00:39:14)

## Quick reference: Anchor constraint/account-type decision tree

```text
Need custom state deserialized?           -> Account<MyState>
Need a signer (keypair)?                  -> Signer
Need system program ownership?            -> SystemAccount
Need either SPL Token or Token 2022?      -> InterfaceAccount<TokenAccount> / Interface<Mint>
Need a sysvar (clock, rent, etc.)?        -> Sysvar
Need to defer checks / pass raw account?  -> UncheckedAccount + document why
Stack overflow on build?                  -> Box<Account<T>> incrementally
Account may not exist?                    -> Option<Account<T>>
Initialize only if absent?                -> init_if_needed
Initialize and fail if exists?            -> init
Need to close account and reclaim rent?   -> close = destination
Need program to sign via PDA?             -> seeds + bump + CpiContext::new_with_signer
```

Related: `token-2022-and-extensions.md` (extension constraints, delegate patterns), `nfts-metaplex-core.md` (PDA authorities, plugin CPIs), `rwa-tokenized-fund.md` (roles, upgrade-authority gating, `emit_cpi!`), `testing-and-debugging.md`.
