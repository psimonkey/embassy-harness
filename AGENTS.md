# Agent Skills and Instructions

This file documents the available skills and standard operating procedures for agents working in this repository.

The host system's target triple is specified in host.md, and the embedded target for firmware development is specified in the AGENTS.md file in each project.  If details of the target are not found, assume it is `thumbv6m-none-eabi`.

Don't use inline env vars, it breaks the harness. Export the env vars before running the command.

Specific commands have been whitelisted so where possible use the exact command line listed, rather than concatenating multiple together.

## Standard Operating Procedures

* Most work should be performed within a project-specific sub-folder of the `projects/` directory. If it is not clear from the context which project you should be operating in, **ask the user** for clarification before proceeding.
* Lean heavily on the examples (embassy/examples/rp/src/bin/*.rs for RP2040, esp-hal/examples/*/src/*.rs and esp-hal/examples/*/*/src/*.rs for ESP32) to guide your implementation.
* There are also examples for other platforms (embassy/examples/*/src/bin/*.rs) that may be helpful for reference but remember you will need to adapt them to the correct target for each project.
* As a fallback, you can also search the full Embassy project source code (embassy/**/*.rs) and ESP-HAL project source code (esp-hal/**/*.rs).

## Available Skills

### Embassy Best Practices
- **File**: `.agents/skills/embassy-best-practices/SKILL.md`
- **Description**: Best practices for writing Embassy firmware. Key points include:
    - Use `#[embassy_executor::main]` instead of `cortex_m_rt::entry`.
    - Use `#[embassy_executor::task]` for background tasks to separate concerns.
    - Use `bind_interrupts!` macro for explicit interrupt binding.
    - Use `embassy_sync` primitives (Mutex, Channel, Signal) for safe resource sharing.

### Create New Project
- **File**: `.agents/skills/new-project/SKILL.md`
- **Description**: Automates the creation of a new Embassy RP project.
    - Creates a folder in `projects/`.
    - Copies configuration from `embassy/examples/rp` or `esp-hal/examples/`.
    - Initializes `main.rs` from a suitable example.
    - Updates `Cargo.toml` and workspace members.
    - Create a AGENTS.md file for the project with information about the target board and anything else relevant to the project.
    - Applies the "Host-Based Unit Testing" project structure.

### Pico Firmware Development
- **File**: `.agents/skills/pico-firmware/SKILL.md`
- **Description**: Specific details for RP2040/Pico development.
    - Hardware details (Cortex-M0+, 264KB SRAM, 2MB Flash).
    - GPIO function mappings reference (`boards/rp2040/gpio-functions.md`).
    - Flashing instructions using `cargo run`.
    - Troubleshooting connection issues (`killall probe-rs`).

### ESP32 Firmware Development
- **File**: `.agents/skills/esp32-firmware/SKILL.md`
- **Description**: Specific details for ESP32 development.
    - Various hardware types, depending on the variant of ESP32 being used.
    - PIN mappings reference (`boards/*/pins.md`).
    - Flashing instructions using `cargo run`.
    - Troubleshooting connection issues (`killall probe-rs`).

### Project Structure (Simple)
- **File**: `.agents/skills/project-structure/SKILL.md`
- **Description**: Configures a project with a `no_std` firmware build.
    - Splits code into `src/lib.rs` (logic) and `src/main.rs` (hardware/entry).
    - Configures `Cargo.toml` to disable binary tests and use conditional dependencies.
    - Adjusts `build.rs` to only link embedded artifacts for target builds.

### Project Structure (Host-Based Unit Testing)
- **File**: `.agents/skills/project-structure-host-debug/SKILL.md`
- **Description**: Configures a project for host-based unit testing while maintaining a `no_std` firmware build.
    - Splits code into `src/lib.rs` (logic) and `src/main.rs` (hardware/entry).
    - Configures `Cargo.toml` to disable binary tests and use conditional dependencies.
    - Adjusts `build.rs` to only link embedded artifacts for target builds.
    - Guards `src/main.rs` with `#[cfg(not(target_os = "none"))]` dummy main.

### Review Project
- **File**: `.agents/skills/review-project/SKILL.md`
- **Description**: Procedure for reviewing and improving firmware code.
    - Fix build warnings.
    - Check against best practices.
    - Compare with `embassy/examples/rp` or `esp-hal/examples/hello_world` and other embassy sources.
    - Refine and validate.
