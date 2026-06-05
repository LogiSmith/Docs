# Tutorial 4 — RISC-V SoC

The top of the ladder: put a whole **RISC-V CPU** on the FPGA and run a **C
program** on it. The program prints over UART — so this ties together everything:
hardware (a CPU + peripherals) *and* software (firmware) in one build.

**You'll learn:** what an SoC is · firmware compiled and baked into the bitstream ·
memory-mapped peripherals (MMIO) · hardware/software co-design.

**Prerequisite:** toolchain [installed](../installation/index.md) (incl. the RISC-V
toolchain — `anvil doctor`); [UART hello](uart-hello.md) done.

## 1. Start from the example

```bash
mkdir soc && cd soc
anvil init --board Nexys-A7-100T --example hello-PicoRV
```

## 2. What's an SoC here?

An **SoC** (System-on-Chip) is a CPU + a bus + peripherals. Adding the SoC module
brings all of that, *and* scaffolds a `firmware/` folder for your C code:

```bash
anvil addmodule picorv32-soc      # CPU + APB bus + interconnect; creates firmware/
```

The top level instantiates the `picorv32_soc` (the PicoRV32 CPU + a 64-slot MMIO
bus) and an `apb_uart` peripheral on **slot 0**:

```systemverilog
picorv32_soc #(.NUM_SLOTS(64)) u_soc ( .clk(clk), .resetn(cpu_resetn), … );
apb_uart      u_uart           ( .PSEL(SpSEL[0]), … .uart_tx(uart_tx) );  // slot 0
```

The board's `led0` is a heartbeat blink and `led1` lights on UART activity — handy
visual confirmation the CPU is alive and transmitting.

## 3. The firmware (`firmware/src/main.cpp`)

This is the program the CPU runs — plain C++:

```cpp
#include "soc.hpp"

int main() {
    uart_puts("Hello from PicoRV32!\n");
    while (1);
}
```

How does writing a string reach the UART hardware? Through **memory-mapped I/O** —
the peripheral lives at a fixed address (`firmware/include/soc.hpp`):

```cpp
#define IO_BASE  0xC0000000
#define UART_TX  (*(volatile unsigned int*)SLOT_ADDR(0, 0))   // slot 0, reg 0

inline void uart_putc(char c) { UART_TX = c; }                // a store sends a byte
inline void uart_puts(const char* s) { while (*s) uart_putc(*s++); }
```

Writing a byte to address `UART_TX` *is* the act of transmitting it — the hardware
on slot 0 sees the bus write and shifts the byte out. That's MMIO: software talks
to hardware through ordinary memory addresses.

## 4. Build (firmware + hardware together)

```bash
anvil build
```

For an SoC project, `anvil build` does **two** things:

1. **`anvil compile`** — compiles your C/C++ with the RISC-V toolchain and bakes
   the program into the CPU's RAM (a generated `ram.v` — see
   [How it works](../how-it-works.md) / [memory init](../troubleshooting.md#faq)).
2. **`anvil synth`** — synthesizes the whole SoC, with that firmware inside, into
   the bitstream.

So the program ships *inside* the bitstream — no separate flashing of code.

## 5. Program and watch

```bash
anvil program
screen /dev/ttyUSB1 9600      # exit: Ctrl+A then K
```

Press reset — the CPU boots and prints **`Hello from PicoRV32!`**. A RISC-V
processor you synthesized is running your C code. 🎉

!!! note "On WSL"
    Needs USB forwarded + the FTDI driver for `/dev/ttyUSB1` — see the
    [WSL2 guide](../installation/wsl.md).

## Make it print your own message

Edit `firmware/src/main.cpp`, then just `anvil build && anvil program`:

```cpp
int main() {
    uart_puts("Booting...\n");
    uart_puts("My RISC-V SoC says hi!\n");

    // print it on a loop, with a crude delay
    while (1) {
        uart_puts("tick\n");
        for (volatile int i = 0; i < 2000000; i++);   // ~delay
    }
}
```

Change the C, rebuild — the new program is baked into the next bitstream.

## What you learned

- An **SoC** = CPU + bus + peripherals, added as one module.
- **Firmware** is compiled and embedded into the bitstream (HW + SW in one build).
- **MMIO** — software drives hardware by reading/writing fixed addresses.
- **Hardware/software co-design** — the essence of embedded systems.

## Where to go from here

You've climbed the whole ladder: an LED, combinational logic, a serial protocol
via modules, and a CPU running C. From here, extend the SoC — add a peripheral on
another slot (GPIO, timer), or write richer firmware.

- [How it works](../how-it-works.md) · [CLI reference](../cli.md)
- [Anvil developer docs](https://logismith.github.io/Anvil/) — SoC internals and
  the `soc.json` build config.
