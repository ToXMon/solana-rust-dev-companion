# Solana Build Workflows

This directory documents the multi-step agent graphs (workflows/loops) that turn an idea into a shipped Solana dapp. Each workflow can be driven by the skills in `skills/` and the custom agent profiles in `agents/`.

## Workflow 1: Idea → Shipped Dapp

```
User idea
    │
    ▼
/solana-architect  ──► design doc (`docs/design.md` in the user's working repo)
    │
    ▼
/solana-build  ──► Anchor program + tests
    │
    ▼
/solana-test  ──► passing local + devnet tests
    │
    ▼
/solana-frontend  ──► React/Next.js client
    │
    ▼
/solana-security  ──► audit and fix loop
    │
    ▼
/solana-research  ──► refresh docs if needed during the build
    │
    ▼
Shipped dapp (program ID + tx links + UI)
```

## Workflow 2: Specialized Build (tokens, NFTs, DeFi, RWA, privacy)

```
User idea
    │
    ▼
/solana-architect  (design doc)
    │
    ▼
/solana-tokens  OR  /solana-nfts  OR  /solana-defi  OR  /solana-rwa  OR  /solana-privacy
    │
    ▼
/solana-test  +  /solana-security
    │
    ▼
Deploy + verify
```

## Workflow 3: Audit Loop

```
Existing code
    │
    ▼
/solana-security  ──► findings doc
    │
    ▼
/solana-build  ──► fixes
    │
    ▼
/solana-test  ──► regression tests
    │
    ▼
back to /solana-security until clean
```

## Workflow 4: Research → Spec → Scaffold

```
New topic or ecosystem change
    │
    ▼
/solana-research  ──► update knowledge/
    │
    ▼
/solana-architect  ──► spec
    │
    ▼
/solana-build  ──► scaffold + implement
```

## Workflow 5: Capstone / LOI Writing

```
Idea
    │
    ▼
Read knowledge/cohort/capstone-loi-and-architecture.md LOI guidance + knowledge/ecosystem/web-resources.md market context
    │
    ▼
Draft value prop, competitors, target market, use cases
    │
    ▼
Red-team with /solana-architect or /solana-security
    │
    ▼
Refine and submit
```

## Workflow 6: Architecture Diagram (capstone deliverable)

```
Approved LOI / use cases
    │
    ▼
Read knowledge/process/architecture-diagramming.md
    │
    ▼
Write numbered, atomic "The protocol shall …" requirements (one action each)  ──► NOT the diagram yet
    │
    ▼
Enumerate actors (direct, beneficiary, admin, third-party, stakeholder) + signer status
    │
    ▼
/solana-architect  ──► spec: account map, PDA seeds, instruction set, state machine, CPI plan,
                       traceability table (requirement ↔ element both ways), MVP scope cut
    │
    ▼
Self-check (atomicity, explicit signers, one instruction handler per state change)
    │
    ▼
AI red-team ──► treat output as findings to evaluate; each member logs override decisions
    │
    ▼
Draw the diagram (consistent arrows, distinct boundaries, consistent labels)
```

## How to run a workflow

You can either:

1. **Use the entry skill** `/turbin3` (harness-agnostic) or `/solana-guide` and describe your goal. It will route you to the right first skill.
2. **Run skills sequentially** by name, e.g., `/solana-architect`, then `/solana-build`, then `/solana-test`.
3. **Spawn subagents** with the agent profiles in `agents/` for parallel work (e.g., security review while tests run).

## Notes

- Workflows are descriptive, not enforced by the CLI. Use them as checklists.
- Always capture devnet/mainnet transaction signatures and Explorer links as evidence.
- After each loop, update the project README with program ID, deployed cluster, and key transaction links.
