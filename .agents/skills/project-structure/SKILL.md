---
name: project-structure
description: Configure an embedded Rust project structure for a standard no_std firmware build.
---

# Target-Only no_std Firmware Pattern for Embedded Rust

This pattern describes a clean project structure for writing and building firmware directly for your embedded target.

## 1. Project Structure

Separate business logic from the platform-specific entry point.

```
my-project/
├── Cargo.toml
├── build.rs
├── src/
│   ├── lib.rs   <-- Business logic and reusable modules
│   └── main.rs  <-- Firmware entry point, hardware init, task setup
```

## 2. Cargo.toml Configuration

Use a normal firmware `[[bin]]` declaration when needed, but keep target-specific build settings in `.cargo/config.toml` rather than splitting behavior for host testing.

```toml
[[bin]]
name = "my-project"
path = "src/main.rs"
```

## 3. build.rs Adjustments

For embedded targets, copy `memory.x` to the output directory and pass the linker arguments required for your board.

```rust
use std::env;
use std::fs::File;
use std::io::Write;
use std::path::PathBuf;

fn main() {
    if env::var("CARGO_CFG_TARGET_OS").unwrap_or_default() == "none" {
        let out = PathBuf::from(env::var_os("OUT_DIR").unwrap());
        File::create(out.join("memory.x"))
            .unwrap()
            .write_all(include_bytes!("memory.x"))
            .unwrap();
        println!("cargo:rustc-link-search={}", out.display());
        println!("cargo:rerun-if-changed=memory.x");

        println!("cargo:rustc-link-arg-bins=--nmagic");
        println!("cargo:rustc-link-arg-bins=-Tlink.x");
        println!("cargo:rustc-link-arg-bins=-Tlink-rp.x");
        println!("cargo:rustc-link-arg-bins=-Tdefmt.x");
    }
}
```

## 4. src/main.rs

Write the firmware entry point for the embedded target using `no_std` and `no_main`.

```rust
#![no_std]
#![no_main]

use embassy_executor::Spawner;

#[embassy_executor::main]
async fn main(_spawner: Spawner) {
    // Hardware init and task spawning...
}
```

## 5. src/lib.rs

Keep reusable logic in `src/lib.rs` so it is easy to reason about and maintain.

```rust
#![no_std]

pub fn calculate_led_color(position: u8) -> [u8; 3] {
    // Business logic
    [255, 0, 0]
}
```

## 6. Building for the Target Architecture

Build directly for the embedded target. Configure the target and runner in `.cargo/config.toml` so `cargo build` and `cargo run` remain straightforward.

Example for ESP32-C3:

```toml
[build]
target = "riscv32imc-esp-esp32c3-none-elf"

[target.'cfg(target_arch = "riscv32")']
runner = "probe-rs run --chip esp32c3"
```

Then use:

```bash
cargo build
cargo run
```
