# STRIDE and Triton production readiness for Solana Rust

Use this module when a homework program starts becoming a portfolio project, devnet demo, capstone, or mainnet candidate.

## Core idea

A Solana program is not production-ready just because the instruction code passes tests.
Production readiness also includes governance, dependencies, release provenance, infrastructure, monitoring, incident response, logging, and transaction delivery.

Primary sources:

- STRIDE: https://stride.asymmetric.re/
- STRIDE framework: https://stride.asymmetric.re/framework
- STRIDE first findings: https://stride.asymmetric.re/first-findings
- Triton One: https://triton.one/
- Triton Solana docs: https://docs.triton.one/chains/solana

## STRIDE-derived release checklist

Score each item as `0 missing`, `1 ad hoc`, `2 documented`, or `3 proven`.

### Program security

- Internal review completed.
- External audit or focused review exists when value at risk justifies it.
- Vulnerability disclosure path exists.
- `solana-security-txt` metadata is planned for public programs.
- Fuzzing, property tests, static analysis, or formal methods are used where practical.
- Defense-in-depth exists for dangerous operations.

### Governance and authority

- Upgrade authority is identified.
- Admin roles are documented.
- Timelock or multisig expectations are explicit.
- Risk parameters and who can change them are listed.
- Emergency powers have scope limits.

### External dependencies

- Oracle, CPI, Token-2022 hook, bridge, keeper, and off-chain dependencies are listed.
- Each dependency has trust assumptions and failure modes.
- Staleness/liveness checks exist where external data affects funds.
- Blast radius is capped when a dependency fails or is compromised.

### Infrastructure

- RPC provider strategy is documented.
- Rate limits and retry behavior are known.
- Fallbacks are planned for production apps.
- DNS, web app, API, and indexing services have owners.
- Secrets never live in source control or shell history.

### Supply chain and release

- Lockfiles are committed.
- Dependencies are pinned or reviewed.
- Branch protection or review policy exists.
- Releases are reproducible or at least documented.
- Program binaries can be verified against source with `solana-verify` or equivalent when appropriate.
- CI/CD secrets and deploy keys are scoped.

### Operations, monitoring, and incident response

- Large transfers are monitored.
- Authority changes are monitored.
- Upgrade authority changes are monitored.
- Abnormal transaction patterns have alerts.
- Circuit breaker or pause criteria are documented.
- Incident runbooks name the first ten minutes, escalation path, evidence to preserve, recovery steps, and postmortem timeline.

## Triton-derived transaction delivery checklist

Use this for clients, bots, keepers, relayers, and production frontends.

- Manage retries in the client instead of relying blindly on default RPC retries.
- Use `maxRetries: 0` when the client owns retry behavior.
- Simulate separately when simulation is needed.
- Use `skipPreflight: true` only when the client already simulated or accepts that tradeoff.
- Use confirmed or finalized blockhashes intentionally.
- Set tight compute unit budgets instead of broad guesses.
- Estimate priority fees from relevant writable account markets, not only network-wide minimums.
- Implement rate-limit backoff.
- Keep submission, confirmation, and retry telemetry.

## When to teach this in homework

For early homework, ask only three questions:

1. What authority can move or upgrade funds?
2. What must be monitored if this were deployed?
3. What proof would convince a stranger this build matches the source?

For capstone or portfolio work, require the full readiness checklist.
