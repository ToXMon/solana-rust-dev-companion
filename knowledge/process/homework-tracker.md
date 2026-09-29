# Solana Builders Cohort Q3 2026 — Homework & LOI Tracker

This tracker is built from the cohort transcripts. Use it to see what's due, what skills to invoke, and where the supporting patterns live.

## Legend

- Status: `Not started` / `In progress` / `Done`
- Evidence: program ID, transaction signature, Explorer link, doc link, or screenshot

---

## Week 1: Foundations

| # | Assignment | Due | Skill | Status | Evidence |
|---|------------|-----|-------|--------|----------|
| 1.1 | Set up toolchain (Rust, Solana CLI, Anchor CLI) | Week 1 | `/solana-build` | Not started | |
| 1.2 | Deploy "Hello Solana" program to devnet | Week 1 | `/solana-build` | Not started | |
| 1.3 | Counter PDA program (initialize + increment) | Week 1 | `/solana-build` | Not started | |

## Week 2: Tokens & First Deployments

| # | Assignment | Due | Skill | Status | Evidence |
|---|------------|-----|-------|--------|----------|
| 2.1 | Mint first SPL token on devnet | Week 2 | `/solana-tokens` | Not started | |
| 2.2 | Token-2022 mint with one extension | Week 2 | `/solana-tokens` | Not started | |
| 2.3 | NFT metadata with Metaplex | Week 2 | `/solana-nfts` | Not started | |

## Week 3: Vaults & Escrow

| # | Assignment | Due | Skill | Status | Evidence |
|---|------------|-----|-------|--------|----------|
| 3.1 | Vault program: `initialize`, `deposit`, `withdraw`, `close` | Week 3 | `/solana-build`, `/solana-defi` | Not started | |
| 3.2 | Escrow program: `make`, `take`, `refund` + tests | Week 3 | `/solana-defi`, `/solana-test` | Not started | |
| 3.3 | Escrow extensions: `update`, optional timed expiry | Week 3 | `/solana-defi` | Not started | |

## Week 4: Programs & Frontend

| # | Assignment | Due | Skill | Status | Evidence |
|---|------------|-----|-------|--------|----------|
| 4.1 | Proposal state machine program | Week 4 | `/solana-build` | Not started | |
| 4.2 | Tip jar CPI program (deposit/withdraw with PDA vault) | Week 4 | `/solana-build` | Not started | |
| 4.3 | Full dapp frontend connected to program | Week 4 | `/solana-frontend` | Not started | |

## Week 5: AMM & DeFi

| # | Assignment | Due | Skill | Status | Evidence |
|---|------------|-----|-------|--------|----------|
| 5.1 | Constant-product AMM: `initialize`, `deposit`, `withdraw`, `swap` | Week 5 | `/solana-defi`, `/solana-test` | Not started | |
| 5.2 | LP token + slippage checks | Week 5 | `/solana-defi` | Not started | |
| 5.3 | Optional AMM extensions: fees, custom curve | Week 5 | `/solana-defi` | Not started | |

## Week 6: Token-2022 + NFT Staking

| # | Assignment | Due | Skill | Status | Evidence |
|---|------------|-----|-------|--------|----------|
| 6.1 | Token-2022 extension integration patterns | Week 6 | `/solana-tokens` | Not started | |
| 6.2 | Confidential transfer state-machine walkthrough | Week 6 | `/solana-privacy` | Not started | |
| 6.3 | CPI guard and permanent delegate awareness | Week 6 | `/solana-tokens`, `/solana-security` | Not started | |
| 6.4 | Metaplex Core NFT staking program: collection, mint, stake, unstake | Week 6 | `/solana-nfts`, `/solana-defi` | Not started | |
| 6.5 | `claim_rewards` while staked (`last_claimed_at` attribute) | Week 6 | `/solana-nfts` | Not started | |
| 6.6 | `burn_staked_nft` — burn delegate + one-time bonus reward | Week 6 | `/solana-nfts` | Not started | |
| 6.7 | Collection-level staking stats via collection Attributes plugin (`total_staked`) | Week 6 | `/solana-nfts` | Not started | |
| 6.8 | Oracle plugin — time-gated transfers (RWA compliance windows) | Week 6 | `/solana-nfts`, `/solana-rwa` | Not started | |

## Week 7+: Capstone / LOI

| # | Assignment | Due | Skill | Status | Evidence |
|---|------------|-----|-------|--------|----------|
| 7.1 | LOI Part 1: value prop, market, competitors, target markets, use cases | Week 7 | `/solana-loi` | In progress | `capstone/LEASH-LOI-Submission.pdf` |
| 7.2 | LOI refinement + red-team | Week 8 | `/solana-loi`, `/solana-research` | In progress | Red-team log in `capstone/` in your working repo (format reference: `examples/loi/` in the companion repo) |
| 7.3 | Architecture diagram / "requirement contract": numbered atomic requirements → actors/signers → traceable diagram (method covered in the Sep 28 enrichment class; see `knowledge/process/architecture-diagramming.md`) | Capstone wk 2 | `/solana-architect` | Not started | |
| 7.4 | Capstone MVP build (group project — confirm teammates are active) | Capstone wks 1–2 | `/solana-architect` → `/solana-build` → `/solana-frontend` | Not started | |
| 7.5 | Capstone demo + final submission | End | all above | Not started | |

---

## How to use this tracker

1. Copy this file to your own notes or project README and update Status/Evidence as you go.
2. For each coding assignment, invoke the linked skill. It will reference the patterns in `knowledge/patterns/code-patterns.md` and the mental models in `knowledge/cohort/README.md`.
3. For the LOI, start with `loi-template.md`, fill it yourself, then use `/solana-loi` for red-teaming.
4. Run `/solana-security` before considering any assignment complete.

## Notes from transcripts

- **Aug 31 assignment:** complete vault program (`initialize`, `deposit`, `withdraw`, `close`). (`knowledge/cohort/capstone-loi-and-architecture.md`)
- **Sep 2 assignment:** complete escrow `make`/`take`/`refund`. (`knowledge/cohort/capstone-loi-and-architecture.md`)
- **Sep 4 assignment:** implement `take` and `update`; optional timed escrow. (`knowledge/cohort/capstone-loi-and-architecture.md`)
- **Sep 7/9 AMM assignment:** implement initialize, deposit, withdraw, swap for CPMM with tests; optional fees. (`knowledge/cohort/capstone-loi-and-architecture.md`)
- **Sep 23 challenge (Week 6 plugins):** `claim_rewards`, `burn_staked_nft`, collection-level `total_staked` stats, oracle-plugin time-gated transfers. (`knowledge/cohort/nfts-metaplex-core.md`)
- **Sep 25 (final class):** TMMF reference repo shared via Classroom/Discord; cohort ended — enrichment week next, architecture-diagram class Monday, capstone is a **group** project (~2 weeks). (`knowledge/cohort/rwa-tokenized-fund.md`)
- **Sep 28 (enrichment class):** Architecture-diagram methodology delivered — requirements first (numbered, atomic "shall" statements), explicit actors/signers, requirement↔diagram bidirectional traceability, MVP scope cut, AI red-team as findings + per-member override reflections. Full guide: `knowledge/process/architecture-diagramming.md`. (`knowledge/cohort/capstone-loi-and-architecture.md`)
- **LOI guidance:** human-first thinking, then AI red-team; submit as Google Doc/PDF. (`knowledge/cohort/capstone-loi-and-architecture.md`)

## Fill in your actual due dates

Replace the placeholder "Week X" due dates with the real deadlines from your cohort calendar once they are published.
