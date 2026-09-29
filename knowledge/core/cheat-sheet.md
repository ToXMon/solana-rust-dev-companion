# Solana / Anchor Cheat Sheet

Quick reference for the most common patterns, program IDs, and pitfalls.

## Essential program IDs

| Program | Address |
|---------|---------|
| System Program | `11111111111111111111111111111111` |
| SPL Token | `TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA` |
| Token-2022 | `TokenzQdBNbW9ZV7FPi1XEG4J8t75VmvFwAPc2gUwzbqb` |
| Associated Token Account | `ATokenGPvbdGVxr1b2hvZbsiqW5xWH25efZZEU9W6eB9v` |
| MPL Core | `CoREENxT6tW1HoK8ypY1SxRMZTcVPm7R94rH4PZNhX7d` |
| ZK ElGamal Proof | `ZkE1Gama1Proof11111111111111111111111111111` |

## Account fields

```rust
pub struct Account {
    pub lamports: u64,      // balance
    pub data: Vec<u8>,      // owner-defined bytes
    pub owner: Pubkey,      // only owner can mutate
    pub executable: bool,   // true for programs
    pub rent_epoch: u64,    // legacy
}
```

## Anchor account types

| Type | Use it for |
|------|------------|
| `Account<'info, T>` | Program-owned account of type `T` |
| `Signer<'info>` | Transaction signer |
| `SystemAccount<'info>` | System Program-owned account (wallet) |
| `UncheckedAccount<'info>` | Account you manually validate; needs `/// CHECK:` doc |
| `AccountLoader<'info, T>` | Large account, zero-copy access |
| `Interface<'info, T>` | Accepts SPL Token or Token-2022 |
| `Box<Account<'info, T>>` | Heap allocation to avoid stack overflow |

## Common constraints

```rust
#[derive(Accounts)]
pub struct InitVault<'info> {
    #[account(mut)]
    pub payer: Signer<'info>,

    #[account(
        init,
        payer = payer,
        seeds = [b"state", user.key().as_ref()],
        bump,
        space = 8 + VaultState::INIT_SPACE
    )]
    pub vault_state: Account<'info, VaultState>,

    #[account(
        seeds = [b"vault", user.key().as_ref()],
        bump
    )]
    pub vault: SystemAccount<'info>,

    pub system_program: Program<'info, System>,
}
```

## PDA signer seeds

```rust
let seeds: &[&[u8]] = &[b"vault", user.key().as_ref(), &[vault_state.vault_bump]];
let signer_seeds = [seeds];
let cpi_ctx = CpiContext::new_with_signer(
    self.system_program.to_account_info(),
    Transfer { from: self.vault.to_account_info(), to: self.user.to_account_info() },
    &signer_seeds,
);
transfer(cpi_ctx, amount)?;
```

## Token CPI patterns

**Create ATA:** use `anchor_spl::associated_token::Create` CPI or client-side helper.

**Transfer checked:**

```rust
let cpi_ctx = CpiContext::new(
    self.token_program.to_account_info(),
    TransferChecked {
        from: self.from_ata.to_account_info(),
        mint: self.mint.to_account_info(),
        to: self.to_ata.to_account_info(),
        authority: self.authority.to_account_info(),
    },
);
transfer_checked(cpi_ctx, amount, self.mint.decimals)?;
```

**PDA-owned vault token account:** the ATA's authority is the PDA, and the program signs with seeds.

## Arithmetic safety

```rust
vault.balance = vault.balance.checked_add(amount).ok_or(ErrorCode::Overflow)?;
```

## Error patterns

```rust
#[error_code]
pub enum ErrorCode {
    #[msg("Amount must be greater than zero")]
    InvalidAmount,
    #[msg("Overflow")]
    Overflow,
}

require!(amount > 0, ErrorCode::InvalidAmount);
```

## Transaction limits

- Max raw tx size: ~1232 bytes
- Max compute units per tx: 1.4M (default 200k if not set)
- Priority fee: set via Compute Budget Program

## Devnet commands

```bash
solana config set --url devnet
solana airdrop 2
anchor test --provider.cluster devnet
anchor deploy --provider.cluster devnet
```

## Common pitfalls

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| `Account not initialized` | Wrong PDA seeds or bump | Re-derive with same seeds, store canonical bump |
| `missing mut` | Account changes data/lamports but `mut` is missing | Add `#[account(mut)]` |
| `Transfer failed` | Wrong decimals or using `transfer` instead of `transfer_checked` | Use `transfer_checked` with mint decimals |
| `Signer privilege escalated` | Using `new` instead of `new_with_signer` for PDA | Use `CpiContext::new_with_signer` |
| `Stack overflow` | Too many accounts on stack | Box large account types |
| `Instruction did not deserialize` | Discriminator mismatch | Check account discriminator / IDL version |
| Test passes locally but fails on devnet | Devnet airdrop / blockhash / account not created | Check transaction signatures on Explorer |

## Verification checklist

- [ ] Local tests pass
- [ ] Devnet tests pass
- [ ] Program ID captured
- [ ] At least one devnet tx signature captured
- [ ] Explorer link works
- [ ] Security review done
