# Security knowledge

Audit-derived Solana security guidance. `security-audit-patterns.md` in this directory is the summary; the detailed rulebooks live in `skills/safe-solana-builder/references/` (Frank Castle's Safe Solana Builder). Read the summary first, then the reference matching your framework.

## In this directory

- `security-audit-patterns.md` — extended security checklist distilled from audit practice: account validation, PDA/bump discipline, CPI safety, arithmetic, authority design.

## Detailed references (`skills/safe-solana-builder/references/`)

| File | Covers |
|------|--------|
| `references/shared-base.md` | Framework-agnostic rules for **every** Solana program: signer/owner checks, PDA validation, CPI safety, arithmetic, token handling. Read this first, always. |
| `references/anchor.md` | Anchor-specific patterns and pitfalls: correct account wrappers (`Account`, `UncheckedAccount`, `InterfaceAccount`, …), constraints, `init`/`close`, discriminators. Read after shared-base when using Anchor. |
| `references/native-rust.md` | Native Rust programs: manual deserialization/validation — every check Anchor does automatically, done by hand. |
| `references/pinocchio.md` | Pinocchio framework (Anza's zero-copy, low-CU framework): patterns, security notes, API reference. Unaudited — use with caution. |
| `references/litesvm.md` | LiteSVM testing reference: in-process VM setup, program loading, time travel. Applies to all frameworks. |

A worked example (NFT whitelist mint + security checklist) is in `skills/safe-solana-builder/examples/`.
