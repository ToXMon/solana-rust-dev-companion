# Token-2022 & extensions — cohort notes

Everything the cohort covered on Token-2022: the three extension integration patterns, confidential transfers, CPI guard, permanent delegate, and delegate-to-program. Read this before touching Token-2022 mints/accounts. Related: `anchor-practical.md`, `mistakes-and-prevention.md`.

**Transcription caveat:** `2026_09_14` is largely unintelligible between ~00:02 and ~00:27; `2026_09_16` degrades after ~01:06; `2026_09_18` is intelligible only for the first ~52 minutes (the confidential-transfer-fees walkthrough is not captured). Insights are drawn from the intelligible segments.

## SPL token basics

- **SPL token operations:** create mint, create/fund token account, mint to ATA, transfer, burn. (`2026_08_26`)
- **Token 2022 program capabilities:** transfer fees, transfer hooks, native metadata, permanent delegate ("God mode" institutional mint/burn authority). (`2026_08_26`, ~00:20:50–00:24:39)
- **Metaplex Token Metadata:** metadata is stored in a PDA derived from the mint. Wrong-mint bugs are common when chaining mint + metadata instructions. (`2026_08_26`, ~01:16:32–01:18:28)

## Three integration patterns for Token-2022 extensions (Sep 14/16)

- **Three scenarios:** (1) Anchor supports it declaratively via extension constraints — just declare accounts and constraints and Anchor CPIs the init sequence for you; (2) Anchor does not support it — you manually allocate, create the account via System Program CPI, initialize the extension(s), then initialize the mint; (3) Anchor's `Mint` type doesn't expose the extensions — you read the raw account bytes yourself and deserialize via `StateWithExtensions` (TLV layout). (`2026_09_14`, ~00:27:29–00:32:28; recapped `2026_09_16`, ~00:01:39–00:18:21)
- **The canonical mint-with-extensions flow:** "You allocate the space, you create the account, you initialize the extensions, *then* you initialize the mint." Extensions can only be appended before mint initialization — once the mint is initialized you cannot add extensions. (`2026_09_16`, ~00:17:28–00:17:47; `2026_09_14`, ~01:22:49–01:23:53)
- **Extensions consume space — allocate at fully extended length.** Always compute size via `ExtensionType` helpers (`get_account_len` / `try_calculate_account_len`) rather than a fixed space constant, because extension sizes vary. (`2026_09_14`, ~01:24:48–01:25:19; `2026_09_16`, ~00:09:54–00:10:28)
- **TLV (type-length-value) layout:** Token-2022 accounts append extension data after the base layout, each entry self-describing. "The layout is expanded to accommodate extensions and it can only be appended from the last index." (`2026_09_14`, ~01:02:25–01:03:18; ~01:31:36–01:32:28)
- **Owner ≠ authority.** "Sometimes they are two different concepts. The owner is not always the authority." (`2026_09_14`, ~00:46:45–00:47:05)

### Practical tips

- Anchor provides only ~7 extension constraints out of ~28 Token-2022 extension types; for the rest (e.g., transfer fee, transfer hook, confidential transfers) you must do the manual init sequence yourself. (`2026_09_14`, ~00:28:10–00:29:34; `2026_09_16`, ~00:04:51–00:06:16)
- Use `Interface` for the token program when you want to accept *any* program implementing the token interface; use `Program<Token2022>`/`Program<Token>` when you want to constrain to exactly one. (`2026_09_14`, ~00:49:44–00:53:16)
- When the program creates/initializes an account itself (rather than `init` in constraints), pass `UncheckedAccount` so Anchor doesn't pre-validate — "we want that to be inferred at runtime." Document why. (`2026_09_14`, ~00:47:46–00:48:40; ~00:55:09–00:55:34)
- A new mint keypair must sign `create_account` — "the address is required to sign to prove that it owns the account." (`2026_09_14`, ~00:54:14–00:54:38; `2026_09_16`, ~00:48:36–00:48:57)
- To read extensions off a mint Anchor can't fully type: get the raw account data and CPI into the Token-2022 library, asking it to "interpret these raw bytes as a mint containing extensions" via `StateWithExtensions`. The same technique works in plain Rust clients. (`2026_09_14`, ~01:10:42–01:15:55)
- To compare SPL Token vs Token-2022 mint layouts hands-on, use Solana CLI account inspection (`solana account` / explorers) and look at the raw data. (`2026_09_14`, ~01:33:06–01:33:35)
- Small programs can keep instructions in `lib.rs`; larger programs should keep separation of concerns (instructions/constants/errors modules). (`2026_09_14`, ~00:34:10–00:35:24)

### Example programs

- **Declarative extension mint:** mint with `metadata pointer` + `close authority` via Anchor extension constraints. (`2026_09_14`, ~00:43:33–00:45:24)
- **Manual extension mint (`create_mint_with_fee`):** transfer fee config (basis points + max fee) + mint close authority initialized via manual CPI sequence. (`2026_09_14`, ~00:46:07–00:46:45; `2026_09_16`, ~00:10:53–00:11:13)
- **Raw-deserialization mint:** unchecked account + owner constraint enforced manually by deserializing and validating the TLV layout — guards against "any account from any program whose bytes look like an initialized mint being accepted." (`2026_09_14`, ~01:01:19–01:04:05)
- Suggested self-directed exploration: research the SPL mint vs. Token-2022 mint byte layout and the TLV format; try rewriting the extension flows in plain Rust to see what Anchor abstracts. (`2026_09_14`, ~00:29:34–00:30:24; ~01:32:28–01:33:35)

## Confidential transfers (Sep 16)

- **Confidential balances are cryptographic state, not plaintext.** A confidential token account holds encrypted representations of balances; the Token-2022 program operates on the ciphertext via ZK/ElGamal proofs rather than `amount -= x` arithmetic like an escrow. (`2026_09_16`, ~00:24:19–00:25:07; ~00:33:27–00:33:43)
- **Pending balance = staging area.** "Always think about the staging area. Anything that has to do with transfer, you have to go through that staging area." Received confidential tokens sit in a pending (encrypted) state until the owner calls `apply_pending_balance`. (`2026_09_16`, ~00:28:01–00:30:47; ~00:33:49–00:34:26)
- **Lifecycle state machine:** configure → deposit → apply pending balance → transfer → apply pending balance → withdraw.
- Deposit into a confidential account needs **no ZK proof** — the amount was already public before moving. Proofs are needed for transfers/withdraws where the amount must stay hidden. (`2026_09_16`, ~01:02:06–01:02:47)
- Apply pending balance **before** withdraw as well as after receiving transfers — withdraw operates only against the available balance, and the program cannot compute the new available balance for you because it has no access to the decryption key; the owner supplies it. (`2026_09_16`, ~00:35:50–00:37:40; ~01:04:52–01:05:39)
- Sender doesn't need to apply pending balance to deposit/send — only the receiver does (and anyone before withdrawing). "It works both ways." (`2026_09_16`, ~00:37:55–00:44:24)
- Extra deps used for confidential transfers: a proof-generation crate, a proof-extraction package, a zk/elgamal proof interface crate, and `bytemuck` for byte-level correctness. (`2026_09_16`, ~00:46:14–00:47:17)
- **Confidential mint + token account example:** configure confidential transfer extension on mint, deposit public→confidential balance, apply pending balance, confidential transfer, withdraw. (`2026_09_16`, ~00:22:43–00:24:18; ~00:48:07–01:06:00)
- **Privacy framing:** on a public ledger "everyone can see all the metadata"; confidential transfers give encrypted-value privacy while keeping auditability via the cryptographic layer. (`2026_09_16`, ~00:20:26–00:21:24)

## CPI guard, permanent delegate, delegate-to-program (Sep 18)

- **Three ways a front-end / dApp can move a user's funds:**
  1. A direct Token Program instruction inside the signed transaction.
  2. An approved instruction + CPI transfer where the **program** is the authority (e.g., AMMs, escrow).
  3. An opaque program instruction + CPI transfer where the **user's** authority is reused — the user signs without visibility into what the program will do. This third path is a "black box" and the threat model CPI guard is designed to mitigate. (`2026_09_18`, ~00:11:51–00:17:15)
- **CPI guard = "prohibit setting actions inside CPIs."** An extension on the token account that tells the Token Program to reject CPI calls that use the user's authority, so a malicious program cannot silently drain a token account during an unrelated user-signed transaction. The user can still choose to disable or override it. (`2026_09_18`, ~00:11:18–00:17:15)
- **Permanent delegate is mint-level, not account-level; CPI guard is account-level.** CPI guard lives on the token account; permanent delegate lives on the mint and can bypass the CPI guard. (`2026_09_18`, ~00:17:40–00:19:46)
- **Permanent delegate as institutional "god mode" / seizure tool.** A mint-level authority that can move or burn any token of that mint without owner consent and cannot be revoked. Useful for compliance/asset seizure, but in a permissionless DeFi context it is usually a bug or attack vector — a reason to *reject* a mint. (`2026_09_18`, ~00:17:16–00:20:43; ~00:43:10–00:47:18)
- **Owner ≠ authority, again.** A permanent delegate does not need the token account owner to sign; the delegate itself is the signer. (`2026_09_18`, ~00:45:50–00:46:02)
- **Delegate-to-program pattern:** a user approves a program PDA to spend tokens on their behalf; Anchor's extension constraints simplify the code compared with a manual CPI sequence. (`2026_09_18`, ~00:32:27–00:44:28)

### Practical tips (Sep 18)

- Prefer `transfer_checked` over `transfer` — validates mint/decimals early, required for Token-2022 compatibility. Used throughout the delegate examples. (`2026_09_18`, ~00:30:20–00:30:34; ~00:44:10–00:44:20)
- Use `Interface` / `InterfaceAccount` to accept both SPL Token and Token-2022. (`2026_09_18`, ~00:27:20–00:27:32)
- When Anchor has a declarative extension constraint, use it. For permanent delegate, Anchor abstracts the manual "allocate → create account → initialize extension → initialize mint" sequence into a single constraint, unlike confidential transfers where manual CPI is required. (`2026_09_18`, ~00:40:14–00:40:54)
- Token account data exposes delegation fields (`delegate`, `delegated_amount`) — useful for debugging and for writing allow/deny logic. (`2026_09_18`, ~00:41:02–00:41:54)

### Example programs (Sep 18)

| Example | What it shows | Source segment |
|---------|---------------|----------------|
| **CPI guard enabled/disabled on token account** | Blocks user-authority CPI calls unless the user explicitly disables the guard; prevents opaque programs from draining tokens. | `2026_09_18`, ~00:15:12–00:17:15 |
| **Delegate-to-program PDA** | User approves a program PDA to spend a fixed amount; program then calls `transfer_checked` with program authority. Anchor abstracts the approve CPI. | `2026_09_18`, ~00:32:27–00:44:28 |
| **Permanent delegate seizure** | A mint-level permanent delegate transfers/burns tokens from any account of that mint without owner consent, and without revocation. | `2026_09_18`, ~00:43:10–00:47:18 |
| **Arbitrary Token-2022 mint allowlist** | When accepting unknown mints, reject those carrying the permanent-delegate extension because it is "more of a problem than a feature" for DeFi protocols. | `2026_09_18`, ~00:46:18–00:47:18 |
| **Confidential transfer fees** | Planned but not captured in intelligible audio; topic was introduced as the most intricate part of the day. | `2026_09_18`, ~00:10:37; ~00:20:57; ~00:50:52 |

Related: `rwa-tokenized-fund.md` (ScaledUiAmount, DefaultAccountState KYC freeze in production), `defi-amm.md`, `quotes.md`.
