# LogiSmith Docs

**An end-to-end, fully open-source FPGA toolchain — built for teaching and
learning digital design.** No proprietary tools, no licenses: synthesize,
simulate and program FPGAs with an open flow (F4PGA = Yosys + VPR), driven by a
small CLI called **Anvil**.

```bash
curl -fsSL https://raw.githubusercontent.com/LogiSmith/toolchain-setup/main/install.sh | bash
```

---

## Start here

1. **[Install](installation/index.md)** the toolchain — [Ubuntu](installation/ubuntu.md) · [WSL2](installation/wsl.md) · [Docker](installation/docker.md)
2. **[Getting started](getting-started.md)** — from an empty folder to a bitstream on the board
3. **[Blinky tutorial](tutorials/blinky.md)** — your first design, explained step by step

---

## Documentation

#### Installation
[Overview](installation/index.md) · [Ubuntu](installation/ubuntu.md) · [WSL2](installation/wsl.md) · [Docker](installation/docker.md)
— set up the toolchain and its dependencies.

#### Learn
[Getting started](getting-started.md) · [How it works](how-it-works.md) · [Tutorials](tutorials/index.md)
— the build→program loop, the open FPGA flow explained, and a guided learning path.

#### Reference
[CLI reference](cli.md) · [Supported boards](supported-boards.md) · [Troubleshooting & FAQ](troubleshooting.md)
— every command, the supported hardware, and fixes for common issues.

#### Developer & source
[Anvil developer docs](https://logismith.github.io/Anvil/) (internals & file formats) ·
[Anvil repo](https://github.com/LogiSmith/Anvil) ·
[toolchain-setup](https://github.com/LogiSmith/toolchain-setup) (the installer)

---

## Why open-source for teaching

- **No licenses, no lock-in** — students keep the toolchain after the course.
- **Reproducible** — one command gives a whole lab the same working environment.
- **Fast & modest hardware** — runs on a laptop, or on Windows via WSL2.
- **Transparent** — every layer is inspectable, from RTL to bitstream.
