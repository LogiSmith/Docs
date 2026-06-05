# Tutorial 2 — Switches & LEDs

Now that [Blinky](blinky.md) covered the build loop and a *clocked* design, this
one is the opposite: **combinational logic** — outputs that follow inputs
instantly, with no clock at all. You'll read the 16 slide switches and mirror
them onto the 16 LEDs.

**You'll learn:** reading inputs · combinational vs sequential logic · buses ·
the `--example` workflow.

**Prerequisite:** toolchain [installed](../installation/index.md); [Blinky](blinky.md) done.

## 1. Start from the example

This design ships as an example, so scaffold straight from it (the example also
brings the matching pin constraints for all 32 signals):

```bash
mkdir sw-leds && cd sw-leds
anvil init --board Nexys-A7-100T --example switches-leds
```

## 2. The design (`top.sv`)

```systemverilog
module top (
    input  logic [15:0] sw,
    output logic [15:0] led
);
    assign led = sw;
endmodule
```

That's the whole design. Two things to notice:

- **No clock.** There's no `always_ff @(posedge clk)` — this is **combinational**
  logic. The output is a direct function of the input, recomputed continuously.
  (Blinky was **sequential**: it had state that updated on a clock edge.)
- **`assign`** is a *continuous assignment* — "led is always whatever sw is."
- **`[15:0]`** is a 16-bit **bus**: 16 switches and 16 LEDs handled as one vector.

## 3. The pins (`.xdc`)

The example's XDC maps each switch and LED to its physical pin, in plain syntax:

```tcl
set_property PACKAGE_PIN J15 [get_ports {sw[0]}]
set_property PACKAGE_PIN L16 [get_ports {sw[1]}]
# … sw[2] … sw[15] …
set_property IOSTANDARD LVCMOS33 [get_ports {sw[*]}]

set_property PACKAGE_PIN H17 [get_ports {led[0]}]
# … led[1] … led[15] …
set_property IOSTANDARD LVCMOS33 [get_ports {led[*]}]
```

Note `[get_ports {sw[*]}]` — one line sets the I/O standard for the whole bus.
There's no `clk` / `create_clock` here, because the design has no clock.

## 4. Build and program

```bash
anvil build
anvil program
```

Flip a switch — the LED above it turns on immediately. No clock, no delay.

## Combinational vs sequential

| | Blinky | Switches & LEDs |
|---|--------|-----------------|
| Has a clock? | yes | no |
| Logic type | sequential (state) | combinational |
| Output depends on | past (counter value) | present inputs only |
| Construct | `always_ff @(posedge clk)` | `assign` |

This is the most important distinction in digital design — almost everything is a
mix of these two.

## Experiment

Edit `top.sv` and rebuild to try real combinational logic:

```systemverilog
assign led = ~sw;                       // invert: LED on when switch is OFF
assign led = {sw[7:0], sw[8 +: 8]};     // swap the two halves
assign led[0] = sw[0] & sw[1];          // an AND gate
assign led[1] = sw[2] ^ sw[3];          // an XOR gate
```

(When driving individual bits, drive the rest too, or tie unused LEDs to 0.)

## What you learned

- Combinational logic with `assign` — outputs follow inputs, no clock.
- Inputs and multi-bit buses.
- Building from a bundled example with `--example`.

## Next

→ **UART hello** — pull in a reusable **module** and send characters to your PC
over serial. *(Write-up soon; run it now with
`anvil init --board Nexys-A7-100T --example uart-hello`.)*
