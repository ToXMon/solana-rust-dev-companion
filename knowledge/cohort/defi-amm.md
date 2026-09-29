# DeFi & AMMs — cohort notes

AMM theory and implementation as taught in the cohort (Sep 7/9), plus the MEV/order-flow discussion. Read this before building or reviewing an AMM or anything that touches on-chain liquidity. Related: `anchor-practical.md`, `token-2022-and-extensions.md`, `quotes.md`.

## Theory mental models (Sep 7)

- **CPMM relative value model.** As demand for token Y rises, the pool holds less Y and more X, so Y becomes relatively more valuable. The keyword is *relative*. (`2026_09_07`, ~00:26:00–00:26:35)
- **Order flow toxicity.** "Informed" vs. "uninformed" traders; toxic vs. non-toxic flow; sandwiching as a whack-a-mole problem. (`2026_09_07`, ~00:35:43–00:52:28)
- Session scope: market makers/order books; CPMM `x*y=k`; CLMM/DLMM/stable swap; arbitrage; impermanent loss; slippage; front-running/sandwiching; block builders/latency. (`2026_09_07`)
- **Block builders and MEV:** Jito and Harmonic mentioned as ways to control block placement/latency; sandwiching and toxic order flow are ongoing issues. (`2026_09_07`, ~00:39:18–00:41:53; ~00:49:42–00:52:28)

## Implementation (Sep 9)

- **AMM (constant product):** Pool holds token X and token Y; `x * y = k`. Instructions: initialize config + LP mint, deposit (mint LP tokens), withdraw (burn LP tokens), swap. Optional fee collection to treasury. (`2026_09_07`; `2026_09_09`)
- Session covered slippage checks, optional fees, testing/modularization, and a constant-product curve library. (`2026_09_09`)
- **Assignment:** implement initialize, deposit, withdraw, swap for a constant-product AMM, with tests. Optional extension: fees and/or re-implement the curve library yourself. (`2026_09_09`, ~00:02:38–00:03:05; ~01:21:36–01:22:46)
- The AMM starter repo may need AVM `1.0.1` if mixed Anchor versions cause build issues. (`2026_09_09`, ~01:15:59–01:16:17)

## DeFi-relevant patterns from other sessions

- **Reject mints with permanent-delegate extension** when accepting arbitrary Token-2022 mints — it is "more of a problem than a feature" for DeFi. (`2026_09_18`, ~00:46:18–00:47:18) — see `token-2022-and-extensions.md`.
- **Delegate-to-program pattern** (user approves a program PDA to spend tokens; `transfer_checked` with program authority) is the standard AMM/escrow custody mechanism. (`2026_09_18`, ~00:32:27–00:44:28)
- **Oracle-gated lifecycle hooks** (time-restricted transfers) and NAV/peg guards are the DeFi-adjacent compliance patterns — see `nfts-metaplex-core.md` and `rwa-tokenized-fund.md`.
