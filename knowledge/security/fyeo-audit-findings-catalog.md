# FYEO audit findings catalog — Solana programs

A class-organized catalog of every finding across **23 FYEO public Solana security reviews** (~170 findings). Source: [`fyeo-io/public-audit-reports`](https://github.com/fyeo-io/public-audit-reports) (`Code Audit Reports/`). Synthesized from the public reports; the PDFs themselves are not redistributed — each entry cites its report and finding ID so you can pull the original.

**Excluded:** `Drift Token - Security Code Review of DriftToken v1.0` — the in-scope code is a Solidity/EVM contract (floating pragma, Uniswap dependency), not a Solana program; it was mislabeled in the corpus matrix. `Solana Foundation - Gov contract fuzzing v1.0` contains no FYEO- findings (it is a Trident fuzzing campaign report — see `audit-methodology.md`). Zero-finding ongoing diff reviews (e.g. `Banger Ongoing 2024.12.20`) contribute no entries but appear in the report index.

Read this file as the **empirical abundance distribution** for Solana programs (see `vulnerability-abundance.md`): where programs actually fail, in what proportions. For how to run a review with it: `AGENTS.md`.

## Section A — Abundance distribution

### Findings by class (n = 170)

| Class | Count | Share |
|-------|------:|------:|
| `account-validation` | 25 | 15% |
| `initialization` | 9 | 5% |
| `access-control` | 13 | 8% |
| `input-bounds` | 19 | 11% |
| `arithmetic` | 4 | 2% |
| `accounting-state` | 13 | 8% |
| `signature-introspection` | 4 | 2% |
| `dos-resource` | 15 | 9% |
| `cpi-external` | 3 | 2% |
| `token-program` | 4 | 2% |
| `randomness-ordering` | 3 | 2% |
| `spec-mismatch` | 7 | 4% |
| `code-quality` | 34 | 20% |
| `testing` | 5 | 3% |
| `observability` | 5 | 3% |
| `upgradability-ops` | 7 | 4% |

### Severity histogram

| Severity | Count |
|----------|------:|
| Critical | 1 |
| High | 16 |
| Medium | 27 |
| Low | 37 |
| Informational | 89 |
| **Total** | **170** |


**Reading the distribution:** ~1 in 5 findings is a missing account check; initialization, access-control, and input-bounds together are another third. High+Critical concentrate in signature-introspection, access-control, cpi-external, and accounting — i.e., the classes that are easiest to eliminate by construction (declarative account types, pinned program IDs, domain-separated signatures, checked math) are also where the catastrophic bugs live.

## Section B — Findings by class


### account-validation — missing owner/mint/delegate/signer/PDA/program checks

#### FYEO-AMULET-ID-01 — Treasury and pool token account mints not verified
- **Severity:** High · **Status:** Remediated · **Protocol:** Amulet Protocol (vault / DeFi (risk-hedge vaults)) · **Report:** `2023/Amulet/MTR_Labs_Pte_Ltd_Secure_Code_Review_of_Amulet_Protocol_v1_0 (1).pdf`
- **Mechanism:** Neither vault_treasury_token_account nor vault_pool_token_account has its mint checked at creation.
- **Prevention rule:** Verify `token::mint` on every stored/custody token account.
- **Anchor/Rust tell:** Token account accepted without mint constraint.

#### FYEO-AMULET-ID-02 — LP mints not checked for zero supply
- **Severity:** High · **Status:** Remediated · **Protocol:** Amulet Protocol (vault / DeFi (risk-hedge vaults)) · **Report:** `2023/Amulet/MTR_Labs_Pte_Ltd_Secure_Code_Review_of_Amulet_Protocol_v1_0 (1).pdf`
- **Mechanism:** CreateVaultMetadataState accepts LP mints without `supply == 0`, so a used/pre-minted mint can be registered.
- **Prevention rule:** For fresh LP/share mints, require supply==0 and authority held by the program PDA.
- **Anchor/Rust tell:** Mint accepted without supply/authority check.

#### FYEO-BANGER-02 — Treasury account can be identical to others
- **Severity:** High · **Status:** Remediated · **Protocol:** Banger (launchpad / raffle sale) · **Report:** `2024/Banger/Banger - Security Code Review of Banger v1.0.pdf`
- **Mechanism:** The treasury account is not constrained to any particular address, so a caller can redirect proceeds to any account.
- **Prevention rule:** Pin every payout destination to a stored config address or well-known constant.
- **Anchor/Rust tell:** `treasury: UncheckedAccount` with no `address`/`has_one` check.

#### FYEO-GOV-02 — Unvalidated vote account enables duplicate supports
- **Severity:** High · **Status:** Remediated · **Protocol:** Solana Staker Vote Override (gov contract) (governance) · **Report:** `2025/Solana Foundation/Turbine - Security Code Review of Solana Staker Vote Override v1.0.pdf`
- **Mechanism:** support_proposal accepts any spl_vote_account matching owner/size but never checks the merkle leaf's vote_account equals it — one validator can double-count stake via multiple Support PDAs.
- **Prevention rule:** Bind supplied accounts to the leaf/proof data they claim to represent.
- **Anchor/Rust tell:** Proof account and vote account checked independently, never against each other.

#### FYEO-1INTRO-01 — Insecure auction token account at pool creation
- **Severity:** Medium · **Status:** Remediated · **Protocol:** 1Intro launchpad (launchpad / AMM) · **Report:** `2024/1DEX/Security Code Review 1INTRO v1.0_public.pdf`
- **Mechanism:** Pool creator can submit a token account carrying delegate or close_authority, enabling third-party token movement.
- **Prevention rule:** Reject custody accounts with delegate/close_authority set.
- **Anchor/Rust tell:** No delegate/close_authority check on pool token accounts.

#### FYEO-1INTRO-02 — Unbounded fee accounts, no delegate check
- **Severity:** Medium · **Status:** Remediated · **Protocol:** 1Intro launchpad (launchpad / AMM) · **Report:** `2024/1DEX/Security Code Review 1INTRO v1.0_public.pdf`
- **Mechanism:** Any number of fee accounts can be supplied with no delegate verification — extra rent overhead and complexity.
- **Prevention rule:** Bound the count and validate each fee account's delegate/authority.
- **Anchor/Rust tell:** Fee accounts iterated with no count cap or delegate check.

#### FYEO-4CAST-01 — Admin can close any user state for rent
- **Severity:** Medium · **Status:** Remediated · **Protocol:** 4CAST (points / rewards program) · **Report:** `2024/4Cast/4CAST - Security Code Review 4Cast Programs v1.0.pdf`
- **Mechanism:** The CloseUserState instruction does not verify that the user_state being closed belongs to the reward_state the admin controls, so an admin can drain rent from unrelated users' accounts.
- **Prevention rule:** When an instruction closes an account on behalf of a user, verify the account's relationship to the authority's scope (seeds/parent account), not just that the caller is an admin.
- **Anchor/Rust tell:** `close`/`admin_close` constraint missing a `has_one` or seeds link to the scoped parent.

#### FYEO-AX-SOL-03 — ITS: missing account checks for InterchainTransfer
- **Severity:** Medium · **Status:** Remediated · **Protocol:** Axelar Solana integration (bridge / interoperability) · **Report:** `2025/Axelar/Axelar Foundation - Security Code Review of Axelar - Solana Integration v1.0.pdf`
- **Mechanism:** The InterchainTransfer handler lacks its account validations (added in a later version).
- **Prevention rule:** Enumerate required account checks per instruction in the spec and verify each exists in code review.
- **Anchor/Rust tell:** Handler takes accounts from raw `&[AccountInfo]` slice without owner/key checks.

#### FYEO-GOV-04 — create_proposal does not bind vote account to merkle data
- **Severity:** Medium · **Status:** Remediated · **Protocol:** Solana Staker Vote Override (gov contract) (governance) · **Report:** `2025/Solana Foundation/Turbine - Security Code Review of Solana Staker Vote Override v1.0.pdf`
- **Mechanism:** Proposals can be created referencing vote accounts not tied to the merkle snapshot.
- **Prevention rule:** Cross-check every referenced account against the snapshot/leaf it claims to map to.
- **Anchor/Rust tell:** Vote account accepted without merkle-leaf equality check.

#### FYEO-1INTRO-02 — Pool token accounts not checked empty
- **Severity:** Low · **Status:** Remediated · **Protocol:** 1DEX swap platform (AMM) · **Report:** `2024/1DEX/SecurityCodeReview1DEX.pdf`
- **Mechanism:** Pool token accounts may be created holding a nonzero balance.
- **Prevention rule:** Require custody/pool accounts to start at amount==0.
- **Anchor/Rust tell:** Pool init accepts token accounts without balance check.

#### FYEO-AMULET-ID-06 — LP mints can be the same account
- **Severity:** Low · **Status:** Remediated · **Protocol:** Amulet Protocol (vault / DeFi (risk-hedge vaults)) · **Report:** `2023/Amulet/MTR_Labs_Pte_Ltd_Secure_Code_Review_of_Amulet_Protocol_v1_0 (1).pdf`
- **Mechanism:** vault_risk_pool_lp_mint and vault_hedge_pool_lp_mint may be the identical account.
- **Prevention rule:** When a design needs N distinct accounts, assert pairwise inequality (`key() !=`).
- **Anchor/Rust tell:** Two same-type accounts accepted with no inequality check.

#### FYEO-AMULET-ID-09 — Mint authority of LP mints not checked
- **Severity:** Low · **Status:** Remediated · **Protocol:** Amulet Protocol (vault / DeFi (risk-hedge vaults)) · **Report:** `2023/Amulet/MTR_Labs_Pte_Ltd_Secure_Code_Review_of_Amulet_Protocol_v1_0 (1).pdf`
- **Mechanism:** Neither LP mint's mint_authority is verified, so the mint may not be controlled by the program.
- **Prevention rule:** Check `mint_authority == program_pda` when the program must control supply.
- **Anchor/Rust tell:** Mint account accepted without authority check.

#### FYEO-AMULET-ID-10 — Token accounts unchecked for delegate/close authority
- **Severity:** Low · **Status:** Remediated · **Protocol:** Amulet Protocol (vault / DeFi (risk-hedge vaults)) · **Report:** `2023/Amulet/MTR_Labs_Pte_Ltd_Secure_Code_Review_of_Amulet_Protocol_v1_0 (1).pdf`
- **Mechanism:** Treasury and pool token accounts don't verify `delegate` and `close_authority` are unset — a retained delegate can move custody funds.
- **Prevention rule:** Custody token accounts must have delegate==None and close_authority==None (or program-owned).
- **Anchor/Rust tell:** Token account accepted without `delegate.is_none()`/`close_authority.is_none()`.

#### FYEO-AMULET-ID-11 — Pool token account balance unchecked at init
- **Severity:** Low · **Status:** Remediated · **Protocol:** Amulet Protocol (vault / DeFi (risk-hedge vaults)) · **Report:** `2023/Amulet/MTR_Labs_Pte_Ltd_Secure_Code_Review_of_Amulet_Protocol_v1_0 (1).pdf`
- **Mechanism:** vault_pool_token_account need not be empty at vault creation — pre-existing balance corrupts accounting.
- **Prevention rule:** Require custody accounts start with amount==0.
- **Anchor/Rust tell:** Vault init accepts a token account with no `amount == 0` check.

#### FYEO-BANGER-01 — init_if_needed ATAs could have other authorities
- **Severity:** Low · **Status:** Remediated · **Protocol:** Banger (ongoing diff review) (launchpad — diff review) · **Report:** `2025/Banger/Banger Ongoing 2025.03.25.pdf`
- **Mechanism:** WithdrawLiquidity uses `init_if_needed` for two ATAs; under Tokenkeg the existing account's authority can differ from the derivation authority — a different ATA could be passed.
- **Prevention rule:** Prefer `init` for admin paths or verify the existing account's mint+authority explicitly; `init_if_needed` is not a validation.
- **Anchor/Rust tell:** `init_if_needed` on an attacker-influenceable token account.

#### FYEO-BANGER-04 — Co-signer of claim must be manually verified
- **Severity:** Low · **Status:** Acknowledged · **Protocol:** Banger (launchpad / raffle sale) · **Report:** `2024/Banger/Banger - Security Code Review of Banger v1.0.pdf`
- **Mechanism:** Claim instructions accept creator/collector co-signers with no relation to other accounts — any keypair can co-sign, and co-signer can equal admin.
- **Prevention rule:** Require co-signers to match addresses stored in program state, not just "some signature".
- **Anchor/Rust tell:** `Signer` accounts with no address/has_one binding.

#### FYEO-BANGER-06 — InitPool takes any curve account
- **Severity:** Low · **Status:** Remediated · **Protocol:** Banger (launchpad / raffle sale) · **Report:** `2024/Banger/Banger - Security Code Review of Banger v1.0.pdf`
- **Mechanism:** InitPool accepts any curve account although the PDA setup permits exactly one valid curve.
- **Prevention rule:** Constrain interchangeable-type accounts to the single expected address (seeds/address check).
- **Anchor/Rust tell:** Account of a validated type but unvalidated identity.

#### FYEO-CRY-03 — Attacker-created middleware accounts block executions
- **Severity:** Low · **Status:** Open · **Protocol:** Cryptid (identity / DID middleware) · **Report:** `2023/Identity/Identity Technologies Inc. - Security Assessment of the Cryptid v1.0.pdf`
- **Mechanism:** Whitelisted middleware accounts can be created by an attacker and used to block other users' transactions.
- **Prevention rule:** Middleware/authority accounts must be provisioned by a trusted path, not user-creatable.
- **Anchor/Rust tell:** Middleware account init is permissionless.

#### FYEO-SAMO-04 — Project owner can withdraw to any account
- **Severity:** Low · **Status:** Remediated · **Protocol:** Samo Airdrop (airdrop) · **Report:** `2024/Samo/Samo - Security Code Review Samo Airdrop v1.0_public.pdf`
- **Mechanism:** withdraw does not check that project_owner_ata is owned by project_owner — funds can be routed anywhere.
- **Prevention rule:** Verify payout token accounts' authority/mint, not just "a token account was passed".
- **Anchor/Rust tell:** Payout token account without `token::authority` check.

#### FYEO-1INTRO-04 — Referrer fee account check could be extended
- **Severity:** Informational · **Status:** Open · **Protocol:** 1DEX swap platform (AMM) · **Report:** `2024/1DEX/SecurityCodeReview1DEX.pdf`
- **Mechanism:** The referrer swap-fee account checks could be strengthened; no current exploit.
- **Prevention rule:** Extend validation on fee-destination accounts even when not strictly required today.
- **Anchor/Rust tell:** Fee account validated only loosely.

#### FYEO-AX-SOL-16 — ITS: mint and token account checks missing
- **Severity:** Informational · **Status:** Acknowledged · **Protocol:** Axelar Solana integration (bridge / interoperability) · **Report:** `2025/Axelar/Axelar Foundation - Security Code Review of Axelar - Solana Integration v1.0.pdf`
- **Mechanism:** For interchain token deployments the program should check mint_authority, freeze_authority, owner, delegate and close_authority on supplied mints/accounts; ATA recipients of bridged funds can have their owner changed.
- **Prevention rule:** On any user/admin-supplied mint or token account, verify mint, owner, delegate==None, close_authority as the design requires — especially before custodying funds.
- **Anchor/Rust tell:** Token account accepted without `token::mint`, `token::authority`, `delegate.is_none()` constraints.

#### FYEO-BANGER-02 — Account documented as ATA is not verified as ATA
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Banger (ongoing diff review) (launchpad — diff review) · **Report:** `2025/Banger/Banger Ongoing 2025.03.25.pdf`
- **Mechanism:** Claim author rewards documents `author_vault_ata` as the ATA but accepts any token account owned by the authority.
- **Prevention rule:** When the design expects an ATA, verify the address equals the ATA derivation, not merely owner/mint.
- **Anchor/Rust tell:** Comment says ATA; constraint only checks `token::authority`.

#### FYEO-GOV-08 — Unchecked deserialization (no discriminator check)
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Solana Staker Vote Override (gov contract) (governance) · **Report:** `2025/Solana Foundation/Turbine - Security Code Review of Solana Staker Vote Override v1.0.pdf`
- **Mechanism:** Handlers skip discriminator checks on consensus_result & meta_merkle_proof accounts, assuming CPI will validate.
- **Prevention rule:** Validate account type/discriminator at your own boundary; never rely on the callee.
- **Anchor/Rust tell:** Raw `try_from_slice`/deserialize on accounts without discriminator check.

#### FYEO-PEGAX-04 — Operator/verifier keys not checked for duplicates
- **Severity:** Informational · **Status:** Remediated · **Protocol:** PegaX (launchpad (Raydium) / AMM) · **Report:** `2025/PegaX/Security Code Review of PegaX v1.0.pdf`
- **Mechanism:** Constraints validate operator/verifier sets but not key uniqueness — duplicates weaken quorum assumptions.
- **Prevention rule:** Enforce distinctness for every key list.
- **Anchor/Rust tell:** List of authority keys accepted without dedup.

#### FYEO-PEGAX-07 — ATA validation quirk with changed authority
- **Severity:** Informational · **Status:** Remediated · **Protocol:** PegaX (launchpad (Raydium) / AMM) · **Report:** `2025/PegaX/Security Code Review of PegaX v1.0.pdf`
- **Mechanism:** Anchor associated_token::authority checks the ATA owner PDA; if a user once called SetAuthority (pre-Token2022), authority != PDA and the instruction fails — availability/edge risk.
- **Prevention rule:** Know the SetAuthority edge on Tokenkeg ATAs; prefer Token-2022 ImmutableOwner or handle the mismatch.
- **Anchor/Rust tell:** ATA constraint on legacy token accounts.


### initialization — front-runnable, re-init, unauthorized init

#### FYEO-AX-SOL-01 — Governance: config can be deleted
- **Severity:** High · **Status:** Remediated · **Protocol:** Axelar Solana integration (bridge / interoperability) · **Report:** `2025/Axelar/Axelar Foundation - Security Code Review of Axelar - Solana Integration v1.0.pdf`
- **Mechanism:** The config account can transfer out its entire lamport balance, falling below rent-exemption and being deleted — after which anyone can re-initialize and become controller.
- **Prevention rule:** Keep config accounts strictly above rent-exemption, prevent deletion, and make re-initialization impossible or authority-gated.
- **Anchor/Rust tell:** Lamport transfer out of a config PDA without a rent-exemption floor check.

#### FYEO-AX-SOL-02 — Governance: initialize is first-come-first-served
- **Severity:** Medium · **Status:** Remediated · **Protocol:** Axelar Solana integration (bridge / interoperability) · **Report:** `2025/Axelar/Axelar Foundation - Security Code Review of Axelar - Solana Integration v1.0.pdf`
- **Mechanism:** The public initializer sets the contract controller; any caller can front-run deployment and take ownership.
- **Prevention rule:** Bind the initializer to the deployer or upgrade authority.
- **Anchor/Rust tell:** Public `initialize` setting `controller = payer` unchecked.

#### FYEO-1INTRO-05 — Insecure initialization of metadata state
- **Severity:** Low · **Status:** Remediated · **Protocol:** 1Intro launchpad (launchpad / AMM) · **Report:** `2024/1DEX/Security Code Review 1INTRO v1.0_public.pdf`
- **Mechanism:** Public initializer can be front-run.
- **Prevention rule:** Gate init to known authority.

#### FYEO-4CAST-04 — Insecure initialization
- **Severity:** Low · **Status:** Remediated · **Protocol:** 4CAST (points / rewards program) · **Report:** `2024/4Cast/4CAST - Security Code Review 4Cast Programs v1.0.pdf`
- **Mechanism:** Initialization is a public permissionless function; a front-runner can call it first and take control of the config.
- **Prevention rule:** Gate `initialize` to a known authority (deployer/upgrade authority) or make initialization atomic with deployment.
- **Anchor/Rust tell:** Public `initialize` handler with no authority constraint.

#### FYEO-AX-SOL-06 — GasService: authority check missing on config init
- **Severity:** Low · **Status:** Remediated · **Protocol:** Axelar Solana integration (bridge / interoperability) · **Report:** `2025/Axelar/Axelar Foundation - Security Code Review of Axelar - Solana Integration v1.0.pdf`
- **Mechanism:** The function checks the system account and PDA but not who initializes the configuration account — unauthorized init possible.
- **Prevention rule:** Check both the account being created and the authority allowed to create it.
- **Anchor/Rust tell:** Init path validates accounts but not the initializing signer.

#### FYEO-AX-SOL-07 — ITS: initialize is first-come-first-served
- **Severity:** Low · **Status:** Remediated · **Protocol:** Axelar Solana integration (bridge / interoperability) · **Report:** `2025/Axelar/Axelar Foundation - Security Code Review of Axelar - Solana Integration v1.0.pdf`
- **Mechanism:** Public initializer sets the program operator; anyone can call it first.
- **Prevention rule:** Same rule: initializer gated to deployer/upgrade authority.
- **Anchor/Rust tell:** Public `initialize` setting `operator` unchecked.

#### FYEO-BANGER-07 — Curve initialized first-come-first-served
- **Severity:** Low · **Status:** Remediated · **Protocol:** Banger (launchpad / raffle sale) · **Report:** `2024/Banger/Banger - Security Code Review of Banger v1.0.pdf`
- **Mechanism:** Public initialization function; an attacker can initialize the curve first.
- **Prevention rule:** Gate initializers to a known authority or pair init with deployment.

#### FYEO-PEGAX-05 — Insecure initialisation
- **Severity:** Informational · **Status:** Remediated · **Protocol:** PegaX (launchpad (Raydium) / AMM) · **Report:** `2025/PegaX/Security Code Review of PegaX v1.0.pdf`
- **Mechanism:** Public init function can be front-run.
- **Prevention rule:** Gate init.

#### FYEO-VOLTR-06 — Insecure initialization; new admin unsigned
- **Severity:** Informational · **Status:** Acknowledged · **Protocol:** Voltr Vault (vault / yield strategy) · **Report:** `2025/Voltr/Voltr - Security Code Review of Voltr Vault v1.0.pdf`
- **Mechanism:** Public init can be front-run, and new_admin does not sign — both takeover and lockout risk.
- **Prevention rule:** Gate init; require new admin co-sign.
- **Anchor/Rust tell:** Public initialize; authority written without signature.


### access-control — missing signer, admin over-reach, no co-signer/rotation

#### FYEO-CRY-01 — Attacker can extend another user's transaction
- **Severity:** High · **Status:** Remediated · **Protocol:** Cryptid (identity / DID middleware) · **Report:** `2023/Identity/Identity Technologies Inc. - Security Assessment of the Cryptid v1.0.pdf`
- **Mechanism:** ExtendTransaction is meant for privileged signers, but under certain conditions an unauthorized attacker can extend someone else's transaction.
- **Prevention rule:** Authorize extensions against the transaction's stored authority set, not just generic signer presence.
- **Anchor/Rust tell:** Permission check reads roles but misses the unauthorized-signer branch.

#### FYEO-SOL-01 — DID lockout via malicious update instruction
- **Severity:** High · **Status:** Remediated · **Protocol:** Sol_Did (identity / DID) · **Report:** `2023/Identity/Identity Technologies Inc. - Security Assessment of the Sol_Did v1.0.pdf`
- **Mechanism:** DID spec requires inability to lock the account by removing the last verification method; checks exist in some files but a path allows lockout anyway.
- **Prevention rule:** Enforce "can never brick the account" invariants at the program level, not in scattered checks.
- **Anchor/Rust tell:** Removal handler lacks a "last verification method" guard.

#### FYEO-SPREE-01 — Any signer can update fee parameters
- **Severity:** High · **Status:** Remediated · **Protocol:** Spree Points (points / transfer hook) · **Report:** `2025/Spree/Spree - Security Code Review of Spree Points v1.0.pdf`
- **Mechanism:** _update_fees() does not check who calls it — any signer mutates fee params.
- **Prevention rule:** Privileged parameter updates need an authority check against stored config.
- **Anchor/Rust tell:** Update handler with no `has_one = admin`/address check.

#### FYEO-4CAST-03 — Instruction does not require signer
- **Severity:** Medium · **Status:** Remediated · **Protocol:** 4CAST (points / rewards program) · **Report:** `2024/4Cast/4CAST - Security Code Review 4Cast Programs v1.0.pdf`
- **Mechanism:** An instruction executes without requiring any signature, so anyone can invoke it.
- **Prevention rule:** Every privileged or state-mutating instruction needs an explicit `Signer` (or PDA `invoke_signed`) — decide who must sign and enforce it.
- **Anchor/Rust tell:** No `Signer`/`signer` account in the context struct for a mutating handler.

#### FYEO-AMULET-ID-04 — Admin can deny withdrawals by pausing settled vaults
- **Severity:** Medium · **Status:** Remediated · **Protocol:** Amulet Protocol (vault / DeFi (risk-hedge vaults)) · **Report:** `2023/Amulet/MTR_Labs_Pte_Ltd_Secure_Code_Review_of_Amulet_Protocol_v1_0 (1).pdf`
- **Mechanism:** Admin pause blocks SettleVault/WithdrawRiskFund/WithdrawHedgeFund even after the vault outcome is decided — a rug/abuse lever on user funds.
- **Prevention rule:** Scope pause so it cannot freeze withdrawals after terminal/settled states; pause new risk, not exits.
- **Anchor/Rust tell:** Global `is_paused` check on withdraw paths post-settlement.

#### FYEO-SOL-02 — Key compromise can strip recovery methods
- **Severity:** Medium · **Status:** Remediated · **Protocol:** Sol_Did (identity / DID) · **Report:** `2023/Identity/Identity Technologies Inc. - Security Assessment of the Sol_Did v1.0.pdf`
- **Mechanism:** With two wallets, an attacker compromising one can remove the user's recovery abilities and lock them out.
- **Prevention rule:** Recovery-critical changes should require the recovery path or multiple keys, not any single compromised key.
- **Anchor/Rust tell:** Single-key authorization for removing recovery methods.

#### FYEO-AMULET-ID-05 — Hardcoded program initializer authority
- **Severity:** Low · **Status:** Remediated · **Protocol:** Amulet Protocol (vault / DeFi (risk-hedge vaults)) · **Report:** `2023/Amulet/MTR_Labs_Pte_Ltd_Secure_Code_Review_of_Amulet_Protocol_v1_0 (1).pdf`
- **Mechanism:** The PIA is hardcoded, making future rotation impossible without redeploy.
- **Prevention rule:** Store privileged authorities in config, never hardcoded.
- **Anchor/Rust tell:** `const PIA: Pubkey = pubkey!(...)` or literal key compare.

#### FYEO-AMULET-ID-08 — New admin not co-signer in UpdateAdmin
- **Severity:** Low · **Status:** Remediated · **Protocol:** Amulet Protocol (vault / DeFi (risk-hedge vaults)) · **Report:** `2023/Amulet/MTR_Labs_Pte_Ltd_Secure_Code_Review_of_Amulet_Protocol_v1_0 (1).pdf`
- **Mechanism:** AdminUpdateAdminAuthInfo changes the admin key without the new key signing — a typo brick the vault's control.
- **Prevention rule:** Authority transfers must be two-step (propose + accept) or require the new authority to co-sign.
- **Anchor/Rust tell:** `admin = new_admin` write without `new_admin.is_signer`.

#### FYEO-SAMO-03 — New owner does not co-sign ownership transfer
- **Severity:** Low · **Status:** Open · **Protocol:** Samo Airdrop (airdrop) · **Report:** `2024/Samo/Samo - Security Code Review Samo Airdrop v1.0_public.pdf`
- **Mechanism:** Owner can be changed without the new owner signing — a typo or wrong key bricks control.
- **Prevention rule:** Two-step or co-signed authority transfer.
- **Anchor/Rust tell:** `owner = new` without new.is_signer.

#### FYEO-WAT-06 — admin_close_account enables replay and cap bypass
- **Severity:** Low · **Status:** Acknowledged · **Protocol:** Watch.fun (launchpad / sales (drops, packages, tickets)) · **Report:** `2026/ChillChat/Chillchat - Security Code Review of Watch.fun v1.0 (1).pdf`
- **Mechanism:** Admin close drains+zeroes any non-Config account: closing a sig/tx receipt re-enables signature replay; closing a DropEntry resets entries_count, bypassing per-user caps and desyncing totals.
- **Prevention rule:** Treat anti-replay receipts and counters as non-closable; scope admin-close to a strict allowlist.
- **Anchor/Rust tell:** Generic admin-close handler over arbitrary account types.

#### FYEO-AMULET-ID-13 — Vault admin should co-sign CreateVault
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Amulet Protocol (vault / DeFi (risk-hedge vaults)) · **Report:** `2023/Amulet/MTR_Labs_Pte_Ltd_Secure_Code_Review_of_Amulet_Protocol_v1_0 (1).pdf`
- **Mechanism:** The per-vault admin is not a co-signer of vault creation.
- **Prevention rule:** Accounts being granted authority should co-sign acceptance.
- **Anchor/Rust tell:** New authority field written without that key signing.

#### FYEO-EXO-06 — SetTieBreaker can set any winning ballot
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Solana Staker Vote Override (governance) · **Report:** `2025/Solana Foundation/Exo Tech - Security Code Review of Solana Staker Vote Override_v1.0.pdf`
- **Mechanism:** SetTieBreaker copies a ballot from input without validating against current operator set or tallies — the tie-breaker can declare anything the winner.
- **Prevention rule:** Privileged overrides must still validate against on-chain state (tally non-zero, operators consistent).
- **Anchor/Rust tell:** Admin/override instruction writing result fields with no state check.

#### FYEO-SAMO-10 — Enable/disable semantics unchecked
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Samo Airdrop (airdrop) · **Report:** `2024/Samo/Samo - Security Code Review Samo Airdrop v1.0_public.pdf`
- **Mechanism:** Separate enable/disable functions neither check current state nor are used consistently — ambiguous pause semantics.
- **Prevention rule:** Use one state transition function that asserts the current state before mutating.
- **Anchor/Rust tell:** Two flag setters with no current-state check.


### input-bounds — missing bounds/zero/threshold checks

#### FYEO-1INTRO-03 — Missing zero amount checks
- **Severity:** Medium · **Status:** Remediated · **Protocol:** 1Intro launchpad (launchpad / AMM) · **Report:** `2024/1DEX/Security Code Review 1INTRO v1.0_public.pdf`
- **Mechanism:** Several functions may attempt to transfer 0 tokens.
- **Prevention rule:** Reject zero amounts before transfer CPIs.

#### FYEO-1INTRO-04 — UpdateWeightsGradually can set weight to 0
- **Severity:** Medium · **Status:** Remediated · **Protocol:** 1Intro launchpad (launchpad / AMM) · **Report:** `2024/1DEX/Security Code Review 1INTRO v1.0_public.pdf`
- **Mechanism:** When new weights equal current weights, the update computes 0 — a single intended weight update can zero the other.
- **Prevention rule:** No-op-safe update math; reject or short-circuit when delta is zero.
- **Anchor/Rust tell:** Weight update formula that returns 0 when new==old.

#### FYEO-4CAST-02 — Incomplete bound checks
- **Severity:** Medium · **Status:** Remediated · **Protocol:** 4CAST (points / rewards program) · **Report:** `2024/4Cast/4CAST - Security Code Review 4Cast Programs v1.0.pdf`
- **Mechanism:** Several configurable parameters accept values with no upper/lower bounds; misconfiguration by accident or malice can break program logic.
- **Prevention rule:** Clamp every configurable value at set-time with explicit min/max checks, not only at use-time.
- **Anchor/Rust tell:** Config setter writes instruction arg to account with no `require!` range check.

#### FYEO-EXO-01 — add_operators allows duplicates and overflows limit
- **Severity:** Medium · **Status:** Remediated · **Protocol:** Solana Staker Vote Override (governance) · **Report:** `2025/Solana Foundation/Exo Tech - Security Code Review of Solana Staker Vote Override_v1.0.pdf`
- **Mechanism:** add_operators dedups existing operators via HashSet but not the input list; duplicates are pushed per occurrence and no capacity check precedes the push.
- **Prevention rule:** De-duplicate instruction-supplied lists and check capacity before mutating bounded collections.
- **Anchor/Rust tell:** Loop `push` over caller-supplied Vec with no contains/dedup or len check.

#### FYEO-1INTRO-07 — Weight adjustment duration unbounded
- **Severity:** Low · **Status:** Remediated · **Protocol:** 1Intro launchpad (launchpad / AMM) · **Report:** `2024/1DEX/Security Code Review 1INTRO v1.0_public.pdf`
- **Mechanism:** No upper bound on the duration of weight adjustments.
- **Prevention rule:** Bound time-based parameters (duration, windows) at set-time.
- **Anchor/Rust tell:** Duration arg stored without max check.

#### FYEO-AMULET-ID-07 — No input validation in CreateVault
- **Severity:** Low · **Status:** Remediated · **Protocol:** Amulet Protocol (vault / DeFi (risk-hedge vaults)) · **Report:** `2023/Amulet/MTR_Labs_Pte_Ltd_Secure_Code_Review_of_Amulet_Protocol_v1_0 (1).pdf`
- **Mechanism:** CreateVaultMetadataState takes parameters with no validation → logical errors from bad config.
- **Prevention rule:** Validate every config parameter (ranges, non-zero, internal consistency) at creation.
- **Anchor/Rust tell:** Multi-arg init handler storing args unchecked.

#### FYEO-TFLM-ID-02 — UTF-8 length miscount lets names exceed fixed arrays
- **Severity:** Low · **Status:** Remediated · **Protocol:** buddlink-beta-sol (identity / social links) · **Report:** `2023/BuddyLink/Buddy Link - Security Assessment of buddlink-beta-sol v1.0.pdf`
- **Mechanism:** `name.chars().count()` counts codepoints, not bytes; with PROFILE_NAME_LENGTH=18 a name can take 72 bytes, overflowing `set_padding`'s 36-byte assumption.
- **Prevention rule:** Validate string length in bytes (`len()`), not chars, before copying into fixed buffers.
- **Anchor/Rust tell:** `chars().count()` compared against a byte-array capacity.

#### FYEO-1INTRO-05 — Swap allowed with 0 amount
- **Severity:** Informational · **Status:** Remediated · **Protocol:** 1DEX swap platform (AMM) · **Report:** `2024/1DEX/SecurityCodeReview1DEX.pdf`
- **Mechanism:** Users can swap with zero amount.
- **Prevention rule:** Reject zero-amount trades/transfers.
- **Anchor/Rust tell:** No amount>0 on swap arg.

#### FYEO-4CAST-08 — Missing checks for 0 amounts
- **Severity:** Informational · **Status:** Remediated · **Protocol:** 4CAST (points / rewards program) · **Report:** `2024/4Cast/4CAST - Security Code Review 4Cast Programs v1.0.pdf`
- **Mechanism:** Several handlers accept a zero amount where a positive value is semantically required.
- **Prevention rule:** Reject zero (and non-positive) amounts on every token-moving instruction.
- **Anchor/Rust tell:** No `require!(amount > 0)` on deposit/withdraw/transfer args.

#### FYEO-AMULET-ID-17 — Withdraw only checks LP balance > 0
- **Severity:** Informational · **Status:** Open · **Protocol:** Amulet Protocol (vault / DeFi (risk-hedge vaults)) · **Report:** `2023/Amulet/MTR_Labs_Pte_Ltd_Secure_Code_Review_of_Amulet_Protocol_v1_0 (1).pdf`
- **Mechanism:** Withdraw functions check lp_token_account.amount > 0 rather than >= lp_amount being burned.
- **Prevention rule:** Check balances against the exact amount being claimed, not just non-zero.
- **Anchor/Rust tell:** `require!(amount > 0)` where `>= requested` is meant.

#### FYEO-AX-SOL-11 — GasService: no checks on transferred amounts
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Axelar Solana integration (bridge / interoperability) · **Report:** `2025/Axelar/Axelar Foundation - Security Code Review of Axelar - Solana Integration v1.0.pdf`
- **Mechanism:** Gas add/fee/refund functions (native and SPL) do not validate the incoming gas_fee_amount/amount/fees values.
- **Prevention rule:** Validate every value that flows into a transfer or accounting write.
- **Anchor/Rust tell:** Transfer CPI executed with unchecked instruction amount arg.

#### FYEO-BANGER-13 — No bound checks on config values such as fees
- **Severity:** Informational · **Status:** Acknowledged · **Protocol:** Banger (launchpad / raffle sale) · **Report:** `2024/Banger/Banger - Security Code Review of Banger v1.0.pdf`
- **Mechanism:** Fee values in a pool have no bounds; a frontend slip or inattentive user sets absurd fees.
- **Prevention rule:** Clamp fee/weight/limit config at set-time with protocol-level maxima.
- **Anchor/Rust tell:** Config write of fee bps with no `require!(x <= MAX)` check.

#### FYEO-BIO-02 — Missing bound checks on user input
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Bio Launchpad (launchpad) · **Report:** `2025/Bio/Bio - Security Code Review of Bio Launchpad v1.0.pdf`
- **Mechanism:** Upper/lower bounds missing on several user inputs.
- **Prevention rule:** Same clamp-at-set-time rule.

#### FYEO-BIO-04 — Withdraw accepts 0 amount
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Bio Launchpad (launchpad) · **Report:** `2025/Bio/Bio - Security Code Review of Bio Launchpad v1.0.pdf`
- **Mechanism:** The withdraw function accepts a zero amount.
- **Prevention rule:** Reject zero amounts on withdrawal paths.

#### FYEO-GOV-07 — Missing active_stake > 0 check
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Solana Staker Vote Override (gov contract) (governance) · **Report:** `2025/Solana Foundation/Turbine - Security Code Review of Solana Staker Vote Override v1.0.pdf`
- **Mechanism:** support_proposal adds meta_merkle_leaf.active_stake to support totals without verifying stake is positive (other handlers do check).
- **Prevention rule:** Apply the same positivity checks uniformly across all handlers touching the same field.
- **Anchor/Rust tell:** One handler missing a `> 0` check siblings have.

#### FYEO-PEGAX-06 — verifier_count_min not bounded by list length
- **Severity:** Informational · **Status:** Remediated · **Protocol:** PegaX (launchpad (Raydium) / AMM) · **Report:** `2025/PegaX/Security Code Review of PegaX v1.0.pdf`
- **Mechanism:** verifier_count_min can exceed len(verifiers), setting an impossible threshold.
- **Prevention rule:** Cross-validate thresholds against the size of the set they reference.
- **Anchor/Rust tell:** Threshold arg stored without `<= set.len()` check.

#### FYEO-SPREE-04 — Whitelist accepts duplicate accounts
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Spree Points (points / transfer hook) · **Report:** `2025/Spree/Spree - Security Code Review of Spree Points v1.0.pdf`
- **Mechanism:** _add_to_whitelist() does not check membership before appending.
- **Prevention rule:** Reject duplicate inserts into sets.
- **Anchor/Rust tell:** `push` without `contains` check.

#### FYEO-VOLTR-03 — Deposit/withdraw amounts not checked positive
- **Severity:** Informational · **Status:** Acknowledged · **Protocol:** Voltr Vault (vault / yield strategy) · **Report:** `2025/Voltr/Voltr - Security Code Review of Voltr Vault v1.0.pdf`
- **Mechanism:** deposit/withdraw accept non-positive amounts yet still bump update_last_updated_ts.
- **Prevention rule:** Require positive amounts on value-moving instructions.
- **Anchor/Rust tell:** Amount arg unconstrained while timestamp updates.

#### FYEO-WAT-11 — DropAcc::create missing param validation
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Watch.fun (launchpad / sales (drops, packages, tickets)) · **Report:** `2026/ChillChat/Chillchat - Security Code Review of Watch.fun v1.0 (1).pdf`
- **Mechanism:** `id` can be empty — the reinit guard `id.is_empty()` then lets a second create() wipe live state; `tickets_per_entry` accepts 0 making entries free.
- **Prevention rule:** Validate semantic non-emptiness and >0 on admin params; never use a field value both as data and as the reinit sentinel.
- **Anchor/Rust tell:** Guard `is_empty()` on a field users can set to empty.


### arithmetic — unchecked math, wrong formulas

#### FYEO-BANGER-01 — Calculation error in sell function
- **Severity:** High · **Status:** Remediated · **Protocol:** Banger (launchpad / raffle sale) · **Report:** `2024/Banger/Banger - Security Code Review of Banger v1.0.pdf`
- **Mechanism:** The `sell` function computes the subtotal using `banger_fee` twice instead of `banger_fee` + `creator_fee`, mispricing every sale.
- **Prevention rule:** Name fee components distinctly and unit-test each accounting formula against a hand-computed example.
- **Anchor/Rust tell:** Two similar fee vars in one expression (`x_fee` used twice).

#### FYEO-BANGER-08 — Unchecked subtraction in sell can underflow
- **Severity:** Low · **Status:** Remediated · **Protocol:** Banger (launchpad / raffle sale) · **Report:** `2024/Banger/Banger - Security Code Review of Banger v1.0.pdf`
- **Mechanism:** `current_supply - (i + 1)` uses plain subtraction; if supply < i+1 it underflows and panics/wraps.
- **Prevention rule:** Use `checked_sub`/checked math anywhere an operand can exceed the minuend; fail with a domain error.
- **Anchor/Rust tell:** Raw `-` on user-influenced values without `checked_*`.

#### FYEO-AMULET-ID-14 — Inconsistent checked math
- **Severity:** Informational · **Status:** Open · **Protocol:** Amulet Protocol (vault / DeFi (risk-hedge vaults)) · **Report:** `2023/Amulet/MTR_Labs_Pte_Ltd_Secure_Code_Review_of_Amulet_Protocol_v1_0 (1).pdf`
- **Mechanism:** overflow-checks=true covers panics but one path misses explicit checked math where the rest of the codebase uses it.
- **Prevention rule:** Use checked math uniformly; do not mix panic-on-overflow with checked_* style.
- **Anchor/Rust tell:** One raw `+`/`-`/`*` among checked_* code.

#### FYEO-SAMO-07 — Overflow concerns
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Samo Airdrop (airdrop) · **Report:** `2024/Samo/Samo - Security Code Review Samo Airdrop v1.0_public.pdf`
- **Mechanism:** Overflow deemed unlikely via token instructions but recommended checked math since the codebase does loose accounting elsewhere.
- **Prevention rule:** checked math everywhere; assumptions fail under future edits.
- **Anchor/Rust tell:** Raw arithmetic in accounting paths.


### accounting-state — ledger drift, stale reads, unclaimed fees

#### FYEO-VOLTR-01 — Unused harvest_fee() leaves accumulated fees unclaimable
- **Severity:** High · **Status:** Remediated · **Protocol:** Voltr Vault (vault / yield strategy) · **Report:** `2025/Voltr/Voltr - Security Code Review of Voltr Vault v1.0.pdf`
- **Mechanism:** harvest_fee() exists but is exposed by no instruction, so accumulated_lp_{manager,admin,protocol}_fees can never be distributed.
- **Prevention rule:** Every recorded liability needs a reachable settle path; check "state written but never payable" in review.
- **Anchor/Rust tell:** Accounting field incremented but no handler reads/pays it.

#### FYEO-WAT-03 — special_limit bypass: counter never written
- **Severity:** High · **Status:** Remediated · **Protocol:** Watch.fun (launchpad / sales (drops, packages, tickets)) · **Report:** `2026/ChillChat/Chillchat - Security Code Review of Watch.fun v1.0 (1).pdf`
- **Mechanism:** BuyPackageCtx declares `package` without `mut`; `purchased()` increments in memory but Anchor never re-serializes, so the purchased counter stays 0 and the cap never engages.
- **Prevention rule:** Mark every account your handler mutates `#[account(mut)]` — and test that counters actually persist.
- **Anchor/Rust tell:** Handler mutates a struct field of a non-`mut` account.

#### FYEO-AX-SOL-04 — Relayer: latest processed signature not updated in config.toml
- **Severity:** Medium · **Status:** Remediated · **Protocol:** Axelar Solana integration (bridge / interoperability) · **Report:** `2025/Axelar/Axelar Foundation - Security Code Review of Axelar - Solana Integration v1.0.pdf`
- **Mechanism:** `latest_processed_signature` is never written back to the TOML config, so a restarted relayer re-processes from a stale checkpoint.
- **Prevention rule:** Off-chain services must persist cursors/state after each processed item; verify the write path exists, not just the read.
- **Anchor/Rust tell:** Config value read at startup with no matching write path.

#### FYEO-AX-SOL-05 — Relayer: potential data loss via mmap flush_async
- **Severity:** Medium · **Status:** Remediated · **Protocol:** Axelar Solana integration (bridge / interoperability) · **Report:** `2025/Axelar/Axelar Foundation - Security Code Review of Axelar - Solana Integration v1.0.pdf`
- **Mechanism:** `mmap.flush_async()` returns before modified pages are durable; crash/restart can lose processed data and metadata timestamps.
- **Prevention rule:** Use synchronous `flush()` on state that must survive restarts before acknowledging processing.
- **Anchor/Rust tell:** `flush_async` (or equivalent fire-and-forget persistence) on critical state.

#### FYEO-PEGAX-01 — Incorrect token accounting (lamports vs token balance)
- **Severity:** Medium · **Status:** Remediated · **Protocol:** PegaX (launchpad (Raydium) / AMM) · **Report:** `2025/PegaX/Security Code Review of PegaX v1.0.pdf`
- **Mechanism:** RaydiumLaunchpadBuy/Sell account for swapped tokens using the token account's lamport balance instead of its SPL token amount.
- **Prevention rule:** Always measure token flows via `TokenAccount.amount` (and reload after CPI), never lamports.
- **Anchor/Rust tell:** `.lamports()` read on a token account for token accounting.

#### FYEO-SAMO-01 — Withdraw does bad accounting
- **Severity:** Medium · **Status:** Remediated · **Protocol:** Samo Airdrop (airdrop) · **Report:** `2024/Samo/Samo - Security Code Review Samo Airdrop v1.0_public.pdf`
- **Mechanism:** withdraw fails to reset total_supply to 0 and disables the project without checking it isn't already disabled — stale state under future changes.
- **Prevention rule:** Every mutating instruction should leave all derived/counter fields consistent; enumerate the full state delta.
- **Anchor/Rust tell:** Handler updates one field while sibling counters/flags are left stale.

#### FYEO-VOLTR-02 — Protocol fees never accumulated
- **Severity:** Medium · **Status:** Remediated · **Protocol:** Voltr Vault (vault / yield strategy) · **Report:** `2025/Voltr/Voltr - Security Code Review of Voltr Vault v1.0.pdf`
- **Mechanism:** accumulated_lp_protocol_fees is declared but never updated or minted — the protocol never receives its share.
- **Prevention rule:** Trace each fee/accounting field end-to-end: accrual write → accumulation → distribution.
- **Anchor/Rust tell:** Fee field declared, never incremented.

#### FYEO-WAT-04 — Special package buyers never receive bonus tickets
- **Severity:** Medium · **Status:** Remediated · **Protocol:** Watch.fun (launchpad / sales (drops, packages, tickets)) · **Report:** `2026/ChillChat/Chillchat - Security Code Review of Watch.fun v1.0 (1).pdf`
- **Mechanism:** buy_package/claim_gift credit only `ticket_count`; the `special_bonus_ticket_count` field stored during set_packages is never read — the special signature grants nothing extra.
- **Prevention rule:** Every stored config field must have a consumer; trace field → read site in review.
- **Anchor/Rust tell:** Config field written but never read anywhere.

#### FYEO-1INTRO-06 — Checks rely on stale token account data
- **Severity:** Low · **Status:** Remediated · **Protocol:** 1Intro launchpad (launchpad / AMM) · **Report:** `2024/1DEX/Security Code Review 1INTRO v1.0_public.pdf`
- **Mechanism:** token0 is meant frozen but several checks read stale account data; amounts must be reloaded after CPI to be trusted.
- **Prevention rule:** `reload()` any account whose data may change across a CPI before checking it again.
- **Anchor/Rust tell:** Account field read after a CPI without `.reload()`.

#### FYEO-EXO-03 — Mid-ballot whitelist changes alter consensus denominator
- **Severity:** Low · **Status:** Acknowledged · **Protocol:** Solana Staker Vote Override (governance) · **Report:** `2025/Solana Foundation/Exo Tech - Security Code Review of Solana Staker Vote Override_v1.0.pdf`
- **Mechanism:** Consensus computes tally_bps from `whitelisted_operators.len()` at vote time; changing the whitelist mid-ballot changes the denominator — spec says thresholds are set at creation.
- **Prevention rule:** Snapshot the voter set/threshold at proposal creation; never read live config for in-flight consensus.
- **Anchor/Rust tell:** Threshold denominator read from live config during an active vote.

#### FYEO-SAMO-02 — Manual airdrop ignores accounting
- **Severity:** Low · **Status:** Remediated · **Protocol:** Samo Airdrop (airdrop) · **Report:** `2024/Samo/Samo - Security Code Review Samo Airdrop v1.0_public.pdf`
- **Mechanism:** manual airdrop writes any amount into the claim account without performing the corresponding token transfer — ledger ≠ balances.
- **Prevention rule:** Never write claim/ledger state without the matching asset movement in the same instruction.
- **Anchor/Rust tell:** State write of an amount with no CPI transfer beside it.

#### FYEO-WAT-09 — claim_gift reads live ticket_count instead of snapshot
- **Severity:** Low · **Status:** Remediated · **Protocol:** Watch.fun (launchpad / sales (drops, packages, tickets)) · **Report:** `2026/ChillChat/Chillchat - Security Code Review of Watch.fun v1.0 (1).pdf`
- **Mechanism:** GiftAcc stores no snapshot; claim_gift credits whatever package.ticket_count is at claim time — admin mutation between purchase and claim changes the yield.
- **Prevention rule:** Snapshot the entitlement at creation time; never re-read mutable config at redemption.
- **Anchor/Rust tell:** Redemption reads a mutable config field instead of a stored entitlement.

#### FYEO-SAMO-11 — Global collection admin-mutable; unclear airdrop end criteria
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Samo Airdrop (airdrop) · **Report:** `2024/Samo/Samo - Security Code Review Samo Airdrop v1.0_public.pdf`
- **Mechanism:** Admin can change the global collection affecting all live projects; no time bound or clear end state for claims — funds withdrawable any time.
- **Prevention rule:** Define explicit lifecycle states for airdrops and freeze shared config while projects are live.
- **Anchor/Rust tell:** Live projects read a mutable global config.


### signature-introspection — Ed25519/header/replay bugs

#### FYEO-WAT-01 — Full operator signature bypass via unchecked Ed25519 headers
- **Severity:** Critical · **Status:** Remediated · **Protocol:** Watch.fun (launchpad / sales (drops, packages, tickets)) · **Report:** `2026/ChillChat/Chillchat - Security Code Review of Watch.fun v1.0 (1).pdf`
- **Mechanism:** parse_ed25519_ix_data never validates num_sigs nor the *_ix_index header fields; num_sigs=0 makes native Ed25519 succeed vacuously, and non-MAX ix_index lets the program read sig/pubkey/message from another instruction.
- **Prevention rule:** Validate every Ed25519 instruction-introspection header field: num_sigs==1, all ix_index fields == u16::MAX, offsets consistent.
- **Anchor/Rust tell:** Parsing `instructions` sysvar Ed25519 data by offsets without checking header fields.

#### FYEO-WAT-02 — Cross-type signature replay via missing domain separation
- **Severity:** High · **Status:** Remediated · **Protocol:** Watch.fun (launchpad / sales (drops, packages, tickets)) · **Report:** `2026/ChillChat/Chillchat - Security Code Review of Watch.fun v1.0 (1).pdf`
- **Mechanism:** Three flows sign the same operator key over byte layouts with no domain tag; crafted ids make one flow's signature valid in another (e.g., special package ↔ claim tickets ↔ payment).
- **Prevention rule:** Bind signed messages to (program, instruction type, recipient, expiry) with an explicit domain separator.
- **Anchor/Rust tell:** Same signer verifies two message layouts that can collide byte-for-byte.

#### FYEO-PEGAX-02 — Insecure ed25519 parsing (unchecked slicing)
- **Severity:** Medium · **Status:** Remediated · **Protocol:** PegaX (launchpad (Raydium) / AMM) · **Report:** `2025/PegaX/Security Code Review of PegaX v1.0.pdf`
- **Mechanism:** parse_ed25519 assumes fixed layout and slices data without validity checks — OOB access or wrong pubkey/message parsing.
- **Prevention rule:** Bounds-check and header-validate all instruction-introspection data before slicing.
- **Anchor/Rust tell:** Direct `data[a..b]` slicing on introspected instruction data.

#### FYEO-WAT-08 — Gift payment signature does not bind flow or recipient
- **Severity:** Low · **Status:** Remediated · **Protocol:** Watch.fun (launchpad / sales (drops, packages, tickets)) · **Report:** `2026/ChillChat/Chillchat - Security Code Review of Watch.fun v1.0 (1).pdf`
- **Mechanism:** SigPackagePayment message = buyer+tx_hash+chain_id+package_id+expiry; no flow discriminator or recipient — a gift payment can be replayed as a self-buy and gifts can be redirected.
- **Prevention rule:** Include action type and beneficiary inside every signed message.
- **Anchor/Rust tell:** Signature message omits recipient/discriminator fields present in the handler.


### dos-resource — unbounded growth, panics, griefing

#### FYEO-SF-01 — Rent exemption not checked on metadata account (create_unchecked)
- **Severity:** High · **Status:** Remediated · **Protocol:** Streamflow protocols (payments / streaming) · **Report:** `2023/Streamflow/Streamflow_Finance_Security_Assessment_of_the_Streamflow_Protocols.pdf`
- **Mechanism:** The metadata account is not verified rent-exempt and could be deleted unexpectedly, breaking dependents.
- **Prevention rule:** Check rent-exemption for accounts created unchecked that must persist.
- **Anchor/Rust tell:** create_unchecked path with no rent-exempt validation.

#### FYEO-CRY-02 — Permissionless ApproveExecution can block others' ExecuteTransaction
- **Severity:** Medium · **Status:** Remediated · **Protocol:** Cryptid (identity / DID middleware) · **Report:** `2023/Identity/Identity Technologies Inc. - Security Assessment of the Cryptid v1.0.pdf`
- **Mechanism:** A permissionless instruction lets attackers interfere with core operations — approving/blocking other users' executions.
- **Prevention rule:** Permissionless handlers must be provably side-effect-free for other users; otherwise gate them.
- **Anchor/Rust tell:** Public instruction mutating shared/per-user execution state.

#### FYEO-EXO-02 — Pushes in cast_vote may exceed limits
- **Severity:** Medium · **Status:** Remediated · **Protocol:** Solana Staker Vote Override (governance) · **Report:** `2025/Solana Foundation/Exo Tech - Security Code Review of Solana Staker Vote Override_v1.0.pdf`
- **Mechanism:** cast_vote unconditionally pushes into ballot_tallies and operator_votes; concurrent operator changes can grow them past safe bounds → panics/account-size/consensus breakage.
- **Prevention rule:** Bound-check every Vec growth at the push site against the allocated max.
- **Anchor/Rust tell:** Unconditional `.push` into account-sized Vecs.

#### FYEO-GOV-03 — Vote tallying panics and DoS paths
- **Severity:** Medium · **Status:** Remediated · **Protocol:** Solana Staker Vote Override (gov contract) (governance) · **Report:** `2025/Solana Foundation/Turbine - Security Code Review of Solana Staker Vote Override v1.0.pdf`
- **Mechanism:** TallyVotes: anyone can trigger; realloc(0) dedup leaves stranded lamports; closed VoteStates between epochs panic; loop decrement without underflow protection.
- **Prevention rule:** Same tally-loop hardening as FYEO-REAL-01 (same codebase lineage).
- **Anchor/Rust tell:** Tally loop over arbitrary accounts.

#### FYEO-REAL-01 — Incomplete vote-tallying logic causes panics/DoS
- **Severity:** Medium · **Status:** Remediated · **Protocol:** validator vote dashboard (governance / analytics) · **Report:** `2025/Solana Foundation/Real-time Dashboard - Security Code Review_v1.0.pdf`
- **Mechanism:** TallyVotes: any validator can initiate tallying; vote accounts deduped via realloc(0) leaving unreclaimable lamports; closed VoteState accounts between epochs panic the tally; vote count decremented in a loop without underflow protection.
- **Prevention rule:** Loop-driven accounting over user-supplied account lists needs permission rules, underflow guards, and liveness assumptions checked per epoch.
- **Anchor/Rust tell:** Tally loop over arbitrary accounts with raw `--` and realloc tricks.

#### FYEO-SPREE-02 — Vec whitelist scales O(n) past compute budget
- **Severity:** Medium · **Status:** Remediated · **Protocol:** Spree Points (points / transfer hook) · **Report:** `2025/Spree/Spree - Security Code Review of Spree Points v1.0.pdf`
- **Mechanism:** Whitelist stores Vec<Pubkey>; contains/remove are O(n) — at hundreds of thousands of entries the transfer hook blows the 200k CU budget.
- **Prevention rule:** Choose O(1)/account-per-entry structures (PDA membership or bitset) for on-chain lists consulted per transfer.
- **Anchor/Rust tell:** Linear `iter().any()`/`position()` over an unbounded on-chain Vec.

#### FYEO-BIO-01 — Low minimum contribution enables contributor-slot DoS
- **Severity:** Low · **Status:** Remediated · **Protocol:** Bio Launchpad (launchpad) · **Report:** `2025/Bio/Bio - Security Code Review of Bio Launchpad v1.0.pdf`
- **Mechanism:** Minimum contribution can be 1 token unit while contributor list is bounded (u16 ≈ 65k); an attacker fills all slots for a few dollars.
- **Prevention rule:** Set economically meaningful minimums for bounded participant lists.
- **Anchor/Rust tell:** Bounded `Vec`/counter of participants + no minimum contribution.

#### FYEO-AX-SOL-19 — Relayer: no check on compute budget limit
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Axelar Solana integration (bridge / interoperability) · **Report:** `2025/Axelar/Axelar Foundation - Security Code Review of Axelar - Solana Integration v1.0.pdf`
- **Mechanism:** computed_units is padded +10% without checking the protocol ceiling, producing transactions the RPC/validator rejects.
- **Prevention rule:** Clamp computed-unit estimates under the Solana limit before submitting.
- **Anchor/Rust tell:** CU budget arg derived by multiplication with no max check.

#### FYEO-AX-SOL-20 — Relayer: unbounded concurrent connections
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Axelar Solana integration (bridge / interoperability) · **Report:** `2025/Axelar/Axelar Foundation - Security Code Review of Axelar - Solana Integration v1.0.pdf`
- **Mechanism:** The REST service accepts unlimited connections — overload/DoS.
- **Prevention rule:** Bound every network-facing resource (connections, queue depth, retries).
- **Anchor/Rust tell:** Server accept loop with no semaphore/rate limit.

#### FYEO-EXO-04 — remove_vote leaves zeroed tallies causing growth
- **Severity:** Informational · **Status:** Acknowledged · **Protocol:** Solana Staker Vote Override (governance) · **Report:** `2025/Solana Foundation/Exo Tech - Security Code Review of Solana Staker Vote Override_v1.0.pdf`
- **Mechanism:** Removing a vote decrements tally but leaves the zeroed BallotTally entry; repeated cycles accumulate dead entries until limits hit.
- **Prevention rule:** Prune zeroed entries or compact the collection on removal.
- **Anchor/Rust tell:** Removal path decrements but never deletes container entries.

#### FYEO-EXO-05 — Off-chain service hardening gaps
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Solana Staker Vote Override (governance) · **Report:** `2025/Solana Foundation/Exo Tech - Security Code Review of Solana Staker Vote Override_v1.0.pdf`
- **Mechanism:** Trusted-operator service has paths for accidental data loss, invalid indexing, operator error, spoofable rate-limit bypass and local FS manipulation (reduced risk because operator is trusted).
- **Prevention rule:** Harden operator-run services like the chain: validate inputs, bound resources, persist cursors.

#### FYEO-EXO-07 — Unbounded gzip decompression
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Solana Staker Vote Override (governance) · **Report:** `2025/Solana Foundation/Exo Tech - Security Code Review of Solana Staker Vote Override_v1.0.pdf`
- **Mechanism:** read_from_bytes_with_hash decompresses without a size cap — crafted payload is a zip bomb.
- **Prevention rule:** Cap decompressed size before allocating.
- **Anchor/Rust tell:** Decompress into unbounded Vec.

#### FYEO-VOLTR-08 — Panic copying strings into fixed arrays at vault init
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Voltr Vault (vault / yield strategy) · **Report:** `2025/Voltr/Voltr - Security Code Review of Voltr Vault v1.0.pdf`
- **Mechanism:** name/description strings copied into fixed byte arrays without length checks — runtime panic for longer inputs.
- **Prevention rule:** Length-check before copying user strings into fixed buffers.
- **Anchor/Rust tell:** `copy_from_slice` into `[u8; N]` from unchecked String.

#### FYEO-WAT-13 — Unbounded Vec pushes exceed max_len allocations
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Watch.fun (launchpad / sales (drops, packages, tickets)) · **Report:** `2026/ChillChat/Chillchat - Security Code Review of Watch.fun v1.0 (1).pdf`
- **Mechanism:** DropAcc winner lists (max_len 5) and DropEntryAcc entries_ranges (max_len 100) are pushed without length guards — the 101st entry permanently locks the user account.
- **Prevention rule:** Check `vec.len() < max_len` before every push; reject or rotate instead of reverting at serialize time.
- **Anchor/Rust tell:** `push` into a `#[max_len]`-sized Vec without a capacity check.

#### FYEO-WAT-14 — sig_receipt PDA griefing via pre-funding
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Watch.fun (launchpad / sales (drops, packages, tickets)) · **Report:** `2026/ChillChat/Chillchat - Security Code Review of Watch.fun v1.0 (1).pdf`
- **Mechanism:** sig_receipt PDA derives from the operator signature; an observer can pre-send lamports so `create_account` (which requires lamports==0) reverts, burning the grant permanently.
- **Prevention rule:** Use Anchor `init` (tolerates prefunded PDAs) or design receipts so prefunding cannot brick them.
- **Anchor/Rust tell:** `system_program::create_account` on a deterministic PDA with a lamports==0 require.


### cpi-external — unpinned programs, unchecked oracles

#### FYEO-AMULET-ID-03 — Oracle account unchecked at vault creation
- **Severity:** High · **Status:** Remediated · **Protocol:** Amulet Protocol (vault / DeFi (risk-hedge vaults)) · **Report:** `2023/Amulet/MTR_Labs_Pte_Ltd_Secure_Code_Review_of_Amulet_Protocol_v1_0 (1).pdf`
- **Mechanism:** vault_oracle_account is not validated — a malicious or mistaken admin can wire a bad oracle into the vault.
- **Prevention rule:** Pin oracles to a whitelist/expected program+address at creation, or store-and-verify on use.
- **Anchor/Rust tell:** Oracle account stored with no owner/address check.

#### FYEO-BANGER-03 — Users can choose rarity via fake switchboard account
- **Severity:** High · **Status:** Remediated · **Protocol:** Banger (launchpad / raffle sale) · **Report:** `2024/Banger/Banger - Security Code Review of Banger v1.0.pdf`
- **Mechanism:** The program never verifies the supplied account belongs to the Switchboard program, so any correctly-sized deserializable account can fake randomness for rarity selection.
- **Prevention rule:** Check external oracle/vrf accounts' owner AND expected program ID before trusting their data.
- **Anchor/Rust tell:** Randomness account validated only by data length/deserialization, no `owner`/`address` check.

#### FYEO-GOV-01 — Snapshot program account not pinned
- **Severity:** High · **Status:** Acknowledged · **Protocol:** Solana Staker Vote Override (gov contract) (governance) · **Report:** `2025/Solana Foundation/Turbine - Security Code Review of Solana Staker Vote Override v1.0.pdf`
- **Mechanism:** snapshot_program is an UncheckedAccount; PDAs are only checked `owner == snapshot_program.key()` — the program itself is never required to be the real snapshot program.
- **Prevention rule:** Pin every CPI/external program account to its known program ID (address check or Program<T>).
- **Anchor/Rust tell:** `UncheckedAccount` used as a program ID reference, owners compared to it.


### token-program — decimals, Token-2022, ATA issues

#### FYEO-SPREE-01 — Init decimals allowed to vary while math assumes 6
- **Severity:** Medium · **Status:** Remediated · **Protocol:** Spree (ongoing diff review) (points — diff review) · **Report:** `2025/Spree/Spree Ongoing 2025.08.11 .pdf`
- **Mechanism:** args.decimals feeds mint::decimals but constants assume SP_DECIMALS=6 and SP_PER_USDC=100 — a different init value breaks conversion math and pricing.
- **Prevention rule:** When arithmetic assumes fixed decimals, pin decimals at init (constant or check) — never accept it as free input.
- **Anchor/Rust tell:** `decimals` taken from args while `SP_DECIMALS` const assumes a value.

#### FYEO-1INTRO-01 — LP token mint allows arbitrary decimals
- **Severity:** Low · **Status:** Remediated · **Protocol:** 1DEX swap platform (AMM) · **Report:** `2024/1DEX/SecurityCodeReview1DEX.pdf`
- **Mechanism:** The user-provided LP mint has no decimals check or consistency requirement.
- **Prevention rule:** Fix or validate decimals for mints your accounting depends on (store decimals and re-verify).
- **Anchor/Rust tell:** User-supplied `Mint` with no decimals constraint.

#### FYEO-BIO-03 — Tokens can only be withdrawn to ATA
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Bio Launchpad (launchpad) · **Report:** `2025/Bio/Bio - Security Code Review of Bio Launchpad v1.0.pdf`
- **Mechanism:** Withdrawal destinations restricted to ATAs; for non-Token2022 tokens a user's ATA authority can have been changed (common scam vector). Team confirmed Token2022 is used, so not exploitable here.
- **Prevention rule:** If you restrict destinations to ATAs, prefer Token-2022 `ImmutableOwner` or verify the ATA authority matches.
- **Anchor/Rust tell:** Withdraw requires `associated_token::authority` but token program is Tokenkeg.

#### FYEO-VOLTR-07 — LP token disallows Token-2022
- **Severity:** Informational · **Status:** Acknowledged · **Protocol:** Voltr Vault (vault / yield strategy) · **Report:** `2025/Voltr/Voltr - Security Code Review of Voltr Vault v1.0.pdf`
- **Mechanism:** Program enforces legacy Tokenkeg for LP tokens — no ImmutableOwner, so ATAs remain vulnerable to authority-change scams.
- **Prevention rule:** Accept Token-2022 via TokenInterface unless there is a reason not to.
- **Anchor/Rust tell:** `Program<Token>` instead of `Interface<TokenInterface>` for user-facing mints.


### randomness-ordering — advisory RNG, mid-sale mutation, races

#### FYEO-WAT-05 — Admin mid-sale mutations rewrite live drop/package state
- **Severity:** Low · **Status:** Acknowledged · **Protocol:** Watch.fun (launchpad / sales (drops, packages, tickets)) · **Report:** `2026/ChillChat/Chillchat - Security Code Review of Watch.fun v1.0 (1).pdf`
- **Mechanism:** update overwrites end_date and caps with no entries_sold gate; set_package uses init_if_needed + unconditional set that resets special_limit_purchased=0 and can flip is_special mid-sale.
- **Prevention rule:** Gate admin mutations on sale state (no live entries) and update only the fields intended to change.
- **Anchor/Rust tell:** `init_if_needed`/`set` that rewrites all fields including counters on a live account.

#### FYEO-WAT-07 — admin_set_winners ignores pending_winners — randomness advisory
- **Severity:** Low · **Status:** Acknowledged · **Protocol:** Watch.fun (launchpad / sales (drops, packages, tickets)) · **Report:** `2026/ChillChat/Chillchat - Security Code Review of Watch.fun v1.0 (1).pdf`
- **Mechanism:** select_winner stores Switchboard-derived winners in pending_winners, but admin_set_winners writes instruction-supplied winners verbatim without comparing — the randomness draw is decorative.
- **Prevention rule:** When randomness drives an outcome, verify the submitted result equals the drawn result (or let the VRF callback write it directly).
- **Anchor/Rust tell:** RNG result stored in one account but a later admin input replaces it unchecked.

#### FYEO-WAT-10 — select_winner randomness handling gaps
- **Severity:** Low · **Status:** Acknowledged · **Protocol:** Watch.fun (launchpad / sales (drops, packages, tickets)) · **Report:** `2026/ChillChat/Chillchat - Security Code Review of Watch.fun v1.0 (1).pdf`
- **Mechanism:** reveal_slot>0 is no freshness check (admin can grind old reveals); only 4 of 32 random bytes used with no attempt cap; no account-type discriminator on the Switchboard account; both devnet and mainnet program IDs accepted in one binary.
- **Prevention rule:** Consume randomness atomically from a fresh, type-checked VRF account; derive all outputs from the full value; pin the program ID per build.
- **Anchor/Rust tell:** Randomness account reused across slots; partial bytes consumed; dual env program IDs compiled in.


### spec-mismatch — implementation ≠ docs, unused fields

#### FYEO-BANGER-05 — InitPool not restricted as documented
- **Severity:** Low · **Status:** Remediated · **Protocol:** Banger (launchpad / raffle sale) · **Report:** `2024/Banger/Banger - Security Code Review of Banger v1.0.pdf`
- **Mechanism:** Docs say InitPool may only be run by an authorized user; the implementation allows anyone.
- **Prevention rule:** Implement what the spec states, or fix the spec — spec-mismatch is itself a finding class.
- **Anchor/Rust tell:** Docstring/comment claims an authorization the code does not enforce.

#### FYEO-GOV-05 — Proposal.voting field unused on-chain
- **Severity:** Low · **Status:** Remediated · **Protocol:** Solana Staker Vote Override (gov contract) (governance) · **Report:** `2025/Solana Foundation/Turbine - Security Code Review of Solana Staker Vote Override v1.0.pdf`
- **Mechanism:** `voting: bool` is used by frontend/tests but never enforced in program logic — state exists that code ignores.
- **Prevention rule:** Delete fields the program never enforces, or enforce them; surface ≠ spec.
- **Anchor/Rust tell:** State field never read by any handler.

#### FYEO-SOL-03 — Missing DID-syntax check on controllers
- **Severity:** Low · **Status:** Remediated · **Protocol:** Sol_Did (identity / DID) · **Report:** `2023/Identity/Identity Technologies Inc. - Security Assessment of the Sol_Did v1.0.pdf`
- **Mechanism:** set_other_controllers does not verify controllers follow the DID syntax spec ("did:method:id").
- **Prevention rule:** Validate protocol-level format rules on every externally supplied identifier.
- **Anchor/Rust tell:** Identifier stored without format validation.

#### FYEO-SAMO-09 — Airdrop PDA only works for NFTs; multi-NFT double claims
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Samo Airdrop (airdrop) · **Report:** `2024/Samo/Samo - Security Code Review Samo Airdrop v1.0_public.pdf`
- **Mechanism:** The claim scheme works only for NFT collections, and users holding multiple NFTs can claim multiple times — likely not intended.
- **Prevention rule:** Model eligibility/claim tracking per intended unit (wallet vs token) and test multi-asset cases.
- **Anchor/Rust tell:** Claim keyed by token with no per-wallet dedup.

#### FYEO-SF-07 — Two create instructions make extra checks optional
- **Severity:** Informational · **Status:** Open · **Protocol:** Streamflow protocols (payments / streaming) · **Report:** `2023/Streamflow/Streamflow_Finance_Security_Assessment_of_the_Streamflow_Protocols.pdf`
- **Mechanism:** A checked and an unchecked create instruction exist; every extra check in the checked path becomes optional — plus duplicated code.
- **Prevention rule:** One create path; gate optional checks behind params if needed, never a second unchecked instruction.
- **Anchor/Rust tell:** Parallel `create`/`create_unchecked` instruction pair.

#### FYEO-SOL-04 — Controller functionality deviates from W3C spec
- **Severity:** Informational · **Status:** Open · **Protocol:** Sol_Did (identity / DID) · **Report:** `2023/Identity/Identity Technologies Inc. - Security Assessment of the Sol_Did v1.0.pdf`
- **Mechanism:** A DID set as controller without proper permissions cannot modify the DID document per W3C semantics.
- **Prevention rule:** Track spec conformance explicitly; deviations need documenting as intentional.
- **Anchor/Rust tell:** Implementation behavior contradicts referenced standard.

#### FYEO-SOL-05 — Self-controller restriction deviates from W3C spec
- **Severity:** Informational · **Status:** Open · **Protocol:** Sol_Did (identity / DID) · **Report:** `2023/Identity/Identity Technologies Inc. - Security Assessment of the Sol_Did v1.0.pdf`
- **Mechanism:** sol:did forbids the DID being its own controller while W3C allows it — deviation from the standard.
- **Prevention rule:** Same spec-conformance rule.
- **Anchor/Rust tell:** Required check contradicts referenced spec text.


### code-quality — dead code, duplication, clarity

#### FYEO-1INTRO-03 — Code clarity
- **Severity:** Informational · **Status:** Remediated · **Protocol:** 1DEX swap platform (AMM) · **Report:** `2024/1DEX/SecurityCodeReview1DEX.pdf`
- **Mechanism:** Readability improvements needed.
- **Prevention rule:** Same hygiene rule.

#### FYEO-1INTRO-09 — Code clarity
- **Severity:** Informational · **Status:** Remediated · **Protocol:** 1Intro launchpad (launchpad / AMM) · **Report:** `2024/1DEX/Security Code Review 1INTRO v1.0_public.pdf`
- **Mechanism:** Rust style/best-practice issues.
- **Prevention rule:** Same hygiene rule.

#### FYEO-4CAST-05 — Code clarity
- **Severity:** Informational · **Status:** Remediated · **Protocol:** 4CAST (points / rewards program) · **Report:** `2024/4Cast/4CAST - Security Code Review 4Cast Programs v1.0.pdf`
- **Mechanism:** Readability issues in several places raise maintenance and review cost.
- **Prevention rule:** Keep the audited code clean; clarity defects hide real bugs.

#### FYEO-4CAST-06 — Duplicate account checks
- **Severity:** Informational · **Status:** Remediated · **Protocol:** 4CAST (points / rewards program) · **Report:** `2024/4Cast/4CAST - Security Code Review 4Cast Programs v1.0.pdf`
- **Mechanism:** Both the Token/System program wrappers and Anchor macros re-check the same account keys — dead duplication.
- **Prevention rule:** Drop redundant manual checks where the framework already enforces them; each check should exist once.
- **Anchor/Rust tell:** Manual `key() == expected` next to `constraint`/`address` on the same account.

#### FYEO-AMULET-ID-12 — Duplicate execution in AttemptToTrigger
- **Severity:** Informational · **Status:** Open · **Protocol:** Amulet Protocol (vault / DeFi (risk-hedge vaults)) · **Report:** `2023/Amulet/MTR_Labs_Pte_Ltd_Secure_Code_Review_of_Amulet_Protocol_v1_0 (1).pdf`
- **Mechanism:** get_latest_trigger_status() runs twice — once via constraint macros, again in process().
- **Prevention rule:** Run each external read once and pass the value through.
- **Anchor/Rust tell:** Same CPI/view call in both constraints and handler.

#### FYEO-AMULET-ID-16 — Unnecessary constraints
- **Severity:** Informational · **Status:** Open · **Protocol:** Amulet Protocol (vault / DeFi (risk-hedge vaults)) · **Report:** `2023/Amulet/MTR_Labs_Pte_Ltd_Secure_Code_Review_of_Amulet_Protocol_v1_0 (1).pdf`
- **Mechanism:** Some checks exist that can be removed (over-constraining adds surface and cost).
- **Prevention rule:** Keep only checks that trace to a threat or spec requirement.
- **Anchor/Rust tell:** Constraints with no matching threat/spec reason.

#### FYEO-AX-SOL-10 — GasService: code duplication
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Axelar Solana integration (bridge / interoperability) · **Report:** `2025/Axelar/Axelar Foundation - Security Code Review of Axelar - Solana Integration v1.0.pdf`
- **Mechanism:** Four functions repeat the same configuration-PDA validation block.
- **Prevention rule:** Extract shared validation into one helper; duplicated checks drift apart.
- **Anchor/Rust tell:** Same validation block copy-pasted across handlers.

#### FYEO-AX-SOL-12 — Gateway: dead code
- **Severity:** Informational · **Status:** Acknowledged · **Protocol:** Axelar Solana integration (bridge / interoperability) · **Report:** `2025/Axelar/Axelar Foundation - Security Code Review of Axelar - Solana Integration v1.0.pdf`
- **Mechanism:** `verify_eddsa_signature` is defined but never called.
- **Prevention rule:** Delete dead code before audit — unused crypto paths confuse reviewers and hide intent.
- **Anchor/Rust tell:** Public helper with zero call sites.

#### FYEO-AX-SOL-13 — Gateway: incorrect names of errors and functions
- **Severity:** Informational · **Status:** Acknowledged · **Protocol:** Axelar Solana integration (bridge / interoperability) · **Report:** `2025/Axelar/Axelar Foundation - Security Code Review of Axelar - Solana Integration v1.0.pdf`
- **Mechanism:** Function and error names do not match the logic they implement.
- **Prevention rule:** Name errors and functions after what they enforce; misleading names cause misapplied checks.
- **Anchor/Rust tell:** Error variant named for a different invariant than the check.

#### FYEO-AX-SOL-15 — Governance: code improvements
- **Severity:** Informational · **Status:** Acknowledged · **Protocol:** Axelar Solana integration (bridge / interoperability) · **Report:** `2025/Axelar/Axelar Foundation - Security Code Review of Axelar - Solana Integration v1.0.pdf`
- **Mechanism:** Minor clarity concerns in the governance module.
- **Prevention rule:** Same hygiene rule as other clarity findings.

#### FYEO-AX-SOL-17 — MultiCall: incorrect account indexing
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Axelar Solana integration (bridge / interoperability) · **Report:** `2025/Axelar/Axelar Foundation - Security Code Review of Axelar - Solana Integration v1.0.pdf`
- **Mechanism:** `MultiCallPayloadBuilder::build()` computes account_start_index/account_end_index wrong for zero-account instructions (start == end).
- **Prevention rule:** Handle the empty/zero case explicitly in index arithmetic.
- **Anchor/Rust tell:** Index computed from a count without the zero branch.

#### FYEO-AX-SOL-18 — Relayer: constant value/name mismatch
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Axelar Solana integration (bridge / interoperability) · **Report:** `2025/Axelar/Axelar Foundation - Security Code Review of Axelar - Solana Integration v1.0.pdf`
- **Mechanism:** A named constant's value does not match its name.
- **Prevention rule:** Keep constants named by meaning and audited against literals.
- **Anchor/Rust tell:** Constant like `X_MS = 5` used as seconds.

#### FYEO-BANGER-09 — Code clarity
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Banger (launchpad / raffle sale) · **Report:** `2024/Banger/Banger - Security Code Review of Banger v1.0.pdf`
- **Mechanism:** Duplicate and inefficient code in places.
- **Prevention rule:** Same hygiene rule.

#### FYEO-BANGER-10 — Constants declared but not used
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Banger (launchpad / raffle sale) · **Report:** `2024/Banger/Banger - Security Code Review of Banger v1.0.pdf`
- **Mechanism:** PDA seed constants exist but are not consistently used.
- **Prevention rule:** Derive seeds from declared constants everywhere; delete unused constants.
- **Anchor/Rust tell:** Literal seed strings duplicated instead of the declared constant.

#### FYEO-BANGER-12 — Manual account size calculation
- **Severity:** Informational · **Status:** Acknowledged · **Protocol:** Banger (launchpad / raffle sale) · **Report:** `2024/Banger/Banger - Security Code Review of Banger v1.0.pdf`
- **Mechanism:** Account sizes are computed manually in several different ways.
- **Prevention rule:** Use `INIT_SPACE`/`8 + T::INIT_SPACE` or a single sizing helper; manual sizes drift from the struct.
- **Anchor/Rust tell:** Hand-summed `space = 8 + 32 + 8 + ...` literals.

#### FYEO-BIO-05 — Unfinished code (TODOs) in scope
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Bio Launchpad (launchpad) · **Report:** `2025/Bio/Bio - Security Code Review of Bio Launchpad v1.0.pdf`
- **Mechanism:** Several TODO comments indicate missing logic around bonding-phase validation and accounts.
- **Prevention rule:** Do not ship/audit code with TODO placeholders in production paths.
- **Anchor/Rust tell:** `TODO`/`FIXME` in handlers.

#### FYEO-EXO-08 — Unnecessary clone allocation
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Solana Staker Vote Override (governance) · **Report:** `2025/Solana Foundation/Exo Tech - Security Code Review of Solana Staker Vote Override_v1.0.pdf`
- **Mechanism:** A possibly large meta_merkle_proof is cloned where a borrow suffices, wasting CUs.
- **Prevention rule:** Borrow over clone for large structures.
- **Anchor/Rust tell:** `.clone()` on large account/proof structs.

#### FYEO-GOV-06 — Code clarity
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Solana Staker Vote Override (gov contract) (governance) · **Report:** `2025/Solana Foundation/Turbine - Security Code Review of Solana Staker Vote Override v1.0.pdf`
- **Mechanism:** Magic numbers, repeated validation, redundant Result wrapping.
- **Prevention rule:** Same hygiene rule.

#### FYEO-PEGAX-03 — General improvements
- **Severity:** Informational · **Status:** Remediated · **Protocol:** PegaX (launchpad (Raydium) / AMM) · **Report:** `2025/PegaX/Security Code Review of PegaX v1.0.pdf`
- **Mechanism:** Maintainability optimizations available.
- **Prevention rule:** Same hygiene rule.

#### FYEO-REAL-02 — Code clarity
- **Severity:** Informational · **Status:** Remediated · **Protocol:** validator vote dashboard (governance / analytics) · **Report:** `2025/Solana Foundation/Real-time Dashboard - Security Code Review_v1.0.pdf`
- **Mechanism:** Magic numbers, repeated validation patterns, redundant Result wrapping.
- **Prevention rule:** Same hygiene rule; extract constants and shared validators.

#### FYEO-SAMO-05 — Constant bank authority regenerated per request
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Samo Airdrop (airdrop) · **Report:** `2024/Samo/Samo - Security Code Review Samo Airdrop v1.0_public.pdf`
- **Mechanism:** A value constant across projects is recomputed on every call.
- **Prevention rule:** Derive once (constant or stored PDA) instead of per-request.
- **Anchor/Rust tell:** Same derivation repeated per instruction.

#### FYEO-SAMO-06 — Code clarity
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Samo Airdrop (airdrop) · **Report:** `2024/Samo/Samo - Security Code Review Samo Airdrop v1.0_public.pdf`
- **Mechanism:** Readability improvements needed.
- **Prevention rule:** Same hygiene rule.

#### FYEO-SAMO-08 — PDA space does not account for strings
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Samo Airdrop (airdrop) · **Report:** `2024/Samo/Samo - Security Code Review Samo Airdrop v1.0_public.pdf`
- **Mechanism:** ProjectPda and ClaimPda store Strings but the space macro makes no room for them (String is 24 bytes header + heap data).
- **Prevention rule:** Size accounts with `INIT_SPACE`/computed space including max string/vec lengths.
- **Anchor/Rust tell:** `space =` literal that omits variable-length fields.

#### FYEO-SAMO-12 — Missing custom errors
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Samo Airdrop (airdrop) · **Report:** `2024/Samo/Samo - Security Code Review Samo Airdrop v1.0_public.pdf`
- **Mechanism:** Macro constraints return unintelligible errors to users.
- **Prevention rule:** Attach ` @ ErrorCode::X` custom errors to user-facing constraints.
- **Anchor/Rust tell:** Constraints without error codes on user paths.

#### FYEO-SF-02 — Code cleanup suggestions
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Streamflow protocols (payments / streaming) · **Report:** `2023/Streamflow/Streamflow_Finance_Security_Assessment_of_the_Streamflow_Protocols.pdf`
- **Mechanism:** Cleanup + cargo clippy recommended.
- **Prevention rule:** Run clippy; keep audit surface clean.

#### FYEO-SF-03 — Error variant never constructed
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Streamflow protocols (payments / streaming) · **Report:** `2023/Streamflow/Streamflow_Finance_Security_Assessment_of_the_Streamflow_Protocols.pdf`
- **Mechanism:** A ProgramError variant is never constructed; the function can be simplified.
- **Prevention rule:** Delete unused error variants and dead branches.
- **Anchor/Rust tell:** Error enum variant with no raise site.

#### FYEO-SF-05 — Payment period oversized at 8 bytes
- **Severity:** Informational · **Status:** Open · **Protocol:** Streamflow protocols (payments / streaming) · **Report:** `2023/Streamflow/Streamflow_Finance_Security_Assessment_of_the_Streamflow_Protocols.pdf`
- **Mechanism:** 8-byte payment period supports 130+ years per unit; 4 bytes suffices.
- **Prevention rule:** Size integer fields to real ranges to save space.
- **Anchor/Rust tell:** u64 field for a small-range value.

#### FYEO-SF-06 — Superfluous return statements
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Streamflow protocols (payments / streaming) · **Report:** `2023/Streamflow/Streamflow_Finance_Security_Assessment_of_the_Streamflow_Protocols.pdf`
- **Mechanism:** match arms use 10 unnecessary `return`s.
- **Prevention rule:** Style cleanup.

#### FYEO-SF-08 — Unnecessary while-loop padding calc
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Streamflow protocols (payments / streaming) · **Report:** `2023/Streamflow/Streamflow_Finance_Security_Assessment_of_the_Streamflow_Protocols.pdf`
- **Mechanism:** Padding computed with a while loop where `+= 8 - (size % 8)` suffices.
- **Prevention rule:** Prefer closed-form arithmetic over loops for sizing.
- **Anchor/Rust tell:** Loop used for a constant-time computation.

#### FYEO-TFLM-ID-04 — Undocumented contract methods
- **Severity:** Informational · **Status:** Remediated · **Protocol:** buddlink-beta-sol (identity / social links) · **Report:** `2023/BuddyLink/Buddy Link - Security Assessment of buddlink-beta-sol v1.0.pdf`
- **Mechanism:** Methods carry no documentation of intent beyond identifier names.
- **Prevention rule:** Document each instruction's intent, signer expectations, and account ownership — it becomes the audit spec.

#### FYEO-TFLM-ID-05 — Duplicate claim arithmetic across functions
- **Severity:** Informational · **Status:** Remediated · **Protocol:** buddlink-beta-sol (identity / social links) · **Report:** `2023/BuddyLink/Buddy Link - Security Assessment of buddlink-beta-sol v1.0.pdf`
- **Mechanism:** The same claiming arithmetic is duplicated in multiple functions.
- **Prevention rule:** Single-source accounting formulas; duplicates diverge.
- **Anchor/Rust tell:** Same mul/div expression copy-pasted in handlers.

#### FYEO-TFLM-ID-06 — Manual reimplementation of account.exit()
- **Severity:** Informational · **Status:** Remediated · **Protocol:** buddlink-beta-sol (identity / social links) · **Report:** `2023/BuddyLink/Buddy Link - Security Assessment of buddlink-beta-sol v1.0.pdf`
- **Mechanism:** `write_data_manually` reimplements `account.exit()` manually.
- **Prevention rule:** Use framework primitives over hand-rolled equivalents.
- **Anchor/Rust tell:** Manual lamport-move + zeroing code duplicating `exit`/`close`.

#### FYEO-TFLM-ID-07 — Redundant multi-pass over same data
- **Severity:** Informational · **Status:** Remediated · **Protocol:** buddlink-beta-sol (identity / social links) · **Report:** `2023/BuddyLink/Buddy Link - Security Assessment of buddlink-beta-sol v1.0.pdf`
- **Mechanism:** `match_owners_pubkey` and `match_owners` iterate the same data multiple times, wasting compute.
- **Prevention rule:** Single-pass matching saves CU and simplifies logic.
- **Anchor/Rust tell:** Two sequential `find`/`filter` passes over one collection.

#### FYEO-VOLTR-04 — Dead code
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Voltr Vault (vault / yield strategy) · **Report:** `2025/Voltr/Voltr - Security Code Review of Voltr Vault v1.0.pdf`
- **Mechanism:** Unused code snippets present.
- **Prevention rule:** Delete dead code.


### testing — missing/incomplete tests

#### FYEO-AX-SOL-08 — Amplifier: incomplete test for invalid characters
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Axelar Solana integration (bridge / interoperability) · **Report:** `2025/Axelar/Axelar Foundation - Security Code Review of Axelar - Solana Integration v1.0.pdf`
- **Mechanism:** The base58 validation test covers invalid chars I/0/O but misses lowercase 'l', which Base58 also forbids.
- **Prevention rule:** Negative-input tests should enumerate the complete invalid set per the spec, not a sample.
- **Anchor/Rust tell:** Test vector lists a partial alphabet of invalid inputs.

#### FYEO-AX-SOL-09 — Amplifier: missing tests
- **Severity:** Informational · **Status:** Acknowledged · **Protocol:** Axelar Solana integration (bridge / interoperability) · **Report:** `2025/Axelar/Axelar Foundation - Security Code Review of Axelar - Solana Integration v1.0.pdf`
- **Mechanism:** Implementation ships with no unit tests validating correctness.
- **Prevention rule:** Every module needs tests before audit scope; untested code is unreviewable.
- **Anchor/Rust tell:** No test files / `#[cfg(test)]` for a module.

#### FYEO-AX-SOL-14 — Gateway: missing tests
- **Severity:** Informational · **Status:** Acknowledged · **Protocol:** Axelar Solana integration (bridge / interoperability) · **Report:** `2025/Axelar/Axelar Foundation - Security Code Review of Axelar - Solana Integration v1.0.pdf`
- **Mechanism:** No unit tests for the gateway implementation.
- **Prevention rule:** Same rule as AX-SOL-09: tests are audit scope.

#### FYEO-BANGER-11 — Improve test cases
- **Severity:** Informational · **Status:** Acknowledged · **Protocol:** Banger (launchpad / raffle sale) · **Report:** `2024/Banger/Banger - Security Code Review of Banger v1.0.pdf`
- **Mechanism:** Test suite lacks coverage of concurrent txs, min/max inputs, and overflow/underflow paths.
- **Prevention rule:** Cover boundary values, concurrent paths, and failure cases — negative test per failure path.
- **Anchor/Rust tell:** Tests only exercise happy path with default values.

#### FYEO-VOLTR-05 — Incomplete test coverage
- **Severity:** Informational · **Status:** Acknowledged · **Protocol:** Voltr Vault (vault / yield strategy) · **Report:** `2025/Voltr/Voltr - Security Code Review of Voltr Vault v1.0.pdf`
- **Mechanism:** Tests cover only creation/deposit/withdraw/config-update basics.
- **Prevention rule:** Cover every instruction incl. failure paths before audit.
- **Anchor/Rust tell:** Tests missing for several instructions.


### observability — missing/wrong/misleading events

#### FYEO-1INTRO-08 — Wrong event emitted
- **Severity:** Low · **Status:** Remediated · **Protocol:** 1Intro launchpad (launchpad / AMM) · **Report:** `2024/1DEX/Security Code Review 1INTRO v1.0_public.pdf`
- **Mechanism:** Emits UpdateToken0WeightEvent when token1 weight is updated.
- **Prevention rule:** Match emitted event type to the actual mutation; indexers trust events.
- **Anchor/Rust tell:** Wrong event variant emitted in a symmetric handler.

#### FYEO-4CAST-07 — Events emitted without action
- **Severity:** Informational · **Status:** Remediated · **Protocol:** 4CAST (points / rewards program) · **Report:** `2024/4Cast/4CAST - Security Code Review 4Cast Programs v1.0.pdf`
- **Mechanism:** Events fire regardless of whether any meaningful state change occurred, making off-chain indexing unreliable.
- **Prevention rule:** Emit events only on actual state transitions and include the fields indexers need.
- **Anchor/Rust tell:** `emit!` called unconditionally at handler exit.

#### FYEO-GOV-01 — Timestamp/epoch emission inconsistencies in ProposalCreated
- **Severity:** Informational · **Status:** Remediated · **Protocol:** gov contract (ongoing diff review) (governance — diff review) · **Report:** `2025/Solana Foundation/Turbine - Ongoing 2025.12.04_v2.1.0.pdf`
- **Mechanism:** ProposalCreated emits snapshot_slot always 0 (uninitialized) and timestamp but not creation_epoch — misleading to indexers.
- **Prevention rule:** Emit only initialized, meaningful fields; keep emitted shape consistent with stored state.
- **Anchor/Rust tell:** Event field that is always default/zero.

#### FYEO-SPREE-03 — No events on key state updates
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Spree Points (points / transfer hook) · **Report:** `2025/Spree/Spree - Security Code Review of Spree Points v1.0.pdf`
- **Mechanism:** _toggle_freeze() and _update_fees() change state without emitting events — changes are invisible to admins/users.
- **Prevention rule:** Emit an event on every privileged mutation.
- **Anchor/Rust tell:** Privileged handler with no `emit!`.

#### FYEO-SPREE-05 — Emits deletion event for nonexistent whitelist entry
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Spree Points (points / transfer hook) · **Report:** `2025/Spree/Spree - Security Code Review of Spree Points v1.0.pdf`
- **Mechanism:** _remove_from_whitelist() emits removal logs/events even when the account was absent — misleading audit trail.
- **Prevention rule:** Emit outcome events that reflect what actually happened (including no-ops distinctly or rejecting).
- **Anchor/Rust tell:** Event/log emitted before checking membership.


### upgradability-ops — no rotation/versioning/cleanup path

#### FYEO-TFLM-ID-01 — Hard-coded global values should be configurable
- **Severity:** Medium · **Status:** Remediated · **Protocol:** buddlink-beta-sol (identity / social links) · **Report:** `2023/BuddyLink/Buddy Link - Security Assessment of buddlink-beta-sol v1.0.pdf`
- **Mechanism:** The contract hardcodes many global values that will need to change over time.
- **Prevention rule:** Put tunable globals in a config account owned by a governed authority instead of constants.
- **Anchor/Rust tell:** Business parameters as `const` in source.

#### FYEO-AMULET-ID-15 — No cleanup path for old accounts
- **Severity:** Informational · **Status:** Open · **Protocol:** Amulet Protocol (vault / DeFi (risk-hedge vaults)) · **Report:** `2023/Amulet/MTR_Labs_Pte_Ltd_Secure_Code_Review_of_Amulet_Protocol_v1_0 (1).pdf`
- **Mechanism:** Settled vaults' many accounts can never be closed; rent is stranded (Token2022 mint close could help).
- **Prevention rule:** Provide close instructions for every account family after its terminal state.
- **Anchor/Rust tell:** No `close`/`close_account` anywhere in the program.

#### FYEO-BANGER-14 — Rent cannot be recovered
- **Severity:** Informational · **Status:** Acknowledged · **Protocol:** Banger (launchpad / raffle sale) · **Report:** `2024/Banger/Banger - Security Code Review of Banger v1.0.pdf`
- **Mechanism:** Certain accounts can never be closed, so their rent is locked forever.
- **Prevention rule:** Provide a close path for every account type the program creates (with proper authority gating).
- **Anchor/Rust tell:** Account type created via `init` with no corresponding `close` path.

#### FYEO-CRY-04 — Super-user middleware cannot be modified or revoked
- **Severity:** Informational · **Status:** Open · **Protocol:** Cryptid (identity / DID middleware) · **Report:** `2023/Identity/Identity Technologies Inc. - Security Assessment of the Cryptid v1.0.pdf`
- **Mechanism:** Cryptid accounts fix their super-user middleware set at creation with no edit/revoke path — permanent over-privilege.
- **Prevention rule:** Provide an owner-gated rotation/revocation path for every delegated authority.
- **Anchor/Rust tell:** Authority set written at init with no update handler.

#### FYEO-SF-04 — Partner list capped at 250 entries by Vec storage
- **Severity:** Informational · **Status:** Open · **Protocol:** Streamflow protocols (payments / streaming) · **Report:** `2023/Streamflow/Streamflow_Finance_Security_Assessment_of_the_Streamflow_Protocols.pdf`
- **Mechanism:** Partner account stores a fixed-size Vec (10,000 bytes ≈ 250 entries) with no growth path.
- **Prevention rule:** Size registries for expected growth or use per-entry PDAs.
- **Anchor/Rust tell:** Fixed-space Vec registry with hard capacity.

#### FYEO-TFLM-ID-03 — No versioning or upgrade path for accounts
- **Severity:** Informational · **Status:** Remediated · **Protocol:** buddlink-beta-sol (identity / social links) · **Report:** `2023/BuddyLink/Buddy Link - Security Assessment of buddlink-beta-sol v1.0.pdf`
- **Mechanism:** Account structures have no version field or migration path for future changes.
- **Prevention rule:** Reserve a version/discriminator scheme and a migration story for account layouts.
- **Anchor/Rust tell:** State struct with no version field and append-only layout assumptions.

#### FYEO-WAT-12 — No on-chain rotation path for admin/operator
- **Severity:** Informational · **Status:** Remediated · **Protocol:** Watch.fun (launchpad / sales (drops, packages, tickets)) · **Report:** `2026/ChillChat/Chillchat - Security Code Review of Watch.fun v1.0 (1).pdf`
- **Mechanism:** ConfigAcc exposes only init; no setter for admin/operator and no delete path — privileged keys are immutable forever.
- **Prevention rule:** Every privileged key needs an on-chain rotation instruction (with co-signer) or a multisig migration plan.
- **Anchor/Rust tell:** Config has init but no `set_admin`/`update_authority` handler.

## Section C — Cross-cutting lessons

Patterns that recur across ≥3 reports:

- **Insecure/first-come initialization is the single most repeated bug** — public initializers or unprotected config across 9 findings and ≥7 reports: FYEO-4CAST-04, FYEO-AX-SOL-01/02/06/07, FYEO-BANGER-07, FYEO-1INTRO-05, FYEO-PEGAX-05, FYEO-VOLTR-06. Gate init to deployer/upgrade authority; keep config non-deletable.
- **Token accounts are under-validated** — mint, delegate, close_authority, owner, balance, ATA-vs-any-account across ≥8 reports: FYEO-AMULET-ID-01/06/09/10/11, FYEO-AX-SOL-16, FYEO-BANGER-ongoing-01/02, FYEO-1INTRO-01/02, FYEO-1DEX-02/04, FYEO-SAMO-04. Verify every field the custody design relies on.
- **Authority transfers without the new key signing** recur in ≥4 reports: FYEO-AMULET-ID-08/13, FYEO-SAMO-03, FYEO-VOLTR-06, FYEO-WAT-12 (worse: no rotation at all). Two-step or co-signed transfer for every privileged key.
- **Zero amounts and unbounded config values** are pervasive — ≥10 reports: FYEO-4CAST-02/08, FYEO-AX-SOL-11, FYEO-BANGER-13, FYEO-BIO-02/04, FYEO-1INTRO-03/07, FYEO-1DEX-05, FYEO-PEGAX-06, FYEO-VOLTR-03, FYEO-WAT-11. Clamp at set-time; reject 0.
- **Accounting drift between ledger and balances** — fees never accrued/distributed, live-vs-snapshot reads, missing `mut`, reload-after-CPI — ≥6 reports: FYEO-VOLTR-01/02, FYEO-WAT-03/04/09, FYEO-1INTRO-06, FYEO-SAMO-01/02/11, FYEO-PEGAX-01. One ledger, one mutation site, invariant tests.
- **Admin/god-mode over-reach** — pause-can-block-withdrawals, arbitrary close, mid-sale mutation, any-signer fee updates — ≥6 reports: FYEO-AMULET-ID-04, FYEO-SPREE-01, FYEO-WAT-05/06, FYEO-CRY-02/04, FYEO-EXO-06, FYEO-4CAST-01. Scope admin powers per state; make receipts/counters non-closable.
- **Signature/introspection is rare but catastrophic** — all 4 findings are High/Critical (FYEO-WAT-01/02/08, FYEO-PEGAX-02): validate every Ed25519 header field and domain-separate signed messages.
- **Unpinned external accounts** (oracles, VRF, other programs) — FYEO-BANGER-03, FYEO-AMULET-ID-03, FYEO-GOV-01, FYEO-WAT-10. Pin program IDs and account identities before trusting data.
- **Unbounded on-chain/off-chain resources** — Vec pushes past max_len, O(n) whitelists, decompression bombs, connection limits, panics on fixed arrays — ≥7 reports: FYEO-WAT-13/14, FYEO-SPREE-02, FYEO-EXO-02/07, FYEO-AX-SOL-19/20, FYEO-BIO-01, FYEO-REAL-01, FYEO-VOLTR-08.
- **Mutating live config underneath in-flight state** — mid-ballot denominator, mid-sale package/whitelist changes — FYEO-EXO-03, FYEO-WAT-05/07/09, FYEO-SAMO-11. Snapshot what a position/vote/gift was promised at creation.
- **Missing or misleading events** — FYEO-4CAST-07, FYEO-SPREE-03/05, FYEO-1INTRO-08, FYEO-GOV-ongoing-01. Emit on every privileged mutation and only for real transitions.
- **Code hygiene findings are the plurality** (34): dead code, duplication, unused constants/errors, manual sizing, unclear naming — they are Informational but they are where the Highs hide. Reports: nearly all.
- **Missing/incomplete tests noted in ≥4 reports**: FYEO-AX-SOL-08/09/14, FYEO-BANGER-11, FYEO-VOLTR-05 — tests are audit scope; negative tests are expected.
- **No cleanup/rotation/versioning path** — FYEO-TFLM-ID-01/03, FYEO-AMULET-ID-05/15, FYEO-BANGER-14, FYEO-WAT-12, FYEO-CRY-04, FYEO-SF-04. Every privileged key rotatable, every account closable.

## Section D — Report index

| Date | Client | Protocol (type) | C | H | M | L | I | Report file | Commit |
|------|--------|-----------------|--:|--:|--:|--:|--:|-------------|--------|

| 2023 | Buddy Link | buddlink-beta-sol (identity / social links) | 0 | 0 | 1 | 1 | 5 | `Code Audit Reports/2023/BuddyLink/Buddy Link - Security Assessment of buddlink-beta-sol v1.0.pdf` | `db6aeb650b45d4875c0d7b5272f820182e5a0d81` |
| 2023 | Identity Technologies Inc. | Cryptid (identity / DID middleware) | 0 | 1 | 1 | 1 | 1 | `Code Audit Reports/2023/Identity/Identity Technologies Inc. - Security Assessment of the Cryptid v1.0.pdf` | `5b6a0630a41ba61acafa1593dd9ccc74e5cb5e95` |
| 2023 | Identity Technologies Inc. | Sol_Did (identity / DID) | 0 | 1 | 1 | 1 | 2 | `Code Audit Reports/2023/Identity/Identity Technologies Inc. - Security Assessment of the Sol_Did v1.0.pdf` | `b4f2b71a8a600009379915ea49612f9dd82d2c59` |
| 2023 | MTR Labs | Amulet Protocol (vault / DeFi (risk-hedge vaults)) | 0 | 3 | 1 | 7 | 6 | `Code Audit Reports/2023/Amulet/MTR_Labs_Pte_Ltd_Secure_Code_Review_of_Amulet_Protocol_v1_0 (1).pdf` | `e182a0ecf4e2f70d701f21a153a76eb84c26bc29` |
| 2023 | Streamflow Finance | Streamflow protocols (payments / streaming) | 0 | 1 | 0 | 0 | 7 | `Code Audit Reports/2023/Streamflow/Streamflow_Finance_Security_Assessment_of_the_Streamflow_Protocols.pdf` | `c82ce81db42b78343826223a77ff50db87dec1e1` |
| 2024 | Banger | Banger (launchpad / raffle sale) | 0 | 3 | 0 | 5 | 6 | `Code Audit Reports/2024/Banger/Banger - Security Code Review of Banger v1.0.pdf` | `49560795d2f1b9909a4e9b2d42074cd4ae57defe` |
| 2024 | 4CAST | 4CAST (points / rewards program) | 0 | 0 | 3 | 1 | 4 | `Code Audit Reports/2024/4Cast/4CAST - Security Code Review 4Cast Programs v1.0.pdf` | `fe17284cc86d8e0705153197a9b7d858320d01a3` |
| 2024 | Samo | Samo Airdrop (airdrop) | 0 | 0 | 1 | 3 | 8 | `Code Audit Reports/2024/Samo/Samo - Security Code Review Samo Airdrop v1.0_public.pdf` | `be80ac41fae5cece855dc0272baabf5117ca7748` |
| 2024 | 1Intro | 1DEX swap platform (AMM) | 0 | 0 | 0 | 2 | 3 | `Code Audit Reports/2024/1DEX/SecurityCodeReview1DEX.pdf` | `41a82354f27b49a1befcdddbb0f4628e3d167e8e` |
| 2024 | 1Intro | 1Intro launchpad (launchpad / AMM) | 0 | 0 | 4 | 4 | 1 | `Code Audit Reports/2024/1DEX/Security Code Review 1INTRO v1.0_public.pdf` | `8f04afb527b40386cb1986d0e73a5ac304da02c2` |
| 2025 | Axelar Foundation | Axelar Solana integration (bridge / interoperability) | 0 | 1 | 4 | 2 | 13 | `Code Audit Reports/2025/Axelar/Axelar Foundation - Security Code Review of Axelar - Solana Integration v1.0.pdf` | `b6be90dbbe5e987bbce94f6d860064ce878b1d22` |
| 2025 | Bio | Bio Launchpad (launchpad) | 0 | 0 | 0 | 1 | 4 | `Code Audit Reports/2025/Bio/Bio - Security Code Review of Bio Launchpad v1.0.pdf` | `—` |
| 2025 | Exo Tech | Solana Staker Vote Override (governance) | 0 | 0 | 2 | 1 | 5 | `Code Audit Reports/2025/Solana Foundation/Exo Tech - Security Code Review of Solana Staker Vote Override_v1.0.pdf` | `—` |
| 2025 | Real-time Dashboard | validator vote dashboard (governance / analytics) | 0 | 0 | 1 | 0 | 1 | `Code Audit Reports/2025/Solana Foundation/Real-time Dashboard - Security Code Review_v1.0.pdf` | `—` |
| 2025 | PegaX | PegaX (launchpad (Raydium) / AMM) | 0 | 0 | 2 | 0 | 5 | `Code Audit Reports/2025/PegaX/Security Code Review of PegaX v1.0.pdf` | `—` |
| 2025 | Spree | Spree Points (points / transfer hook) | 0 | 1 | 1 | 0 | 3 | `Code Audit Reports/2025/Spree/Spree - Security Code Review of Spree Points v1.0.pdf` | `—` |
| 2025 | Turbine | Solana Staker Vote Override (gov contract) (governance) | 0 | 2 | 2 | 1 | 3 | `Code Audit Reports/2025/Solana Foundation/Turbine - Security Code Review of Solana Staker Vote Override v1.0.pdf` | `—` |
| 2025 | Voltr | Voltr Vault (vault / yield strategy) | 0 | 1 | 1 | 0 | 6 | `Code Audit Reports/2025/Voltr/Voltr - Security Code Review of Voltr Vault v1.0.pdf` | `—` |
| 2026 | Chillchat | Watch.fun (launchpad / sales (drops, packages, tickets)) | 1 | 2 | 1 | 6 | 4 | `Code Audit Reports/2026/ChillChat/Chillchat - Security Code Review of Watch.fun v1.0 (1).pdf` | `—` |
| 2025-03-25 | Banger | Banger (ongoing diff review) (launchpad — diff review) | 0 | 0 | 0 | 1 | 1 | `Code Audit Reports/2025/Banger/Banger Ongoing 2025.03.25.pdf` | `—` |
| 2025-08-11 | Spree | Spree (ongoing diff review) (points — diff review) | 0 | 0 | 1 | 0 | 0 | `Code Audit Reports/2025/Spree/Spree Ongoing 2025.08.11 .pdf` | `—` |
| 2025-12-04 | Turbine | gov contract (ongoing diff review) (governance — diff review) | 0 | 0 | 0 | 0 | 1 | `Code Audit Reports/2025/Solana Foundation/Turbine - Ongoing 2025.12.04_v2.1.0.pdf` | `—` |
| 2024-12-20 | Banger | Banger (ongoing diff review) | 0 | 0 | 0 | 0 | 0 | `Code Audit Reports/2024/Banger/Banger Ongoing 2024.12.20_v1.0.pdf` | — |

Ongoing/diff-review engagements (Banger, Spree, Turbine rows above) review a commit range rather than a full codebase and typically produce few or zero findings.
