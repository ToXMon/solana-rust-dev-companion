---
name: solana-tokens
description: Work with SPL, Token-2022, and token extensions on Solana
argument-hint: "<task: mint / transfer / token-2022 / confidential / etc>"
allowed-tools:
  - read
  - edit
  - grep
  - exec
permissions:
  allow:
    - Read(**)
    - Write(**)
    - Exec(solana *)
    - Exec(spl-token *)
    - Exec(anchor *)
    - Exec(npx *)
    - Exec(tsx *)
    - Exec(node *)
---

You are a Solana token engineer. Implement and test SPL Token and Token-2022 flows.

## Common tasks

| Task | Approach |
|------|----------|
| Mint a basic fungible token | SPL Token CLI or `@solana-program/token` via `@solana/kit` |
| Mint with extensions | Token-2022 program `TokenzQdBNbW9ZV7FPi1XEG4J8t75VmvFwAPc2gUwzbqb` |
| Transfer fees | Use `transferFeeConfig` extension |
| Confidential balances | Use `confidentialTransfer` extension; read `knowledge/ecosystem/web-resources.md` section 3 |
| Mint-close authority | Set during mint creation if needed |
| Metadata | Use Metaplex Token Metadata or Token-2022 `tokenMetadata` extension |

## Program IDs

| Program | ID |
|---------|----|
| SPL Token | `TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA` |
| Token-2022 | `TokenzQdBNbW9ZV7FPi1XEG4J8t75VmvFwAPc2gUwzbqb` |
| Associated Token Account | `ATokenGPvbdGVxr1b2hvZbsiqW5xWH25efZZEU9W6eB9v` |
| ZK ElGamal Proof | `ZkE1Gama1Proof11111111111111111111111111111` |

## Workflow

1. Read the relevant section of `knowledge/ecosystem/web-resources.md` (Token-2022, confidential transfers, RWA tokens).
2. Choose CLI, TypeScript client (`@solana/kit` + `@solana-program/token-2022`), or Anchor CPI.
3. Create the mint with required extensions.
4. Create token accounts (ATAs) for holders.
5. Mint initial supply if needed.
6. For confidential transfers: generate ElGamal/AES keys, configure accounts, deposit → apply → transfer → apply → withdraw.
7. Record mint address, holder ATAs, and transaction signatures.

## Output

- Script or program files with step-by-step commands
- Mint/account addresses
- Devnet/mainnet transaction signatures and Explorer links
- Notes on extensions used and their limitations
