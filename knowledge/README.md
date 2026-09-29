# Knowledge base index

Every knowledge file, grouped by category. Files are organized by *how an agent looks things up*, not by source. Each file states its purpose in its first lines.

## `core/` — foundations

| File | Purpose |
|------|---------|
| `core/solana-core-concepts.md` | Solana fundamentals: accounts, programs, transactions, PDAs, CPIs, tokens |
| `core/rust-for-anchor.md` | The Rust you actually need to read/write Anchor programs |
| `core/cheat-sheet.md` | Quick reference: common patterns, program IDs, pitfalls |
| `core/mental-model-primer.md` | 5-minute mental-model read before writing or reviewing Solana code |

## `process/` — how to work

| File | Purpose |
|------|---------|
| `process/architecture-diagramming.md` | Atomic-requirements-first methodology for architecture diagrams (capstone deliverable) |
| `process/loi-template.md` | Letter of Intent / capstone proposal template |
| `process/homework-tracker.md` | What's due when, which skill to invoke, where supporting patterns live |

## `patterns/` — reusable build material

| File | Purpose |
|------|---------|
| `patterns/code-patterns.md` | Reusable program/test/frontend patterns distilled from real code |
| `patterns/use-case-recipes.md` | End-to-end build recipes for hot Solana use cases |
| `patterns/stack-selector.md` | Decision aid for programs, SDKs, and patterns per build |

## `security/`

| File | Purpose |
|------|---------|
| `security/security-audit-patterns.md` | Extended audit-derived security guidance; complements the PRINCIPLES.md checklist |
| `security/vulnerability-abundance.md` | Why no checklist/audit is complete; class elimination + prioritization mindset |
| `security/fyeo-audit-findings-catalog.md` | All ~170 findings from 23 FYEO public Solana audits by class, with prevention rules |
| `security/audit-methodology.md` | FYEO review pipeline, severity definitions, Trident fuzzing method, finding template |
| `security/audit-readiness-checklist.md` | Pre-audit requirements: pinned commit, spec, threat model, tests, authority plan |
| `security/AGENTS.md` | Agent playbook for running a security review with this directory |
| `security/README.md` | Index into `skills/safe-solana-builder/references/` |

## `ecosystem/` — current state of Solana

| File | Purpose |
|------|---------|
| `ecosystem/solana-right-now.md` | Current ecosystem priorities (Alpenglow, RWAs, perps, agents, p-token) |
| `ecosystem/web-resources.md` | Synthesized external resources by topic (Metaplex Core, RWA, privacy, DeFi) |
| `ecosystem/static-resources.md` | Curated links: SDKs, frameworks, testing, tokens, Rust |

## `cohort/` — Turbin3 Q3 2026 transcript insights

Indexed by topic, not chronology. Start at `cohort/README.md` for the index and curriculum map.

| File | Purpose |
|------|---------|
| `cohort/fundamentals-and-accounts.md` | Account/PDA/ATA/CPI mental models as taught |
| `cohort/anchor-practical.md` | Anchor account types, constraints, init/close, decision tree |
| `cohort/testing-and-debugging.md` | Test strategy, LiteSVM, time-travel, debugging tips |
| `cohort/mistakes-and-prevention.md` | Beginner mistakes table + prevention |
| `cohort/token-2022-and-extensions.md` | Extensions, confidential transfers, CPI guard, permanent delegate |
| `cohort/nfts-metaplex-core.md` | NFT standards + Metaplex Core staking workshop |
| `cohort/defi-amm.md` | AMM theory/implementation, MEV, order-flow toxicity |
| `cohort/rwa-tokenized-fund.md` | Tokenized MMF reference: NAV, roles, KYC gating |
| `cohort/capstone-loi-and-architecture.md` | LOI guidance, assignments, atomic-requirements session |
| `cohort/ecosystem-and-tooling.md` | Anchor CLI/AVM, Iris, Umi, toolchain notes |
| `cohort/quotes.md` | All verbatim instructor quotes by topic |
| `cohort/slides/` | Extracted slide-deck text |
