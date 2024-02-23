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

# First thing is to generate/extract `keypair` and store in `.env` file
- filename: `src/main.rs`
```zsh
// region:    --- Crate Modules

use dotenvy::dotenv;
use solana_sdk::signer::keypair::Keypair;
use std::path::Path;
use std::env;
use std::fs::{File, OpenOptions};
use std::io::Write;

// endregion: --- Crate Modules

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
- Imports necessary crates: This includes modules for environment variable management, file handling, and Solana keypair functionality.

`initialize_keypair` : 
- This asynchronous function is responsible for initializing a Solana keypair.
- It first checks if a `.env` file exists in the current directory. If not, it creates one.
- It then attempts to read a private key from an environment variable `PRIVATE_KEY`. If the variable exists, it `parses` the private key from a string into a `byte array` and creates a `keypair` from it.
- If the `PRIVATE_KEY` environment variable does not exist, it generates a new `keypair`, converts it to a `byte array`, and writes it to the .env file.

# Check the balance of `Pubkey` address by `RpcClient`
- filename: continue `src/main.rs`  
```zsh
// region:    --- Crate Modules···
use solana_sdk::native_token::LAMPORTS_PER_SOL;

// region:    --- Keypair···

// region:    --- Balance

pub async fn get_balance_in_sol(client: &RpcClient, pubkey: &Pubkey) -> f64 {
    let lamports = LAMPORTS_PER_SOL as f64;
    let balance = client.get_balance(&pubkey).unwrap() as f64;

    balance / lamports
}

pub async fn get_balance_in_lamports(client: &RpcClient, pubkey: &Pubkey) -> u64 {
    let balance = client.get_balance(&pubkey).unwrap();
    balance
}

// endregion:    --- Balance
```
This code defines two functions related to Solana account balances:

`get_balance_in_sol`:

- Takes an `RpcClient` and a `Pubkey` as input.
- Retrieves the balance of the account associated with the `Pubkey` using `get_balance()` method.
- Converts the balance from lamports to SOL by dividing it by `LAMPORTS_PER_SOL` (a constant representing the number of lamports per SOL).
- Returns the balance as a `f64`- (floating-point number).
 
 `get_balance_in_lamports`:

- This also takes an `RpcClient` and a `Pubkey` as input.
- Retrieves the balance of the account associated with the `Pubkey` using `get_balance()` method.
- Returns the balance directly as a `u64` (unsigned 64-bit integer), without conversion.

# Solana Devnet Airdrops
- filename : continue `src/main.rs`
```zsh
// region:    --- Crate Modules···

use solana_sdk::signature::Signature;
use solana_sdk::signer::Signer;

// region:    --- Keypair···

// region:    --- Balance···

// region:    --- Airdrop

pub async fn airdrop_possible(client: &RpcClient, pubkey: &Pubkey) -> bool {
    let balance = get_balance_in_lamports(client, pubkey).await;

    let result = !(balance > LAMPORTS_PER_SOL);
    result
}

pub async fn airdrop(client: &RpcClient, pubkey: &Pubkey) -> Option<Signature> {
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

#[tokio::main]
async fn main() {
    dotenv().ok();

    let keypair = initialize_keypair().await;

    println!("{:?}\n", keypair);

    let public_key = keypair.pubkey();

    let url = String::from("https://api.devnet.solana.com");
    let client = RpcClient::new(url); 

    let airdrop = airdrop(&client, &public_key).await;

    if let Some(sig) = airdrop {
        println!("airdrop signature : {}\n", sig);
    } 
    println!("airdrop failed : becoz your balance is more then 1 SOL\n");


    let balance = get_balance_in_sol(&client, &public_key).await;

    println!("before balance : {} SOL\n", balance);
}
```
implemented two functions for solana airdrops:

`airdrop_possible`:

- Takes an `RpcClient` and a `Pubkey` as input.
- Retrieves the balance using `get_balance_in_lamports` async function.
- Checks if the balance is less than 1 SOL (LAMPORTS_PER_SOL).
- Returns true if eligible for an airdrop (balance < 1 SOL), false otherwise.
 
`airdrop`:

- Takes an `RpcClient` and a `Pubkey` as input.
- Calls `airdrop_possible` to check eligibility.
- If eligible, retrieves the latest blockhash and requests an airdrop for 1 SOL.
- Returns the `Signature` of the airdrop transaction (if successful), otherwise `None`.

# Let's see how can we make Transactions in Solana
- filename: continue `src/main.rs`
```zsh
// region:    --- Crate Modules···

use solana_sdk::system_instruction;
use solana_sdk::transaction::Transaction;
use std::str::FromStr;

// region:    --- Keypair···

// region:    --- Balance···

// region:    --- Airdrop···

// region:    --- Transaction

pub async fn create_transfer_account(
    client: &RpcClient,
    sender: &Keypair,
    reciever: &Pubkey,
    amount: u64
) -> Option<Signature> {
    let instr = system_instruction::transfer(
        &sender.pubkey(),
        &reciever,
        amount,
    );

    let recent_blockhash = client.get_latest_blockhash().unwrap();
    
    let transaction = Transaction::new_signed_with_payer(
        &[instr], 
        Some(sender.pubkey()).as_ref(), 
        &[sender], 
        recent_blockhash
    );

    let sig = client.send_and_confirm_transaction(&transaction).unwrap();

    Some(sig)

}

// endregion: --- Transaction
```
Initiates a Solana transaction to transfer SOL funds from a sender account to a receiver account.

`create_transfer_account` takes four arguments:

`client` : An `RpcClient` object used to interact with the Solana node.  
`sender` : A `Keypair` object representing the sender's Solana account.  
`receiver` : A `Pubkey` object representing the receiver's Solana account.  
`amount` : The amount of SOL to transfer, represented as a `u64` integer.  

`1`. Create transfer instruction:

- Uses `system_instruction::transfer` to create a Solana instruction object representing the transfer operation.
- This object specifies the sender's public key, receiver's public key, and the transfer amount.

`2`. Get latest blockhash:
- Retrieves the most recent blockhash from the Solana node using `client.get_latest_blockhash()`.
- The blockhash is used for transaction signing and ensures the transaction is valid for the current network state.
  
`3`. Create and sign transaction:
- Constructs a new `Transaction` object with the created transfer instruction.
- Uses `Transaction::new_signed_with_payer` to sign the transaction with the sender's private key and set the fee payer to the sender's public key.

`4`. Send and confirm transaction:
- Submits the signed transaction to the Solana node using `client.send_and_confirm_transaction()`.
- This method waits for the transaction to be processed and included in a block on the blockchain.

`5`. Return signature:
- If successful, returns the transaction signature (a unique identifier for the transaction) as a `Signature`. Otherwise, returns `None`.

# Let's run the code using cli `cargo run`
- filename: `src/main.rs`

```zsh
#[tokio::main]
async fn main() {
    dotenv().ok();

    let keypair = initialize_keypair().await;

    println!("{:?}\n", keypair);

    let public_key = keypair.pubkey();

    let url = String::from("https://api.devnet.solana.com");
    let client = RpcClient::new(url); 

    let airdrop = airdrop(&client, &public_key).await;

    if let Some(sig) = airdrop {
        println!("airdrop signature : {}\n", sig);
    } 
    println!("airdrop failed : becoz your balance is more then 1 SOL\n");


    let balance = get_balance_in_sol(&client, &public_key).await;

    println!("before balance : {} SOL\n", balance);

    let reciever = Pubkey::from_str("Fudp7uPDYNYQRxoq1Q4JiwJnzyxhVz37bGqRki3PBzS").unwrap();
    let transfer_lamports = 10000000; // 0.01 SOL


    let tx = create_transfer_account(&client, &keypair, &reciever, transfer_lamports).await;

    if let Some(sig) = tx {
        println!("tx : {}\n", sig);
    } 

    let balance = get_balance_in_sol(&client, &public_key).await;

    println!("after balance : {}\n", balance);
}
```

Output : 

```sh
Keypair(Keypair { secret: SecretKey: [38, 196, 247, 2, 108, 189, 145, 71, 90, 34, 243, 23, 97, 67, 76, 27, 238, 127, 81, 223, 63, 169, 243, 31, 96, 234, 73, 146, 222, 98, 165, 111], public: PublicKey(CompressedEdwardsY: [125, 210, 157, 111, 216, 68, 183, 45, 14, 241, 150, 186, 109, 50, 10, 244, 180, 219, 137, 210, 176, 243, 218, 17, 117, 120, 48, 227, 188, 158, 235, 159]), EdwardsPoint{
        X: FieldElement51([1728613258012327, 343252765223047, 425545071446415, 960337167879222, 623386839219603]),
        Y: FieldElement51([2046021213213309, 1608941425509814, 606306906941641, 1190259680991337, 561549455274759]),
        Z: FieldElement51([1, 0, 0, 0, 0]),
        T: FieldElement51([275103477731411, 1733262979526940, 388164583281727, 1881120732385144, 2057187608693722])
}) })

airdrop failed : becoz your balance is more then 1 SOL

before balance : 1.969455 SOL

tx : 2dgbRbiAp17JRhssGDE8tZvJ1GMvfgMZGyR5mgAQMdjjzwbLBUvyYggPo3DArQwUZL2wEgh7XhELZhrMLLaNZ7m6

after balance : 1.95945
```

checkout the tx in solana explorer (devnet) : https://explorer.solana.com/tx/2dgbRbiAp17JRhssGDE8tZvJ1GMvfgMZGyR5mgAQMdjjzwbLBUvyYggPo3DArQwUZL2wEgh7XhELZhrMLLaNZ7m6?cluster=devnet

----------------------------------------------------- Remember, don't share your `SecretKey` with anyone -----------------------------------------------------

# Folder Structure
- If you are interested in folder structure
- checkout src:[github](https://github.com/isamyakt/solana-client-rs)
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

