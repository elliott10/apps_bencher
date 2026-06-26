# ArceOS Task Scheduling Benchmark

A benchmark suite for evaluating OS task scheduling performance, designed for the [ArceOS](https://github.com/rcore-os/arceos) operating system.

## Overview

This benchmark measures the performance of key task scheduling operations:

- **rdtsc**: Timer read overhead (TSC on x86_64, CNTVCT_EL0 on aarch64)
- **spawn**: Thread creation and termination latency
- **switch**: Context switch latency between two threads
- **condvar**: Condition variable wait/notify performance (optional)

## Supported Platforms

| Architecture | QEMU | RK3588 Hardware |
|--------------|------|-----------------|
| x86_64       | ✓    | -               |
| aarch64      | ✓    | ✓               |

## Building

### For ArceOS (aarch64)

```bash
# Build for QEMU
cargo build --features "axstd,qemu" --target aarch64-unknown-none-softfloat

# Build for RK3588
cargo build --features "axstd" --target aarch64-unknown-none-softfloat
```

### For Linux/macOS (native)

```bash
cargo build --no-default-features
```

## Running

### On QEMU

```bash
# From ArceOS root directory
make run A=apps/benchmarks/arceos-bencher ARCH=aarch64 FEATURES=qemu
```

### On RK3588 Hardware

The benchmark outputs GPIO signals for hardware-level measurement:

- **GPIO3_C6**: Pulse signal for thread switching visualization
  - High level: Thread 0 running
  - Low level: Thread 1 running
- **UART7**: Serial output for benchmark results
- **LED indicators**:
  - Red LED (GPIO3_B2): Initialization complete
  - Green LED (GPIO3_C0): Benchmark running

Connect a logic analyzer or oscilloscope to GPIO3_C6 to measure actual context switch timing.

## Benchmark Details

### rdtsc
Measures the overhead of reading the system timer counter.
- x86_64: Uses `__rdtscp` instruction
- aarch64: Reads `CNTVCT_EL0` register

### spawn
Measures thread creation latency by spawning and joining threads in a loop.
- 500,000 iterations (ArceOS)
- 200,000 iterations (Linux/macOS)

### switch
Measures context switch latency between two cooperating threads.
- Each thread yields `iter/2` times
- Single switch time = total_time / iterations
- Default: 100,000,000 switches per measurement, repeated 100 times

### condvar (optional)
Measures condition variable signaling overhead.
- 5,000,000 iterations
- Uses wait queue on ArceOS, standard Condvar on Linux/macOS

## Measurement Methodology

### Timing Sources

- **x86_64**: TSC (Time Stamp Counter) at assumed 4 GHz
- **aarch64**: 
  - `CNTVCT_EL0`: Generic timer counter (typically 24 MHz)
  - `PMCCNTR_EL0`: CPU cycle counter (CPU frequency dependent)

### PMU Configuration

On aarch64, the benchmark enables user-mode access to Performance Monitoring Unit (PMU):
- `PMUSERENR_EL0`: Enables EL0 access to performance monitors
- `PMCR_EL0`: Enables all counters with 64-bit overflow
- `PMCNTENSET_EL0`: Enables cycle count register

### Output Format

```
Benchmark: switch
  Iterations: 100000000
  Benchmark total duration: 42 s
  Average timer nanoseconds: 420 ns
  Min CPU cycles: 1008
  Average CPU cycles: 1008
  Max CPU cycles: 1008
```

## Dependencies

- `axstd`: ArceOS standard library (optional, for ArceOS builds)
- `dw_apb_uart`: UART driver for RK3588 hardware output
- `aarch64-cpu`: AArch64 register access (aarch64 only)

