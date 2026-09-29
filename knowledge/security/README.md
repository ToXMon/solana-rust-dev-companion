# Security knowledge

Audit-derived Solana security guidance. `security-audit-patterns.md` in this directory is the checklist summary; `AGENTS.md` is the review playbook; the detailed rulebooks live in `skills/safe-solana-builder/references/` (Frank Castle's Safe Solana Builder).

## In this directory

- `security-audit-patterns.md` — extended security checklist distilled from audit practice: account validation, PDA/bump discipline, CPI safety, arithmetic, authority design.
- `vulnerability-abundance.md` — why no checklist/audit is complete; class elimination, the abundance distribution, prioritization, discoverability.
- `fyeo-audit-findings-catalog.md` — every finding from 23 FYEO public Solana audits, organized by vulnerability class, with prevention rules and grep-able "tells".
- `audit-methodology.md` — how FYEO structures a review (Kickoff→Ramp-up→Review→Reporting→Verify), exact severity definitions, fuzzing methodology, finding template.
- `audit-readiness-checklist.md` — what a team must have before requesting an external audit.
- `AGENTS.md` — instructions for an agent performing a security review using this directory (load order, feature→class mapping, grep tells, finding format).

## Detailed references (`skills/safe-solana-builder/references/`)

| File | Covers |
|------|--------|
| `references/shared-base.md` | Framework-agnostic rules for **every** Solana program: signer/owner checks, PDA validation, CPI safety, arithmetic, token handling. Read this first, always. |
| `references/anchor.md` | Anchor-specific patterns and pitfalls: correct account wrappers (`Account`, `UncheckedAccount`, `InterfaceAccount`, …), constraints, `init`/`close`, discriminators. Read after shared-base when using Anchor. |
| `references/native-rust.md` | Native Rust programs: manual deserialization/validation — every check Anchor does automatically, done by hand. |
| `references/pinocchio.md` | Pinocchio framework (Anza's zero-copy, low-CU framework): patterns, security notes, API reference. Unaudited — use with caution. |
| `references/litesvm.md` | LiteSVM testing reference: in-process VM setup, program loading, time travel. Applies to all frameworks. |

A worked example (NFT whitelist mint + security checklist) is in `skills/safe-solana-builder/examples/`.
