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
```zsh
use dotenvy::dotenv;
use solana_sdk::signer::keypair::Keypair;
use std::path::Path;
use std::env;
use std::fs::{File, OpenOptions};
use std::io::Write;

async fn initialize_keypair() -> Keypair {
    let file_path = ".env";

    // Check if the file exists
    let file_exists = Path::new(file_path).exists();

    if !file_exists {
        File::create(file_path).unwrap();
    }

    match env::var("PRIVATE_KEY") {
        Ok(_) => {
            let env_private_key = env::var("PRIVATE_KEY").unwrap();
            let trim_private_key = env_private_key
                .trim_matches(|c| c == '[' || c == ']' || c == '"');

            let vec_private_key: Vec<u8> = trim_private_key
                .split(",")
                .map(| s| s.trim().parse().unwrap())
                .collect();

            let bytes_private_key: [u8; 64] = vec_private_key
                .try_into()
                .expect("Expected a Vec of length 64");

            let keypair = Keypair::from_bytes(&bytes_private_key).unwrap();

            keypair
        },
        Err(_) => {
            let keypair = Keypair::new();
            let keypair_bytes = keypair.to_bytes();

            let mut file = OpenOptions::new()
                .write(true)
                .append(true)
                .open(".env")
                .unwrap();
            
            writeln!(file, "PRIVATE_KEY=\"{:?}\"", keypair_bytes).unwrap();

            keypair
        }
    }
}

#[tokio::main]
async fn main() {
    dotenv().ok();

    let keypair = initialize_keypair().await;

    println!("{:?}\n", keypair);
}
```
- Imports necessary crates: This includes modules for environment variable management, file handling, and Solana keypair functionality.
- initialize_keypair function: This asynchronous function is responsible for initializing a Solana keypair.
- It first checks if a `.env` file exists in the current directory. If not, it creates one.
- It then attempts to read a private key from an environment variable `PRIVATE_KEY`. If the variable exists, it `parses` the private key from a string into a `byte array` and creates a `keypair` from it.
- If the `PRIVATE_KEY` environment variable does not exist, it generates a new `keypair`, converts it to a `byte array`, and writes it to the .env file.
