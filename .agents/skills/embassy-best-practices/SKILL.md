---
name: embassy-best-practices
description: Best practices for writing Embassy firmware, including async entry points, task separation, interrupt handling, resource sharing, and timing.
---

# Embassy Best Practices

Below are some Embassy best practices that you should follow:

## 1. Use the Async Entry Point

Instead of the traditional `cortex_m_rt::entry`, use the `#[embassy_executor::main]` attribute. This macro sets up the Embassy executor, initializes the heap (if configured), and allows `main` to be `async`. It also provides a `Spawner` handle for spawning other tasks immediately.

**Example from `blinky.rs`:**

```rust
#![no_std]
#![no_main]

use embassy_executor::Spawner;
use embassy_rp::gpio::{Level, Output};
use embassy_time::Timer;

#[embassy_executor::main]
async fn main(_spawner: Spawner) {
    let p = embassy_rp::init(Default::default());
    let mut led = Output::new(p.PIN_25, Level::Low);

    loop {
        led.set_high();
        Timer::after_secs(1).await;
        led.set_low();
        Timer::after_secs(1).await;
    }
}
```

## 2. Separate Concerns with Tasks

Use `#[embassy_executor::task]` to define background tasks. This keeps your main loop clean and encourages a modular design. Tasks must be `async` and take arguments that are `'static` or effectively static (like `&'static Mutex`).

**Example from `multiprio.rs`:**

```rust
#[embassy_executor::task]
async fn run_high() {
    loop {
        // High priority work
        Timer::after_ticks(1000).await;
    }
}

#[embassy_executor::main]
async fn main(spawner: Spawner) {
    // Spawn a background task
    spawner.spawn(run_high()).unwrap();
    
    // Continue with main task
}
```

## 3. Bind Interrupts Explicitly

Embassy drivers often require interrupt handlers. Instead of manually defining `extern "C"` functions, use the `bind_interrupts!` macro. This bundles interrupt handlers into a struct that acts as a type-safe token proving that the interrupts are handled.

**Example from `usb_logger.rs`:**

```rust
use embassy_rp::bind_interrupts;
use embassy_rp::peripherals::USB;
use embassy_rp::usb::InterruptHandler;

// Define a struct to hold the interrupt handlers
bind_interrupts!(struct Irqs {
    USBCTRL_IRQ => InterruptHandler<USB>;
});

#[embassy_executor::main]
async fn main(_spawner: Spawner) {
    let p = embassy_rp::init(Default::default());
    
    // Pass the Irqs struct to the driver
    let driver = Driver::new(p.USB, Irqs);
}
```

## 4. Share Resources Safely

Sharing data between async tasks requires thread-safe synchronization primitives, even if you are on a single core, because tasks behave like threads.
*   Use `embassy_sync::mutex::Mutex` for sharing async resources (like I/O drivers).
*   Use `embassy_sync::blocking_mutex::Mutex` for sharing synchronous data.
*   For complex signaling, use `Channels` or `Signals`.

**Example from `sharing.rs`:**

```rust
use embassy_sync::blocking_mutex::raw::CriticalSectionRawMutex;
use embassy_sync::mutex::Mutex;
use static_cell::StaticCell;

// Define a StaticCell to hold the mutex globally/statically
static UART_MUTEX: StaticCell<Mutex<CriticalSectionRawMutex, UartTx>> = StaticCell::new();

#[embassy_executor::main]
async fn main(spawner: Spawner) {
    // ... init uart ...
    
    // Initialize the mutex
    let uart_safe = UART_MUTEX.init(Mutex::new(uart));

    // Pass reference to multiple tasks
    spawner.spawn(task_a(uart_safe)).unwrap();
    spawner.spawn(task_b(uart_safe)).unwrap();
}
```

## 5. Use `Timer` and `Ticker` for Timing

Never use blocking delays (like `cortex_m::asm::delay`) in an async context, as this halts the entire executor. Instead, use `embassy_time::Timer`. For periodic tasks, `Ticker` ensures drift-free scheduling.

**Example from `sharing.rs`:**

```rust
use embassy_time::{Duration, Ticker, Timer};

#[embassy_executor::task]
async fn periodic_task() {
    // Ticker helps maintain a steady 1Hz rate even if work takes time
    let mut ticker = Ticker::every(Duration::from_secs(1));
    
    loop {
        // Do work...
        
        // Wait for next tick
        ticker.next().await;
    }
}
```
