# Solana Custom Agent Profiles

These profiles live under `agents/` and can be spawned as subagents for parallel, focused work.

## Available profiles

| Profile | Best for |
|---------|----------|
| `solana-architect` | Design-only work before implementation |
| `solana-programmer` | Writing and deploying Anchor programs |
| `solana-frontend-dev` | Building React/TypeScript clients |
| `solana-security-reviewer` | Security audits and findings reports |
| `solana-researcher` | Fetching and summarizing latest ecosystem info |

## How to use

From any skill or the main agent, describe the task and ask to use the profile. Example:

> "Use the solana-security-reviewer subagent to audit the escrow program while I finish the frontend."

The subagent will read the code, apply its system prompt, and return a findings summary.

## Design notes

- Profiles are model-agnostic; they only restrict tools to read/edit/grep/exec as appropriate.
- They are read/write capable except for the researcher, which also has `web_search` and `webfetch`.
- Security reviewer does not apply fixes unless explicitly asked; it produces a report.
