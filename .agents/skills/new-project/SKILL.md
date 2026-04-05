---
name: new-project
description: Create a new embassy-rp project based on an appropriate example. This skill sets up the folder structure, project-specific AGENTS.md files, copies necessary config files, initializes main.rs, and updates Cargo.toml.
---

# Create New Embassy RP Project

This skill automates the creation of a new embedded Rust project using the Embassy framework.

## Approach

Files will be copied from an appropriate example in the `embassy/examples` or `esp-hal/examples` directory, depending on the target board.  For RP2040 projects, the `embassy/examples/rp` example is a good starting point. For ESP32 projects, the `esp-hal/examples/hello_world` example is a good starting point. The new project will be configured to follow the host-based unit testing structure defined in the `project-structure` skill.

## Steps

1.  **Create Project Folder**: Create a new sub-folder in the `projects` folder at the root of this repository. The folder name should correspond to the project name (ask the user if not provided).

2.  **Create AGENTS.md**: Inside the new project folder, create an `AGENTS.md` file. This file should include:
    - A brief description of the project.
    - The target board (e.g., Raspberry Pi Pico).
    - The target triple (e.g., `thumbv6m-none-eabi`).
    - Any specific details relevant to the project. This will help guide future agents working on this project.

3.  **Copy Configuration**:
    - Copy `.cargo` to the new project folder.
    - Make sure `.cargo/config.toml` is updated for the target board. For ESP32-C3 projects, this should set `[build] target = "riscv32imc-esp-esp32c3-none-elf"` and a `runner` such as `probe-rs run --chip esp32c3`.
    - Copy all files from the example directory (excluding subdirectories other than .cargo/src if needed, but primarily the root configs) to the new project folder.

4.  **Initialize Source**:
    - For RP2040 projects, copy `embassy/examples/rp/src/bin/blinky.rs` to `src/main.rs` in the new project.
    - For ESP32 projects, copy `esp-hal/examples/hello_world/src/main.rs` to `src/main.rs` in the new project.

5.  **Update Dependencies**:
    - Update `Cargo.toml` with the new project name.
    - Update embassy crate references to use git dependencies targeting the same commit as the embassy git submodule.
    - Update esp-hal crate references to use git dependencies targeting the same commit as the esp-hal git submodule.
    - Update the workspace settings to include the new project as a linked project.

6.  **Documentation**:
    - Update the comment at the top of `src/main.rs` with the project name and a brief description of the project.

7.  **Apply Project Structure**:
    - Refactor the new project to follow the host-based unit testing pattern defined in the `project-structure` skill. This includes creating a `lib.rs` for business logic, updating `Cargo.toml` with conditional dependencies, and adjusting `build.rs` and `main.rs`.

8.  **Verification**:
    - Run `cargo check` on the new project to ensure everything is set up correctly.
