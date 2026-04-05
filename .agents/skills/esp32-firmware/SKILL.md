---
name: esp32-firmware
description: Firmware development for ESP32 using the Embassy framework
---

# ESP32 Firmware Development

This skill covers the details of developing firmware for several variants of the ESP32 chip, and several types of hardware modules based on those chips, using the Embassy framework.

## Hardware Details

The firmware is being developed for the ESP32, which has several variants:

- ESP32: The original ESP32 features a dual-core Xtensa processor, 520KB of SRAM, and support for various peripherals including GPIO, I2C, SPI, UART, and USB. The ESP32 board also includes 4MB of onboard flash memory for storing the firmware.
- ESP32-C3: The ESP32-C3 is a more recent variant that features a single-core RISC-V processor, 400KB of SRAM, and support for similar peripherals as the original ESP32. It also includes 4MB of onboard flash memory.

Several hardware modules are based on these chips, such as the ESP32-C3 Supermini, which is a compact module that includes the ESP32-C3 chip along with necessary components for power regulation and USB connectivity.

The GPIO function mappings are detailed in files named pins.md in various subfolders of the `boards` folder. For example, the ESP32-C3 Supermini pin mappings can be found in `boards/esp32-c3-supermini/pins.md`.

Different variants require different target triples for compilation. The original ESP32 typically uses `xtensa-esp32-none-elf`, while the ESP32-C3 uses `riscv32imc-unknown-none-elf`. The specific target triple for each project should be documented in the AGENTS.md file for that project.

## ESP-HAL

The ESP-HAL crate provides hardware abstraction layers for various ESP32 variants, including the original ESP32 and the ESP32-C3. When developing firmware for these boards, you will typically use the `esp-hal` crate to interact with the hardware peripherals. The `esp-hal` crate includes support for GPIO, I2C, SPI, UART, and other common peripherals, as well as board-specific support for different ESP32 variants.

## Flashing and Debugging

### Flashing with `cargo run`

To successfully flash firmware to the device, use `cargo run`. This command compiles the project and then runs the cargo runner configured in the project's `.cargo/config.toml` file. For ESP32-C3 projects, that file should set a default target triple and a runner using `probe-rs`.

For example, a typical ESP32-C3 `.cargo/config.toml` includes:

```toml
[build]
target = "riscv32imc-unknown-none-elf"

[target.'cfg(target_arch = "riscv32")']
runner = "probe-rs run --chip esp32c3"
```

This lets you use the simpler command:

```bash
cargo run --release
```

If your project does not have a local `.cargo/config.toml`, or if it uses a different runner, then you may need to pass the target explicitly:

```bash
cargo run --release --target riscv32imc-unknown-none-elf
```

### Troubleshooting Probe Issues

If you encounter issues connecting to the debug probe (e.g., "Probe not found" or similar errors likely due to USB congestion or a hung process), try resetting the probe software stack by killing the `probe-rs` process:

```bash
killall probe-rs
```

Then retry the `cargo run` command.