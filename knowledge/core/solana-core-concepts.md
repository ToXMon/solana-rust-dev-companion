---
title: "Solana Core Concepts"
description: "Beginner Solana concept module for the builder cohort."
tags: ["solana", "core-concepts", "accounts", "programs", "instructions", "transactions", "fees", "pda", "cpi", "tokens", "clusters", "rpc"]
---

# Solana Core Concepts

Beginner Solana concept module for the builder cohort.

- Docs: <https://solana.com/docs/core>

## Table of Contents

- [0. The one mental shift (start here)](#0-the-one-mental-shift-start-here)
- [1. Accounts](#1-accounts)
- [2. Programs](#2-programs)
- [3. Instructions](#3-instructions)
- [4. Transactions and fees](#4-transactions-and-fees)
- [5. Program Derived Addresses (PDAs)](#5-program-derived-addresses-pdas)
- [6. Cross-Program Invocations (CPIs) and tokens](#6-cross-program-invocations-cpis-and-tokens)
- [7. Clusters and endpoints](#7-clusters-and-endpoints)

## 0. The one mental shift (start here)

Not a docs page, this is the mental model everything else hangs on.

- **State lives in accounts, not in programs.** On Ethereum a contract holds its code and its storage at one address. Solana splits them.
- **Programs are stateless.** A program is pure code with an ID. It holds no data of its own.
- **Accounts hold the state.** Every piece of state (your balance, a token, config) is a separate account with its own address.
- **Everything a transaction touches is declared up front.** The runtime knows exactly which accounts a transaction will read or write before it runs.

> **EVM contrast to say out loud:** "A program is a function. Accounts are its arguments and its outputs."

### Learner check

1. Explain to someone from Ethereum why the program and its data live at different addresses.
2. Finish the sentence: "A Solana program is a function, and accounts are its ___."

## 1. Accounts

- Docs: <https://solana.com/docs/core/accounts>

Everything on Solana is an **account**. An account is a record in a big key-value store, keyed by a 32-byte address.

### The fields every account has

| Field | Description |
|-------|-------------|
| `address` | The account's 32-byte public key, its identity and how you look it up. |
| `lamports` | The SOL balance (1 SOL = 1,000,000,000 lamports). |
| `owner` | The program ID that controls this account. Only the owner can change the data or reduce the lamports. |
| `data` | A raw byte buffer. Has no built-in structure; the owning program decides what the bytes mean. |
| `executable` | A true/false flag. If true, this account is a program. |
| `rent_epoch` | A legacy field, mostly ignored today. |

### Types of accounts a builder will meet

| Type | Description |
|------|-------------|
| **Wallet account** | Owned by the System Program, holds SOL, controlled by a keypair. |
| **Data / state account** | Owned by your program, holds your program's data (your `struct` as bytes). |
| **Program account** | `executable`, its `data` field holds the compiled program bytecode. |
| **Native program accounts** | Built-in programs like System and Token (more in topic 2). |
| **Sysvar accounts** | System data the runtime exposes, like `Clock` and `Rent`. |

### Rent (why accounts cost SOL)

- Every validator stores every account, so accounts must pay to exist.
- An account must hold a minimum balance to be **rent-exempt**. The bigger the `data`, the higher the minimum.
- You do not compute it by hand; the runtime gives you the number for a given size.
- Drop below the minimum and the account can be deleted.
- **Rent is a refundable deposit, not a fee.** Close the account and you get it back.

### Learner check

1. Name the six fields of an account.
2. What does the `owner` field enforce, and why is it the core security rule?
3. Why does an account have to pay rent?
4. What is the difference between a wallet account and a program account?

## 2. Programs

- Docs: <https://solana.com/docs/core/programs>

A **program** is Solana's version of a smart contract: compiled code with an address, and nothing else.

- **Stateless code.** A program is compiled to sBPF bytecode and stored in a program account (`executable = true`). It holds no data of its own.
- **Program ID.** The program's address. Clients and other programs use it to call the program.
- **It owns accounts to store state.** When a program creates a data account, it sets that account's `owner` field to its own program ID. From then on only that program can write to it.

### Native (built-in) programs you will actually use

| Program | Purpose |
|---------|---------|
| **System Program** | Creates accounts, transfers SOL, assigns ownership. |
| **Token Program** (and Token-2022) | Creates and manages token mints and token balances. |
| **BPF Loader** | Deploys and upgrades programs. |
| **Compute Budget Program** | Sets the compute limit and priority fee for a transaction. |
| **Stake and Vote programs** | Validator infrastructure (awareness only). |

### Upgradeable vs frozen

A program can have an **upgrade authority** that can ship new code, or that authority can be removed to make it immutable.

### Learner check

1. Why is a Solana program called stateless?
2. Which native program creates accounts and transfers SOL?
3. How does a program come to "own" a data account?

## 3. Instructions

- Docs: <https://solana.com/docs/core/transactions>

An **instruction** is the smallest unit of work: one request to run one program. Transactions are built out of these.

### An instruction has three parts

1. **Program ID** — which program should run.
2. **Accounts list** — every account this instruction will touch, each tagged with two flags:
   - `is_signer` — did this account sign the transaction?
   - `is_writable` — is this instruction allowed to change it?
3. **Data** — a blob of bytes telling the program which action to run and with what arguments.

### Key ideas

- A transaction bundles one or more instructions, run in order.
- Everything the instruction can touch is listed explicitly. The program can not reach for an account that was not passed in.
- This up-front declaration is what lets Solana run non-overlapping transactions in parallel.

### Learner check

1. What are the three parts of an instruction?
2. What do `is_signer` and `is_writable` mean?
3. Why does every account have to be listed on the instruction instead of discovered while it runs?

## 4. Transactions and fees

- Docs: <https://solana.com/docs/core/transactions> and <https://solana.com/docs/core/fees>

A **transaction** is the atomic unit of execution. All of its instructions succeed together, or the whole thing is rolled back.

### What a transaction contains

- **Signatures** — one per required signer. The first signer is the fee payer.
- **Message** — the account addresses it touches, a recent blockhash, and the compiled instructions.

### Things to teach

- **Atomic.** If any instruction fails, every change in the transaction is undone. Nothing partial lands.
- **Recent blockhash.** Proves the transaction is fresh and prevents it from being replayed later. It expires after roughly 60 to 90 seconds, so transactions have to be sent promptly.
- **Size limit.** A transaction is capped at about 1232 bytes; everything has to fit.
- **Confirmation levels** (how sure you are it happened): `processed` (a validator saw it), `confirmed` (a supermajority voted), `finalized` (permanent).

### Fees (two kinds, keep them separate from rent)

| Fee | Description |
|-----|-------------|
| **Base fee** | 5,000 lamports per signature. Flat and predictable. |
| **Priority fee** | Optional. You pay a bit per compute unit to jump the queue when the network is busy. Set via the Compute Budget program. |

**Rent is not a fee.** It is the refundable deposit from topic 1. Do not let learners confuse the two.

### Learner check

1. What makes a transaction atomic?
2. What is a recent blockhash for, and why do transactions expire?
3. Base fee vs priority fee vs rent: which is flat, which is optional, which is refundable?
4. Name the three confirmation levels.

## 5. Program Derived Addresses (PDAs)

- Docs: <https://solana.com/docs/core/pda>

A **PDA** is an account address a program can control and "sign" for, without anyone holding a private key. This is how programs own structured, per-user data.

- **Deterministic address.** Derived from the program ID plus some seeds (for example the text `"vault"` plus a user's public key). Same inputs always give the same address.
- **No private key.** A PDA is deliberately placed off the normal key curve, so no keypair exists for it. Only the program can act for it.
- **The bump.** The derivation adds one extra byte, the bump, to land on a valid off-curve address. You store the bump and re-derive it later to verify the account.
- **Program signing.** A program signs on behalf of its PDA by passing the seeds back in (`invoke_signed`, covered in topic 6).

### What they are for

Per-user state (the Solana answer to Ethereum's `mapping[address => data]`), vaults that hold funds, and config or authority accounts.

### Learner check

1. What is a PDA, and why does it have no private key?
2. How does a program prove it controls a PDA?
3. You had `mapping[address => Balance]` in Solidity. How do you do the same thing on Solana?
4. What is the bump and why do you store it?

## 6. Cross-Program Invocations (CPIs) and tokens

- Docs: <https://solana.com/docs/core/cpi> and <https://solana.com/docs/core/tokens>

### CPIs — programs calling programs

- One program calls another mid-transaction (for example your program calls the System Program to create an account, or the Token Program to move tokens).
- `invoke` — a plain call to another program.
- `invoke_signed` — a call where your program signs using a PDA's seeds. This is how a program authorizes actions on accounts it controls.
- Signer and writable privileges carry through to the program being called.
- Calls can nest, up to a depth of 4.

### Tokens (SPL Token)

- Tokens on Solana are not their own smart contracts. They are accounts managed by the shared Token Program.
- **Mint account** — defines a token: its supply, decimals, and who can mint.
- **Token account** — one holder's balance of one specific token. Owned by the Token Program, associated with the holder's wallet.
- **Token-2022** — a newer version with extra features like transfer fees (awareness only for beginners).

### Learn more

- [SPL Token basics](https://solana.com/docs/tokens/basics)
- [SPL Token extensions](https://solana.com/docs/tokens/extensions)

### Learner check

1. What is a CPI, and what does `invoke_signed` add over `invoke`?
2. What is the difference between a mint account and a token account?
3. Which program owns a user's token account?

## 7. Clusters and endpoints

- Docs: <https://solana.com/docs/references/clusters>

The practical wrap: where code actually runs and how your app talks to it.

### Clusters — separate Solana networks

| Cluster | Purpose |
|---------|---------|
| `mainnet-beta` | Real network, real money. |
| `devnet` | Free playground with fake SOL you can airdrop. Where the cohort builds. |
| `testnet` | For validator and release testing. |
| `localnet` | A validator running on your own machine (`solana-test-validator`) for fast local testing. |

### RPC endpoint

The URL your app or CLI points at to talk to a cluster. It sends transactions and reads account data (JSON-RPC over HTTP; WebSocket for live subscriptions).

### Getting test SOL

Airdrop it on `devnet`, either from the CLI or a faucet.

### Basic toolchain

The Solana CLI, a wallet, a local validator, and a framework (Anchor or native) to write the program.

### Learner check

1. Name the clusters and what each one is for.
2. Where do you get SOL to test with, and on which cluster?
3. What is an RPC endpoint and what do you use it for?
