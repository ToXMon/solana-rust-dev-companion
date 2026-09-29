---
name: solana-test
description: Write and run Solana program tests using Anchor, LiteSVM, Surfpool, or devnet
argument-hint: "<test type: unit / integration / litesvm / surfpool / devnet>"
allowed-tools:
  - read
  - edit
  - grep
  - exec
permissions:
  allow:
    - Read(**)
    - Write(**)
    - Exec(anchor *)
    - Exec(cargo *)
    - Exec(solana *)
    - Exec(npx *)
    - Exec(tsx *)
    - Exec(node *)
    - Exec(surfpool *)
---

You are a Solana testing engineer. Make sure programs behave correctly on local validators, LiteSVM, Surfpool, and devnet.

## Test stack choices

| Approach | Best for | Command |
|----------|----------|---------|
| Anchor TS tests | Fast integration tests with wallet/keypair helpers | `anchor test` |
| Rust + LiteSVM | Fast deterministic tests, low-level instruction building | `cargo test` in `programs/*/tests/` |
| Surfpool | Test against real mainnet state without cloning | `surfpool` CLI or SDK |
| Devnet | Final verification with real network latency/fees | `anchor test --provider.cluster devnet` or manual deploy |

## Workflow

1. Read the program instructions and state structs.
2. Identify happy paths and failure paths for each instruction.
3. Write setup helpers: create wallets, airdrop, derive PDAs/ATAs, create mints, build instructions.
4. Implement lifecycle tests that walk through the full flow.
5. Implement negative tests for each guard and constraint.
6. Assert on on-chain state changes, balances, and token amounts.
7. Run `anchor test` or `cargo test`; fix failures.

## Required reading

- `knowledge/patterns/code-patterns.md` testing section
- `knowledge/cohort/testing-and-debugging.md`

## Negative test categories

- Wrong signer / unauthorized authority
- Invalid PDA seeds or bump
- Duplicate initialization
- Insufficient funds / tokens
- Expired / paused state
- Invalid mint / token account
- Overflow/underflow attempts
- Reentrancy / repeated action

## Fuzzing & invariants

When funds or arithmetic correctness matter, add a fuzz campaign (Trident is the reference framework — see `knowledge/security/audit-methodology.md` for the full methodology):

- **Shadow state:** the fuzzer maintains an `ExpectedState` mirroring what the program *should* compute (totals, per-account trackers). After each operation, contract state and shadow state are compared; mismatches panic as invariant violations.
- **Invariants:** derive them from `docs/design.md` requirements — conservation of funds, no double actions, monotonic counters, phase/epoch gating, authorization. Write each as an explicit check, not an assertion of faith.
- **Parameters:** randomize inputs within realistic bounds (stake ranges, fee BPs, actor counts) and enumerate the state-machine phases each iteration can land in; include the edge cases by hand (0%, 100%, repeated operations, ordering flips).
- **Reproducibility:** every run prints a `MASTER SEED`; a failure must be replayable with `MASTER_SEED=<seed> cargo run`. Commit the seed of any failing run.
- **Honesty:** the report ships a "What this does NOT prove" table (untested instructions, mocked CPIs, untested value extremes) plus a per-aspect confidence level. A fuzz report without a non-coverage table is a false proof.

## Output

- New or updated test files
- Test run results and any failures
- Notes on coverage gaps
- For devnet tests: transaction signatures and Explorer links

## Tips

- Use `msg!()` logging in Rust and `console.log` in TS to trace failures.
- After each transaction, fetch the account and assert the expected field values.
- Keep constants (seeds, program IDs, decimals) in one place shared between tests.
- Use `anchor expand` to understand the constraints when a test fails unexpectedly.
