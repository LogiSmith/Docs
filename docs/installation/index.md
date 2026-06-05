# Installation

How to install Anvil and the external tools it drives. Pick the guide for your
platform:

- [Ubuntu (native)](ubuntu.md) — bare-metal or VM Linux
- [WSL2](wsl.md) — Windows + Ubuntu under WSL2 (adds USB forwarding)
- [Docker](docker.md) — containerised toolchain

After installing, verify everything from one command:

```bash
anvil doctor
```

## Dependencies at a glance

Anvil itself is pure-Python (standard library only) — the "dependencies" are the
external CLI tools it invokes.

| Tool | Needed for | Required? |
|------|-----------|-----------|
| Python 3.6+ | running `anvil` | yes |
| F4PGA + Conda | synthesis & place/route (`anvil synth`) | yes |
| sv2v | SystemVerilog → Verilog (every build) | yes |
| Icarus Verilog | simulation (`anvil test`) | optional |
| openFPGALoader | flashing the board (`anvil program`) | optional |
| RISC-V toolchain | SoC firmware (`anvil compile`) | optional |
| usbipd-win | forwarding USB to WSL (Windows host only) | WSL only |

!!! note "Stub"
    Skeleton — the per-platform guides below are still to be filled in.
