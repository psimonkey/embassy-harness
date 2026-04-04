---
name: new-project
description: Create a new embassy-rp project based on the blinky example. This skill sets up the folder structure, copies necessary config files, initializes main.rs, and updates Cargo.toml.
---

# Create New Embassy RP Project

This skill automates the creation of a new Raspberry Pi Pico (RP2040) project using the Embassy framework.

## Steps

1.  **Create Project Folder**: Create a new sub-folder in the `projects` folder at the root of this repository. The folder name should correspond to the project name (ask the user if not provided).

2.  **Copy Configuration**:
    - Copy `embassy/examples/rp/.cargo` to the new project folder.
    - Copy all files from `embassy/examples/rp/` (excluding subdirectories other than .cargo/src if needed, but primarily the root configs) to the new project folder.

3.  **Initialize Source**:
    - Copy `embassy/examples/rp/src/bin/blinky.rs` to `src/main.rs` in the new project.

4.  **Update Dependencies**:
    - Update `Cargo.toml` with the new project name.
    - Update embassy crate references to use git dependencies targeting the same commit as the embassy git submodule.
    - Update the workspace settings to include the new project as a linked project.

5.  **Documentation**:
    - Update the comment at the top of `src/main.rs` with the project name and a brief description of the project.

6.  **Apply Project Structure**:
    - Refactor the new project to follow the host-based unit testing pattern defined in the `project-structure` skill. This includes creating a `lib.rs` for business logic, updating `Cargo.toml` with conditional dependencies, and adjusting `build.rs` and `main.rs`.

7.  **Verification**:
    - Run `cargo check` on the new project to ensure everything is set up correctly.
