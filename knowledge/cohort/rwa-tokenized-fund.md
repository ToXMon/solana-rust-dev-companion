# RWA: tokenized money-market fund — cohort notes

The Sep 25 final live class: a complete Anchor codebase walkthrough of a tokenized money-market fund (TMMF) — the cohort's RWA capstone reference. Read this before building tokenization/RWA capstones or any program with roles, NAV math, or KYC gating. Related: `token-2022-and-extensions.md`, `anchor-practical.md`.

## Mental models

- **NAV = (total assets − liabilities) / shares outstanding.** The program only cares about the published NAV; how the fund computes it off-chain (T-bill interest, fees) is the issuer's regulatory problem. (`2026_09_25`, ~00:14:50–00:16:50; ~00:35:00–00:36:10)
- **NAV only moves via interest accrual.** Subscriptions and redemptions move assets and shares together — NAV is unchanged. It changes only when the NAV manager posts accrued interest. (`2026_09_25`, ~00:31:46–00:33:04)
- **Dust is a feature.** Integer division floors share/USDC math; the remainder silently accrues to remaining holders ("remove 1% of the pie, 99 people share 100 pieces"). Always decide which direction rounding favors. (`2026_09_25`, ~00:20:46–00:25:29)
- **Token ≠ ownership.** "Even though you hold the token, that does not prove you are the owner of the underlying portfolio share" — the issuer's books are the legal record; the token is a representation. (`2026_09_25`, ~00:26:16–00:27:15)
- **Tokenization benefits:** 24/7 settlement, instant redemption, P2P transfers between whitelisted wallets, collateral composability, programmable administration, stablecoin backing, treasury management. (`2026_09_25`, ~00:10:42–00:14:45)
- **Bank-run guard:** custodian withdrawals must leave a liquidity floor in the vault (e.g., keep ~30% of assets) so redemptions can always be served. (`2026_09_25`, ~01:21:47–01:23:42)

## Practical tips

- **Role-based access via PDA accounts.** `Role` account seeds = `role_type.seed() + wallet`; enum variants (`FundAdmin`, `NavManager`, `Pauser`) each map to a distinct seed constant via a `match` helper. Pauser can pause but not resume — implemented with a bitwise `|=` style operation so the resume bit needs higher authority. (`2026_09_25`, ~00:42:50–00:46:40; ~01:19:00–01:20:20)
- **Gate admin instructions to the upgrade authority.** Pass the program account + `ProgramData` account, check `program_data.upgrade_authority == authority.key()`, and **re-derive the program-data PDA from the program id** (`require_own_program_data`-style) — otherwise an attacker can pass a program-data account from a different program they deployed. Then rotate the upgrade authority to a multisig post-deploy. (`2026_09_25`, ~00:56:19–01:04:30)
- **Validate external mints through stored state, not hardcoding.** `initialize_fund` records `usdc_mint` on the `Fund` account (admin-trusted); user-facing instructions verify it via `has_one`/address check. Keeps the program generic across stablecoins/Token-2022 variants. (`2026_09_25`, ~01:08:00–01:10:11)
- **Skip deserialization when you only need the address.** The USDC mint is an `UncheckedAccount` — Anchor won't deserialize it, saving CUs; decimals are stored on `Fund` and re-verified for consistency. (`2026_09_25`, ~01:14:40–01:16:14)
- **`u64`→`u128` `mul_div` for share math.** `a * b / c` computed in u128 then coerced back to u64; fail the transaction if it doesn't fit. (`2026_09_25`, ~00:52:50–00:54:30)
- **Token-2022 extensions used:** `ScaledUiAmount` (wallets display `shares × NAV multiplier`; takes an `f64`, so cast at the boundary — display-only, precision non-critical) and `DefaultAccountState` initialized frozen so only program-thawed (KYC'd) token accounts can hold shares (`approve_investor` thaws, `revoke_investor` re-freezes). (`2026_09_25`, ~00:30:09–00:31:46; ~01:10:49–01:12:22)
- **NAV safety checks:** reject non-positive NAV, cap per-update deviation in bps (e.g., 2% = 200 bps), store `last_nav_update` and reject stale reads on subscribe/redeem. Optional oracle check guards against stablecoin depeg (e.g., USDC ≠ $1 → halt). (`2026_09_25`, ~00:27:46–00:30:09; ~00:50:50–00:52:36)
- **`emit_cpi!` vs `emit!`.** `emit!` writes to program logs and can be truncated; `emit_cpi!` self-CPIs the event into inner-instruction data (indexer-friendly, not truncated) at the cost of one CPI depth slot + two extra accounts. (`2026_09_25`, ~00:54:38–01:18:10)
- **Redemption windows:** `redemption_cap`, `window`, `window_start` fields rate-limit redemptions per period for protocol stability. (`2026_09_25`, ~00:52:20–00:52:36)
- **Helper placement rule of thumb:** a check used in ≥2 instructions goes in `utils.rs` (e.g., the upgrade-authority check); account methods like `fund.usdc_to_shares()`, `fund.require_fresh_nav()`, `fund.check_usdc_peg()` live in `impl` blocks on the state structs. (`2026_09_25`, ~01:25:26–01:27:01)

## Example programs

| Example | What it shows | Source segment |
|---------|---------------|----------------|
| Tokenized MMF program | Full RWA pattern: subscribe (USDC→shares at NAV), redeem (burn→USDC), `publish_nav`, role grant/revoke, pause flags, custodian withdrawal with liquidity floor, KYC freeze gating. | `2026_09_25`, ~00:18–01:23 |
| `grant_role` upgrade-authority gate | Only the deployer/upgrade authority can grant roles; program-data PDA re-derivation closes the cross-program spoof. | `2026_09_25`, ~00:56–01:05 |
| Oracle peg guard | Optional oracle feed halts dealing if the stablecoin depegs. | `2026_09_25`, ~00:50:50–00:52:20 |

## Logistics notes

- Cohort live classes ended here; capstone period begins. Enrichment week follows; an extra Monday class covers architecture diagrams. (`2026_09_25`, ~01:30:00–01:33:21)
- Capstone is a group project — teammate assignments are out; report unreachable/inactive teammates to staff. Two weeks of capstone work remain. (`2026_09_25`, ~01:33:23–01:35:18)
- The TMMF repo was promised to the class via Classroom/Discord — a complete RWA reference implementation worth studying before building a capstone in the RWA/Tokenization categories.
