# LogiSmith Docs

User documentation for the **LogiSmith open-source FPGA toolchain** — an
end-to-end, fully open-source flow for synthesizing, simulating and programming
FPGAs, built for **teaching and learning** digital design without proprietary
tools or licenses.

At its core is **Anvil**, a small CLI that hides the raw F4PGA toolchain behind a
few commands, plus a reusable RTL module system and a RISC-V SoC flow.

---

## New here? Start here

1. **[Install](installation/index.md)** the toolchain for your platform —
   [Ubuntu](installation/ubuntu.md) · [WSL2](installation/wsl.md) · [Docker](installation/docker.md)
   *(one command:* `curl -fsSL …/install.sh | bash`*)*
2. **[Getting started](getting-started.md)** — from an empty folder to a bitstream
   on the board.
3. **[CLI reference](cli.md)** — every `anvil` command.

---

## What's here

| Section | For |
|---------|-----|
| [Installation](installation/index.md) | Setting up the toolchain + dependencies |
| [Getting started](getting-started.md) | Your first project, end to end |
| [CLI reference](cli.md) | All `anvil` commands and what they do |

## Beyond the basics

| Resource | What it is |
|----------|-----------|
| [Anvil developer docs](https://logismith.github.io/Anvil/) | Internals & reference: architecture, file formats (`config.json`, `module.json`, `soc.json`), the module system, contributing |
| [Anvil source](https://github.com/LogiSmith/Anvil) | The CLI itself |
| [toolchain-setup](https://github.com/LogiSmith/toolchain-setup) | The installer (`install.sh`) and pinned toolchain versions |

---

## Why open-source for teaching

- **No licenses, no vendor lock-in** — students keep the toolchain after the course.
- **Reproducible setup** — one command gives a whole lab the same working environment.
- **Fast & modest hardware** — runs on a laptop, or on Windows via WSL2.
- **Transparent** — every layer is inspectable, from RTL to bitstream.
