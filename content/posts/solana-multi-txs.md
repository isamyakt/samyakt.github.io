---
title: Solana - Sending multiple transaction in single blockhash using rust
description: Sending multiple transfer intructions (ix) to single transaction (tx) by using `solana-sdk` crate For Client-Side application & `solana-client` for communication with a solana node over RPC
date: "2023-09-26"
---

# Environment Setup
- **Rust**, You can download rust from their [website](https://www.rust-lang.org/learn/get-started)
- cargo comes with rust, so you don't have to worry about it.
- Initialize the Project
```zsh
$ cargo new --bin solana-multi-txs
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

# Note: 

- Before going through this complicated tranasactions understand my previous blog about [airdrop & transaction](https://isamyakt.github.io/website/posts/solana-airdrop-tx/)

# First thing first: Initialize Keypair
- filename: `src/main.rs`
```zsh
use dotenvy::dotenv;
use std::env;
use std::fs::{File, OpenOptions};
use std::path::Path;
use std::io::Write;

use solana_sdk::signature::Keypair;

// region:    --- Keypair

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

// endregion: --- Keypair

#[tokio::main]
async fn main() {
    dotenv().ok();

    let keypair = initialize_keypair().await;

    println!("{:?}\n", keypair);
}

```
open terminal and run command `cargo run`

Output:
```bash
Keypair(Keypair { secret: SecretKey: [169, 33, 219, 109, 248, 202, 18, 201, 9, 247, 83, 133, 224, 172, 121, 44, 53, 69, 20, 73, 66, 108, 154, 50, 235, 58, 204, 240, 13, 99, 154, 1], public: PublicKey(CompressedEdwardsY: [173, 83, 149, 245, 30, 168, 154, 80, 60, 159, 234, 145, 233, 45, 223, 139, 134, 180, 5, 20, 213, 124, 35, 163, 250, 94, 225, 83, 119, 214, 70, 61]), EdwardsPoint{
        X: FieldElement51([569245570616498, 2003359860844829, 2125784894344511, 735460460889380, 912158005314796]),
        Y: FieldElement51([747800876110765, 1462613865630227, 25091638262967, 2108114127973002, 1077991282392597]),
        Z: FieldElement51([1, 0, 0, 0, 0]),
        T: FieldElement51([1301069849558026, 1235770193232742, 738204870145171, 1637446764606588, 1392849794349830])
}) })
```
Dependencies used to generate/extract existing keypair
- `dotenvy::dotenv`: This is a dependency used for loading environment variables from a .env file.
- `std::env`: This module provides access to environment variables.
- `std::fs::{File, OpenOptions}`: These modules are used for file operations.
- `std::path::Path`: This module provides facilities to work with file paths.
- `std::io::Write`: This trait provides the write method to write bytes to a file.
- `solana_sdk::signature::Keypair`: This is a type representing a cryptographic key pair used for signing transactions in Solana blockchain.

`initialize_keypair()` Function :
This asynchronous function initializes a cryptographic key pair. Let's break it down:
```zsh
async fn initialize_keypair() -> Keypair {
    let file_path = ".env";

    // Check if the file exists
    let file_exists = Path::new(file_path).exists();

    if !file_exists {
        File::create(file_path).unwrap();
    }
```
- It initializes the file path where the private key will be stored as `.env`.
- Checks if the file exists, and if not, creates an empty file.
```zsh
    match env::var("PRIVATE_KEY") {
        Ok(_) => {
            // Logic for handling existing private key in environment variables
        },
        Err(_) => {
            // Logic for generating and saving a new private key
        }
    }
```
- It checks if an environment variable named `PRIVATE_KEY` is set.
- If the `PRIVATE_KEY` environment variable is found, it parses the key, constructs a keypair, and returns it.
- If the `PRIVATE_KEY` environment variable is not found, it generates a new `Keypair`, saves it to the `.env` file, and returns the generated `Keypair`.

# Now, let's focus on Airdrop and Checking Balance
- filename : `src/main.rs`
```zsh
use solana_client::rpc_client::RpcClient;
use solana_sdk::pubkey::Pubkey;
use solana_sdk::signature::Signature;
use solana_sdk::native_token::LAMPORTS_PER_SOL;
use solana_sdk::signer::Signer;

// region:    --- Keypair···

// region:    --- Balance

async fn get_balance_in_sol(client: &RpcClient, pubkey: &Pubkey) -> f64 {
    let lamports = LAMPORTS_PER_SOL as f64;
    let balance = client.get_balance(&pubkey).unwrap() as f64;

    balance / lamports
}

async fn get_balance_in_lamports(client: &RpcClient, pubkey: &Pubkey) -> u64 {
    let balance = client.get_balance(&pubkey).unwrap();
    balance
}

// endregion:    --- Balance

// region:    --- Airdrop

async fn airdrop_possible(client: &RpcClient, pubkey: &Pubkey) -> bool {
    let balance = get_balance_in_lamports(client, pubkey).await;

    let result = !(balance > LAMPORTS_PER_SOL);
    result
}

async fn airdrop(client: &RpcClient, pubkey: &Pubkey) -> Option<Signature> {
    let airdrop_available = airdrop_possible(client, pubkey).await;

    if airdrop_available  {
        let recent_blockhash = client
            .get_latest_blockhash()
            .unwrap();

        let lamports = 1000000000;

        let airdrop_sig = client
            .request_airdrop_with_blockhash(&pubkey, lamports, &recent_blockhash)
            .unwrap();

        return Some(airdrop_sig);
    } 

    None
} 

// endregion: --- Airdrop
```
Let's update the `main` function :
```zsh
#[tokio::main]
async fn main() {
    dotenv().ok();

    let keypair = initialize_keypair().await;

    // println!("{:?}\n", keypair);

    let public_key = keypair.pubkey();

    let url = String::from("https://api.devnet.solana.com");
    let client = RpcClient::new(url); 

    let airdrop = airdrop(&client, &public_key).await;

    if let Some(sig) = airdrop {
        println!("airdrop signature : {}\n", sig);
    }
}
```
open terminal and run command `cargo run`

Output:
```bash
airdrop signature : 4QejmCBPByoVzKySYHcTDeBVFyox8UjVcWYP9doB3fKiPSXFg5zizamz9Cds88tQbduGTbD5g1GBQJNiwWDH2pVn
```
Dependencies used to get balance and airdrop SOL tokens
- `solana_client::rpc_client::RpcClient`: This module provides a client for interacting with the Solana RPC (Remote Procedure Call) API.
- `solana_sdk::pubkey::Pubkey`: This module provides types and utilities for dealing with public keys on the Solana blockchain.
- `solana_sdk::signature::Signature`: This module provides types and utilities for handling cryptographic signatures.
- `solana_sdk::native_token::LAMPORTS_PER_SOL`: This constant represents the number of lamports in one SOL (the native token of the Solana blockchain).
- `solana_sdk::signer::Signer`: This module provides traits for types that can sign messages

Balance Functions :

```zsh
async fn get_balance_in_sol(client: &RpcClient, pubkey: &Pubkey) -> f64 { ... }
```
- This asynchronous function retrieves the balance of an account in SOL and returns it as a floating-point number.
```zsh
async fn get_balance_in_lamports(client: &RpcClient, pubkey: &Pubkey) -> u64 { ... }
```
- This asynchronous function retrieves the balance of an account in lamports and returns it as a u64 integer.

Airdrop Functions :
```zsh
async fn airdrop_possible(client: &RpcClient, pubkey: &Pubkey) -> bool { ... }
```
- This asynchronous function checks if an airdrop is possible for the given account by comparing its balance with a threshold (here, the value of `LAMPORTS_PER_SOL`).
```zsh
async fn airdrop(client: &RpcClient, pubkey: &Pubkey) -> Option<Signature> { ... }
```
- This asynchronous function attempts to initiate an airdrop to the given account if it's possible (determined by `airdrop_possible` function).
- It requests an airdrop of 1 SOL (converted to lamports) to the specified account using the Solana RPC client.
- If the airdrop is successful, it returns the signature of the transaction; otherwise, it returns `None`

Main Function :
```zsh
#[tokio::main]
async fn main() { ... }
```
- This is the entry point of the program, marked with `#`[`tokio::main`] to indicate asynchronous execution.
- It loads environment variables, initializes a keypair (which is assumed to be done elsewhere, perhaps as in your previous code), gets the public key, and creates an RPC client for the Solana network.
- It attempts to perform an airdrop to the account and prints the signature if the airdrop is successful.

# Now, let's perform multiple Transactions
- filename: `src/main.rs`
```zsh
use std::str::FromStr;

use solana_sdk::transaction::Transaction;
use solana_sdk::system_instruction;

// region:    --- Mutli-Tx

async fn create_multi_tx_account(
    client: &RpcClient,
    sender: &Keypair,
    recievers: &[&Pubkey],
    amount: &[u64],
) -> Option<Signature> {
    let first_transfer_ix = system_instruction::transfer(
        &sender.pubkey(), 
        recievers[0], 
        amount[0]
    );

    let second_transfer_ix = system_instruction::transfer(
        &sender.pubkey(), 
        recievers[1], 
        amount[1]
    );

    let third_transfer_ix = system_instruction::transfer(
        &sender.pubkey(), 
        recievers[2], 
        amount[2]
    );

    let fourth_transfer_ix = system_instruction::transfer(
        &sender.pubkey(), 
        recievers[3], 
        amount[3]
    );

    let recent_blockhash = client.get_latest_blockhash().unwrap();

    let tx = Transaction::new_signed_with_payer(
        &[first_transfer_ix, second_transfer_ix, third_transfer_ix, fourth_transfer_ix], 
        Some(sender.pubkey()).as_ref(), 
        &[sender], 
        recent_blockhash
    );

    let sig = client.send_and_confirm_transaction(&tx).unwrap();

    Some(sig)
}

async fn multi_tx(client: &RpcClient, keypair: &Keypair) {
    // change the public key if you want
    let recievers = [
        &Pubkey::from_str("7QnSXgoZHi9FGCwaziaEMsUtmWZUbuvg3qq5UCGVJFat").unwrap(),
        &Pubkey::from_str("9o7acD8UP8DDKEDZ1LFzuajC7bwG2WZJXRdG1i5FAfD3").unwrap(),
        &Pubkey::from_str("G2MeMHLr84SbTWVfBj7HSLqPLNQqmR9T8Mkepxi2Ag8V").unwrap(),
        &Pubkey::from_str("EtGf3KRUT2R21mAPCyZBXb7GFQy1sAeAfwBsHtCeBXP8").unwrap(),
    ];

    // sending amounts to corresponding to recievers public key
    let amount_lamports: [u64; 4] = [
        1_000_000, 
        2_000_000, 
        3_000_000,
        4_000_000
    ];

    let multi_tx_sign = create_multi_tx_account(
        client, 
        &keypair, 
        &recievers, 
        &amount_lamports, 
    ).await;

    if let Some(sig) = multi_tx_sign {
        println!("multi tx : {}\n", sig);
    }
}

// endregion: --- Mutli-Tx

#[tokio::main]

async fn main() {
    dotenv().ok();

    let keypair = initialize_keypair().await;

    // println!("{:?}\n", keypair);

    let public_key = keypair.pubkey();

    let url = String::from("https://api.devnet.solana.com");
    let client = RpcClient::new(url); 

    let airdrop = airdrop(&client, &public_key).await;

    if let Some(sig) = airdrop {
        println!("airdrop signature : {}\n", sig);
    } 

    let balance = get_balance_in_sol(&client, &public_key).await;
    println!("before balance : {} SOL\n", balance);

    multi_tx(&client, &keypair).await;

    let balance = get_balance_in_sol(&client, &public_key).await;
    println!("after multi-tx balance : {}\n", balance);

}
```
open terminal and run command `cargo run`

Output:
```bash
before balance : 1.983995 SOL

multi tx : 4ofVdWL5aUyNmtbhALf4HKZrdLdGNbHNY1nq63XTdAaGczRHhVJrtShNVaVtuPChYf8ovVpyUK77SXj6Gbgji83v

after multi-tx balance : 1.97399
```

{{< figure src="https://raw.githubusercontent.com/isamyakt/website/72f82d6ebd1569ece7a3ca47210c37a8ffd15b2e/assets/four-txs.png" title="Solana FM Devnet" >}}

- checkout (make sure you’re in devnet) :
https://solana.fm/tx/4ofVdWL5aUyNmtbhALf4HKZrdLdGNbHNY1nq63XTdAaGczRHhVJrtShNVaVtuPChYf8ovVpyUK77SXj6Gbgji83v

{{< figure src="https://raw.githubusercontent.com/isamyakt/website/main/assets/four-txs-flow.png" title="Solana FM Transaction Flow" >}}

# Code Explanation :
- `create_multi_tx_account` Function : 
```zsh
async fn create_multi_tx_account(
    client: &RpcClient,
    sender: &Keypair,
    receivers: &[&Pubkey],
    amount: &[u64],
) -> Option<Signature> {
    // Creating multiple transfer instructions for different receivers
    // ...

    // Building a transaction with all transfer instructions
    // ...

    // Sending and confirming the transaction
    // ...

    Some(sig)
}
```
- This asynchronous function creates and sends a transaction that performs multiple transfers from the sender to different receivers.
- `multi_tx` Function : 
```zsh
async fn multi_tx(client: &RpcClient, keypair: &Keypair) {
    // Define receivers and amounts for each transfer
    // ...

    // Call create_multi_tx_account to create and send the multi-transaction
    // ...
}
```
- This asynchronous function orchestrates the multi-transaction by defining receivers and amounts for each transfer and then calling `create_multi_tx_account`.

# Let's add Few more transfer instruction's
- in `create_multi_tx_account` fn, `let tx = Transaction::new_signed_with_payer(...)` replace with below code  
```zsh
    let tx = Transaction::new_signed_with_payer(
        &[
            first_transfer_ix.clone(), 
            second_transfer_ix.clone(), 
            third_transfer_ix.clone(), 
            fourth_transfer_ix, 
            third_transfer_ix, 
            second_transfer_ix, 
            first_transfer_ix
        ], 
        Some(sender.pubkey()).as_ref(), 
        &[sender], 
        recent_blockhash
    );
```
- now, we can added 7 ix in single tx.
- let's run the code `cargo run`

Output :
```bash
before balance : 1.97399 SOL

multi tx : 4KTVSbtrXc9FCiqkW5udvAL84dz3y5SnGWtwZNk9tFnDreJ73scpQ6UFsryvq6hUSqpsm5MJsp9gADEnZNtbDRVr

after multi-tx balance : 1.957985
```

{{< figure src="https://raw.githubusercontent.com/isamyakt/website/main/assets/seven-txs.png" title="Solana FM Devnet" >}}

- checkout (make sure you're in devnet): https://solana.fm/tx/4ofVdWL5aUyNmtbhALf4HKZrdLdGNbHNY1nq63XTdAaGczRHhVJrtShNVaVtuPChYf8ovVpyUK77SXj6Gbgji83v