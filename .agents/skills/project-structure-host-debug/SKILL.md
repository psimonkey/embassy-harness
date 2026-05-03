---
name: project-structure-host-debug
description: Configure an embedded Rust project structure to support host-based unit testing and dual-target builds. This pattern allows you to write and run unit tests for your embedded business logic on your host machine while maintaining a standard no_std firmware build.
---

# Host-Based Unit Testing Pattern for Embedded Rust

This pattern allows you to write and run unit tests for your embedded business logic on your host machine (e.g., macOS, Linux, Windows) while maintaining a standard `no_std` firmware build for your target device (e.g., RP2040, STM32).

## 1. Project Structure

Move your pure logic into `src/lib.rs` so it can be tested independently. Keep your hardware setup and main loop in `src/main.rs`.

```
my-project/
├── Cargo.toml
├── build.rs
├── src/
│   ├── lib.rs   <-- Business logic (pure functions, state machines) & Unit Tests
│   └── main.rs  <-- Firmware entry point (Embassy tasks, hardware init)
```

## 2. Cargo.toml Configuration

### Disable Binary Tests
Disable the default test harness for the binary so `cargo test` doesn't try to run unit tests inside `main.rs`.

*Note: Cargo will still compile `main.rs` during tests to check for errors, which is why we still need the `cfg` guards in Step 4.*

```toml
[[bin]]
name = "my-project"
test = false
```

### Conditional Dependencies
Move microcontroller-specific crates (like HALs, runtimes) behind a target gate so they don't break the host build.

```toml
[dependencies]
# Dependencies compatible with both host and device (no_std friendly)
smart-leds = "0.4.0"
heapless = "0.9"
log = "0.4"

# ...

# Firmware-only dependencies
[target.'cfg(target_os = "none")'.dependencies]
embassy-rp = { ... }
embassy-executor = { ... }
embassy-time = { ... }
defmt = "..."
cortex-m = "..."
```

## 3. build.rs Adjustments

Linker arguments (like memory layout scripts) crash the host linker. Wrap them in a condition so they only apply when building for the embedded target. We also need to copy `memory.x` to the output directory and add it to the linker search path for the embedded target.

```rust
use std::env;
use std::fs::File;
use std::io::Write;
use std::path::PathBuf;

fn main() {
    // Only configure linker scripts for embedded targets
    if env::var("CARGO_CFG_TARGET_OS").unwrap_or_default() == "none" {
        // Put `memory.x` in our output directory and ensure it's
        // on the linker search path.
        let out = &PathBuf::from(env::var_os("OUT_DIR").unwrap());
        File::create(out.join("memory.x"))
            .unwrap()
            .write_all(include_bytes!("memory.x"))
            .unwrap();
        println!("cargo:rustc-link-search={}", out.display());
        println!("cargo:rerun-if-changed=memory.x");

        println!("cargo:rustc-link-arg-bins=--nmagic");
        println!("cargo:rustc-link-arg-bins=-Tlink.x");
        // Required for RP2040 to link boot2
        println!("cargo:rustc-link-arg-bins=-Tlink-rp.x");
        println!("cargo:rustc-link-arg-bins=-Tdefmt.x");
    }
}
```

## 4. src/main.rs

Guard the firmware implementation so it is ignored by the host compiler. Provide a dummy `main` for the host to satisfy the compiler entry point requirement (though it won't be run during tests).

```rust
#![cfg_attr(target_os = "none", no_std)]
#![cfg_attr(target_os = "none", no_main)]

// Dummy main for host builds
#[cfg(not(target_os = "none"))]
fn main() {}

// Firmware implementation
#[cfg(target_os = "none")]
mod firmware {
    use embassy_executor::Spawner;
    // ... other imports ...

    #[embassy_executor::main]
    async fn main(_spawner: Spawner) {
        // ... hardware init and task spawning ...
    }
}
```

## 5. src/lib.rs

Write your logic here. Using `#![no_std]` ensures you don't accidentally use standard library features unavailable on your device.

```rust
#![no_std]

pub fn calculate_led_color(position: u8) -> [u8; 3] {
    // ... logic ...
    [255, 0, 0]
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_color_calculation() {
        assert_eq!(calculate_led_color(0), [255, 0, 0]);
    }
}
```

## 6. Running Tests and Builds

**To Run Unit Tests (Host):**
Specify your host's target triple to ensure `target_os="none"` is false.
```bash
# macOS (Apple Silicon)
cargo test --target aarch64-apple-darwin

# macOS (Intel) or Linux (x86_64)
cargo test --target x86_64-apple-darwin
```

**To Build Firmware (Device):**
Use your standard build command (assuming the project's `.cargo/config.toml` sets the default target and runner). For example, an ESP32-C3 project can define:

```toml
[build]
target = "riscv32imc-esp-esp32c3-none-elf"

[target.'cfg(target_arch = "riscv32")']
runner = "probe-rs run --chip esp32c3"
```

Then you can run:

```bash
cargo build
# or
cargo run
```
