# Getting started

From an empty folder to a design running on the board. Assumes the toolchain is
already [installed](installation/index.md) — check with:

```bash
anvil doctor
```

!!! warning "WSL only — attach the board first"
    On WSL2 the board is visible to Windows and invisible to Linux until you
    forward its USB into the distro, so `anvil program` below will not find it.
    Do it before you start, and again after every reboot or replug:
    [USB forwarding](installation/usb-forwarding.md). On native Linux there is
    nothing to do.

## Option A — start from an example (recommended first) { .step }

The fastest way to see the whole flow work:

```bash
anvil examples --board Nexys-A7-100T          # list what's available
mkdir hello && cd hello
anvil init --board Nexys-A7-100T --example uart-hello
anvil build                                   # sources → bitstream
anvil program                                 # flash the board
```

That's the full loop: scaffold → build → program.

## Option B — start a project from scratch { .step }

```bash
mkdir blinky && cd blinky
anvil init --board Nexys-A7-100T
```

This creates:

```
blinky/
├── config.json     # board + module config
├── top.sv          # your top-level design (edit this)
├── *.xdc           # pin constraints (copied from the board master)
├── tb/             # testbenches
└── Makefile        # auto-generated
```

Then:

1. **Edit `top.sv`** — write your design.
2. **Edit the `.xdc`** — uncomment the pins you use (it ships with all board pins).
3. **Build and program:**

   ```bash
   anvil build
   anvil program
   ```

## Adding reusable blocks (modules)

Pull in shared RTL (UART, APB, PicoRV32, …) instead of copy-pasting:

```bash
anvil modules                 # list available modules
anvil addmodule uart          # adds uart + its dependencies
anvil build
```

## Simulating a testbench

```bash
anvil test tb/my_tb.sv        # runs Icarus Verilog, writes tb/my_tb.vcd
gtkwave tb/my_tb.vcd &        # view waveforms
```

## Building a RISC-V SoC

Add a SoC module and Anvil scaffolds a `firmware/` folder; your C/C++ is compiled
and baked into the bitstream:

```bash
anvil addmodule picorv32-soc  # SoC detected → firmware/ template created
# edit firmware/src/main.cpp
anvil build                   # compiles firmware + synthesizes
anvil program
```

## Where to go next

- [CLI reference](cli.md) — every command and its options.
- [Anvil internals](https://logismith.github.io/Anvil/) — how modules, SoCs
  and the build flow work under the hood, and the `config.json` / `module.json` /
  `soc.json` schemas.

!!! tip "Serial output"
    Designs that use UART expose it over USB (e.g. `/dev/ttyUSB1`). View it with
    `sudo screen /dev/ttyUSB1 9600` (exit: `Ctrl+A` then `K`). The `sudo` is
    needed because `/dev/ttyUSB*` is root-owned unless your user is in the
    `dialout` group.
