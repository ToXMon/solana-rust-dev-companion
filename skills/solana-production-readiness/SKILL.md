---
name: solana-production-readiness
description: Review a Solana Rust, Anchor, Native, or Pinocchio project for production readiness using STRIDE-style security, release, infrastructure, monitoring, and Triton transaction-delivery guidance. Use before devnet/mainnet deployment, capstone submission, public demo, audit prep, or when asked for production readiness.
argument-hint: <project path or deployment target>
---

# Solana production readiness

Load these resources first:

- `resources/knowledge/production/stride-triton-production-readiness.md`
- `resources/checklists/solana-security.md`
- `resources/checklists/devnet-mainnet-readiness.md`

## Procedure

1. Identify the target: homework, devnet demo, capstone, or mainnet candidate.
2. Inventory program authorities, upgrade authority, admin keys, treasury accounts, oracles, CPI targets, token programs, and off-chain services.
3. Review code for account validation, checked arithmetic, state transitions, signer checks, owner checks, and PDA validation.
4. Review release evidence: lockfiles, toolchain, reproducible-build notes, deploy command, program ID, and verification plan.
5. Review infrastructure: RPC provider, retry behavior, priority fees, rate limits, monitoring, and incident response.
6. Return only actionable gaps.

## Output

Return:

1. Production readiness score by category.
2. Critical blockers.
3. Mainnet/devnet risks.
4. Minimal fixes before deployment.
5. Monitoring and incident-response additions.
6. Evidence the project should capture before publishing.
