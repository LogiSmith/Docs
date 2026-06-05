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

## Updating

Update the **whole toolchain** (Anvil + dependencies) to the latest tested set:

```bash
anvil update
```

This re-runs the [installer](https://github.com/LogiSmith/toolchain-setup), which
is idempotent: it moves Anvil to its latest published release and picks up any
bumped dependency versions. The long integration build is skipped by default
(`anvil doctor` still runs); pass flags through, e.g. `anvil update --minimal`.

## Checking your version

```bash
anvil version        # e.g. "anvil 1.0.0 (ff70179)"
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

!!! note "Status"
    The [Ubuntu](ubuntu.md) guide is complete. [WSL2](wsl.md) and
    [Docker](docker.md) are still skeletons.
