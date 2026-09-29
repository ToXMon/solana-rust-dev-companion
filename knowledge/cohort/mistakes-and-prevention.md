# Beginner mistakes & prevention — cohort notes

The mistakes instructors saw repeatedly, plus prevention strategies. Read this as a pre-flight checklist before submitting a program or assignment. Related: `testing-and-debugging.md`, `anchor-practical.md`.

## Mistakes table

| Mistake | Why it fails / risk | Prevention |
|---------|---------------------|------------|
| Reusing the wrong mint/account from a previous step | Custom program errors, failed metadata/token transactions | Name variables clearly, assert the expected mint/account before sending |
| Forgetting `mut` on accounts that change lamports/data | Transaction fails at runtime | Ask: "Is this account's data or balance changing?" |
| Using `transfer` instead of `transfer_checked` | Decimal/mint mismatches, especially with Token 2022 | Default to `transfer_checked` |
| Storing the bump as a field but not using it in signer seeds | PDA signature verification fails | Store bump and pass seeds consistently in `new_with_signer` |
| Assuming `init` will silently re-initialize | `init` panics if account exists; use `init_if_needed` only when intended | Choose constraint based on business logic |
| Treating account type and constraints as interchangeable | Under-validation leads to security bugs | Use both; account type handles deserialization/owner, constraints enforce business rules |
| Writing all test setup inline for every instruction | Redundant, brittle tests | Extract setup helpers, instruction builders, and constants |
| Joining class from a mobile phone | Cannot follow code/screenshare, disengagement | Always join from a laptop |
| Pasting the LOI assignment into AI before thinking | Shallow, non-iterative strategy | Do human-first market/strategy thinking, then use AI for red-teaming |
| Copy-pasting reference repo instead of building from scratch | Surface-level understanding | Use reference repo as guide, initialize fresh repo, document why each constraint exists |

## Mistakes called out in later sessions

- **Naming an account `admin` is not a security policy.** "Just because I say this is an admin doesn't mean anything." Production code should maintain a whitelist/multisig/timelock for privileged instructions. (`2026_09_21`, ~00:43:48–00:44:37)
- **Accepting arbitrary Token-2022 mints without checking extensions.** A mint with a permanent-delegate extension is "more of a problem than a feature" for DeFi — reject it. (`2026_09_18`, ~00:46:18–00:47:18) — see `token-2022-and-extensions.md`.
- **Trusting a program-data account without re-deriving it.** An attacker can pass a program-data PDA from a different program they deployed; re-derive it from the program id. (`2026_09_25`, ~00:56:19–01:04:30) — see `rwa-tokenized-fund.md`.
- **Wiping unrelated attributes when updating a Core Attributes plugin.** Iterate the fetched list, copy unrelated key/values, modify only your keys. (`2026_09_23`, ~00:30:51–00:31:52) — see `nfts-metaplex-core.md`.
- **Double-paying rewards on unstake.** Use `last_claimed_at` as the accrual base after a `claim_rewards` run. (`2026_09_23`, ~01:25:40–01:28:32)
- **Assuming the newest standard/toolchain is usable.** MPL Core + Anchor version mismatch (`2026_09_21`, ~01:08:55–01:09:40); Anchor's modified SBF toolchain lags stable Rust (`2026_09_04`, ~00:35:16–00:36:43); newer token standards may lack wallet/explorer support (`2026_08_28`, ~01:04:43–01:05:24).
- **Wrong-mint bugs when chaining mint + metadata instructions** are common — Metaplex metadata is a PDA derived from the mint. (`2026_08_26`, ~01:16:32–01:18:28)
