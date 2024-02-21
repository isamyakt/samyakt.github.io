---
title: Airdrop and Transaction in Solana Devnet using rust
description: To write Client-Side application we'll use `solana-sdk` crate & for communication with a solana node over RPC we'll use `solana-client`
date: "2023-09-08"
---

# Environment Setup
- **Rust**, You can download rust from their [website](https://www.rust-lang.org/learn/get-started)
- cargo comes with rust, so you don't have to worry about it.
- Initialize the Project
```zsh
$ cargo new --bin solana-client-rs
```

- **solana-sdk** & **solana-client** can be installed via
```zsh
$ cargo add solana-sdk
$ cargo add solana-client
```
- install **tokio** for writing asynchronous I/O
```zsh
$ cargo add tokio -F full
```
- **dotenvy** for environment variables
```zsh
$ cargo add dotenvy
```

|| OR ||
- open `Cargo.toml` to add dependencies
```
[dependencies]
dotenvy = "0.15.7"
solana-client = "1.18.2"
solana-sdk = "1.18.2"
tokio = { version = "1.36.0", features = ["full"] }
```

# Folder Structure
- first we'll focus on `main.rs` file and try to initialize keypair, airdrop and make transaction.
- the we'll focus on making folder structure.

```
.
├── src
│   ├── main.rs
│   ├── keypair.rs
│   ├── balance.rs
│   ├── airdrop.rs
│   └── transaction.rs
├── .gitignore
├── Cargo.lock
└── Cargo.toml
```

# First only focus on `main.rs`
- filename: `src/main.rs`
