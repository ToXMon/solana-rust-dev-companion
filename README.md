# Turbin3 Solana Companion

A harness-agnostic knowledge base and skill set for building professional-grade, **security-first** Solana dapps end to end — built alongside the Turbin3 Q3 2026 Builders Cohort. It encodes three working agreements: **security is a design input**, **simplicity beats cleverness**, and **requirements come before diagrams, diagrams before code**. It works with any coding agent that can read files: Claude Code, Cursor, Devin, Codex, Windsurf, OpenCode, and friends.

## Quick start

**Option 1 — skills installer (any harness):**

```bash
npx skills add ToXMon/turbin3-solana-companion -g
```

Then ask: `/turbin3 help me design my capstone vault program`

**Option 2 — sibling clone (recommended):**

```bash
git clone https://github.com/ToXMon/turbin3-solana-companion ~/turbin3-solana-companion
```

Keep it next to your working repo and tell your agent: *"read `../turbin3-solana-companion/ENTRY.md`"*.

**Option 3 — Claude Code plugin marketplace:**

```
/plugin marketplace add ToXMon/turbin3-solana-companion
/plugin install turbin3-solana-companion@turbin3-solana-companion
```

More per-harness adapters (Cursor rules, Devin, Windsurf, AGENTS.md): `adapters/README.md`.

## How it works

```
PRINCIPLES.md          ENTRY.md               knowledge/  skills/  workflows/  agents/
(behavioral            (router — picks        (what to    (how to  (multi-step  (subagent
 contract)      ──►    the files for           know)       do it)   graphs)     profiles)
                        your ask)
                              │
                              ▼
                    output lands in YOUR working repo
                    (designs, programs, LOI drafts — never here)
```

## What's inside

| Directory | Contents |
|-----------|----------|
| `knowledge/core/` | Solana fundamentals, Rust-for-Anchor, cheat sheet, mental models |
| `knowledge/process/` | Architecture-diagram methodology, LOI template, homework tracker |
| `knowledge/patterns/` | Code patterns, use-case recipes, stack selector |
| `knowledge/security/` | Audit-derived patterns, FYEO findings catalog (~170 real findings by class), review playbook, methodology, readiness checklist |
| `knowledge/ecosystem/` | What's current on Solana, synthesized web resources, curated links |
| `knowledge/production/` | STRIDE-style release readiness, Triton transaction delivery, Fumarole/Jetstreamer indexing, monitoring, and incident response |
| `knowledge/gtm/` | Builder positioning, demo, launch, and getting-noticed lessons for Solana builders |
| `knowledge/cohort/` | Turbin3 transcript insights indexed by topic + slide extracts |
| `skills/` | 18 invocable skills (`/turbin3`, `/solana-architect`, `/safe-solana-builder`, `/solana-production-readiness`, `/solana-streaming-indexing`, `/solana-gtm`, …) |
| `agents/` | Subagent profiles (architect, programmer, frontend, security, researcher) |
| `workflows/` | Multi-step build graphs: idea → shipped, audit loop, arch diagram, LOI |
| `examples/` | Worked artifacts from one real capstone (LEASH): LOI, requirements, red-team log |
| `adapters/` | Install instructions per harness |

## Example asks

- "Do my week-3 homework" → homework tracker → matching skill, build from scratch
- "Produce my capstone architecture diagram" → numbered atomic requirements first, then the diagram
- "Red-team my LOI draft" → adversarial findings log, you decide what to adopt
- "Audit this escrow program" → security checklist + audit patterns + findings doc
- "Build me a full-stack dapp" → Workflow 1: architect → build → test → frontend → security

## Using with `sb2026`

`sb2026/` is the companion's sibling working repo (homework programs, capstone, video transcripts). Read this repo for *how*; read `sb2026/` for *what exists*. Outputs always land in the working repo. See `adapters/README.md` and `ENTRY.md`.

## Credits

- **Turbin3 instructors** — the cohort content distilled in `knowledge/cohort/`
- **Frank Castle** — `skills/safe-solana-builder` security rules
- **multica-ai/andrej-karpathy-skills** (MIT) — behavioral guideline patterns in `PRINCIPLES.md`
- **kunchenguid/kun** — task-routing pattern in `PRINCIPLES.md` Part A

## License

MIT — see `LICENSE`.
