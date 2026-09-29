# Solana Security Audit Patterns

This module makes the repository's extended audit guidance discoverable from the main knowledge base. It complements—not replaces—the cohort checklist in `AGENTS.md`, `/solana-security`, and the implementation recipes in `code-patterns.md`.

## Source and provenance

Imported from [Frankcastleauditor/safe-solana-builder](https://github.com/Frankcastleauditor/safe-solana-builder) at commit `9e94436dcf4b5dfd6d837eb70cf88b9048e72e5d`.

The repository-native copy lives at:

- `skills/safe-solana-builder/SKILL.md`
- `skills/safe-solana-builder/references/shared-base.md`
- `skills/safe-solana-builder/references/anchor.md`
- `skills/safe-solana-builder/references/native-rust.md`
- `skills/safe-solana-builder/references/pinocchio.md`
- `skills/safe-solana-builder/references/litesvm.md`
- `skills/safe-solana-builder/examples/`

## How to use it

### Before architecture

1. Classify the program as low, medium, or critical risk.
2. Identify every signer, authority, PDA, vault, token mint, CPI target, oracle, admin role, and terminal state.
3. Write the state-transition matrix and financial invariants before implementing handlers.
4. For vaults and recurring-payment systems, document spendable balance, reserved liabilities, withdrawal paths, expiry behavior, and emergency recovery.

### Before implementation

1. Invoke `/safe-solana-builder` for new Solana program builds.
2. Read `shared-base.md` plus the selected framework reference.
3. Read `litesvm.md` when using LiteSVM.
4. Apply the repository's existing `code-patterns.md` recipes for PDAs, CPIs, `TokenInterface`, `transfer_checked`, and negative tests.

### Before review or deployment

1. Invoke `/solana-security` for an adversarial review.
2. Verify every invariant with positive and negative tests.
3. Review Token-2022 extensions on accepted mints.
4. Record admin, upgrade-authority, oracle, and external-protocol trust assumptions.
5. Treat audits as risk reduction, not proof of safety; deploy to devnet before any mainnet consideration.

## Extended checklist

### Accounts, signers, and PDAs

- Verify account ownership, discriminator/type, signer status, mutability, and relationships.
- Use purpose-specific, user-scoped PDA seeds with canonical stored bumps.
- Prevent seed collisions, namespace capture, reinitialization, and zombie-account reuse.
- Never elevate a non-signer through CPI or accept arbitrary CPI program IDs.

### Arithmetic and state machines

- Use checked arithmetic for all financial values and counters.
- Define valid transitions explicitly; terminal states must be absorbing unless a documented recovery path exists.
- Update coupled fields and counters atomically.
- Use one time unit consistently and test exact boundaries around start, expiry, and cooldown timestamps.

### Vaults, solvency, and withdrawals

- Avoid a single global user-funds vault when user-specific vaults can reduce blast radius.
- Every PDA-controlled vault needs an authorized withdrawal or recovery path.
- Separate principal, fees, rewards, reserves, and pending liabilities in both state and reasoning.
- Verify backing and solvency after state-changing operations.
- Rewards must come from a funded reward source or realized yield, never other users' principal.

### Tokens and Token-2022

- Use `transfer_checked` with the correct mint and decimals.
- Reload token accounts after CPI before using changed balances.
- Use balance-delta accounting for transfer-fee mints.
- Reject or explicitly support dangerous extensions: `PermanentDelegate`, external freeze authority, `TransferHook`, confidential transfers, and fee extensions.
- Validate token program IDs and all required transfer-hook accounts.

### Fees, slippage, and precision

- Name gross, fee, and net amounts unambiguously.
- Apply fees and slippage checks consistently across every execution path.
- Avoid division-before-multiplication and unsafe narrowing casts.
- Test dust values, maximum values, rounding boundaries, and repeated partial operations.

### Rewards, shares, and pools

- Settle accrued rewards before shrinking a position.
- Update reward debt on every payout path.
- Do not apply changed rates retroactively.
- Protect share systems against first-depositor/inflation attacks and dead exchange rates.

### Administration and configuration

- Validate every config write, not only initialization.
- Enforce cross-field invariants before atomically committing updates.
- Use two-step admin rotation for critical authority changes.
- Recommend multisig and timelock controls for production-critical operations.
- Ensure fee recipients and treasury token accounts remain sweepable by the intended authority.

### Runtime and panic safety

- Keep BPF stack limits in mind; box only the large accounts that need it.
- Do not use `unwrap()` or `expect()` on user-controlled paths.
- Bound all user-supplied strings and metadata; allowlist URI schemes where relevant.

## Relationship to the working repo's guidance

When guidance conflicts, use this priority:

1. Current canonical Solana/Anchor/Token documentation and the repository's pinned dependencies.
2. `AGENTS.md` security and build conventions.
3. Project-specific architecture and invariants.
4. `skills/safe-solana-builder/references/` and this summary.
5. Older cohort examples, which may use different Anchor or SDK versions.

Always verify live APIs and dependency versions before implementation.