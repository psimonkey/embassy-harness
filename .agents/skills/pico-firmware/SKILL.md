---
name: pico-firmware
description: Firmware development for Raspberry Pi Pico (RP2040) using the Embassy framework
---

# Raspberry Pi Pico Firmware Development

This skill covers the details of developing firmware for the Raspberry Pi Pico (RP2040) using the Embassy framework.

## Hardware Details

The firmware is being developed for the Raspberry Pi Pico, which is based on the RP2040 microcontroller. The RP2040 features a dual-core ARM Cortex-M0+ processor, 264KB of SRAM, and support for various peripherals including GPIO, I2C, SPI, UART, and USB. The Pico board also includes 2MB of onboard flash memory for storing the firmware.

The GPIO function mappings are detailed in gpio-functions.md.

## Flashing and Debugging

### Flashing with `cargo run`

To successfully flash firmware to the device, use `cargo run`. This command compiles the project and uses `probe-rs` to flash the binary to the RP2040 and attach a debugger/logger.

For faster edit/flash/test cycles, use release mode:

```bash
cargo run --release --target thumbv6m-none-eabi
```

This is especially helpful on Pico W projects that include large Wi-Fi blobs.

### Wi-Fi firmware blobs: download once and use a static path

For Pico W (`cyw43`) projects, avoid repeatedly copying/downloading blob files per project. Keep them in a single static host location and point your code to that location.

```rust
use cyw43::aligned_bytes;
let fw = aligned_bytes!("../../../embassy/cyw43-firmware/43439A0.bin");
let clm = aligned_bytes!("../../../embassy/cyw43-firmware/43439A0_clm.bin");
let nvram = aligned_bytes!("../../../embassy/cyw43-firmware/nvram_rp2040.bin");
```

Pre-flash the blobs once:

```bash
probe-rs download "../../embassy/cyw43-firmware/43439A0.bin" --binary-format bin --chip RP2040 --base-address 0x10100000
probe-rs download "../../embassy/cyw43-firmware/43439A0_clm.bin" --binary-format bin --chip RP2040 --base-address 0x10140000
probe-rs download "../../embassy/cyw43-firmware/nvram_rp2040.bin" --binary-format bin --chip RP2040 --base-address 0x10148000
```

Then reference them as follows:

```rust
let fw = unsafe { core::slice::from_raw_parts(0x10100000 as *const u8, 231077) };
let clm = unsafe { core::slice::from_raw_parts(0x10140000 as *const u8, 984) };
let nvram = unsafe { core::slice::from_raw_parts(0x10148000 as *const u8, 742) };
```

### Troubleshooting Probe Issues

If you encounter issues connecting to the debug probe (e.g., "Probe not found" or similar errors likely due to USB congestion or a hung process), try resetting the probe software stack by killing the `probe-rs` process:

```bash
killall probe-rs
```

Then retry the `cargo run` command.
