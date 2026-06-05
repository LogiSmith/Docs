# Tutorial 1 — Blinky

The "hello world" of FPGAs: blink an on-board LED. You'll go through the whole
loop — scaffold a project, write a tiny design, map pins, build, and program —
and meet the two most basic building blocks: a **clock** and a **counter**.

**You'll learn:** the build → program loop · what a clock is for · a counter as a
clock divider · mapping a signal to a physical pin (XDC).

**Prerequisite:** toolchain [installed](../installation/index.md) — `anvil doctor`.

## 1. Scaffold a project

```bash
mkdir blinky && cd blinky
anvil init --board Nexys-A7-100T
```

## 2. The design (`top.sv`)

Replace `top.sv` with:

```systemverilog
// Blinky — toggle LED0 at ~1.5 Hz from the 100 MHz clock
module top (
    input  logic clk,    // 100 MHz board clock
    output logic led      // LED0
);
    logic [25:0] count = 0;

    always_ff @(posedge clk)
        count <= count + 1;

    assign led = count[25];
endmodule
```

**Why a counter?** The board clock runs at 100 MHz — far too fast to see. The
counter increments every clock tick; its top bit `count[25]` flips only every
2²⁵ ticks (~0.34 s), so the LED toggles a few times per second. A counter used
this way is a simple **clock divider**.

## 3. Map the pins (`.xdc`)

Synthesis only knows the signal names `clk` and `led` — the XDC ties them to
real pins on *this* board. Replace the project's `.xdc` with:

```tcl
set_property PACKAGE_PIN E3 [get_ports {clk}]
set_property IOSTANDARD LVCMOS33 [get_ports {clk}]
create_clock -period 10.0 [get_ports {clk}]

set_property PACKAGE_PIN H17 [get_ports {led}]
set_property IOSTANDARD LVCMOS33 [get_ports {led}]
```

- `PACKAGE_PIN` — which physical pin (E3 is the 100 MHz clock; H17 is LED0).
- `IOSTANDARD` — the pin's voltage standard (3.3 V CMOS here).
- `create_clock -period 10.0` — tells the tools the clock period is 10 ns (100 MHz).

!!! warning "Plain syntax only"
    F4PGA needs plain `set_property` lines — **not** Vivado's
    `set_property -dict { … }` form. See [Troubleshooting](../troubleshooting.md#constraints-xdc).

## 4. Build and program

```bash
anvil build       # sources → bitstream (~20 s)
anvil program     # flash the board
```

**LED0 should now blink ~1.5 times a second.** 🎉

## What you learned

- The full flow: `init` → edit HDL → map pins → `build` → `program`.
- A clock drives sequential logic (`always_ff @(posedge clk)`).
- A counter divides a fast clock down to something visible.
- The XDC binds HDL ports to physical pins.

## Experiment

- Change `count[25]` to `count[24]` (faster) or `count[26]` (slower).
- Drive a second LED from a different bit.
- Output the whole counter to `led[7:0]` for a binary "scanner".

## Next

→ **Switches & LEDs** — reading inputs and combinational logic. *(Write-up soon;
run it now with `anvil init --board Nexys-A7-100T --example switches-leds`.)*
