# From std to no_std

We have successfully flashed and run our first program, which creates a blinking effect. However, we have not yet explored the code or the project structure in detail. In this section, we will recreate the same project from scratch. I will explain each part of the code and configuration along the way. Are you ready for the challenge?

> NOTE: If you find this chapter overwhelming, especially if you're just working on a hobby project, feel free to skip it for now. You can come back to it later after building some fun projects and working through exercises.

## Create a Fresh Project

We will start by creating a standard Rust binary project. Use the following command:

```rust
cargo new pico-from-scratch
```

At this stage, the project will contain the usual files as expected.

```sh
├── Cargo.toml
└── src
    └── main.rs
```

Our goal is to reach the following final project structure:

```sh
├── build.rs
├── .cargo
│   └── config.toml
├── Cargo.toml
├── memory.x
├── rp235x_riscv.x
├── src
│   └── main.rs
```

