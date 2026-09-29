# Solana Use-Case Recipes

End-to-end build recipes for the hottest Solana use cases right now. Each recipe lists the skills to invoke, the key primitives, and the verification steps.

## Recipe 1: Launch a Token-2022 Fungible Token with Metadata

**Goal:** Create a USDC-like or project token with on-chain metadata extension.

**Skills:** `/solana-tokens` → `/solana-test`

**Steps**

1. Create a Token-2022 mint with `tokenMetadata` extension.
2. Create the associated token account (ATA) for the authority.
3. Mint the initial supply to the authority ATA.
4. Optionally add `transferFeeConfig` or `confidentialTransfer` extension.
5. Write tests for mint creation, minting, and transfers.

**Key program IDs**

- Token-2022: `TokenzQdBNbW9ZV7FPi1XEG4J8t75VmvFwAPc2gUwzbqb`
- Associated Token Program: `ATokenGPvbdGVxr1b2hvZbsiqW5xWH25efZZEU9W6eB9v`

**Verification**

- [ ] Devnet mint address captured
- [ ] ATA balance matches minted supply
- [ ] Explorer link for mint transaction

---

## Recipe 2: Build a Metaplex Core NFT Collection with Royalties

**Goal:** Launch a PFP or art collection with enforced creator royalties.

**Skills:** `/solana-nfts` → `/solana-frontend`

**Steps**

1. Create a Core Collection with name, URI, and update authority.
2. Add a Royalties plugin at the Collection level (e.g., 2.5% to creator).
3. Generate off-chain metadata JSON and upload to permanent storage (Arweave, IPFS, Irys).
4. Create Core Assets individually or in batch, pointing to the collection.
5. Build a mint/frontend page using Umi + mpl-core SDK.

**Key program ID**

- MPL Core: `CoREENxT6tW1HoK8ypY1SxRMZTcVPm7R94rH4PZNhX7d`

**Verification**

- [ ] Collection address captured
- [ ] Sample Asset address captured
- [ ] Royalties plugin visible on Explorer or via SDK
- [ ] Frontend can fetch collection and display assets

---

## Recipe 3: Build an Escrow Marketplace

**Goal:** Atomic token swap between two parties with refund and optional expiry.

**Skills:** `/solana-defi` → `/solana-test` → `/solana-security`

**Steps**

1. Design PDA seeds: `[b"escrow", maker, seed.to_le_bytes()]`.
2. Create escrow state account and a PDA-owned ATA vault for token A.
3. Implement `make`: maker deposits token A into vault, sets desired token B amount and expiry.
4. Implement `take`: taker sends token B to maker, receives token A from vault, closes escrow.
5. Implement `refund`: after expiry, maker reclaims token A and rent.
6. Implement `update`: maker changes desired amount before expiry.
7. Write lifecycle and negative tests.

**Key primitives**

- PDA state + PDA-owned ATA vault
- `transfer_checked` CPIs (maker deposit, taker payment, vault release)
- `close_account` CPI to return rent and vault token account

**Verification**

- [ ] Happy path `make` → `take` test passes
- [ ] Refund after expiry test passes
- [ ] Unauthorized take/refund tests fail
- [ ] Devnet deployment proof

---

## Recipe 4: Build a Constant-Product AMM

**Goal:** A mini Uniswap-style pool with LP tokens and swap fees.

**Skills:** `/solana-defi` → `/solana-test` → `/solana-security`

**Steps**

1. Design pool state: token reserves, LP token mint, fee basis points.
2. PDA seeds: `[b"pool", mint_a, mint_b]` (ordered so pair is canonical).
3. `initialize`: create pool, mint LP tokens to first depositor.
4. `deposit`: add liquidity proportional to current reserves; mint LP tokens.
5. `withdraw`: burn LP tokens; return proportional reserves.
6. `swap`: enforce `x * y = k` invariant; apply fee to input; check slippage.
7. Implement LP token mint/burn via Token Program CPI.
8. Test invariants, fees, and slippage failures.

**Key primitives**

- Constant-product curve: `x * y = k`
- LP token as shares of pool
- Slippage checks on swap output
- Fee accrual to reserves or fee account

**Verification**

- [ ] Invariant holds after deposit/withdraw/swap
- [ ] LP token math matches reserves
- [ ] Swap with excessive slippage fails
- [ ] Devnet deployment + swap transaction proof

---

## Recipe 5: Tokenized Money-Market Fund (RWA)

**Goal:** On-chain token representing shares in a T-bill/money-market fund with KYC and NAV oracle.

**Skills:** `/solana-rwa` → `/solana-tokens` → `/solana-test` → `/solana-security`

**Steps** (grounded in the Sep 25 cohort reference implementation)

1. Design legal/off-chain structure (SPV, custodian, transfer agent, fund administrator). The program only consumes a published NAV — portfolio valuation stays off-chain.
2. `Fund` PDA (static seed — only one fund) stores `share_mint`, `stable_mint` + decimals, `nav`, `last_nav_update`, `max_nav_change_bps`, pause flags, `redemption_cap`/`window`, and `oracle_enabled`.
3. `Role` PDAs seeded by `role_type.seed() + wallet`; enum `RoleType { FundAdmin, NavManager, Pauser }`. Pauser can pause but not resume.
4. Gate `grant_role`/`initialize_fund` to the program's **upgrade authority**: pass program + `ProgramData` accounts, check `upgrade_authority == signer`, and re-derive the program-data PDA from the program id to prevent cross-program spoofing. Rotate upgrade authority to a multisig after deploy.
5. Create the share mint with Token-2022 `ScaledUiAmount` (wallets display shares × NAV) and `DefaultAccountState::Frozen` — new share accounts are frozen until `approve_investor` thaws them (KYC gate); `revoke_investor` re-freezes.
6. `subscribe`: require `!subscription_paused`, fresh NAV (`last_nav_update` within window), optional oracle peg check on the stablecoin, then `shares = usdc_in * NAV_SCALE / nav` via u128 `mul_div`; `transfer_checked` USDC to vault + `mint_to` shares — atomic.
7. `redeem`: burn shares + pay out `shares * nav / NAV_SCALE` USDC from vault in one tx; enforce `redemption_cap` per window.
8. `publish_nav` (NavManager only): reject non-positive NAV, cap deviation vs previous NAV in bps, update `nav` + `last_nav_update` + the ScaledUiAmount multiplier.
9. `set_custodian`/withdrawals: always keep a liquidity floor (e.g., ~30% of vault) for redemptions — bank-run guard.
10. Emit `emit_cpi!` events (self-CPI into inner-instruction data; not truncated like `emit!` logs) for subscribe/redeem/role events; costs one CPI depth slot + two accounts.
11. Round toward remaining holders: floor share/USDC division — dust accrues to the pool, never to the transacting user.

**Key primitives**

- NAV oracle = trusted `NavManager` role signer, not a price feed; staleness + bps-deviation checks are the safety surface
- Token-2022 `ScaledUiAmount` + `DefaultAccountState::Frozen` for KYC'd shares
- Upgrade-authority gating for bootstrap admin (then multisig)
- Role PDA accounts; `u128` `mul_div`; floor rounding; redemption rate-limit windows; `emit_cpi!`

**Verification**

- [ ] Non-upgrad-authority signer cannot `grant_role` (including spoofed program-data accounts)
- [ ] KYC-blocked (frozen) share account cannot receive/hold shares
- [ ] Subscription mints `usdc * NAV_SCALE / nav` shares; stale NAV rejected
- [ ] NAV update beyond `max_nav_change_bps` rejected; zero/negative NAV rejected
- [ ] Redemption over the window cap rejected; burn and payout are atomic
- [ ] Custodian withdrawal cannot take vault below the liquidity floor
- [ ] Devnet deployment + subscription transaction proof

---

## Recipe 6: Confidential Payroll / Private Stablecoin Transfer

**Goal:** Pay a team in stablecoins without revealing amounts on-chain.

**Skills:** `/solana-privacy` → `/solana-tokens` → `/solana-test`

**Steps**

1. Create a Token-2022 mint with `confidentialTransfer` extension and optional auditor key.
2. Each recipient generates ElGamal + AES keys client-side.
3. Configure confidential token accounts for sender and recipients.
4. Employer deposits public balance, applies to confidential available balance.
5. For each recipient, confidentially transfer amount; recipient applies pending balance.
6. Recipients can withdraw to public balance when needed.

**Key primitives**

- ElGamal public key per account
- Pending/available balance staging
- ZK range, equality, and ciphertext-validity proofs

**Verification**

- [ ] Confidential transfer succeeds with valid proofs
- [ ] Transfer with invalid range proof fails
- [ ] Auditor can decrypt amounts if configured
- [ ] Devnet transaction signatures for deposit/transfer/withdraw

---

## Recipe 8: Core NFT Staking with Rewards

**Goal:** Lock NFTs in a Metaplex Core collection to earn a fungible reward token over time.

**Skills:** `/solana-nfts` → `/solana-tokens` → `/solana-test` → `/solana-security`

**Steps**

1. Create a Core Collection with a PDA as update authority (the PDA only signs, no initialization needed).
2. Create the staking program config PDA seeded by the collection key.
3. Create a reward mint PDA seeded by the config PDA.
4. Users mint or bring a Core Asset from the collection.
5. `stake`: add a `FreezeDelegate` plugin with `frozen = true`; set `Attributes` plugin with `stake = true` and `staked_at` timestamp.
6. `unstake`: validate freeze period has elapsed; thaw the asset; mint reward tokens to the user's reward ATA (use `init_if_needed`).
7. Stretch: `claim_rewards` mints accrued rewards and resets `staked_at` while keeping the asset frozen.

**Key primitives**

- MPL Core CPI builders (`CreateCollectionV2CpiBuilder`, `AddPluginV1CpiBuilder`, etc.)
- `FreezeDelegate` + `Attributes` plugins for staking state
- Reward calculation in basis points scaled by mint decimals
- PDA authority that does not require initialization

**Verification**

- [ ] Collection and asset created on devnet
- [ ] Stake transaction freezes the asset and sets attributes
- [ ] Unstake before freeze period fails; after freeze period mints correct reward
- [ ] Program ID and transaction links captured

---

## Recipe 9: AI Agent Payments (x402)

**Goal:** Enable an AI agent to pay for APIs or compute with Solana stablecoins.

**Skills:** `/solana-research` → `/solana-tokens` → `/solana-defi` → `/solana-frontend`

**Steps**

1. Read latest x402 specs and Agent Registry docs.
2. Set up a server endpoint that returns `402 Payment Required` with payment terms.
3. Build an agent client that signs a stablecoin transfer or Token-2022 transfer on Solana.
4. Server verifies the payment on-chain before fulfilling the request.
5. Optional: use Solana Developer MCP for on-chain interaction.

**Key primitives**

- HTTP 402 status + payment terms payload
- Solana stablecoin transfer with memo
- On-chain payment verification

**Verification**

- [ ] 402 handshake works
- [ ] Payment transaction lands and is verifiable
- [ ] Server only responds after confirmed payment

---

## How to run any recipe

1. Start with `/turbin3` (or `/solana-guide`) and describe the recipe goal.
2. Move to the domain skill (e.g., `/solana-rwa`) for implementation.
3. Use `/solana-test` after each instruction.
4. Use `/solana-security` before devnet deployment.
5. Record program IDs and transaction links in your project notes.
