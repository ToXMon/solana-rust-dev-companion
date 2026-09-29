---
title: "Static Resources"
description: "Comprehensive resources to build knowledge in all aspects of Solana development."
tags: ["solana", "resources", "docs", "frontend", "frameworks", "testing", "tokens", "rust", "creators", "workshops", "simd", "advanced", "examples"]
---

# Static Resources

Comprehensive resources to build knowledge in all aspects of Solana development.

## Table of Contents

- [General](#general)
- [Frontend](#frontend)
- [Frameworks](#frameworks)
- [Testing](#testing)
- [Tokens](#tokens)
- [Advanced Solana](#advanced-solana)
- [Example programs](#example-programs)
- [Rust-specific material](#rust-specific-material)
- [Creators](#creators)
- [Must watch](#must-watch)
- [Must follow](#must-follow)

## General

| Resource | Link | Notes |
|----------|------|-------|
| **Solana Documentation** | [solana.com/docs](https://solana.com/docs) | The OG Solana docs, mostly updated. Solid foundation on basic Solana concepts and more. |
| **Solana Developers Portal** | [solana.com/developers](https://solana.com/developers) | Tools and resources for generic Solana development. |
| **Solana CLI** | [docs.anza.xyz/cli](https://docs.anza.xyz/cli/) | Command-line tools for installing Solana, managing keys, deploying programs, and local testing. Necessary to make Anchor work. |

## Frontend

| Resource | Link | Notes |
|----------|------|-------|
| **@solana/web3.js** (legacy JS/TS SDK) | [solana.com/docs/clients/official/javascript](https://solana.com/docs/clients/official/javascript) | Legacy but still widely used client library for Solana interactions in TypeScript/JavaScript. Handles RPC calls, transactions, and wallets. Most existing programs and Anchor clients rely on it. |
| **@solana/kit** (new JS/TS SDK) | [solanakit.com/docs](https://www.solanakit.com/docs) | New-generation, recommended TypeScript SDK for Solana (formerly web3.js v2). More functional, modular, and type-safe. Ideal for new projects and production client-side apps. |
| **Gill SDK** | [github.com/gillsdk/gill](https://github.com/gillsdk/gill) | Modern, lightweight TypeScript/JavaScript client library, compatible with kit. Good for Node, web, or React Native with simplified APIs. |
| **@solana-program/token** | [solanakit.com/docs/programs/token](https://www.solanakit.com/docs/programs/token) | Modern library for Solana tokens (integrated with kit): creating, minting, transferring. Replaces legacy `spl-token` in new setups. |
| **@solana/wallet-adapter** | [github.com/anza-xyz/wallet-adapter](https://github.com/anza-xyz/wallet-adapter) | Popular TypeScript library for integrating wallet connections (e.g., Phantom, Solflare) in client-side apps. |

## Frameworks

| Resource | Link | Notes |
|----------|------|-------|
| **Anchor Framework** | [anchor-lang.com](https://www.anchor-lang.com/) / [docs](https://www.anchor-lang.com/docs) | The framework used to develop ~80% of current mainnet programs. Built-in testing, security features, IDL generation, and everything needed for easy Solana program development. |
| **Pinocchio** | [github.com/anza-xyz/pinocchio](https://github.com/anza-xyz/pinocchio) | Zero-external-dependency library for Solana programs in Rust. Uses only Solana SDK types for on-chain programs. Efficient zero-copy library optimized for compute-unit consumption and binary size. |
| **Codama** | [github.com/codama-idl/codama](https://github.com/codama-idl/codama) | Tool for generating type-safe client code (TypeScript/JavaScript) from Solana program IDLs. Works with Anchor, Shank, and Solana Kit. |
| **StarFrame** (honorable mention) | [build.staratlas.com/dev-resources/star-frame](https://build.staratlas.com/dev-resources/star-frame) | — |
| **Steel** (honorable mention) | [github.com/regolith-labs/steel](https://github.com/regolith-labs/steel) | — |

## Testing

| Resource | Link | Notes |
|----------|------|-------|
| **Surfpool** | [surfpool.run](https://surfpool.run/) | Drop-in replacement for `solana-test-validator`. Enables local testing with real Mainnet state, perfect for simulating programs and transactions without full chain downloads or manual account clones. |
| **LiteSVM** | [litesvm.com](https://www.litesvm.com/) / [Anchor docs](https://www.anchor-lang.com/docs/testing/litesvm) | Fast, lightweight in-process Solana VM for testing programs in Rust, TypeScript, or Python. Optimized for quick iterations; up to 25x faster than traditional validators. |

## Tokens

| Resource | Link | Notes |
|----------|------|-------|
| **SPL Token basics** | [solana.com/docs/tokens/basics](https://solana.com/docs/tokens/basics) | Official docs covering token mints, accounts, transfers, and the Token Program. |
| **SPL Token extensions** | [solana.com/docs/tokens/extensions](https://solana.com/docs/tokens/extensions) | Official docs for Token-2022 extensions: transfer fees, confidential transfers, memo, etc. |
| **Solana Foundation tokens repo** | [github.com/solana-foundation/tokens](https://github.com/solana-foundation/tokens) | Reference implementations and examples for working with Solana tokens. |

## Advanced Solana

| Resource | Link | Notes |
|----------|------|-------|
| **Engineering Solana (Aseneca)** | [aseneca.systems/engineering-solana](https://aseneca.systems/engineering-solana) | A practitioner's systems-engineering deep dive into the SVM, AccountsDB, TPU, consensus, and protocol tooling. |

## Example programs

| Resource | Link | Notes |
|----------|------|-------|
| **AMM example program (Q3 cohort)** | [github.com/ShrinathNR/amm_q2_26](https://github.com/ShrinathNR/amm_q2_26) | Example AMM program implementation for the builder cohort. |
| **SPL NFT example program (Q3 cohort)** | [github.com/ShrinathNR/spl-nft-q326](https://github.com/ShrinathNR/spl-nft-q326) | Example SPL NFT program implementation for the builder cohort. |

## Rust-specific material

| Resource | Link | Notes |
|----------|------|-------|
| **The Rust Book** | [doc.rust-lang.org/book](https://doc.rust-lang.org/book/) | Official Rust programming language documentation. The first 6 chapters are sufficient to build basic Anchor programs. The whole book is a must for advanced tasks. |
| **Rust by Example** | [doc.rust-lang.org/rust-by-example](https://doc.rust-lang.org/rust-by-example/) | Runnable examples demonstrating Rust features from basics to advanced topics like traits and generics. Free and official. |
| **Rustlings** | [github.com/rust-lang/rustlings](https://github.com/rust-lang/rustlings) | Small command-line exercises to practice reading, writing, and fixing Rust code, including handling compiler errors. Free and highly effective for building intuition. |
| **Crust of Rust (YouTube)** | [YouTube playlist](https://www.youtube.com/playlist?list=PLqbS7AVVErFiWDOAVrPt7aYmnuuOLYvOa) | Intermediate videos exploring Rust topics like lifetimes and standard library internals through code walkthroughs. |
| **Let's Get Rusty (YouTube)** | [YouTube channel](https://www.youtube.com/c/LetsGetRusty) | Beginner-to-intermediate videos on Rust fundamentals and projects, including a "Rust Job-Ready Roadmap." Great for visual learners. |

## Creators

| Creator | Channel | Focus |
|---------|---------|-------|
| Jon Gjengset | [youtube.com/@jonhoo](https://www.youtube.com/@jonhoo) | Low-level Rust development. |
| Brimigs | [youtube.com/@brimigs](https://www.youtube.com/@brimigs) | Solana development. |
| Jonas Hahn | [youtube.com/@jonashahnsolana](https://www.youtube.com/@jonashahnsolana) | Solana development, IoT. |

## Must watch

Showcasing multiple tools by the developers: [Toolapalooza 2025](https://www.youtube.com/live/fdOI3zJR6A8).

| Workshop | Notes |
|----------|-------|
| Magic Block Workshop | — |
| Switchboard Workshop | — |
| Yellowstone Vixen Workshop | — |
| LiteSVM Workshop | — |
| Surfpool Workshop | — |
| Turbin3 Tribe Jam | — |
| Carbon Workshop | — |
| Codama Workshop | — |
| GILL SDK Workshop | — |

## Must follow

- **Solana SIMD documents:** [github.com/solana-foundation/solana-improvement-documents](https://github.com/solana-foundation/solana-improvement-documents)
