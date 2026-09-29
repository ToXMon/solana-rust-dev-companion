# Cohort transcript insights — topic index

Distilled notes from the Turbin3 Builders Cohort Q3 2026 session recordings, reorganized **by topic** so you can look things up directly. Read this index to pick the right file; read the topic file for the insight.

## Files

| File | Covers |
|------|--------|
| `fundamentals-and-accounts.md` | Accounts, programs, rent, PDAs, ATAs, CPI mental models |
| `anchor-practical.md` | Anchor account types, constraints, init/close, discriminators, token CPIs, vault design, decision tree |
| `testing-and-debugging.md` | Test strategy, assertions, LiteSVM setup, debugging tips, time-travel testing |
| `mistakes-and-prevention.md` | Beginner mistakes table + prevention strategies |
| `token-2022-and-extensions.md` | Token-2022 extensions, confidential transfers, CPI guard, permanent delegate |
| `nfts-metaplex-core.md` | NFT standards, Metaplex Core plugins, NFT staking workshop |
| `defi-amm.md` | AMM theory (CPMM/CLMM), order-flow toxicity, MEV, AMM implementation |
| `rwa-tokenized-fund.md` | Tokenized money-market fund reference program (NAV, roles, KYC gating) |
| `capstone-loi-and-architecture.md` | LOI guidance, assignments, architecture-diagram/atomic-requirements methodology |
| `ecosystem-and-tooling.md` | Anchor CLI, AVM, Crucible, Iris, Umi, toolchain notes |
| `quotes.md` | All verbatim instructor quotes, grouped by topic |
| `slides/` | Extracted slide-deck text (Solana intro, tokens, AMM theory, NFTs) |

## Curriculum map

| Date | Main topic | Key sub-topics | Source file |
|------|------------|----------------|-------------|
| **2026-08-24** | Solana fundamentals | Accounts, programs, instructions; transaction lifecycle; account ownership; rent/rent exemption; program-data accounts; PDAs; ATAs; CPIs; atomicity; transaction size/versioned transactions; bundles/block placement. | `Builders Cohort Q3 2026 - 2026_08_24 14_19 WEST - Recording.txt` |
| **2026-08-26** | Tokens, Token 2022, minting | PDA/ATA relationship; SPL token transfers; mint vs. burn vs. transfer; Token 2022 extensions; mint account creation flow; commitment levels; Metaplex metadata PDAs. | `Builders Cohort Q3 2026 - 2026_08_26 14_41 WEST - Recording.txt` |
| **2026-08-28** | NFTs, metadata, ecosystem standards | Programmable NFTs; rule sets; compressed vs. core NFTs; file/metadata upload (Iris); standalone vs. collection mints; ecosystem adoption tradeoffs; Anchor v2/Pinocchio/Quasar. | `Builders Cohort Q3 2026 - 2026_08_28 14_40 WEST - Recording.txt` |
| **2026-08-31** | Anchor fundamentals and vault program | Anchor runtime/entry point/dispatch; account validation; `anchor init/build/expand/test`; AVM; Crucible; vault PDA design; `init` vs. `init_if_needed`; stored bumps; System Program CPI. | `Builders Cohort Q3 2026 - 2026_08_31 14_49 WEST - Recording.txt` |
| **2026-09-02** | Escrow + Anchor account types deep dive | Anchor account types (`Account`, `UncheckedAccount`, `AccountLoader`, `Box`, `Interface`/`InterfaceAccount`, `Signer`, `SystemAccount`, etc.); constraints vs. account types; one-account vs. two-account vault; escrow `make`/`take`/`refund`; custom discriminators; PDA signer seeds; transfer checks. | `Builders Cohort Q3 2026 - 2026_09_02 14_52 WEST - Recording.txt` |
| **2026-09-04** | Testing and transaction anatomy | Building a transaction message manually; lite SVM test setup; program loading; instruction builders; assertions; IDL usage; Anchor build/toolchain quirks; ABI v2. | `Builders Cohort Q3 2026 - 2026_09_04 14_39 WEST - Recording.txt` |
| **2026-09-07** | AMM theory + capstone LOI guidance | LOI part 1: value prop, market/competitor analysis, target markets, use cases; market makers/order books; CPMM `x*y=k`; CLMM/DLMM/stable swap; arbitrage; impermanent loss; slippage; front-running/sandwiching; block builders/latency. | `Builders Cohort Q3 2026 - 2026_09_07 14_48 WEST - Recording.txt` |
| **2026-09-09** | AMM implementation | Initialize, deposit, withdraw, swap; LP tokens; constant-product curve library; slippage checks; optional fees; testing/modularization; capstone meet-and-greet preview. | `Builders Cohort Q3 2026 - 2026_09_09 14_53 WEST - Recording.txt` |
| **2026-09-14** | Token-2022 extensions in Anchor (three integration patterns) | Declarative extension constraints (metadata pointer, close authority); manual extension initialization via CPI (transfer fee, transfer hook, mint close authority); reading raw account data with `StateWithExtensions`/TLV when Anchor types don't expose extensions; `Interface`/`InterfaceAccount` vs. concrete token program; fully-extended-length allocation. | `Builders Cohort Q3 2026 - 2026_09_14 14_44 WEST - Recording.txt` |
| **2026-09-16** | Confidential transfers (Token-2022) + recap of extension patterns | Recap of the three extension approaches; privacy motivation on a public ledger; encrypted balances vs. plaintext; confidential transfer lifecycle as a state machine (configure → deposit → apply pending balance → transfer → apply pending balance → withdraw); pending balance as staging area; ZK/ElGamal proofs; code walkthrough of confidential mint creation, deposit, apply pending balance. | `Builders Cohort Q3 2026 - 2026_09_16 14_55 WEST - Recording.txt` |
| **2026-09-18** | CPI guard + permanent delegate | Team formation update; Token-2022 CPI guard; permanent delegate extension; delegate-to-program / PDA approval patterns. Planned but not transcribed: confidential transfer fees. | `Builders Cohort Q3 2026 - 2026_09_18 14_55 WEST - Recording (1).txt` |
| **2026-09-21** | Metaplex Core NFT staking | Core asset + plugin model; collections; freeze-delegate + attributes; PDA authorities without initialization; staking/unstaking rewards; CPI builders; time-travel testing with Surfpool. | `Builders Cohort Q3 2026 - 2026_09_21 14_55 WEST - Recording.txt` |
| **2026-09-23** | Core plugin breakout workshop | Challenge tasks: claim rewards while staked, burn-staked-NFT bonus, collection-level staking stats via attributes, oracle plugin for time-based transfers. Live pair-programming of collection init, config PDA, mint asset, stake, unstake, claim-rewards design. | `Builders Cohort Q3 2026 - 2026_09_23 14_58 WEST - Recording.txt` |
| **2026-09-25** | Tokenized money market fund (RWA capstone reference) | NAV math + staleness/deviation checks; subscribe/redeem with atomic mint/burn; roles (FundAdmin/NavManager/Pauser) via PDA accounts; upgrade-authority gating; ScaledUiAmount + DefaultAccountState KYC freeze; redemption caps; `emit_cpi!` events; mul_div u128 math. Final live class; enrichment week + capstone logistics. | `Builders Cohort Q3 2026 - 2026_09_25 14_56 WEST - Recording.txt` |
| **2026-09-28** | Architecture diagrams & atomic requirements (enrichment week) | Requirements-first methodology: numbered atomic "shall" statements before any drawing; actor/signer enumeration; state ownership + PDA seeds; requirement↔diagram bidirectional traceability; reference tables; MVP scope cutting; AI red-team as findings; breakout-room requirement critiques (private-payroll, dead-man's switch examples). See `knowledge/process/architecture-diagramming.md`. | `Builders Cohort Q3 2026 - Arch Diagram - 2026_09_28 14_56 WEST - Recording.txt` |

## Transcription caveats

- Source: fifteen dated session recordings from `2026_08_24` through `2026_09_28` (the complete cohort plus the enrichment-week architecture-diagram class), transcribed by `mlx-community/whisper-small.en-mlx-q4`. Speaker identities are not assigned; text is automatic speech recognition and is **not human verified**. Timestamps point to the original recordings. Slides and silent code are not captured.
- Direct quotes are transcript wording. Where ASR clearly garbled a term (e.g., "PDF" for PDA, "lambdas" for lamports, "Solano" for Solana, "CPA" for CPI), the intended term is shown in brackets with `[ASR corrected]`.
- `2026_09_14` is largely unintelligible between ~00:02 and ~00:27; `2026_09_16` degrades after ~01:06 — insights are drawn from the intelligible segments.
- `2026_09_18`: only the first ~52 minutes are intelligible; the remainder is silent live-coding. Confidential transfer fees were planned but are not captured.
- Citations use `(YYYY_MM_DD, ~HH:MM)` and refer to the original recordings in `sb2026/video/transcripts/`.
