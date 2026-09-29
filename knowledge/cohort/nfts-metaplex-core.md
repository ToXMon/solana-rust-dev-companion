# NFTs & Metaplex Core — cohort notes

NFT standards (Token Metadata vs compressed vs Core) and the full Metaplex Core NFT-staking workshop (Sep 21/23). Read this before building anything NFT-related. Related: `anchor-practical.md`, `testing-and-debugging.md`, `quotes.md`.

**Transcription caveat:** `2026_09_21` (01:31:38) is mostly narrated but ASR occasionally garbles terms ("feast period" for freeze period, "soul" for SOL); intended terms are bracketed. `2026_09_23` is a breakout/pair-programming workshop.

## NFT standards overview (Aug 28 + Sep 21)

- **NFT workflows:** upload image → upload metadata JSON → mint. Standalone mints do not require a collection. (`2026_08_28`, ~00:35:18–00:40:00; ~01:10:30–01:11:30)
- **Programmable NFTs:** frozen by default; rule-set program invoked via CPI from metadata program. (`2026_08_28`, ~00:16:19–00:18:21)
- **Compressed NFTs:** lower on-chain storage via Merkle trees, but worse indexing/dev UX. (`2026_08_28`, ~00:18:21–00:20:23)
- **Core NFTs:** one asset = one account, plugin-based extensions, simplified collections. (`2026_08_28`, ~00:20:30–00:22:08)
- **Core is "NFT-first"; Token Metadata is "NFT as an afterthought on top of SPL tokens."** Token Metadata needs a mint account, an associated token account, a metadata account, and often a master-edition account; Core collapses this into a single asset account. (`2026_09_21`)
- **Collections are first-class accounts.** In Token Metadata a collection is just another NFT linked via metadata; in Core a collection is a separate account type, and collection-level plugins (e.g., royalties) propagate to every asset unless an asset-level plugin overrides them. (`2026_09_21`)
- **Plugins are lifecycle hooks.** Core checks every attached plugin on `create`, `transfer`, `burn`, or `update`; each plugin can approve, reject, or mutate state. (`2026_09_21`, ~00:19:11–00:19:35)
- **Three plugin trust classes:**
  - *Owner-managed* — e.g., freeze delegate, transfer/burn delegate.
  - *Authority-managed* — e.g., royalties, on-chain attributes (controlled by update authority).
  - *Permanent* — can only be added at creation and never removed; used for soulbound tokens and non-burnable assets. (`2026_09_21`, ~00:20:36–00:21:12)
- **Staking = "lock asset, get rewarded over time."** The specific reward curve is business logic; the protocol-level action is just freezing the asset and recording a timestamp. (`2026_09_21`)
- **"Just because I say this is an admin doesn't mean anything."** Naming an account `admin` is not a security policy; production code should maintain a whitelist/multisig/timelock for privileged instructions. (`2026_09_21`, ~00:43:48–00:44:37)
- **Clock syscall vs. Clock sysvar account.** Calling `sol_get_unix_timestamp` has a base cost of ~100 CUs; reading the `Clock` sysvar as an account can avoid that syscall overhead. (`2026_09_21`, ~00:53:50–00:54:57)
- **Transaction size and compute are the real plugin limits.** The number of plugins you can attach in one instruction is bounded by account/data size and CUs, not by Core itself; the move from 1 KB to 4 KB transactions materially helps. (`2026_09_21`, ~01:19:47–01:20:33)
- **Attributes plugin doubles as program state.** Staking status (`stake`, `staked_at`, `last_claimed_at`) lives on the asset's Attributes plugin rather than a separate PDA — indexable by API providers for free. (`2026_09_23`, ~00:54–01:15)

## Practical tips (Sep 21 + Sep 23)

- **Use Core CPI builders.** `CreateCollectionV2CpiBuilder`, `CreateV2CpiBuilder`, `AddPluginV1CpiBuilder`, `UpdatePluginV1CpiBuilder`. Chain configuration methods (accounts, plugin type, authority, system program) and finish with `.invoke_signed()`. (`2026_09_21`, ~00:35:44–00:36:05)
- **PDA update authority for collections/assets.** Derive an update-authority PDA from a seed like `"update_authority"` + `collection.key()`. Pass it as an `UncheckedAccount` signer and provide seeds + bump in `invoke_signed`; it does not need to be created because it holds no data — and the constraint still verifies it derives from the expected seeds. (`2026_09_21`, ~00:36:40–00:39:54; `2026_09_23`, ~00:38:26–00:39:00)
- **Make config and reward mint unique per collection.** The config PDA uses the collection key as a seed; the reward mint PDA uses the config as a seed. (`2026_09_21`, ~00:33:05–00:33:31)
- **Represent reward rate in basis points.** `rewards_bps` (10,000 = 100%); `reward = staked_days * rewards_bps * 10^decimals / 10_000`, cast through wider ints to avoid overflow; mint via `mint_to` with the config PDA as mint authority. (`2026_09_21`, ~00:34:13–00:35:14; `2026_09_23`, ~01:15:34–01:19:12)
- **Attribute plugin as staking state.** On stake, fetch the existing `Attributes` plugin; if it already contains `stake = true`, fail. Otherwise preserve all unrelated attributes, set `stake = true`, and store `staked_at` as a string timestamp. (`2026_09_21`, ~00:50:51–00:53:43)
- **Add-vs-update plugin pattern.** If the Attributes plugin is present use `UpdatePluginV1CpiBuilder`, else `AddPluginV1CpiBuilder` with an explicit `init_authority`. Both signed by the update-authority PDA via `invoke_signed`. (`2026_09_23`, ~00:57:20–00:58:26)
- **Preserve unrelated attributes.** Copy every unrelated key/value into the new list and only modify the staking keys — otherwise you wipe user/app metadata. (`2026_09_23`, ~00:30:51–00:31:52)
- **Use `match` over if/else chains for multi-key attribute checks** (`stake`, `staked_at`, `last_claimed_at`), with a default arm. (`2026_09_23`, ~01:32:28–01:36:49)
- **Freeze via `FreezeDelegate` plugin.** Add freeze-delegate with `frozen = true` on stake; on unstake update it to `frozen = false` before transferring/minting rewards. (`2026_09_21`, ~00:56:53–00:57:19; ~01:04:33–01:04:53)
- **Validate the freeze period before unstaking.** `staked_days = (current_timestamp - staked_at) / seconds_per_day`; revert if below the configured freeze period. (`2026_09_21`, ~01:01:52–01:03:20)
- **`init_if_needed` for the user's reward ATA** — it may or may not exist yet. (`2026_09_21`, ~01:08:02–01:08:12)
- **Custom errors can live directly in constraints.** `has_one = ... @ ErrorCode::...` or `constraint = ... @ ErrorCode::...` so invalid accounts fail before the handler runs. (`2026_09_21`, ~00:47:27–00:47:57)
- **Claim-while-staked bookkeeping.** Add a `last_claimed_at` attribute. Reward = `now - max(staked_at, last_claimed_at)`; on claim, update `last_claimed_at` but keep `staked=true` and the freeze. On unstake, use `last_claimed_at` as the accrual base or you pay double rewards. (`2026_09_23`, ~01:25:40–01:28:32)
- **MPL Core + Anchor version compatibility.** The MPL Core Anchor crate currently targets ~Anchor 0.32.2 and also works with 0.31.x/0.30.x; you may need to downgrade from the latest Anchor, or write a custom CPI builder from the raw Core SDK for newer versions. (`2026_09_21`, ~01:08:55–01:09:40)

## Example programs

| Example | What it shows | Source segment |
|---------|---------------|----------------|
| **Core collection + asset mint** | One-account NFT creation vs. Token Metadata's multi-account model; `CreateCollectionV2CpiBuilder` + `CreateV2CpiBuilder`. | `2026_09_21`, ~00:35:44–00:43:06 |
| **NFT staking program** | Config (freeze period + reward bps) + reward mint; stake freezes asset and writes attributes; unstake thaws asset and mints rewards. | `2026_09_21`, ~00:31:03–01:08:21 |
| **Freeze-delegate plugin** | Lock an asset in the owner's own token account so it cannot be transferred while staked. | `2026_09_21`, ~00:56:53–00:57:19 |
| **Attributes plugin** | Store `stake`/`staked_at` key-value pairs on-chain, indexable by API providers. | `2026_09_21`, ~00:51:02–00:53:43 |
| **Soulbound / non-burnable assets** | Permanent freeze or permanent burn plugins added at creation; cannot be removed later. | `2026_09_21`, ~00:21:12–00:21:58 |
| **Oracle plugin** | Reject lifecycle actions based on external data (e.g., "no transfers outside business hours"). | `2026_09_21`, ~00:24:33–00:25:16 |
| **App-data plugin** | Third-party app gets an exclusive data partition on the asset (game stats, ticketing, loyalty points). | `2026_09_21`, ~00:25:22–00:26:12 |
| **Niche-data oracle** | If no price feed exists for your data (wave heights, sensor readings), you must operate the sensor yourself or partner with/incentivize the provider. | `2026_09_21`, ~01:28:13–01:30:10 |
| `claim_rewards` | Proportional reward accrual keyed off `last_claimed_at` vs `staked_at`; NFT stays frozen. | `2026_09_23`, ~01:24–01:37 |
| `burn_staked_nft` | Burn delegate plugin + one-time bonus reward; permanent destruction as an economic lever. | `2026_09_23`, ~00:14:23–00:14:59 |
| Collection staking stats | Collection-level Attributes plugin as an aggregate counter (`total_staked`). | `2026_09_23`, ~00:15:00–00:15:38 |
| Oracle plugin / time-gated transfers | External validation account gates lifecycle hooks — model for compliance windows (RWA transfer hours, market hours). | `2026_09_23`, ~00:15:57–00:17:15 |

## Assignments from these sessions

- **Due Sep 23 class:** complete the staking program through `unstake` (collection creation, asset mint, config init, stake, unstake). (`2026_09_21`, ~01:30:44–01:31:15)
- **Self-directed:** add a plugin before Wednesday — royalties, oracle validation, app-data partition, or permanent freeze. (`2026_09_21`, ~01:30:27–01:30:44)
- **Stretch:** `claim_rewards` while still staked (`2026_09_21`, ~01:05:30–01:06:26); APY/oracle reward curve (`2026_09_21`, ~01:06:51–01:07:21).
- **Sep 23 challenge tasks:** (a) `claim_rewards` while staked; (b) `burn_staked_nft` for a one-time bonus; (c) collection-level staking stats; plus oracle plugin for time-gated transfers. Week 4 assignment takes priority. (`2026_09_23`, ~00:13:45–00:17:19; ~01:37:09–01:37:16)

Related: `testing-and-debugging.md` (time-travel tests used here), `rwa-tokenized-fund.md` (oracle-gated transfers applied to RWA), `capstone-loi-and-architecture.md`.
