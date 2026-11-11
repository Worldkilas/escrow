# Token Escrow

A Solana program for trust-minimized, atomic swaps between two SPL tokens. A maker deposits token A into a program-controlled vault and specifies how much token B they expect. A taker can complete the swap, or the maker can reclaim the deposited tokens.

The program is built with [Anchor](https://www.anchor-lang.com/) and uses Anchor's token interface so the instructions can work with compatible token programs.

## How it works

1. **Make:** The maker creates an escrow PDA and deposits token A into its associated token account. The escrow records the maker, both mints, the requested token B amount, a seed, and an expiration timestamp.
2. **Take:** A taker supplies token B and receives the token A held in the vault. On success, the escrow and vault are closed.
3. **Refund:** The maker returns token A from the vault to their associated token account. On success, the escrow and vault are closed.

The escrow PDA is derived from the `escrow` seed, the maker's public key, and a caller-provided `u64` seed. This allows a maker to create multiple escrows.

## Program details

| Property | Value |
| --- | --- |
| Program name | `escrowq32026` |
| Program ID | `5KNduHpBhC9cMc6CP5rpR6FUJajwcJMT4Rabay12fjc` |
| Framework | Anchor 1.1.2 |
| Rust edition | 2021 |
| Rust version | 1.89.0 or newer |

## Repository layout

```text
programs/escrowq32026/
├── src/
│   ├── instructions/
│   │   ├── make.rs
│   │   ├── take.rs
│   │   └── refund.rs
│   ├── constants.rs
│   ├── error.rs
│   ├── lib.rs
│   └── state.rs
└── tests/
    └── mod.rs

arch/                       # Instruction flow diagrams
```

## Prerequisites

- Rust 1.89.0 or newer (the repository includes a `rust-toolchain.toml`)
- Solana CLI and Anchor CLI compatible with Anchor 1.1.2
- An installed Rust target/toolchain capable of building Solana programs if building the on-chain artifact

## Build and test

Build the Anchor program:

```sh
anchor build
```

Run the Rust test suite:

```sh
cargo test
```

The current LiteSVM test covers creating an escrow and refunding it. It does not yet test completing a swap with `take`.

## Instructions

### `make`

Creates an escrow and deposits token A from the maker's associated token account into the escrow vault.

Arguments:

| Argument | Type | Description |
| --- | --- | --- |
| `seed` | `u64` | Per-maker value used to derive the escrow PDA |
| `deposit` | `u64` | Token A amount to deposit, in the mint's base units |
| `receive` | `u64` | Token B amount requested by the maker, in the mint's base units |
| `expiration` | `i64` | Expiration timestamp stored in the escrow |

The maker must authorize the instruction and have enough token A to cover the deposit. The associated token account for the escrow vault is initialized as part of the instruction.

### `take`

Completes a swap and closes the escrow and vault. The escrow account contains the seed needed to validate its PDA, so `take` has no instruction arguments.

The taker must authorize the instruction and provide the required token accounts. The expected behavior is for the taker to pay the recorded token B amount to the maker and receive the token A deposit.

### `refund`

Returns the token A remaining in the vault to the maker and closes the escrow and vault. Only the maker can authorize a refund.

## Current status and limitations

This repository is an in-progress program, not production-ready:

- The `take` instruction is implemented but is not covered by the current tests. Review and test its token-transfer amounts and mint/account wiring before use.
- `expiration` is stored in the escrow, but the current instruction logic does not enforce an expiration condition.
- The recorded `receive` amount is not currently enforced by the instruction logic.

Do not use the program with valuable tokens until these behaviors are implemented or verified and the program has been thoroughly tested and independently reviewed.

## Instruction diagrams

See the diagrams in [`arch/`](arch/):

- [Make escrow](arch/make.png)
- [Take escrow](arch/take.png)
- [Refund escrow](arch/refund.png)
