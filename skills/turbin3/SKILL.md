---
name: turbin3
description: >
  Turbin3 Solana Companion — guide the user through professional-grade, security-first Solana
  dapp development end to end: homework programs, architecture diagrams / atomic requirements,
  LOI brainstorming and red-teaming, Anchor programs, tokens, NFTs, DeFi, RWA, privacy, frontends,
  tests, and audits. Use on /turbin3 or whenever the user is working on Solana or Turbin3 cohort work.
user-invocable: true
metadata:
  short-description: "Security-first Solana full-stack companion for Turbin3 builders."
---

# /turbin3

This skill is intentionally thin. The real instructions live in the repo `ToXMon/turbin3-solana-companion`. Load them, then follow `ENTRY.md`.

## Loading (session-cached)

If the repo is already checked out locally (look for a directory containing `PRINCIPLES.md` and `ENTRY.md` — commonly `../turbin3-solana-companion` or `~/turbin3-solana-companion`), read from disk:

1. `PRINCIPLES.md`
2. `ENTRY.md`

Otherwise fetch the **full** content of:

- `https://raw.githubusercontent.com/ToXMon/turbin3-solana-companion/main/PRINCIPLES.md`
- `https://raw.githubusercontent.com/ToXMon/turbin3-solana-companion/main/ENTRY.md`

Fallback: `https://cdn.jsdelivr.net/gh/ToXMon/turbin3-solana-companion@main/<file>`

Then fetch/read only the additional `knowledge/`, `skills/`, `workflows/`, or `examples/` files that `ENTRY.md` routes you to for this ask, using the same base URL or local path.

## Rules

1. If a file was already fully read earlier in this session, don't re-read it unless asked to refresh.
2. If files cannot be loaded, stop and say so. Do not guess their contents.
3. After loading, follow `ENTRY.md` exactly. `PRINCIPLES.md` overrides your defaults where they conflict.
4. Outputs (designs, code, LOI drafts) go in the user's working repo, not in the companion repo.
