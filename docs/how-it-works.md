# How it works

A conceptual tour of what happens between your HDL and a running FPGA — no
commands here, just the mental model. (For the code-level details, see the
[Anvil developer docs](https://logismith.github.io/Anvil/architecture/).)

## The open FPGA flow

Turning a design into something running on the chip is a pipeline. Vivado hides it
behind one button; here each stage is a separate open-source tool, and **Anvil
orchestrates them**:

```
  your design (top.sv) + modules + firmware
        │
        ▼  sv2v          SystemVerilog → Verilog
   Verilog sources
        │
        ▼  Yosys         synthesis: HDL → netlist of FPGA primitives
     netlist                 (LUTs, flip-flops, block RAM, …)
        │
        ▼  VPR           pack · place · route: map the netlist onto THIS
   placed & routed           specific chip's resources and wire them up
        │
        ▼  write_fasm → write_bitstream
   top.bit               the binary the FPGA loads to "become" your circuit
        │
        ▼  openFPGALoader
   FPGA running your design
```

### What each stage does

- **sv2v** — converts SystemVerilog to plain Verilog, because the synthesis flow
  expects Verilog.
- **Synthesis (Yosys)** — translates your HDL into a *netlist*: a graph of generic
  FPGA building blocks (look-up tables, flip-flops, block RAMs) and how they connect.
  Nothing about the physical chip yet.
- **Place & route (VPR)** — takes that netlist and decides *which actual resources*
  on your specific FPGA implement each block, and how the signals are wired through
  the chip's routing fabric.
- **Bitstream (FASM → `.bit`)** — serializes the placed-and-routed design into the
  binary configuration the FPGA loads at power-up to physically become your circuit.
- **Programming (openFPGALoader)** — sends that bitstream to the board over USB.

## What Anvil adds on top

The tools above are low-level and fiddly to wire together. Anvil adds a small
**project model** so you work in concepts, not plumbing:

- **`config.json`** — your project: which board, which modules, parameters.
- **Modules** — reusable, versioned RTL blocks (UART, APB, a CPU core…) with
  dependencies, instead of copy-pasting Verilog between projects.
- **SoC + firmware** — drop in a SoC module and your C/C++ is compiled and **baked
  into the design's memory** (see [memory init](troubleshooting.md#faq)), so the
  CPU boots your program from the bitstream.
- **One build command** — Anvil resolves sources, runs sv2v, generates the build
  files, and drives the whole F4PGA pipeline for you.

## Key concepts

- **Module** — a self-contained RTL block you can reuse. A project lists the
  modules it needs; each module declares its own dependencies, pulled in
  automatically.
- **SoC** — a special module that bundles a CPU (RISC-V PicoRV32) + a bus +
  peripherals, and carries the build config for compiling firmware.
- **Firmware** — the C/C++ program the SoC's CPU runs; compiled and embedded into
  on-chip RAM as part of the bitstream.

## Coming from Vivado?

| Vivado | Here (open-source) |
|--------|--------------------|
| Vivado IDE | `anvil` CLI + F4PGA tools |
| Synthesis | Yosys |
| Implementation (place & route) | VPR |
| Program device | openFPGALoader |
| XDC with `-dict` | XDC, **plain** `set_property` syntax |
| Block Memory Generator / `.coe` | generated `ram.v` (no external hex) |
| IP catalog | the module system |
| Project (`.xpr`) | `config.json` |

## Going deeper

- [Anvil developer docs](https://logismith.github.io/Anvil/) — the build flow in
  code, and the `config.json` / `module.json` / `soc.json` file formats.
- [CLI reference](cli.md) — the commands that drive each stage above.
