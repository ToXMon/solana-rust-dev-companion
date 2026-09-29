# Ecosystem & tooling — cohort notes

Tools, frameworks, and ecosystem trends the instructors mentioned in passing. Read this when picking tooling or checking what versions/standards are actually usable. Related: `knowledge/ecosystem/solana-right-now.md` for current ecosystem state.

- **Anchor CLI:** `anchor init`, `anchor build`, `anchor expand`, `anchor test`, `anchor test --skip-build`. (`2026_08_31`, ~00:40:30–00:42:42; `2026_09_04`, ~00:04:34–00:04:36)
- **Anchor Version Manager (AVM):** use it to switch Anchor versions. The AMM starter repo may need AVM `1.0.1` if mixed versions cause build issues. (`2026_08_31`, ~00:42:42–00:45:14; `2026_09_09`, ~01:15:59–01:16:17)
- **Crucible:** testing tool from Asymmetric Research mentioned for Anchor testing. (`2026_08_31`, ~00:42:42–00:45:14)
- **Anchor v2 / Pinocchio / Quasar:** Anchor v2 will use the Pinocchio entry point; zero-copy types may narrow the performance gap. Pinocchio is faster but more boilerplate; Quasar is more ergonomic but had maintenance concerns. (`2026_08_28`, ~01:31:33–01:35:15; `2026_08_31`, Anchor performance segment)
- **Token standards adoption caveat:** newer standards (Token 2022, Core NFTs, compressed NFTs) may be technically superior but lack full explorer/wallet/app support; evaluate real-world adoption before committing. (`2026_08_28`, ~01:04:43–01:05:24)
- **Metaplex Token Metadata:** metadata is stored in a PDA derived from the mint. Wrong-mint bugs are common when chaining mint + metadata instructions. (`2026_08_26`, ~01:16:32–01:18:28)
- **Iris uploader:** used in class for image/metadata storage; conceptually similar to IPFS/Pinata. (`2026_08_28`, ~00:35:18–00:40:00)
- **Umi:** Metaplex JS framework referenced for NFT work. (`2026_08_28`)
- **SBF ABI v2:** Firedancer/Anza working on a better ABI; not yet usable on mainnet; SIMD still in progress. (`2026_09_04`, ~00:47:32–00:48:21)
- **Solana SVM/LLVM toolchain:** additional resources on SVM internals, `cargo build-sbf`, LLVM, and Agave/Solana-LLVM were promised for interested students. (`2026_09_09`, ~01:27:03–01:28:25)
- **Rust toolchain pinning:** `rust-toolchain.toml` pins stable Rust, but Anchor uses a modified/older toolchain for SBF builds; experimental features stable in current Rust may still be experimental inside Anchor's toolchain. (`2026_09_04`, ~00:35:16–00:36:43)
- **Light SVM / lite SVM:** in-process VM used in tests. Good for quick iteration; `solana-test-validator` / SurfPool gives a fuller validator if needed. (`2026_09_04`, ~00:28:17–00:28:44)
- **MPL Core + Anchor version compatibility:** MPL Core Anchor crate targets ~Anchor 0.32.2, works with 0.31.x/0.30.x — see `nfts-metaplex-core.md`. (`2026_09_21`, ~01:08:55–01:09:40)

MEV/block-builder notes (Jito, Harmonic, sandwiching) live in `defi-amm.md`.
