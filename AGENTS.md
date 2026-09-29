# AGENTS.md

This repository is the **Turbin3 Solana Companion**: a harness-agnostic distro of principles, knowledge, skills, and workflows for building professional-grade, security-first Solana dapps end to end. It works with any coding agent that can read files (Claude Code, Cursor, Devin, Codex, Windsurf, OpenCode, Cline, …).

## On every session

1. Read `PRINCIPLES.md` — the behavioral contract. Non-negotiable.
2. Read `ENTRY.md` — the router. Load only the knowledge/skills the ask needs.
3. Apply the security checklist in `PRINCIPLES.md` Part B to any program code you produce or review.

## Invoking skills

Skills live in `skills/<name>/SKILL.md` with YAML frontmatter. Harnesses that support slash-skills can invoke `/solana-guide`, `/solana-architect`, `/safe-solana-builder`, etc. Harnesses that don't: read the SKILL.md and follow it as instructions.

`skills/turbin3/SKILL.md` is the thin entry skill (`/turbin3 <anything>`) — it loads `PRINCIPLES.md` + `ENTRY.md` and routes.

## Conventions

- Knowledge files are Markdown, self-describing in their first lines, organized by *how an LLM looks things up* (`knowledge/<category>/`), not by where they came from.
- Skills reference knowledge by repo-relative path (`knowledge/patterns/code-patterns.md`), never by the old `resources/…` paths.
- Cohort-derived material cites the source transcript filename and approximate timestamp; ASR is unverified.
- Do not add dependencies, build systems, or code here. This repo is docs + skills only. Examples are Markdown artifacts, not runnable projects.

## Safety

- Never commit private keys, wallet JSONs, `.env`, or API keys — here or in the user's working repo.
- Devnet first. Mainnet only with explicit user confirmation and a security review.

## Keeping it alive

- New learnings → append to the relevant `knowledge/` file (or add one + register in `knowledge/README.md`).
- New cohort transcript → summarize into `knowledge/cohort/` using `skills/solana-research`, keep `knowledge/cohort/README.md` index current.
- When you change a principle, update `PRINCIPLES.md` and every adapter in `adapters/` that mirrors it.
