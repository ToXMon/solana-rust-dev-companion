# Solana Stack Selector

Quick decision aid for choosing the right programs, SDKs, and patterns for a Solana build.

## Program framework

| Situation | Use |
|-----------|-----|
| New dapp, fast iteration, standard security | **Anchor 0.31+** |
| Performance-critical, token-heavy, tiny binary | **Pinocchio** (maybe + Steel) |
| Porting from Solidity | **eth-to-sol-skill** two-pass method |
| Need formal verification | **QEDGen/solana-skills** or manual Kani/Lean |
| Game with Unity/React Native | **solana-game-skill** + Solana.Unity-SDK |

## Token standard

| Need | Standard |
|------|----------|
| Basic fungible token | SPL Token |
| Transfer fees, confidential balances, metadata extension, memo | **Token-2022** |
| KYC-gated transfers | Token-2022 + `transferHook` extension |
| Private balances/amounts | Token-2022 + `confidentialTransfer` extension |
| NFT collection with royalties/plugins | **Metaplex Core** |
| Legacy NFT ecosystem compatibility | Metaplex Token Metadata |
| Compressed NFTs | Metaplex Bubblegum |

## Client SDK

| Project type | SDK |
|--------------|-----|
| New TypeScript/web project | **`@solana/kit`** |
| Existing project using web3.js | `@solana/web3.js` or migrate to kit |
| Lightweight Node/RN scripts | **Gill SDK** |
| React wallet integration | `@solana/wallet-adapter-react` |
| Generated type-safe clients | **Codama** from IDL |

## Testing

| Goal | Tool |
|------|------|
| Fast unit/integration tests in Rust | **LiteSVM** |
| Integration tests with Anchor helpers | **Anchor TS tests** |
| Test against real mainnet state | **Surfpool** |
| Final devnet verification | `anchor test --provider.cluster devnet` |
| Security fuzzing / formal methods | Trail of Bits scanner, QEDGen tools |

## Data / RPC

| Need | Provider / Tool |
|------|-----------------|
| General RPC + websocket | Helius, QuickNode, Triton |
| Live on-chain data in AI IDE | **Helius MCP**, **solana-dev-mcp** |
| Program parsing/decoding | Codama, Anchor IDL |
| Indexer for complex queries | Helius APIs, DAS, custom indexer |

## Hot use cases and starter paths

| Use case | Starter skill | Key primitives |
|----------|---------------|----------------|
| Token launch with metadata | `/solana-tokens` | Token-2022, metadata extension, `@solana/kit` |
| NFT collection + royalties | `/solana-nfts` | Metaplex Core, Royalties plugin |
| Escrow marketplace | `/solana-defi` | PDA vault, `transfer_checked`, atomic CPI |
| AMM / DEX | `/solana-defi` | Constant-product curve, LP token, pool invariant checks |
| Money-market / RWA fund token | `/solana-rwa` | Token-2022, oracle/attestation, KYC gating, rate limits |
| Private payroll / payments | `/solana-privacy` | Confidential transfers, ElGamal/AES keys |
| AI agent payments | `/solana-defi` + `/solana-research` | x402, Agent Registry, micropayments |
| Perps / derivatives vault | `/solana-defi` | Oracle integration, collateral, liquidation math |

## Verification discipline

Every build should end with:
1. Local tests passing (`anchor test` or `cargo test`).
2. Devnet deployment with captured program ID.
3. At least one successful devnet transaction with Explorer link.
4. Security checklist completed (use `/solana-security`).
