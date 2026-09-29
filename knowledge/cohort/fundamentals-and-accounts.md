# Fundamentals & accounts — cohort mental models

Core Solana mental models as taught in the cohort: accounts, rent, PDAs, ATAs, CPIs, atomicity. Read this when you need the intuition behind the account model. For instructor phrasing, see `quotes.md`; for Anchor mechanics, see `anchor-practical.md`.

## Mental models, analogies, and repeated teaching phrases

- **Accounts are the unit of state; programs are the unit of code.** "Everything on Solana is an account." (`2026_08_24`, ~00:03–00:05)
- **Rent is more like a deposit than a fee.** It can be reclaimed when an account closes. (`2026_08_24`, ~00:21:45–00:22:51)
- **PDAs = "program-controlled accounts without a private key."** They let a program authorize actions that would otherwise require a keypair, which is powerful for vaults, escrow, recurring payments, multisig, game state, and agents. (`2026_08_24`, ~01:19:39–01:23:56)
- **"All ATAs are PDAs, but not all PDAs are ATAs."** ATAs are a specific PDA derived from `(wallet, mint, token_program)`. (`2026_08_26`, ~00:18:42–00:20:44)
- **Atomicity as a safety guarantee.** "We want all of this to happen in the program… no trusted parties." In escrow, the maker deposit + taker payment + vault release must happen atomically or roll back. (`2026_09_02`, ~00:35:00–00:36:54)
- **Reason from actions first, then state, then accounts.** Repeated instructor prompt: *What action are we performing? What state must persist? Which accounts are read? Which are mutated? Who pays? Which seeds identify the state?* (`2026_08_31`, vault section; `2026_09_02`, ~00:46:38–00:47:11)
- **"Transfer moves balances; mint increases supply; burn decreases supply."** (`2026_08_26`, ~00:20:50–00:24:39)
- **A vault program needs two conceptual buckets:** a *state account* (who owns it, bumps, metadata) and a *token account* (the actual balance). Whether these live in one or two on-chain accounts is a design/optimization choice, not a hard rule. (`2026_09_02`, ~00:22:50–00:31:40)

## PDAs, seeds, and bumps

- PDA derivation searches bumps from 255 down to 0 to find an off-curve address. The probability of failing to find one is astronomically low. (`2026_08_26`, ~00:04:12–00:05:50)
- Use the **canonical bump** (the first valid one found) and store it to avoid repeated derivation cost. (`2026_08_31`, ~01:03:47–01:09:46)
- If an account is a PDA and the program must sign for it, use `CpiContext::new_with_signer` and pass the signer seeds. For a plain keypair signer you can use `CpiContext::new`. (`2026_09_02`, ~01:14:10–01:16:48)
- A PDA used only as an authority does not need to be initialized. If the program only needs its signature and stores no data, deriving the address is enough. (`2026_09_21`, ~00:36:40–00:39:54) — see `nfts-metaplex-core.md`.
- Verify PDA authorities even when unchecked. The collection update-authority PDA is an `UncheckedAccount` holding no data, but the constraint still verifies it derives from the expected seeds (`"update_authority"` + collection key + stored bump). (`2026_09_23`, ~00:38:26–00:39:00)

## PDA-driven use cases

- **Recurring payments / subscriptions:** Use PDAs to hold scheduled state without a private key. (`2026_08_24`, ~01:19:39–01:23:56)
- **Multisig coordination and game state:** PDAs provide deterministic, program-signable state. (`2026_08_24`, ~01:19:39–01:23:56)
- **Agent-controlled transactions:** PDAs let programs act on-chain without holding a user keypair. (`2026_08_24`, ~01:19:39–01:23:56)

Related: `anchor-practical.md` (mechanics), `quotes.md` (verbatim phrasing), `mistakes-and-prevention.md` (common account-model bugs).
