# ENTRY.md

You have been pointed at the **Turbin3 Solana Companion** — a harness-agnostic knowledge base and skill set for building professional-grade, security-first Solana dapps end to end. Read `PRINCIPLES.md` first, then use this file to decide what to load.

Load only what the ask needs. Every file below is self-contained and states its purpose in its first lines.

## Repo map

```
PRINCIPLES.md              behavioral contract (think → simplify → surgical → verify; security first)
ENTRY.md                   this router
knowledge/
  core/                    Solana fundamentals, Rust-for-Anchor, cheat sheet, mental-model primer
  process/                 how to work: architecture diagramming, LOI template, homework tracker
  patterns/                code patterns, use-case recipes, stack selector
  security/                audit-derived patterns, FYEO findings catalog, review playbook, methodology
  ecosystem/               what's hot right now, synthesized web resources, static links
  cohort/                  Turbin3 Q3 2026 transcript insights (indexed by topic)
skills/                    invocable skills (SKILL.md each) — see skills/README.md
agents/                    subagent role profiles (architect, programmer, frontend, security, researcher)
workflows/                 multi-step build graphs (idea → shipped, audit loop, arch diagram, LOI)
examples/                  worked artifacts: LEASH LOI, atomic requirements, red-team log
adapters/                  drop-in files for Claude Code, Cursor, Devin, Codex, Windsurf, etc.
```

## How to route the ask

| The user wants to… | Do this |
|---|---|
| Learn / is vague | `skills/solana-guide` → ask 1–3 questions, then route |
| Brainstorm or revise an LOI / capstone proposal | `skills/solana-loi` + `knowledge/process/loi-template.md` + `examples/loi/`. Human-first: red-team what they have, don't draft from nothing |
| Produce the architecture diagram / requirements | `skills/solana-architect` + `knowledge/process/architecture-diagramming.md`. **Requirements before diagrams.** Use `examples/requirements/` as the model format |
| Do a homework assignment | `knowledge/process/homework-tracker.md` for what's due → the linked skill. Build from scratch, reference repo as guide only |
| Write / fix an Anchor program | `skills/safe-solana-builder` (always) + `skills/solana-build` + `knowledge/patterns/code-patterns.md` + `knowledge/security/` |
| Tokens / NFTs / DeFi / RWA / privacy | the matching domain skill under `skills/` + topic section in `knowledge/ecosystem/web-resources.md` |
| Frontend | `skills/solana-frontend` |
| Tests | `skills/solana-test` |
| Security review / audit | `skills/solana-security` + `knowledge/security/AGENTS.md` + `knowledge/security/fyeo-audit-findings-catalog.md`; prep for an external audit with `skills/solana-audit-prep` + `knowledge/security/audit-readiness-checklist.md` |
| Understand a concept | `knowledge/core/` first; `knowledge/cohort/` for how the instructors taught it |
| Update the knowledge base | `skills/solana-research` |
| Full-stack dapp end to end | `workflows/README.md` → Workflow 1 |

## Working rules

1. **Never skip the requirements step** for anything bigger than a single instruction. If no numbered atomic requirements exist, produce them (or ask the user to) before designing or coding.
2. **Security checklist in `PRINCIPLES.md` Part B applies to every program**, including homework. Homework is practice for production habits.
3. **Test after each instruction.** Don't implement three instructions and test once.
4. **Devnet receipts.** A build isn't done until you can show a program ID and transaction signature.
5. **Validation is adversarial.** After implementation, run a fresh-context review (subagent/separate session) against intent + checklist. Report findings; the human decides.
6. **Companion to a working repo.** This repo holds knowledge and skills. The user's actual code lives elsewhere (e.g., `sb2026/`). Write designs and outputs into *their* repo, not here — unless they're adding knowledge.
7. **Cite what you used.** When you rely on a knowledge file, name it so the user can read the source.
8. **If this repo doesn't cover it, say so** and fall back to your own knowledge — clearly labeled as such.

## Using with the sb2026 working repo

`sb2026/` is the user's Turbin3 working repo (homework programs, capstone, video transcripts). Typical pairing:
- Read this repo for *how*; read `sb2026/` for *what exists*.
- Homework: check `knowledge/process/homework-tracker.md` here, then work inside the matching `sb2026/<project>/`.
- LOI / architecture: current drafts live in `sb2026/capstone/`; the worked examples here in `examples/` are snapshots of those.
- New cohort transcripts: `sb2026/video/transcripts/` → summarize into `knowledge/cohort/` via `skills/solana-research`.
