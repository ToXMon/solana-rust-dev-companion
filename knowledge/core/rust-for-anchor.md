---
title: "Rust for Anchor"
description: "The Rust concepts behind Anchor. Each topic takes something you write in Anchor and explains the Rust underneath it."
tags: ["rust", "anchor", "solana", "structs", "traits", "macros", "ownership", "borrowing", "generics", "lifetimes", "error-handling", "enums", "option"]
---

# Rust for Anchor

The Rust concepts behind Anchor. Each topic takes something you write in Anchor and explains the Rust underneath it.

- Anchor docs: <https://www.anchor-lang.com/docs>
- The Rust Book: <https://doc.rust-lang.org/book>

## Table of Contents

- [0. The one mental shift (start here)](#0-the-one-mental-shift-start-here)
- [1. Structs and types](#1-structs-and-types)
- [2. Traits](#2-traits)
- [3. Macros](#3-macros)
- [4. Ownership and borrowing](#4-ownership-and-borrowing)
- [5. Generics and lifetimes](#5-generics-and-lifetimes)
- [6. Error handling](#6-error-handling)
- [7. Enums, Option, and pattern matching](#7-enums-option-and-pattern-matching)

## 0. The one mental shift (start here)

Not a docs page. This is the model everything else hangs on.

- **Anchor is a set of Rust macros, not a new language.** You write plain Rust (`struct`s, `fn`s, attributes) and Anchor's macros expand them at compile time into the raw account-checking and serialization code you would otherwise write by hand.
- **The attributes are not magic.** `#[account]`, `#[program]`, `#[derive(Accounts)]` are Rust macros. Before your code compiles, they rewrite themselves into ordinary Rust.
- **You write what, Rust generates how.** You declare "this account must be mutable and owned by my program." The macro writes the checks that enforce it.
- **It is still Solana underneath.** Every concept from Solana core (accounts, instructions, owners, rent) is still there. Anchor just stops you from hand-writing the boilerplate for each one.

> **Say out loud:** "Anchor is to raw Solana what a web framework is to writing HTTP by hand. The macros are the framework."

### Learner check

1. What is Anchor, in terms of Rust?
2. Do Anchor's attributes turn into real code at compile time or at runtime?
3. Finish the sentence: "In Anchor you declare what an account must be; the macro generates the ___ that checks it."

## 1. Structs and types

- Docs: <https://doc.rust-lang.org/book/ch05-00-structs.html>

A `struct` is how Rust groups related data under one name. Your account data is a `struct`.

```rust
#[account]
pub struct Vault {
    pub owner: Pubkey,
    pub balance: u64,
    pub bump: u8,
}
```

### The pieces

| Term | Meaning |
|------|---------|
| `struct` | A custom type that bundles named fields. `Vault` is the type; `owner`, `balance`, `bump` are its fields. |
| `pub` | Short for public. Makes the `struct` and each field visible outside the module. Without it, other code can not read the field. |

### Field types

| Type | Size / Use |
|------|------------|
| `Pubkey` | 32-byte address. The type Solana and Anchor use for any wallet or account key. |
| `u64` | Unsigned 64-bit integer, 0 up to about `1.8e19`. Balances, amounts, counters. |
| `u8` | Unsigned 8-bit integer, 0 to 255. One byte. A bump is a `u8`. |
| `bool`, `i64`, `[u8; 32]`, `String`, `Vec<T>` | Other types you will hit. The fixed-size ones are cheapest. |

### Why Anchor cares about size

The account's `data` field is a raw byte buffer (core concepts, topic 1). To create the account, Anchor has to reserve exactly the right number of bytes and pay rent for them. It gets that number by adding up the size of every field, so a `String` or `Vec` (which can grow) needs you to declare how much room to leave.

### Learner check

1. What is a `struct`, and what is a field?
2. What does `pub` do, and what breaks without it?
3. Why does Anchor need to know the exact size of each field type?
4. Which type holds a 32-byte address?

## 2. Traits

- Docs: <https://doc.rust-lang.org/book/ch10-02-traits.html>

A **trait** is a set of behavior a type promises to provide. If you know interfaces from TypeScript or Java, that is the closest match. A trait says "any type that implements me can do these methods."

### The pieces

- **trait** — a named contract of methods. Example: a `Discriminator` trait requires a method that returns the type's 8-byte tag.
- **implementing a trait** — writing the actual methods for a specific type. "`Vault` implements `Discriminator`."

### Why Anchor lives on traits

To load your `Vault` out of an account's raw bytes, the type has to know how to serialize (`struct` to bytes) and deserialize (bytes to `struct`). Those abilities are traits: `AccountSerialize` and `AccountDeserialize`. Anchor also needs `Owner` (which program owns this) and `Discriminator` (the 8-byte type tag that lets it tell one account `struct` from another).

You do not write those by hand. That is what `#[account]` did back in topic 1. The macro implemented all of those traits for `Vault` so you did not have to. That is the link between traits and macros: a macro's job is often to write trait implementations you would rather not.

> **Say out loud:** "A trait is an interface. Anchor requires a handful of them, and the macros implement them for you."

### Learner check

1. What is a trait, in one sentence, using a word a TypeScript dev already knows?
2. Name two things Anchor needs your account type to be able to do (two traits).
3. Who writes the trait implementations for your `#[account]` `struct`?

## 3. Macros

- Docs: <https://doc.rust-lang.org/book/ch19-06-macros.html>

A **macro** is code that writes code. It runs at **compile time**, takes the tokens you wrote, and expands into more Rust before the real compiler sees it. Anchor is mostly macros.

### Two shapes you will see

**Attribute macros** — written as `#[name]` above an item. They take the thing below them and rewrite it.

- `#[program]` — marks the module holding your instruction handlers. Expands into the dispatch code that routes an incoming instruction to the right function.
- `#[account]` — marks a data `struct`. Adds the 8-byte discriminator and implements the serialization and owner traits from topic 2.

**Derive macros** — written as `#[derive(...)]`. They generate trait implementations for the `struct` below.

- `#[derive(Accounts)]` — the big one. You put it on your context `struct` and it generates the account-validation code: is this mutable, is it the right owner, does this PDA match these seeds.

```rust
#[derive(Accounts)]
pub struct Deposit<'info> {
    #[account(mut)]
    pub vault: Account<'info, Vault>,
    #[account(mut)]
    pub user: Signer<'info>,
    pub system_program: Program<'info, System>,
}
```

The inner `#[account(mut)]` lines are attribute helpers the derive reads. `mut` means "this instruction will write to this account," which becomes an `is_writable` check.

Expanded, this `struct` turns into dozens of lines of "load this account, check its owner, confirm it is writable, deserialize it." You wrote nine lines.

### Why it matters

Almost everything that looks like Anchor magic is a macro expansion. When you want to see the real generated Rust, `cargo expand` prints it.

### Learner check

1. What does a macro do, and when does it run?
2. What is the difference between an attribute macro and a derive macro?
3. What does `#[derive(Accounts)]` generate?
4. What is `#[account(mut)]` actually asking for, in Solana terms?

## 4. Ownership and borrowing

- Docs: <https://doc.rust-lang.org/book/ch04-00-understanding-ownership.html>

This is the Rust idea with no equivalent in most languages, and the one the compiler enforces hardest. It is how Rust guarantees memory safety with no garbage collector.

### The three rules, plain

1. Every value has exactly one owner, the variable that holds it.
2. When the owner goes out of scope, the value is freed. No manual free, no GC pass.
3. You can lend a value out with a reference instead of giving it away. That is borrowing.

### Borrowing has two forms

| Form | Meaning | Allowed count |
|------|---------|---------------|
| `&T` | Shared, read-only borrow. | Many readers at once. |
| `&mut T` | Exclusive, read-write borrow. | Exactly one at a time, and no shared borrows while it is out. |

The rule the borrow checker enforces: **many readers or one writer, never both.** That is what stops data races at compile time.

### Where you meet it in Anchor

```rust
pub fn deposit(ctx: Context<Deposit>, amount: u64) -> Result<()> {
    let vault = &mut ctx.accounts.vault;
    vault.balance += amount;
    Ok(())
}
```

`&mut ctx.accounts.vault` — you take an exclusive borrow because you are about to change `balance`. If you were only reading it, `&` would do.

You rarely fight the borrow checker in day-one Anchor code, but this is why `&` and `&mut` are everywhere, and why the compiler sometimes refuses two mutable borrows of the same account at once.

> **Say out loud:** "Rust has no garbage collector. Ownership and borrowing are the rules that replace it, checked at compile time."

### Learner check

1. State the one-owner rule.
2. What is the difference between `&T` and `&mut T`, and how many of each can exist at once?
3. In `let vault = &mut ctx.accounts.vault;`, why `&mut` and not `&`?
4. What does Rust use ownership in place of?

## 5. Generics and lifetimes

- Docs: <https://doc.rust-lang.org/book/ch10-01-syntax.html> and <https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html>

Two things share the `<...>` brackets and trip up beginners together, so learn them together. You see both in `Context<'info, T>` and `Account<'info, T>`.

### Generics — a type with a hole in it

A **generic** lets one definition work for many types. `Account<'info, T>` means "an account holding some type `T`," where `T` is your `Vault`, or a `Mint`, or anything else.

The compiler fills the hole. When you write `Account<'info, Vault>`, `T` becomes `Vault` and you get all of `Vault`'s fields, fully type-checked.

This is why Anchor ships one `Account` wrapper that works for every account `struct` you will ever define. That is the payoff: one implementation, many types, no loss of type safety.

### Lifetimes — a label for how long a reference is valid

A **lifetime** like `'info` is a promise to the compiler, not a piece of data: "this reference lives at least this long."

Anchor loads all your accounts' bytes at the start of the instruction. Your `Account<'info, Vault>` holds a reference into that loaded data. The `'info` lifetime ties every account reference to the span of the instruction, so the compiler can prove no account reference outlives the data it points at.

You rarely invent lifetimes yourself. You mostly copy `'info` into the spots where Anchor's types ask for it. Knowing what it means keeps the errors from looking terrifying.

### Reading the signature

In `Account<'info, Vault>`:

- `'info` is the lifetime (how long the account data is valid).
- `Vault` is the generic type (which `struct` lives in the account).

> **Say out loud:** "Generics are holes for types. Lifetimes are labels for how long references live. `Account<'info, T>` uses both."

### Learner check

1. What does a generic let you avoid writing?
2. In `Account<'info, Vault>`, what is the `T` and what is the `'info`?
3. Is a lifetime a piece of data? If not, what is it?
4. Why does Anchor tie account references to the `'info` lifetime?

## 6. Error handling

- Docs: <https://doc.rust-lang.org/book/ch09-00-error-handling.html>

Rust has no exceptions. A function that can fail says so in its return type, using `Result`. Every Anchor handler returns one.

### The pieces

- **`Result<T, E>`** — an enum with two variants: `Ok(T)` for success carrying a value, or `Err(E)` for failure carrying an error. You return it rather than throw it, and you have to handle it.
- In Anchor, handlers return `Result<()>`. The `()` is the unit type, meaning "no meaningful value, just success or failure." `Ok(())` is the normal happy ending.

### The `?` operator

The workhorse. Put it after any call that returns a `Result`. If the call is `Ok`, `?` unwraps the value and continues. If it is `Err`, `?` returns that error out of your function right away. It is early-return-on-error in one character.

```rust
pub fn deposit(ctx: Context<Deposit>, amount: u64) -> Result<()> {
    require!(amount > 0, VaultError::ZeroAmount);
    let vault = &mut ctx.accounts.vault;
    vault.balance = vault.balance
        .checked_add(amount)
        .ok_or(VaultError::Overflow)?;
    Ok(())
}
```

- `require!(cond, Error)` — an Anchor macro. If the condition is false, it returns your error. Same idea as a guard clause.
- `.checked_add(...)` returns an `Option` (topic 7). `.ok_or(err)?` turns a `None` into your error and `?` propagates it. That is how you avoid a silent overflow instead of crashing.

### Your own errors

```rust
#[error_code]
pub enum VaultError {
    #[msg("Amount must be greater than zero")]
    ZeroAmount,
    #[msg("Balance overflow")]
    Overflow,
}
```

`#[error_code]` — a macro that turns a plain enum into Anchor errors, each with a code and the message you give in `#[msg(...)]`.

> **Say out loud:** "No exceptions. Failure is a value you return, and `?` passes it up the chain."

### Learner check

1. What are the two variants of `Result`?
2. What does `Ok(())` mean, and what is the `()`?
3. In plain words, what does `?` do when it hits an `Err`?
4. What does `#[error_code]` turn a plain enum into?

## 7. Enums, Option, and pattern matching

- Docs: <https://doc.rust-lang.org/book/ch06-00-enums.html>

The last core Rust piece, and the one that makes `Result` and `Option` work.

### The pieces

- **enum** — a type that is exactly one of a fixed set of variants. `Result` is an enum (`Ok` or `Err`). Your error type was an enum. Instruction routing under the hood is an enum.
- **`Option<T>`** — Rust's answer to null, and it is an enum too: `Some(T)` when there is a value, `None` when there is not. There is no `null` in Rust. A maybe-missing value has to be an `Option`, so you cannot forget to handle the empty case.
- **match** — checks a value against each variant and forces you to handle them. The compiler will not accept a `match` that misses a case. That exhaustiveness is the whole point.

```rust
match maybe_bump {
    Some(b) => msg!("bump is {}", b),
    None => msg!("no bump stored"),
}
```

Anchor uses `Option<T>` for optional accounts and for values that may not be set. When `.checked_add()` handed you an `Option` back in topic 6, that was the same machinery.

You will often reach for shortcuts (`if let`, `.ok_or()`, `.unwrap_or()`) instead of a full `match`, but they are all built on the same enum.

### Now go build

- **The toolchain:** `anchor init`, `anchor build`, `anchor test`. Anchor's CLI wraps the Solana CLI from core concepts topic 7.
- **`cargo expand`** when something feels like magic and you want to read the Rust the macros produced.
- Everything from Solana core concepts is still true underneath. Anchor is the Rust layer that writes the boilerplate for you.

### Learner check

1. What is an enum?
2. What are the two variants of `Option`, and what does Rust use it in place of?
3. What does `match` force you to do that an `if-else` does not?
4. Name one place Anchor hands you an `Option`.
