# CLI reference

Every `anvil` command. Run `anvil --help` for the same list, or `anvil <cmd>`
with no project to see usage hints.

## Project

| Command | What it does |
|---------|--------------|
| `anvil init --board <name>` | Scaffold a new project in the current folder |
| `anvil init --board <name> --example <name>` | Scaffold from a bundled example |
| `anvil status` | Show project info (board, modules, SoC, bitstream) |
| `anvil clean` | Remove the `build/` directory |

## Build & program

| Command | What it does |
|---------|--------------|
| `anvil compile` | Build firmware (C/C++ → `ram.v`) — SoC projects only |
| `anvil synth` | Synthesize the bitstream |
| `anvil build` | `compile` + `synth` |
| `anvil rebuild` | `clean` + `compile` + `synth` |
| `anvil program` | Flash the bitstream to the board |

Output bitstream: `build/<target>/top.bit`.

## Simulation

| Command | What it does |
|---------|--------------|
| `anvil test tb/<tb>.{v,sv} [src …]` | Compile + run a testbench (Icarus Verilog); writes `tb/<name>.vcd` |

## Modules

| Command | What it does |
|---------|--------------|
| `anvil modules` | List available modules (and which are in this project) |
| `anvil addmodule <name> …` | Add module(s) + their dependencies |
| `anvil removemodule <name> …` | Remove module(s) (refuses if still depended on) |
| `anvil installmodule` | Install the current directory as a reusable module |

See the [module system](https://logismith.github.io/Anvil/modules/) for how
modules and dependencies work.

## Boards & examples

| Command | What it does |
|---------|--------------|
| `anvil boards` | List supported boards |
| `anvil examples --board <name>` | List examples for a board |

## Toolchain

| Command | What it does |
|---------|--------------|
| `anvil doctor` | Check that all external tools are installed |
| `anvil update` | Update the whole toolchain (Anvil + dependencies) |
| `anvil version` | Print the installed Anvil version |

`anvil update` re-runs the [installer](https://github.com/LogiSmith/toolchain-setup)
(idempotent); the long integration build is skipped by default. Pass flags
through, e.g. `anvil update --minimal`.

---

For the file formats these commands read (`config.json`, `module.json`,
`soc.json`, `boards.json`), see the
[Anvil developer docs](https://logismith.github.io/Anvil/file-formats/).
