# Solana Code Patterns

Distilled from `escrow-q3-26-upstream` (programs `escrowq32026` + `q3_26_vault`),
`pre-req-vault`, `vault-reference`, and `spl-nft-q326`. All Anchor programs use
`anchor-lang`/`anchor-spl` **1.1.2**.

---

## 1. Program architecture patterns

### Module layout (identical in every Anchor program)

`lib.rs` re-exports flat modules; `instructions.rs` is a per-instruction
submodule barrel:

```rust
// escrow-q3-26-upstream/programs/escrowq32026/src/lib.rs:1-10
pub mod constants;
pub mod error;
pub mod instructions;
pub mod state;
pub use constants::*;
pub use instructions::*;
pub use state::*;
```

```rust
// escrow-q3-26-upstream/programs/escrowq32026/src/instructions.rs:1-11
pub mod make;   pub use make::*;
pub mod refund; pub use refund::*;
pub mod take;   pub use take::*;
pub mod update; pub use update::*;
```

### Thin `#[program]` handlers delegate to `impl` blocks on the Accounts struct

Each instruction file defines `#[derive(Accounts)]` + `impl<'info> X<'info>` with
the business logic. The `lib.rs` entrypoint only validates args then calls
`ctx.accounts.<method>()`:

```rust
// escrow-q3-26-upstream/programs/escrowq32026/src/lib.rs:27-42
#[instruction(discriminator = 0)]
pub fn make(ctx: Context<Make>, seed: u64, deposit: u64, receive: u64, expiration: i64) -> Result<()> {
    require!(deposit > 0 && receive > 0, crate::error::ErrorCode::InvalidAmount);
    ctx.accounts.init_escrow(seed, receive, &ctx.bumps, expiration)?;
    ctx.accounts.deposit(deposit)
}
```

Note the explicit `#[instruction(discriminator = N)]` attributes — Anchor 1.x
manual discriminators (escrow uses 0–3).

### State structs

```rust
// escrow-q3-26-upstream/programs/escrowq32026/src/state.rs:3-13
#[derive(InitSpace)]
#[account(discriminator = 1)]   // custom account discriminator
pub struct Escrow {
    pub seed: u64,
    pub maker: Pubkey,
    pub mint_a: Pubkey,
    pub mint_b: Pubkey,
    pub receive: u64,
    pub bump: u8,          // canonical bump stored on-chain
    pub expiration: i64,
}
```

```rust
// escrow-q3-26-upstream/programs/q3_26_vault/src/state.rs:3-8
#[account]
#[derive(InitSpace)]
pub struct VaultState {
    pub vault_bump: u8,
    pub state_bump: u8,   // stores bumps for BOTH PDAs
}
```

Space is always computed as `DISCRIMINATOR.len() + INIT_SPACE`
(e.g. `make.rs:33`); the older variant uses the literal `8 + VaultState::INIT_SPACE`
(`pre-req-vault/.../instructions/initialize.rs`).

### Error handling

One `error.rs` per program with `#[error_code]` + `#[msg]`:

```rust
// escrow-q3-26-upstream/programs/q3_26_vault/src/error.rs:3-11
#[error_code]
pub enum ErrorCode {
    #[msg("The amount must be greater than zero")]
    InvalidAmount,
    #[msg("The vault must retain rent and the amount must fit")]
    InsufficientFunds,
    #[msg("The vault must be empty before closing")]
    VaultNotEmpty,
}
```

Validation happens twice: `require!` in `lib.rs` entrypoints for arg checks,
`require!`/`require_keys_eq!` inside instruction methods for semantic checks.

### Seed constants

```rust
// escrow-q3-26-upstream/programs/q3_26_vault/src/constants.rs:3-7
#[constant]
pub const VAULT_SEED: &[u8] = b"vault";
#[constant]
pub const STATE: &[u8] = b"state";
```
`pre-req-vault` uses inline `b"vault"`/`b"state"` literals instead — the
constant-based version is the cleaned-up recipe.

---

## 2. PDA design recipes

| Program | PDA | Seeds | Bump |
|---|---|---|---|
| escrowq32026 | `escrow` (state) | `[b"escrow", maker, seed.to_le_bytes()]` (`make.rs:32`) | stored in `escrow.bump`, re-checked via `bump = escrow.bump` (`take.rs:33`) |
| escrowq32026 | `vault` (token acct) | ATA of `escrow` PDA — not a program PDA (`make.rs:40-43`) | n/a |
| q3_26_vault | `vault_state` | `[b"state", user]` (`initialize.rs:15`) | stored as `state_bump` |
| q3_26_vault | `vault` (SOL) | `[b"vault", user]` (`initialize.rs:22`) | stored as `vault_bump` |
| pre-req-vault | `vault` | `[b"vault", vault_state.key()]` — **seeds off the state PDA**, not user | stored |
| pre-req-vault | `application_account` | `[b"prereqs", user]` with `seeds::program = application_program.key()` — verifies a PDA owned by a foreign program (`withdraw.rs` in pre-req-vault) | `bump` (canonical) |

Recipes:
- **User-scoped state**: `seed = [label, user.key()]`. All later instructions
  re-derive with `seeds = [STATE, user.key().as_ref()], bump = vault_state.state_bump`.
- **Multi-instance state**: add a caller-chosen `u64` seed in seeds + store it in
  the account so later instructions can re-derive (`escrow.seed.to_le_bytes()`).
- **Two-PDA vault**: a data PDA (`VaultState` storing both bumps) + a
  `SystemAccount` PDA holding lamports (no data). Bumps captured once at `init`
  via `ctx.bumps` and persisted (`initialize.rs:38-39`), then reused in
  `seeds`/`bump` constraints — never `find_program_address` on-chain again.
- Derivation in tests mirrors seeds exactly:
  `Pubkey::find_program_address(&[STATE, user.as_ref()], &program_id)`
  (`q3_26_vault/tests/test_negative.rs:59-65`).

---

## 3. Token handling patterns

- **`TokenInterface` + `InterfaceAccount` everywhere** (not `Program<Token>`),
  so both SPL Token and Token-2022 work:
  `pub token_program: Interface<'info, TokenInterface>` (`make.rs:46`).
- **Mints validate their token program**:
  `#[account(mint::token_program = token_program)]` (`make.rs:14-21`).
- **ATA constraints in `#[account]`**: `associated_token::mint`,
  `associated_token::authority`, `associated_token::token_program`
  (`make.rs:22-28`, `refund.rs:14-19`).
- **PDA-owned vault as an ATA**: the escrow vault is just the ATA whose
  authority is the escrow PDA — `associated_token::authority = escrow`
  (`make.rs:41`). Token PDA vaults need no extra state account.
- **`transfer_checked`** is used for every token move (decimals passed from
  `mint.decimals`), never bare `transfer` (`make.rs:71-82`, `take.rs:60-86`).
- **Atomic swap ordering in `take`**: taker pays maker first (plain signer CPI),
  then vault releases to taker (PDA-signed CPI), then `close_account` on the
  vault returns rent to maker (`take.rs:60-97`). Refund does
  transfer-back-then-close identically (`refund.rs:51-70`).
- **Post-condition checks on ATAs** passed as unconstrained token accounts:
  `require_keys_eq!(self.taker_ata_a.mint, self.mint_a.key())` etc.
  (`take.rs:44-51`) — belt-and-suspenders when accounts aren't ATA-constrained.

---

## 4. CPI patterns

| Target | Mode | Example |
|---|---|---|
| System `transfer` (user → vault PDA) | `CpiContext::new` | `q3_26_vault/.../deposit.rs:30-35` |
| System `transfer` (vault PDA → user) | `CpiContext::new_with_signer` | `withdraw.rs:36-48` |
| Token `transfer_checked` (signer) | `CpiContext::new` | `make.rs:79-81` |
| Token `transfer_checked` / `close_account` (PDA authority) | `CpiContext::new_with_signer` | `take.rs:78-97` |
| Cross-program Anchor CPI | `declare_program!` + generated `cpi::accounts`/`cpi::initialize` | `pre-req-vault/.../withdraw.rs` |

PDA signer-seed idiom (stored-bump variant):

```rust
// escrow-q3-26-upstream/programs/q3_26_vault/src/instructions/withdraw.rs:36-48
let seeds: &[&[u8]] = &[VAULT_SEED, self.user.key.as_ref(), &[self.vault_state.vault_bump]];
let signer_seeds = [seeds];
let context = CpiContext::new_with_signer(self.system_program.key(), accounts, &signer_seeds);
transfer(context, amount)
```

Escrow rebuilds full seeds from state (`take.rs:53-58`):
`[ESCROW_SEED, escrow.maker.as_ref(), &escrow.seed.to_le_bytes(), &[escrow.bump]]`.

Cross-program CPI via `declare_program!` (pre-req-vault withdraw):

```rust
// pre-req-vault/programs/pre-req-vault/src/instructions/withdraw.rs
declare_program!(registration);
use registration::cpi::{accounts::Initialize, initialize};
...
let cpi_accounts = Initialize {
    user: self.user.to_account_info(),
    account: self.application_account.to_account_info(),
    system_program: self.system_program.to_account_info(),
};
let cpi_ctx = CpiContext::new(self.application_program.key(), cpi_accounts);
initialize(cpi_ctx, github)?;   // withdraw also registers user in external program
```

---

## 5. Security checks & Anchor constraints

- `init, payer = X, seeds = [...], bump, space = D::len() + S::INIT_SPACE` — canonical init (`make.rs:29-36`).
- `has_one = maker/mint_a/mint_b` on the escrow account binds state fields to passed accounts (`take.rs:29-31`, `update.rs:10`).
- `close = maker` / `close = user` returns state rent to the rightful owner (`take.rs:28`, `close.rs:15`).
- Seeds re-verification on every access: `seeds = [...], bump = state.bump`.
- `Signer<'info>` for authorities; `UncheckedAccount` + `/// CHECK:` doc for
  lamport-only recipients (`take.rs:13-15` maker).
- `require!` amount > 0 on deposits/withdraws/makes; `require!` on
  `amount <= vault.lamports() - rent` prevents draining rent
  (`withdraw.rs:31-34`); close requires `vault.lamports() <= rent`
  (`close.rs:31-32`).
- `Box<>` on large accounts in `Take` to save stack (`take.rs:21-37`).
- Negative space: `saturating_sub` for rent math (`withdraw.rs:33`).

---

## 6. Testing patterns

### LiteSVM (Rust integration tests under `programs/*/tests/`)

Common scaffolding (identical shape across vault + escrow tests):

```rust
// escrow-q3-26-upstream/programs/q3_26_vault/tests/test_lifecycle.rs:15-28
fn send(svm: &mut LiteSVM, payer: &Keypair, ix: Instruction) {
    svm.expire_blockhash();
    let msg = Message::new_with_blockhash(&[ix], Some(&payer.pubkey()), &svm.latest_blockhash());
    let tx = VersionedTransaction::try_new(VersionedMessage::Legacy(msg), &[payer]).unwrap();
    svm.send_transaction(tx).unwrap();
}
fn ix<T: InstructionData, A: ToAccountMetas>(program_id: Pubkey, data: T, accounts: A) -> Instruction {
    Instruction::new_with_bytes(program_id, &data.data(), accounts.to_account_metas(None))
}
```

- Program binary loaded from `include_bytes!(concat!(env!("CARGO_TARGET_TMPDIR"), "/../deploy/<name>.so"))` (`test_lifecycle.rs:38-42`) — requires `anchor build`/`cargo build-sbf` first.
- `svm.airdrop`, `svm.get_balance`, `svm.get_account(...).is_none()` for closed-account asserts.
- Instructions built via generated `program::instruction::X` + `program::accounts::X` structs — no manual serialization.
- Token setup via `litesvm_token::{CreateMint, CreateAssociatedTokenAccount, MintTo}` builders (`test_take.rs:58-91`).
- Deserialization checks: `Escrow::try_deserialize(&mut acct.data.as_ref())` and `spl_token::state::Account::unpack` (`test_take.rs:142-145`, `168-171`).

### Negative tests

Dedicated `test_negative.rs` files with a `fails()` helper returning
`send_transaction().is_err()` (`test_negative.rs:22-27`). Coverage:
zero amounts, seed reuse, wrong-signer (attacker keypair), overdraw,
re-init, wrong mint substitution, take-after-refund, close-while-nonempty
(escrow `test_negative.rs`; vault `test_negative.rs:79-287`).

### Anchor TS test (pre-req-vault only)

```ts
// pre-req-vault/tests/pre-req-vault.ts:58-66
const tx = await program.methods.initialize()
  .accountsStrict({ user, vaultState: vaultStatePda, vault: vaultPda, systemProgram: SystemProgram.programId })
  .rpc();
```
`accountsStrict` + manual `confirmTx` with `getLatestBlockhash` + `confirmTransaction`;
PDA derivation via `PublicKey.findProgramAddressSync`. Note: the Rust LiteSVM
test file there is entirely commented out (`test_initialize.rs`).

### Read-only verification script (spl-nft-q326)

`tests/verify.ts` — always exits 0; `pass`/`warn` helpers check mint supply,
ATA balances, MPL-Core asset existence on devnet without sending txs.

---

## 7. Frontend / client integration patterns

`spl-nft-q326` is pure client-side TS (no program). Two stacks:

### @solana/kit functional pipeline (SPL scripts)

```ts
// spl-nft-q326/src/spl/spl_init.ts:70-92
const msg = createTransactionMessage({ version: 0 });
const msgWithPayer = setTransactionMessageFeePayerSigner(feePayer, msg);
const msgWithLifetime = setTransactionMessageLifetimeUsingBlockhash(latestBlockhash, msgWithPayer);
const txMessage = appendTransactionMessageInstructions([createAccountIx, initializeMintIx], msgWithLifetime);
const signedTx = await signTransactionMessageWithSigners(txMessage);
```

- Instruction builders from `@solana-program/token` / `@solana-program/system`.
- `findAssociatedTokenPda` + `getCreateAssociatedTokenIdempotentInstructionAsync`
  for safe re-runnable ATA creation (`spl_mint.ts:45-58`).
- **`SEND=1` dry-run gate**: every script builds+signs, prints public plan data,
  simulates via `rpc.simulateTransaction`, and only broadcasts when
  `process.env.SEND === "1"` (`spl_init.ts:39`, `115-137`). Wallet loaded from
  JSON; key material never logged.

### Umi / Metaplex (metadata + MPL-Core NFT scripts)

```ts
// spl-nft-q326/src/nft/nft_mint.ts:36-64
const umi = createUmi(RPC).use(mplCore());
const signer = createSignerFromKeypair(umi, fromWeb3JsKeypair(Keypair.fromSecretKey(new Uint8Array(wallet))));
umi.use(keypairIdentity(signer));
const asset = generateSigner(umi);            // fresh keypair = asset address
const result = await createV1(umi, { asset, name: NAME, uri: URI }).sendAndConfirm(umi);
```

- `spl_metadata.ts` uses `createMetadataAccountV3` (mpl-token-metadata) for a
  fungible SPL token; `nft_*` uses MPL-Core `createV1`/`updateV1`/`fetchAsset`.
- Storage via `irysUploader` with env-override + fallback endpoint and a
  `getUploadPrice` probe before upload (`nft_image.ts:56-71`).
- Chained workflow: image → metadata JSON → mint → update → verify, addresses
  pasted between scripts via constants.

A fuller React + wallet-adapter frontend exists in
`solana-bootcamp/exercises/full-dapp-flow/` (`src/lib/program.ts`,
`src/hooks/useProposal.ts`, `src/providers/SolanaProvider.tsx`) but was outside
the requested project list.

---

## 8. Per-project summary

| Project | What it demonstrates |
|---|---|
| `escrow-q3-26-upstream/programs/escrowq32026` | Full SPL-token escrow: `make`/`take`/`refund`/`update`, PDA state (`[b"escrow", maker, seed]`), ATA vault owned by PDA, `transfer_checked` + `close_account` CPIs with signer seeds, `close =` rent reclaim, Token-2022-compatible `TokenInterface`, extensive LiteSVM positive + negative tests. |
| `escrow-q3-26-upstream/programs/q3_26_vault` | Clean SOL vault: two PDAs (`[b"state", user]` storing both bumps + `[b"vault", user]` `SystemAccount`), system `transfer` CPI in/out with `new_with_signer`, rent-exempt floor enforcement, close-only-when-empty, lifecycle + 6 negative LiteSVM tests. The polished reference version. |
| `vault-reference` | Earlier/simplified draft of the same vault (seeds `[b"vault", user]`), `withdraw`/`close` commented out, less validation. Useful as a "before" comparison. |
| `pre-req-vault` | Bootcamp prerequisite variant: vault PDA seeded off the *state account* (`[b"vault", vault_state]`), and a cross-program CPI via `declare_program!(registration)` — `withdraw` also registers a `github` handle in an external program PDA `[b"prereqs", user]`. Has the only Anchor TS test (`accountsStrict`). Rust LiteSVM test commented out. |
| `spl-nft-q326` | Client-only SPL Token + Metaplex scripts: mint init/mint-to/transfer with `@solana/kit`, metadata via mpl-token-metadata `createMetadataAccountV3`, NFT via MPL-Core `createV1`/`updateV1`, Irys uploads with endpoint fallback, `SEND=1` dry-run safety gate, read-only `verify.ts` checker. |
