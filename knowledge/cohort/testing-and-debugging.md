# Testing & debugging — cohort notes

How the instructors taught testing and debugging: assertions, LiteSVM setup, instruction builders, time-travel testing, and common failure patterns. Read this before writing program tests. Related: `mistakes-and-prevention.md`.

## Core guidance

- **Always write tests after implementing each instruction.** Do not batch-implement multiple instructions and then test once. (`2026_08_31`, ~01:46:31–01:49:37)
- **Assertions are the heart of a test.** "Any transaction could randomly pass. You might not know exactly the result… assert expected data, account changes, balances, lamports, tokens." (`2026_09_04`, ~00:42:33–00:43:12)
- Use **instruction builders** and **setup functions** to reduce test redundancy; define constants in one place so changes stay consistent. (`2026_09_04`, ~00:37:50–00:42:28)
- When a transaction fails, first verify you are passing the correct account/mint/address from the prior step. Example: a metadata transaction failed because the wrong mint was reused; fix was to use the mint created earlier. (`2026_08_26`, ~01:16:32–01:17:12)
- For mint creation, reason from on-chain requirements: (1) determine account size, (2) calculate minimum rent-exempt balance, (3) create/allocate via System Program, (4) initialize via Token Program. (`2026_08_26`, ~00:38:12–00:44:07)
- Use `anchor expand` to inspect macro-generated code and understand what constraints are actually enforcing. (`2026_08_31`, ~00:40:30–00:42:42)
- For manual/lite SVM tests: instantiate the SVM, `add_program` with compiled `.so` bytes, create signers, airdrop lamports, derive PDAs/ATAs, build the instruction with accounts + data, wrap in a message with recent blockhash, sign, send, and assert. (`2026_09_04`, ~00:07:04–00:32:34)
- `anchor test` can be run with `--skip-build` after a build, or use `cargo test` directly with logging flags to see logs on success. (`2026_09_04`, ~00:32:01–00:32:34)
- When stack-size errors appear, the compiler reports how many bytes are over; box accounts incrementally until it compiles. (`2026_09_02`, ~00:10:15–00:10:44)

## Tips from later sessions

- **Test failure paths and use time travel.** Advance the validator clock (Surfpool's time-travel endpoint or LightSVM) so you can test both the "unstake too early" failure and the later success. (`2026_09_21`, ~01:11:07–01:18:23) — see `nfts-metaplex-core.md`.
- **For tests that print values, isolate test threads.** The instructor mentioned running `cargo test` with a thread-count flag so each test runs in its own thread and does not block others; this matters when tests depend on shared on-chain state or when you want clean, per-test output. (`2026_09_18`, ~00:09:28–00:09:48; wording was garbled — verify the exact flag in the repo.)
- **Negative tests matter.** "We want to test for both the cases… we want to test for the failure case as well." (`2026_09_21`, ~01:11:18–01:11:22)
- Token-2022 session tests demonstrated: declarative mint produces expected extensions, manual `create_mint_with_fee` yields a working transfer-fee mint, and transfer fee correctly applies to token accounts. (`2026_09_16`, ~00:16:10–00:17:13)
- **TypeScript client account helpers.** `.accounts()` auto-resolves derived accounts from the IDL, `.accountsPartial()` lets you pass a subset and derive the rest, and `.accountsStrict()` requires every account to be supplied explicitly. (`2026_09_21`, ~01:13:54–01:15:37; ASR was garbled — verify exact semantics in the IDL.)
- **Inspect raw account data to debug extensions/delegation.** Compare SPL Token vs Token-2022 mint layouts with `solana account` or explorers (`2026_09_14`, ~01:33:06–01:33:35); token account data exposes `delegate` and `delegated_amount` fields useful for debugging (`2026_09_18`, ~00:41:02–00:41:54).
