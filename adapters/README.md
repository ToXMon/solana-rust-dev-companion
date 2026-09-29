# Adapters — using the companion with your agent harness

The repo is harness-agnostic: `ENTRY.md` routes any agent that can read files. These adapters wire specific harnesses to it.

## Any harness: `npx skills`

```bash
npx skills add ToXMon/turbin3-solana-companion -g
```

Then invoke `/turbin3 …` (the thin entry skill — it loads `PRINCIPLES.md` + `ENTRY.md` and routes).

## Claude Code

- Plugin marketplace (installs every skill under `skills/`):

  ```
  /plugin marketplace add ToXMon/turbin3-solana-companion
  /plugin install turbin3-solana-companion@turbin3-solana-companion
  ```

- Or copy root `CLAUDE.md` into your working repo (it embeds the principles + a pointer to `ENTRY.md`).

## Cursor

Copy `.cursor/rules/turbin3-solana.mdc` into your working repo's `.cursor/rules/`. It has `alwaysApply: true` frontmatter.

## Devin

Copy each `skills/<name>/` directory into your working repo's `.devin/skills/`, or point Devin at `AGENTS.md` (it reads it natively).

## Codex / OpenCode / any AGENTS.md-reading harness

Nothing to install — the root `AGENTS.md` instructs the agent to read `PRINCIPLES.md` then `ENTRY.md`. Clone this repo as a sibling of your working repo.

## Windsurf

Copy the root `CLAUDE.md` content into `.windsurfrules` in your working repo (Windsurf reads `.windsurfrules`; the content is identical to the Cursor rule minus frontmatter).

## Using alongside `sb2026` (the working repo)

The recommended layout is a sibling clone:

```
~/sb2026/                      ← your working repo (code, capstone, transcripts)
~/turbin3-solana-companion/    ← this repo (knowledge + skills)
```

Then tell your agent: **"read `../turbin3-solana-companion/ENTRY.md`"**. The agent loads principles + router from the companion, reads your working repo for existing code, and writes outputs (designs, programs, LOI drafts) into *your* repo — not the companion.
