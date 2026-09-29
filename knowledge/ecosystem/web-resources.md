# Solana Web Resources Synthesis

A structured synthesis of docs, guides, and repos covering Metaplex Core, tokenized RWAs / MMFs, privacy on Solana (especially confidential transfers), current Solana ecosystem priorities, and practical build recipes.

---

## 1. Metaplex Core — Assets, Collections, and Plugins

### 1.1 What Metaplex Core Is

Metaplex Core is Solana’s next-generation NFT standard. It replaces the older Token Metadata pattern with a single-account Asset model and a built-in, composable **plugin system** that runs on-chain.

- One Core Asset = one Solana account (no separate metadata/mint account needed).
- Behavior and data are added through **plugins** that hook into lifecycle events.
- Plugins can be attached at the Asset level or the Collection level, and they inherit/propagate rules.
- Full SDK support: JavaScript and Rust.

### 1.2 Program ID

| Network | Address                                       |
| --------- | --------------------------------------------- |
| Mainnet   | `CoREENxT6tW1HoK8ypY1SxRMZTcVPm7R94rH4PZNhX7d` |
| Devnet    | `CoREENxT6tW1HoK8ypY1SxRMZTcVPm7R94rH4PZNhX7d` |

### 1.3 Core Collections

A **Core Collection** is a Solana account that groups related Assets under shared metadata and collection-wide plugins.

| Field           | Description                                              |
| --------------- | -------------------------------------------------------- |
| `updateAuthority` | Authority of the collection                              |
| `name`          | Collection name                                          |
| `uri`           | Off-chain JSON metadata URL                              |
| `numMinted`     | Total Assets ever created in the collection              |
| `currentSize`   | Number of Assets currently in the collection             |

**Key mechanics**

- Cost: ~0.0015 SOL rent; total creation cost ≈ 0.002 SOL including tx fee.
- Assets can be standalone or point to a Collection via `collection`.
- Collection-level plugins propagate to member Assets unless overridden at the Asset level.
- You cannot close a Collection while it still contains Assets.

**Membership operations**

| Operation                         | How to do it (SDK)                                  | How to do it (CLI)                                      |
| --------------------------------- | --------------------------------------------------- | ------------------------------------------------------- |
| Add an Asset to a Collection      | `update(asset, { newCollection: collectionId, ... })` | `mplx core asset update <assetId> --collection <id>`      |
| Move an Asset between Collections | `update(...)` with new collection + new authority     | `mplx core asset update <assetId> --collection <newId>` |
| Remove an Asset from a Collection | `update` with `newUpdateAuthority: updateAuthority('Address', [wallet])` | `mplx core asset update <assetId> --remove-collection`    |

### 1.4 Plugins Overview

Plugins are on-chain extensions that add behavior or data storage. They validate lifecycle events and can `approve`, `reject`, or `forceApprove`.

**Plugin categories**

| Category          | Who can add it          | When it can be added     | Examples                                          |
| ----------------- | ----------------------- | ------------------------ | ------------------------------------------------- |
| Owner Managed     | Asset owner signature   | Anytime after creation   | Transfer Delegate, Freeze Delegate, Burn Delegate   |
| Authority Managed | Update authority        | Anytime after creation   | Royalties, Attributes, Update Delegate            |
| Permanent         | Defaults to update auth | Only at creation time    | Permanent Transfer/Freeze/Burn Delegate           |

**Plugin priority**

- If an Asset and its Collection both have the same plugin type, **Asset-level wins**.
- This lets a collection set a default royalty (e.g., 2%) while a rare Asset overrides it (e.g., 5%).

### 1.5 Plugin Lifecycle Validation Rules

Plugins are evaluated during create, transfer, burn, update, add/remove plugin, and authority operations:

1. If any plugin issues a **forceApprove** → event is approved.
2. Else if any plugin issues a **reject** → event is rejected.
3. Else if any plugin issues an **approve** → event is approved.
4. Else → reject by default.

**Force-approve plugins**

- Permanent Transfer Delegate
- Permanent Burn Delegate
- Permanent Freeze Delegate

These override ordinary rejections (e.g., a Permanent Burn can still execute even if an Asset is frozen).

### 1.6 Built-In Plugins Quick Reference

| Plugin                          | Type        | Use Case                                                   |
| ------------------------------- | ----------- | ---------------------------------------------------------- |
| Transfer Delegate               | Owner       | Escrowless marketplace listings, game item transfers         |
| Freeze Delegate                 | Owner       | Escrowless staking, freezing items                         |
| Burn Delegate                   | Owner       | Allow a game/program to burn the Asset                     |
| Royalties                       | Authority   | Enforce creator royalties on sale                          |
| Update Delegate                 | Authority   | Delegate metadata/plugin updates to a third party          |
| Attribute                       | Authority   | On-chain traits/stats for games or membership            |
| Permanent Transfer Delegate     | Permanent   | Permanent transfer rights (e.g., lock-in staking program)  |
| Permanent Freeze Delegate       | Permanent   | Permanent freeze authority                                 |
| Permanent Burn Delegate         | Permanent   | Permanent burn authority                                   |
| AddBlocker                      | Authority   | Prevent further plugins from being added                   |
| Autograph                       | Authority   | Signed/verified autographs                                 |
| Edition / Master Edition        | Authority   | Edition numbering and master-edition behavior              |
| Immutable Metadata              | Authority   | Lock metadata permanently                                  |
| Verified Creators               | Authority   | On-chain verified creator attribution                      |
| Bubblegum                       | Authority   | Bridge compressed NFTs into Core Assets                      |
| Oracle (external)               | External    | Pull in off-chain data/conditions                        |
| AppData (external)              | External    | Store app-specific external data                           |

### 1.7 Common Use Cases & Recommended Plugins

| Use Case                        | Recommended Plugin(s)                              |
| ------------------------------- | ---------------------------------------------------- |
| Enforce creator royalties       | Royalties plugin (Collection level)                |
| Escrowless staking              | Freeze Delegate                                    |
| Marketplace listings            | Freeze Delegate + Transfer Delegate                  |
| On-chain game stats             | Attributes plugin                                  |
| Allow third-party burns         | Burn Delegate                                      |
| Permanent staking program       | Permanent Freeze Delegate                          |

### 1.8 Code Patterns

**Create a Collection**

```typescript
import { generateSigner, publicKey } from '@metaplex-foundation/umi';
import { createCollection } from '@metaplex-foundation/mpl-core';

const collection = generateSigner(umi);
await createCollection(umi, {
  collection: collection,
  name: 'My Collection',
  uri: 'https://example.com/collection.json',
  plugins: [
    {
      type: 'Royalties',
      data: {
        basisPoints: 500, // 5%
        creators: [{ address: publicKey('...'), percentage: 100 }],
        ruleSet: ruleSet('None'), // or custom ruleSet
      },
    },
  ],
}).sendAndConfirm(umi);
```

**Create an Asset in a Collection**

```typescript
import { create, ruleSet } from '@metaplex-foundation/mpl-core';

await create(umi, {
  asset: generateSigner(umi),
  collection: collection.publicKey,
  name: 'Asset #1',
  uri: 'https://example.com/1.json',
  plugins: [
    {
      type: 'Royalties',
      data: {
        basisPoints: 1000, // overrides collection
        creators: [{ address: publicKey('...'), percentage: 100 }],
        ruleSet: ruleSet('None'),
      },
    },
  ],
}).sendAndConfirm(umi);
```

**Add a plugin after creation**

```typescript
import { addPlugin } from '@metaplex-foundation/mpl-core';

await addPlugin(umi, {
  asset: asset.publicKey,
  plugin: {
    type: 'Attributes',
    data: { attributeList: [{ key: 'level', value: '5' }] },
  },
}).sendAndConfirm(umi);
```

### 1.9 Key Developer Notes

- Only built-in plugins are supported; no custom plugins yet.
- Plugins increase account rent; most are ~0.001 SOL, but data-heavy plugins (Attributes, AppData) cost more.
- Owner Managed plugins have their authority auto-revoked on transfer; Authority Managed and Permanent plugins persist.
- For compressed NFT migration, look at the Bubblegum plugin.

---

## 2. Tokenized RWAs / MMFs

### 2.1 What Are Tokenized RWAs?

Tokenized real-world assets (RWAs) put traditional financial instruments on-chain as tokens. Common categories include US Treasuries, money-market funds (MMFs), equities/ETFs, private credit, commodities (gold), real estate, and bonds.

### 2.2 Tokenized MMF Lifecycle

A tokenized money-market fund converts traditional fund shares into blockchain tokens backed by short-term, low-risk instruments (T-bills, repos, agency debt, commercial paper).

| Step              | What Happens                                                            |
| ----------------- | ----------------------------------------------------------------------- |
| 1. Onboarding     | Investor completes KYC; wallet is added to an allowlist                 |
| 2. Subscription   | Investor sends fiat or stablecoin to the fund’s smart contract/vault   |
| 3. Token Minting  | Contract mints tokens = subscribed amount ÷ current NAV per token       |
| 4. Recordkeeping  | On-chain ledger records ownership in real time                          |
| 5. Redemption     | Investor returns tokens; contract burns them and returns cash/stablecoin |

### 2.3 On-Chain vs Off-Chain Split

| Function             | Off-Chain                                       | On-Chain                                            |
| -------------------- | ----------------------------------------------- | --------------------------------------------------- |
| NAV calculation      | Fund administrator calculates NAV daily         | Oracle/data feed publishes NAV to chain             |
| Compliance/KYC       | Issuer verifies identity and maintains allowlist  | Smart contract enforces whitelist during transfers  |
| Settlement           | Traditional T+1/T+2 for underlying assets       | Near-instant or same-day token transfer/redemption  |
| Ownership registry   | Transfer agent (traditional)                  | Distributed ledger                                  |
| Yield accrual        | Portfolio interest collected off-chain        | Accumulating NAV or rebasing token balance          |

### 2.4 Yield Distribution Models

| Model        | How It Works                                                    | Example          |
| ------------ | ---------------------------------------------------------------- | ---------------- |
| Accumulating | Token price (NAV) rises as interest accrues; balance stays same | Ondo OUSG        |
| Distributing / Rebasing | Token price stays ~$1; balance increases periodically | Franklin Templeton BENJI, Ondo rOUSG |
| Separate distribution | Stablecoin payouts sent to holders              | Some fund structures |

### 2.5 Major Tokenized Treasury / MMF Products

| Product | Issuer                | Blockchains                                                    | AUM / Notes              |
| ------- | --------------------- | -------------------------------------------------------------- | ------------------------ |
| BUIDL   | BlackRock / Securitize | Ethereum, Solana, Aptos, Arbitrum, Avalanche, Optimism, Polygon, BNB | ~$2.2–2.4B; institutional min ~$5M |
| BENJI   | Franklin Templeton    | Stellar, Polygon, Ethereum, Arbitrum, Aptos, Avalanche, BNB, Canton | ~$742M; retail via Benji app |
| MONY    | J.P. Morgan Asset Mgmt | Ethereum (Kinexys)                                            | Not publicly disclosed   |
| WTGXX   | WisdomTree            | Ethereum                                                       | First approved 24/7 USDC settlement (Feb 2026) |
| OUSG / rOUSG / USDY | Ondo Finance | Ethereum, Solana, others                                       | Accruing and rebasing treasury tokens |
| FOBXX   | Franklin Templeton    | Stellar                                                        | First U.S. mutual fund on public blockchain (2021) |

### 2.6 RWA Category Snapshot (April 2026)

| Category            | Market Size    | Typical Yield | Yield Type                    | Liquidity   | Key Consideration                  |
| ------------------- | -------------- | ------------- | ----------------------------- | ----------- | ---------------------------------- |
| US Treasuries       | ~$12.88B       | ~3–5%         | Income (accruing or rebasing) | High        | Counterparty, custodial            |
| Equities / ETFs     | ~$1B+          | Variable      | Price appreciation + dividends | Medium      | Securities law, market risk        |
| Private Credit      | ~$5B distributed (~$18–19B broad) | ~8–15% | Income (periodic) | Low         | Credit default, structuring          |
| Commodities         | ~$7.37B        | None          | Price appreciation            | Medium      | Custody, verification              |
| Real Estate         | Low hundreds of millions | ~4–10% | Income + appreciation        | Low         | Valuation, property management     |
| Bonds (non-Treasury)| ~$1.77B        | ~4–8%         | Income (coupon)               | Low–medium  | Credit, interest-rate              |

### 2.7 Solana Relevance

- Solana’s RWA value grew from ~$1.4B (Jan 2026) to ~$3.6B (early July 2026), with 277k+ holders and ~692 products.
- Tokenized stocks on Solana did ~$5.77B spot volume in Q2 2026; Solana captures >96% of all tokenized-stock trading across chains.
- Institutions active or piloting on Solana: Bitwise, State Street, Galaxy, Amundi, Citigroup.
- Composability is emerging: Jupiter Lend accepts tokenized stocks as collateral.
- Ondo Global Markets launched on Solana in early 2026 with 100+ tokenized stocks/ETFs (AAPL, NVDA, TSLA, SPY, QQQ, etc.).

### 2.8 Ondo Global Markets Solana Program

Repository: `ondoprotocol/global-markets-solana`

- Built with **Anchor 0.32.1**, `spl-token-2022` 8.0.0, and Pyth oracle SDK.
- Key components:

| Component             | Purpose                                                    |
| --------------------- | ---------------------------------------------------------- |
| `GMTokenManagerState` | Global program config: pause states, attestation signer    |
| `USDonManagerState`   | USDon mint + vault + USDC oracle settings                  |
| `TokenLimit`          | Per-token mint/redeem rate limits and pause flags          |
| `OndoUser`            | Per-user rate-limit tracking for mint/redeem               |
| `Whitelist`           | Access control for swap operations                         |
| `Attestation`         | Single-use attestation accounts for replay protection      |

- Token-2022 extensions used:
  - `ScaledUiAmount`
  - `MetadataPointer`
  - `Pausable`
  - `ConfidentialTransferMint`
  - `TransferHook`
- Security model:
  - All mint/redeem operations require a signed `secp256k1` attestation.
  - Attestation hash includes chain ID, attestation ID, side (buy/sell), user, asset, price, amount, expiration.
  - Two-tier rate limiting (token-level and user-level with time-decay).
  - Pyth price feeds with max deviation/staleness checks.
  - Multi-layered pause controls.

### 2.9 RWA Risks

**Inherited MMF risks**

- Credit risk
- Interest-rate risk
- Liquidity risk

**Tokenization-specific risks**

- Smart contract bugs
- Oracle/data feed errors
- Blockchain infrastructure outages
- Regulatory uncertainty
- Liquidity mismatch between 24/7 on-chain redemption and T+1 underlying settlement

### 2.10 Builder Whitespace on Solana

Infrastructure, not front-ends:

- Compliance/KYC rails
- RWA-specific oracles and NAV feeds
- Credit/lending primitives against RWA collateral
- Cross-venue liquidity routing
- Private/confidential RWA transfer rails
- MMF redemption/swap vaults

---

## 3. Privacy on Solana

### 3.1 Privacy Spectrum

Privacy on Solana is not one technology; it is a spectrum from fully transparent (default SPL) to encrypted balances (Confidential Transfers) to zero-knowledge / MPC-based private computation.

| Layer                         | Technology                                | What It Hides                                 |
| ----------------------------- | ----------------------------------------- | --------------------------------------------- |
| Transparent                   | SPL Token / Token-2022 default           | Nothing                                       |
| Encrypted balances & amounts    | Token-2022 Confidential Transfer/Balances | Transfer amounts and balances                 |
| Selective disclosure / compliance | Auditor keys, Range proofs                | Amounts from public, not from authorized auditor |
| Private computation           | ZK circuits, MPC (Arcium), TEEs (MagicBlock) | Computation logic and state                  |
| Stealth / mixer-style         | Shielded pools, privacy pools, mixers      | Transaction graph / counterparty linkage      |

### 3.2 Privacy Tools & Projects on Solana

| Project / Tool     | What It Does                                                      |
| ------------------ | ----------------------------------------------------------------- |
| **Token-2022 Confidential Transfer** | Native extension: encrypted balances & hidden amounts |
| **Arcium**         | MPC network for verifiable trustless private computation          |
| **Light Protocol** | ZK compression for scalability + privacy                        |
| **Noir (Aztec)**   | ZK language; Sunspot verifier runs Noir/Groth16 on Solana       |
| **Inco Lightning** | Confidential apps on SVM                                          |
| **MagicBlock**     | Private ephemeral rollups with TEEs                               |
| **Privacy.cash**   | Private SOL/SPL transfers using ZK proofs and privacy pools       |
| **encrypt.trade**  | Encrypted private DeFi using encryption + TEEs                    |
| **Umbra**          | Shielded-pool private transactions (built on Arcium)               |
| **Hush**           | Privacy-first wallet with stealth addresses                       |
| **Radr Labs**      | ShadowWire/ShadowSwap/ShadowTrade private DeFi using ZK             |
| **Range**          | Compliant-privacy pre-screening and selective disclosure          |

### 3.3 Confidential Transfers / Confidential Balances

**What it is**

A Token-2022 extension that lets users transfer tokens without revealing the **transfer amount** or **account balance**. Account addresses, sender, and receiver remain public; only amounts are encrypted.

**Cryptographic building blocks**

- **Homomorphic encryption** — math on encrypted values without decrypting them.
- **ElGamal** — account-specific encryption keys (separate from the owner’s wallet key).
- **Zero-knowledge proofs** — prove validity without revealing the amount.
- **ZK ElGamal Proof Program** — native Solana program for proof verification.
  - Address: `ZkE1Gama1Proof11111111111111111111111111111`

### 3.4 Confidential Mint & Account State

**ConfidentialTransferMint**

```rust
pub struct ConfidentialTransferMint {
    pub authority: OptionalNonZeroPubkey,
    pub auto_approve_new_accounts: PodBool,
    pub auditor_elgamal_pubkey: OptionalNonZeroElGamalPubkey,
}
```

- `authority` — manages confidential-transfer settings.
- `auto_approve_new_accounts` — if false, new token accounts must be approved.
- `auditor_elgamal_pubkey` — optional auditor public key that can decrypt amounts.

**ConfidentialTransferAccount**

```rust
pub struct ConfidentialTransferAccount {
    pub approved: PodBool,
    pub elgamal_pubkey: PodElGamalPubkey,
    pub pending_balance_lo: EncryptedBalance,
    pub pending_balance_hi: EncryptedBalance,
    pub available_balance: EncryptedBalance,
    pub decryptable_available_balance: DecryptableBalance,
    pub allow_confidential_credits: PodBool,
    pub allow_non_confidential_credits: PodBool,
    pub pending_balance_credit_counter: PodU64,
    pub maximum_pending_balance_credit_counter: PodU64,
    pub expected_pending_balance_credit_counter: PodU64,
    pub actual_pending_balance_credit_counter: PodU64,
}
```

- **Pending balance** — where incoming transfers land first.
- **Available balance** — funds that can be transferred or withdrawn.
- Credit counters prevent front-running by correlating pre/post balances.

### 3.5 Confidential Transfer Flow

```
1. Initialize mint with Confidential Balances
2. Create sender token account + configure for confidential transfers
3. Create recipient token account + configure for confidential transfers
4. Mint tokens to sender (public balance)
5. Deposit public balance → confidential pending balance
6. Apply pending balance → confidential available balance
7. Confidentially transfer from sender available to recipient pending
8. Recipient applies pending → available
9. Recipient withdraws from available → public balance (optional)
```

### 3.6 Key Confidential Transfer Instructions

| Instruction                         | Purpose                                                                          |
| ----------------------------------- | -------------------------------------------------------------------------------- |
| `InitializeMint`                    | Enable Confidential Balances on a mint (same tx as standard mint init)             |
| `UpdateMint`                        | Update approval policy and optional auditor ElGamal pubkey                       |
| `ConfigureAccount`                  | Enable Confidential Balances on a token account                                |
| `ApproveAccount`                    | Approve a token account if the mint requires manual approval                     |
| `EmptyAccount`                      | Empty pending/available balances so the account can be closed                    |
| `Deposit`                           | Convert public balance → confidential pending balance                            |
| `ApplyPendingBalance`               | Move pending → available (only account owner)                                    |
| `Transfer`                          | Confidentially transfer between accounts                                         |
| `TransferWithFee`                   | Confidential transfer with a fee                                                 |
| `Withdraw`                          | Convert available confidential balance → public balance                          |
| `EnableConfidentialCredits`         | Allow incoming confidential transfers                                            |
| `DisableConfidentialCredits`        | Block incoming confidential transfers                                            |
| `EnableNonConfidentialCredits`      | Allow incoming public transfers                                                  |
| `DisableNonConfidentialCredits`     | Block public transfers (receive-only confidential)                               |
| `ConfigureAccountWithRegistry`      | Configure account using an `ElGamalRegistry` instead of a pubkey-validity proof  |

### 3.7 ZK Proofs Required

**For a confidential transfer**

| Proof | Purpose |
| ----- | ------- |
| Equality proof | Same amount is deducted from sender and added to recipient |
| Ciphertext validity proof | Encrypted amount is well-formed for sender, receiver, and auditor keys |
| Range proof | Sender has sufficient available balance and amount is positive |

**For a withdrawal**

| Proof | Purpose |
| ----- | ------- |
| Equality proof | Amount is correctly represented in encrypted and plaintext forms |
| Range proof | Withdrawal amount is positive and within available balance |

**Proof program instruction references**

- `ProofInstruction::VerifyCiphertextCommitmentEquality`
- `ProofInstruction::VerifyBatchedGroupedCiphertext3HandlesValidity`
- `ProofInstruction::VerifyBatchedRangeProofU128`
- `ProofInstruction::VerifyBatchedRangeProofU64`
- `ProofInstruction::CloseContextState` — reclaim rent after proof verification.

### 3.8 Important Practical Notes

- Transaction-size limit (1,232 bytes) means proof-heavy flows often need multiple sequential transactions or Jito bundles.
- Lost encryption keys = permanent inability to decrypt confidential balances.
- Metadata (sender, receiver, timestamp) remains public; only amounts are hidden.
- Pattern analysis can still leak information.
- Mints can set an optional auditor key for compliance/regulatory visibility.

### 3.9 Example Code Pattern (Conceptual)

```typescript
// 1. Create a Token-2022 mint with ConfidentialTransferMint extension
const mint = await createMintWithConfidentialTransferExtension(
  connection,
  payer,
  mintAuthority,
  null, // freeze authority
  2,    // decimals
  undefined,
  { confidentialTransferMint: { authority: mintAuthority, autoApproveNewAccounts: true } }
);

// 2. Create & configure token accounts for sender and recipient
const senderTokenAccount = await createAccount(connection, payer, mint, sender.publicKey);
await configureAccount(connection, payer, mint, senderTokenAccount, sender, elgamalKeypair, aesKey);

// 3. Mint, deposit, apply
await mintTo(connection, payer, mint, senderTokenAccount, mintAuthority, amount);
await deposit(connection, payer, senderTokenAccount, sender, BigInt(amount), decimals);
await applyPendingBalance(connection, payer, senderTokenAccount, sender, elgamalKeypair, aesKey, ...);

// 4. Generate proofs off-chain, create proof context accounts via ZK ElGamal Proof Program,
//    then execute Transfer instruction with proof accounts, then close proof accounts.
```

For real code, use the [Confidential Balances Cookbook](https://github.com/solana-developers/Confidential-Balances-Sample) and the [CLI example script](https://github.com/solana-program/token-2022/blob/main/clients/cli/examples/confidential-transfer.sh).

---

## 4. Solana Ecosystem Priorities Right Now

Solana is positioning itself as the settlement and execution layer for internet-scale finance (“Internet Capital Markets”). Almost every priority serves one of three goals:

1. Faster and fairer finality
2. Cheaper compute
3. New financial and AI-agent primitives

### 4.1 Alpenglow — Consensus Overhaul

| Attribute | Detail |
| --------- | ------ |
| What      | Replaces Proof of History + Tower BFT with Votor (voting/finality) and Rotor (block propagation) |
| Effect    | Finality drops from ~12.8s to ~100–150ms (~100x faster); removes validator vote transactions from the chain, freeing ~75% of vote block space |
| Status    | Approved Sep 2025 (98.27% stake in favor); live on community test cluster since May 2026; mainnet expected late Q3/Q4 2026 |
| Caveat    | At activation only Votor ships; Rotor is deferred later |

### 4.2 Constellation — Fair Transaction Ordering

| Attribute | Detail |
| --------- | ------ |
| What      | Multiple Concurrent Proposers (MCP) with attesters holding leaders accountable; 50ms economic cycle |
| Goal      | Reduce MEV and leader-monopoly reordering/censorship; improve market fairness for perps/order books |
| Status    | Design proposal v0.9 (Mar 2026); builds on Alpenglow; intermediate wins (200ms slots, two-slot leader windows) expected before full Constellation |

### 4.3 RWAs — Tokenized Real-World Assets

- Fastest-growing category on Solana.
- From ~$1.4B → ~$3.6B in H1 2026; 277k+ holders; ~692 products.
- Tokenized stocks: $5.77B spot volume in Q2 2026; Solana >96% of all on-chain tokenized-stock volume.
- Active/piloting institutions: Bitwise, State Street, Galaxy, Amundi, Citigroup.
- Composability emerging: Jupiter Lend accepts tokenized stocks as collateral.

### 4.4 Perpetual Futures (Perps)

- Major venues: Drift (hybrid orderbook + JIT auction + AMM backstop), Jupiter Perps (oracle-priced).
- Solana perp venues grew ~57% YoY in H1 2026 vs ~6% for Hyperliquid.
- Strategic frontier: **equity perps** on tokenized stocks; blocked historically by hedging/limited traditional market hours.
- Caution: governance/oracle/admin-key risk matters as much as smart-contract risk (Drift $285M compromise, Apr 1 2026, was a social-engineering/durable-nonce governance attack, not a contract bug).

### 4.5 Agents and x402 — Machine-to-Machine Payments

| Attribute | Detail |
| --------- | ------ |
| What      | Open payment standard for AI agents; uses HTTP `402 Payment Required`; agent requests resource, gets 402 terms, retries with signed stablecoin payment |
| Backing   | Linux Foundation; Coinbase, Google, Stripe, AWS, Visa, Mastercard |
| Solana relevance | Sub-cent fees + sub-second finality make micro-payments viable; Solana drives ~65% of x402 volume; ~15M agent-initiated payments |
| Solana pieces | Agent Registry (on-chain identity), `pay.sh` tooling |

### 4.6 p-token and Pinocchio — Cheaper Compute, Live Now

| Attribute | Detail |
| --------- | ------ |
| Pinocchio | Zero-dependency, zero-copy Rust library for Solana programs; minimizes compute and binary size |
| p-token   | SPL Token program rewritten with Pinocchio; drop-in replacement |
| Impact    | Standard transfer cost fell from ~4,645 CUs to ~76 CUs (~98% cheaper) |
| Status    | **Live on mainnet** — the highest-impact item you can use today |
| Use case  | Reach for Pinocchio on performance-critical program paths |

### 4.7 How to Keep Up

| Source                          | Best For                                              |
| ------------------------------- | ----------------------------------------------------- |
| @solana_devs / Weekly Changelog | What shipped this week (clients, SDKs, SIMDs)         |
| Anza blog                       | Deep protocol context on Alpenglow, Constellation   |
| Helius blog                     | Best long-form technical explainers                   |
| Colosseum / blog.colosseum      | What to build; hackathon winners and accelerator picks |
| Messari State of Solana         | Quarterly data (TVL, DEX volume, RWA, perps, agents)  |
| RWA.xyz                         | Live tokenized-asset data                             |
| Blueshift                       | Hands-on Rust/Anchor/TypeScript courses               |
| DefiLlama                       | Live TVL and volume                                   |
| solana.com/developers + MCP     | Canonical docs and quickstarts                        |
| SIMD repo                       | Actual protocol specs (e.g., SIMD-0326 = Alpenglow)  |

---

## 5. Curated Resource Links

### 5.1 Metaplex Core

| Resource | Link | One-line description |
| -------- | ---- | -------------------- |
| Metaplex Core docs | https://www.metaplex.com/docs/smart-contracts/core | Next-gen NFT standard docs (assets, collections, plugins) |
| Core Collections guide | https://www.metaplex.com/docs/smart-contracts/core/collections | Creating and managing Core Collections |
| Core Plugins overview | https://www.metaplex.com/docs/smart-contracts/core/plugins | Full plugin table, lifecycle rules, and use cases |
| MPL Core JS SDK typedoc | https://mpl-core.typedoc.metaplex.com | API reference for the JavaScript SDK |
| Core program ID | `CoREENxT6tW1HoK8ypY1SxRMZTcVPm7R94rH4PZNhX7d` | Same address on mainnet and devnet |

### 5.2 RWAs / Tokenized MMFs

| Resource | Link | One-line description |
| -------- | ---- | -------------------- |
| CoinPaprika — Tokenized MMF primer | https://coinpaprika.com/education/tokenized-money-market-fund-how-it-works-and-who-uses-it | Lifecycle, issuers, yield models, risks |
| MetaMask — RWA categories 2026 | https://metamask.io/news/types-of-tokenized-real-world-assets-rwa-categories | Market-size comparison across Treasuries, equities, credit, commodities, real estate, bonds |
| RWA.xyz dashboard | https://app.rwa.xyz | Live tokenized-asset data and market size |
| Ondo Global Markets Solana program | https://github.com/ondoprotocol/global-markets-solana | Open-source Anchor program for tokenized stocks with rate limits, oracles, and confidential transfers |
| Ondo docs | https://docs.ondo.finance | Official Ondo Finance documentation |
| Securitize | https://securitize.io | BlackRock BUIDL tokenization partner platform |

### 5.3 Privacy on Solana

| Resource | Link | One-line description |
| -------- | ---- | -------------------- |
| Awesome Privacy on Solana | https://github.com/catmcgee/awesome-privacy-on-solana | Curated list of privacy tools, hackathon tracks, and education resources |
| Solana Confidential Balances docs | https://solana.com/docs/tokens/extensions/confidential-transfer | Official Token-2022 confidential transfer docs |
| Confidential Balances Cookbook | https://github.com/solana-developers/Confidential-Balances-Sample | Code snippets and recipes for confidential transfers |
| ZK ElGamal Proof Program overview | https://github.com/anza-xyz/agave/blob/master/docs/src/runtime/zk-elgamal-proof.md | Native proof program used by confidential transfers |
| QuickNode confidential transfer guide | https://www.quicknode.com/guides/solana-development/spl-tokens/token-2022/confidential | Developer walkthrough of the full confidential flow |
| Confidential Transfer CLI example | https://github.com/solana-program/token-2022/blob/main/clients/cli/examples/confidential-transfer.sh | Shell script reference for CLI usage |
| Arcium docs | https://docs.arcium.com | MPC framework for private DeFi on Solana |
| Light Protocol | https://github.com/Lightprotocol/light-protocol | ZK compression for scalability + privacy |
| Sunspot (Noir verifier) | https://github.com/reilabs/sunspot | Noir/Groth16 verifier on Solana |
| Noir on Solana examples | https://github.com/solana-foundation/noir-examples | Example ZK apps using Noir |

### 5.4 Ecosystem Priorities & General Solana Dev

| Resource | Link | One-line description |
| -------- | ---- | -------------------- |
| Solana Right Now (this repo) | `knowledge/ecosystem/solana-right-now.md` | Internal snapshot of Alpenglow, Constellation, RWAs, perps, agents, p-token |
| Static Resources (this repo) | `knowledge/ecosystem/static-resources.md` | Curated Solana dev resources: SDKs, frameworks, testing, tokens, Rust |
| Alpenglow blog (Anza) | https://www.anza.xyz/blog/alpenglow-a-new-consensus-for-solana | Deep dive into the new consensus protocol |
| SIMD-0326 | https://forum.solana.com/t/simd-0326-proposal-for-the-new-alpenglow-consensus-protocol/4236 | Alpenglow proposal on the Solana forum |
| Constellation overview | https://constellation.anza.xyz | Interactive MCP design demo and docs |
| Pinocchio | https://github.com/anza-xyz/pinocchio | Zero-dependency, zero-copy Rust library for cheap Solana programs |
| p-token upgrade | https://solana.com/upgrades/p-token | Drop-in SPL Token replacement, ~98% cheaper |
| x402 on Solana | https://solana.com/x402/what-is-x402 | Agentic machine-to-machine payments standard |
| Solana dev MCP | https://mcp.solana.com | Official Solana developer MCP tooling |
| SIMD repo | https://github.com/solana-foundation/solana-improvement-documents | Protocol specs and proposals |

---

## 6. End-to-End Use Case Recipes

### 6.1 Build a Private RWA Fund Token

**Goal**: Launch a KYC-gated, yield-bearing tokenized fund on Solana where transfers can optionally hide amounts.

**Architecture**

```
Investor (KYC’d wallet)
   │
   ├─► Whitelist gate (on-chain whitelist or registry)
   ├─► Token-2022 mint with:
   │    • TransferHook (KYC enforcement on transfer)
   │    • ConfidentialTransferMint (optional hidden amounts)
   │    • MetadataPointer (on-chain metadata)
   │    • Pausable (emergency pause)
   │
   ├─► Vault receives stablecoin subscription
   ├─► Attestation-signed mint instruction (like Ondo GM pattern)
   │    • secp256k1 signature over chainId, user, asset, price, amount, expiry
   └─► Oracle NAV feed (Pyth) validates subscription/redemption price
```

**Steps**

1. Create an Anchor program that owns a `Whitelist` and `UserLimit` accounts.
2. Create a Token-2022 mint with `ConfidentialTransferMint`, `TransferHook`, `Pausable`, and `MetadataPointer`.
3. Implement `mint` and `redeem` instructions that:
   - Verify sender is whitelisted.
   - Require an attestation signature and check replay via single-use `Attestation` accounts.
   - Validate NAV from a Pyth price feed with deviation/staleness checks.
   - Enforce per-token and per-user rate limits with time-decay capacity.
4. For optional privacy, configure token accounts with `ConfigureAccount` and route secondary transfers through confidential transfers.
5. For compliance, set an `auditor_elgamal_pubkey` on the mint so the issuer/regulator can decrypt amounts.

**Key programs/accounts**

- Token-2022 program
- ZK ElGamal Proof Program (`ZkE1Gama1Proof11111111111111111111111111111`)
- Pyth oracle price feed
- Custom Anchor program for whitelist + attestation + rate limits

**References**

- Ondo Global Markets Solana program (repo + README)
- Solana confidential balances docs
- Pyth Solana receiver SDK

---

### 6.2 Build an NFT Collection with Royalties

**Goal**: Create a Metaplex Core collection that enforces creator royalties across all sales.

**Architecture**

```
Collection account (Core)
   └─ Royalties plugin (authority-managed, default 5%)
        ├─ basisPoints: 500
        ├─ creators: [{ address: treasury, percentage: 100 }]
        └─ ruleSet: custom or None

Core Assets
   ├─ Reference collection
   ├─ Inherit collection royalties by default
   └─ Optional: Asset-level Royalties plugin to override (e.g., rare 10%)
```

**Steps**

1. **Create the Collection**

   ```typescript
   import { createCollection } from '@metaplex-foundation/mpl-core';

   const collection = generateSigner(umi);
   await createCollection(umi, {
     collection,
     name: 'Royalty Collection',
     uri: 'https://example.com/collection.json',
     plugins: [
       {
         type: 'Royalties',
         data: {
           basisPoints: 500,
           creators: [{ address: treasury.publicKey, percentage: 100 }],
           ruleSet: ruleSet('None'),
         },
       },
     ],
   }).sendAndConfirm(umi);
   ```

2. **Mint Assets**

   ```typescript
   await create(umi, {
     asset: generateSigner(umi),
     collection: collection.publicKey,
     name: 'Asset #1',
     uri: 'https://example.com/1.json',
   }).sendAndConfirm(umi);
   ```

3. **(Optional) Override royalties on rare Assets**

   ```typescript
   await addPlugin(umi, {
     asset: rareAsset.publicKey,
     plugin: {
       type: 'Royalties',
       data: {
         basisPoints: 1000,
         creators: [{ address: treasury.publicKey, percentage: 100 }],
         ruleSet: ruleSet('None'),
       },
     },
   }).sendAndConfirm(umi);
   ```

4. **Marketplace integration**: on `transfer`, the Royalties plugin rejects unless the marketplace instruction includes the royalty payment to the creators.

**Key plugins**

- `Royalties` (authority-managed)
- `Update Delegate` if you want a partner marketplace to manage membership

**References**

- Metaplex Core Collections docs
- Metaplex Core Plugins docs

---

### 6.3 Build a Tokenized Stock Vault

**Goal**: Mirror a traditional stock’s total return on Solana with whitelisted transfers, oracle pricing, and optional confidential balances.

**Architecture**

```
Investor (KYC + accreditation)
   │
   ▼
Issuer program (Anchor)
   ├─ Whitelist: only approved wallets can hold/trade
   ├─ Rate limits: daily mint/redeem caps
   ├─ Oracle: Pyth equity price feed for NAV sanity checks
   ├─ Attestation: off-chain issuer signs every mint/redeem
   └─ Token-2022 mint:
        • TransferHook → KYC check on every transfer
        • Pausable → emergency halt
        • MetadataPointer → on-chain metadata
        • ScaledUiAmount → display scaling
        • ConfidentialTransferMint → optional hidden P2P amounts
   │
   ▼
Vault holds stablecoin collateral / issuer custody the underlying security off-chain
```

**Steps**

1. **Deploy the issuer program** with:
   - `Whitelist` account per user
   - `TokenLimit` per stock token
   - `OndoUser`-style per-user mint/redeem capacity
   - Pause flags at program, token, and user level
2. **Create the Token-2022 mint** with the extensions above.
3. **Mint flow**
   - Investor deposits USDC into program vault.
   - Issuer backend creates a `secp256k1` attestation covering user, stock, price, amount, expiry.
   - Program verifies signature, checks oracle price sanity, enforces rate limit, then mints tokens.
4. **Transfer flow**
   - Every transfer routes through `TransferHook` to verify both sender and recipient are whitelisted.
   - Optional: enable confidential transfers for peer-to-peer trades so amounts stay private.
5. **Redemption flow**
   - Investor burns tokens; program validates attestation and oracle price; releases USDC.

**Key code components**

- Anchor program: whitelist + rate-limit + attestation verification
- Token-2022: mint with `TransferHook`, `Pausable`, `ScaledUiAmount`, `MetadataPointer`, `ConfidentialTransferMint`
- Pyth SDK for oracle price sanity checks
- ZK ElGamal Proof Program if confidential transfers are enabled

**References**

- Ondo Global Markets Solana program (open-source reference)
- Token-2022 extension docs
- Pyth Solana receiver SDK

---

## 7. Quick Decision Tables

### 7.1 When to Use What Token Program / Standard

| Need                                            | Use This                                   |
| ----------------------------------------------- | ------------------------------------------ |
| Standard fungible token                         | SPL Token or p-token (live, much cheaper)  |
| Fungible token with fees, confidential balances, hooks | Token-2022                          |
| NFTs with on-chain plugins and royalties        | Metaplex Core                              |
| Mass minting / compressed NFTs                  | Metaplex Bubblegum                         |
| Game items with on-chain stats/attributes       | Metaplex Core + Attributes plugin          |

### 7.2 Privacy Technology by Use Case

| Use Case                                | Technology                                      |
| --------------------------------------- | ----------------------------------------------- |
| Hide transfer amounts for an SPL token | Token-2022 Confidential Transfer                |
| Hide balances from public but allow auditor | Token-2022 + auditor ElGamal key             |
| Private DeFi swaps/lending/borrowing     | Arcium MPC or Radr Labs ZK tools                |
| Shielded transactions / privacy pools    | Privacy.cash, Umbra, Hush                       |
| ZK verification on Solana                | Sunspot (Noir/Groth16), Light Protocol           |
| Private ephemeral compute / TEEs          | MagicBlock Private Ephemeral Rollups            |

---

## 8. Glossary

| Term | Definition |
| ---- | ---------- |
| Asset (Core) | A single-account Metaplex Core NFT |
| Collection (Core) | A grouping account for related Assets with shared metadata/plugins |
| Plugin | An on-chain extension that adds behavior/data to a Core Asset or Collection |
| Force approve | A permanent plugin’s ability to override other rejections |
| NAV | Net asset value; fund assets minus liabilities divided by outstanding shares/tokens |
| MMF | Money-market fund |
| RWA | Real-world asset (tokenized) |
| KYC | Know-your-customer identity verification |
| Confidential balance | Encrypted token balance using ElGamal encryption |
| Pending balance | Incoming confidential funds awaiting owner application |
| Available balance | Confidential funds ready to transfer or withdraw |
| Attestation | A cryptographically signed authorization for mint/redeem |
| Rate limit capacity | Time-decaying mint/redeem budget per token or user |
| p-token | Pinocchio-optimized drop-in SPL Token program |
| Alpenglow | Solana’s new consensus protocol (Votor + Rotor) |
| Constellation | Multiple Concurrent Proposers design to reduce MEV/leader monopoly |
| x402 | HTTP 402-based agentic payment standard |

---

*Synthesized from Metaplex docs, CoinPaprika, MetaMask RWA guide, Ondo Global Markets Solana repo, Solana confidential-transfer docs, QuickNode guide, Awesome Privacy on Solana, and internal Solana ecosystem docs.*
