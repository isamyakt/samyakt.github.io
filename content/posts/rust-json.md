---
title: How to read and write JSON Data in Rust?
description: Rust structs that can be serialized to and deserialized from JSON using the `serde` crate
date: "2023-08-17"
---

# Environment Setup
- **Rust**, You can download rust from their [website](https://www.rust-lang.org/learn/get-started)
- cargo comes with rust, so you don't have to worry about it.
- Initialize the Project
```zsh
$ cargo new --bin rust-json
```

- **serde** & **serde_json** can be installed via
```zsh
$ cargo add serde -F derive
$ cargo add serde_json
```
|| OR ||
- open `Cargo.toml` to add `serde` and `serde_json` dependencies crate
```
[dependencies]
serde = { version = "1.0.196", features = ["derive"] }
serde_json = "1.0.113"
```

# Folder Structure
The `target` folder will be automatically created after running `cargo build`.

```
.
├── src
│   ├── json
│   │     ├── mod.rs
│   │     ├── read_json.rs
│   │     └── write_json.rs
│   └── main.rs
├── .gitignore
├── Cargo.lock
└── Cargo.toml

```
- `json/read_json.rs` is the file we're reading the json data
- `json/write_json.rs` is the file we're writing the json data
- `main.rs` is the file we'll run the code and understand the json behaviours

# Rust file modules
- filename: `src/main.rs`
```
mod json;

fn main() {
}
```

- filename: `src/json/mod.rs`
```
pub mod read_json;
pub mod write_json; 
```

# Write Json in rust
- filename: `src/json/write_json.rs`
```
use serde::Serialize;

#[derive(Serialize)]
pub struct WriteGameLevel {
    pub level_desc: String,
}

#[derive(Serialize)]
pub struct WriteGame {
    pub name: String,
    pub creator: String,
    pub levels: Vec<WriteGameLevel>,
}


pub fn write_json(game: &WriteGame) -> String  {
    let json = serde_json::to_string(game).unwrap();
    json
}
```
Let's import the necessary traits from the `serde` crate.
`Serialize` is used for serializing Rust structures into JSON data.

Rust Structs:

- Let's have a Rust struct named `WriteGameLevel`. It has one field `level_desc` of type `String`.
- We have another Rust struct named `WriteGame`. With the fields: `name`, `creator`, `levels`.

The `levels` field is of type `Vec<WriteGameLevel>`, meaning it's a vector containing multiple `WriteGameLevel` structs. Like `WriteGameLevel`, `WriteGame` also derives `Serialize`.

`#`[`derive(Serialize)`] attribute indicates that Rust data structure is converted into a format that can be stored or transmitted, such as JSON, YAML or binary.

function `write_json(game: &WriteGame)` takes reference to `WriteGame` as parameter and returns `String`.

`serde_json::to_string(&article)` serializes the `WriteGame` struct into a JSON string.
`to_string` returns a `Result<String, serde_json::Error>`, so `unwrap()` is used to handle any potential errors and extract the resulting JSON string.

The resulting JSON string is returned.

# Let's Run the Write Json code
- filename: `src/main.rs`
```
mod json;
use crate::json::write_json::{
    WriteGame, 
    WriteGameLevel, 
    write_json
};

fn stringfy_json() {
    let game: WriteGame = WriteGame {
        name: String::from("rust game"),
        creator: String::from("samyakt"),
        levels: vec![
            WriteGameLevel {
                level_desc: String::from("basic level")
            },
            WriteGameLevel {
                level_desc: String::from("medium level")
            },
            WriteGameLevel {
                level_desc: String::from("hard level")
            }
        ]
    };

    let json = write_json(&game);

    println!("The JSON is: {}", json);
}

fn main() {
    // write json
    stringfy_json();
}
```
open terminal and run command
```cargo run```

Output:
```
The JSON is: {"name":"rust game","creator":"samyakt","levels":[{"level_desc":"basic level"},{"level_desc":"medium level"},{"level_desc":"hard level"}]}
```
In the `main` function `stringfy_json()` is called, where it contains a `WriteGame` struct is created with some example data. 

This struct represents an game with a game name (`name`), a creator (`creator`), and a list of game levels (`level_desc`).

The `write_json` function is called to serialize the Rust struct `WriteGame` into an JSON string.

# Write Json in rust
- filename: `src/json/read_json.rs`
```
use serde::Deserialize;

#[derive(Deserialize)]
pub struct ReadGameLevel {
    pub level_desc: String,
}

#[derive(Deserialize)]
pub struct ReadGame {
    pub name: String,
    pub creator: String,
    pub levels: Vec<ReadGameLevel>,
}

pub fn read_json_typed(raw_json: &str) -> ReadGame {
    let parsed = serde_json::from_str(raw_json).unwrap();
    parsed
}
```
Let's import the `Deserialize` traits from the `serde` crate.
`Deserialize` is used for deserializing JSON data into Rust structures

Rust Structs:

- Let's have a Rust struct named `ReadGameLevel`. It has one field `level_desc` of type `String`.
- We have another Rust struct named `ReadGame`. With the fields: `name`, `creator`, `levels`.

The `levels` field is of type `Vec<ReadGameLevel>`, meaning it's a vector containing multiple `ReadGameLevel` structs. Like `ReadGameLevel`, `ReadGame` also derives `Deserialize`.

`#`[`derive(Deserialize)`] attribute when you want to create a Rust data structure from a serialized format like JSON, YAML or binary.

The `read_json_typed` function takes a reference to a string `(raw_json: &str)` as input and returns an `ReadGame` struct

It deserializes the JSON string into an `ReadGame` struct using `serde_json::from_str` and any errors encountered during deserialization are unwrapped using `unwrap()`

# Let's Run the Read Json code
- filename: `src/main.rs`
```
mod json;
use crate::json::read_json::read_json_typed;

fn parsed_json() {
    let json = r#"
        {
            "name" : "rust game",
            "creator" : "samyakt",
            "levels" : [
                {
                    "level_desc":"basic level"
                },
                {
                    "level_desc":"medium level"
                },
                {
                    "level_desc":"hard level"
                }
            ]
        }
    "#;

    let parsed = read_json_typed(json);

    println!("Game name is : {}", parsed.name);
    println!("Game creator name : {}", parsed.creator);
    println!("The first level : {}", parsed.levels[0].level_desc);
    println!("The third level : {}", parsed.levels[2].level_desc);
}

fn main() {
    // read json
    parsed_json();

    // stringfy_json(); // comment for now
}
```
open terminal and run command
```cargo run```

Output:
```
Game name is : rust game
Game creator name : samyakt
The first level : basic level
The third level : hard level
```
In the `main` function the `parsed_json()` function is called,  a JSON string representing a game is defined. This JSON string includes an game name, creator name, and a list of levels, each containing a different levels.

The `read_json_typed` function is called to deserialize the JSON string into an `ReadGame` struct.

This program essentially demonstrates how to define Rust structs that can be serialized to and deserialized from JSON using the `serde` crate.
It also shows how to use these structs to read and manipulate JSON data.

# Run both read and write json code
- filename: `src/main.rs` (full code)
```
mod json;

use crate::json::write_json::{WriteGame, WriteGameLevel, write_json};
use crate::json::read_json::read_json_typed;

fn parsed_json() {
    let json = r#"
        {
            "name" : "rust game",
            "creator" : "samyakt",
            "levels" : [
                {
                    "level_desc":"basic level"
                },
                {
                    "level_desc":"medium level"
                },
                {
                    "level_desc":"hard level"
                }
            ]
        }
    "#;

    let parsed = read_json_typed(json);

    println!("Game name is : {}", parsed.name);
    println!("Game creator name : {}", parsed.creator);
    println!("The first level : {}", parsed.levels[0].level_desc);
    println!("The third level : {}", parsed.levels[2].level_desc);
}

fn stringfy_json() {
    let game: WriteGame = WriteGame {
        name: String::from("rust game"),
        creator: String::from("samyakt"),
        levels: vec![
            WriteGameLevel {
                level_desc: String::from("basic level")
            },
            WriteGameLevel {
                level_desc: String::from("medium level")
            },
            WriteGameLevel {
                level_desc: String::from("hard level")
            }
        ]
    };

    let json = write_json(&game);

    println!("The JSON is: {}", json);
}


fn main() {
    // read json
    parsed_json();

    println!("\n");

    // write json
    stringfy_json();
}
```

Let me know what you think! 🦀

code src: [github](https://github.com/isamyakt/rust-json) 