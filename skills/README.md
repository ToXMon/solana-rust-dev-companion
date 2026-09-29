# Solana Skills Index

All skills live in named subdirectories with a `SKILL.md` file. Invoke them with `/skill-name` in harnesses that support slash-skills, or read the SKILL.md and follow it.

## Navigation

- `/turbin3` — harness-agnostic entry skill; loads `PRINCIPLES.md` + `ENTRY.md` and routes any Solana/Turbin3 ask
- `/solana-guide` — interactive navigator; routes to the right skill for any goal

## Building

- `/solana-architect` — design a Solana program: numbered atomic requirements → actors/signers → traceable spec (methodology: `knowledge/process/architecture-diagramming.md`)
- `/solana-build` — implement, test, and deploy an Anchor program
- `/solana-frontend` — build a React/TypeScript client

## Domains

- `/solana-tokens` — SPL, Token-2022, extensions
- `/solana-nfts` — Metaplex Core assets, collections, plugins
- `/solana-defi` — escrow, vault, AMM, perp integrations
- `/solana-rwa` — tokenized real-world assets and money-market funds
- `/solana-privacy` — confidential transfers and privacy primitives

## Quality

- `/safe-solana-builder` — security-first scaffolding and implementation guidance for Anchor, Native Rust, or Pinocchio
- `/solana-security` — adversarial review driven by the FYEO findings catalog (`knowledge/security/`)
- `/solana-audit-prep` — readiness checklist + `docs/audit-scope.md` before an external audit
- `/solana-test` — testing with Anchor, LiteSVM, Surfpool, devnet, fuzz/invariants

## Maintenance

- `/solana-research` — fetch and synthesize latest ecosystem resources

## Cohort / academic

- `/solana-loi` — help draft and review Letters of Intent, capstone proposals, and weekly homework

## How to add a new skill

1. Create `skills/<name>/SKILL.md`.
2. Add YAML frontmatter with `name`, `description`, `argument-hint`, `allowed-tools`, and `permissions`. Skills are model-agnostic — do not pin a `model`.
3. Write a focused prompt body that references the knowledge base in `knowledge/`.
4. Register it here.

## Subagent profiles

Specialized agent profiles are in `agents/` and can be invoked as subagents for parallel work:
- `solana-architect`
- `solana-programmer`
- `solana-frontend-dev`
- `solana-security-reviewer`
- `solana-researcher`
