# Project Template with `cargo-generate`

"cargo-generate is a developer tool to help you get up and running quickly with a new Rust project by leveraging a pre-existing git repository as a template."

Read more about on [cargo-generate GitHub project](https://github.com/cargo-generate/cargo-generate).

## Prerequisites

Before starting, ensure you have the following tools installed:

- [Rust](https://www.rust-lang.org/tools/install)
- [cargo-generate](https://github.com/cargo-generate/cargo-generate) for generating the project template.


Install the OpenSSL development package first because it is required by cargo-generate:
```sh
sudo apt install  libssl-dev
```

You can install `cargo-generate` using the following command:

```sh
cargo install cargo-generate
```

## Step 1: Generate the Project
Run the following command to generate the project from the template:

```sh
cargo generate --git https://github.com/ImplFerris/pico2-template.git --tag v0.3.2
```

This will prompt you to answer a few questions:

- Project name: Name your project.
- HAL choice: You can choose between `embassy` or `rp-hal`.

## Step 2: Default LED Blink Example
By default, the project will be generated with a simple LED blink example. The code structure may look like this:

`src/main.rs`: Contains the default blink logic.

`Cargo.toml`: Includes dependencies for the selected HAL.

## Step 3: Choose Your HAL and Modify Code
Once the project is generated, you can decide to keep the default LED blink code or remove it and replace it with your own code based on the HAL you selected.

## Removing Unwanted Code
You can remove the blink logic from `src/main.rs` and replace it with your own code. Modify the `Cargo.toml` dependencies and project structure as needed for your project.

