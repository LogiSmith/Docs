# Supported boards

The boards Anvil knows about. The live list is always:

```bash
anvil boards
```

| Board | FPGA | Part | Family |
|-------|------|------|--------|
| **Nexys A7 100T** (`Nexys-A7-100T`) | Xilinx Artix-7 (xc7a100t) | `xc7a100tcsg324-1` | artix7 |

!!! note "More boards coming"
    Only the Digilent **Nexys A7 100T** is supported for now (the board this was
    developed and tested on). Other Artix-7 / 7-series boards will be added — the
    F4PGA flow already covers several (e.g. Arty A7, Basys 3, Nexys Video).

## Selecting your board

Pass the board name (the key from `anvil boards`) when creating a project:

```bash
anvil init --board Nexys-A7-100T
```

## Adding a board yourself

A board is just an entry in `boards.json` plus a master XDC file:

1. Add an entry to `boards.json` (description, target, part, device, openFPGALoader
   board id, XDC filename).
2. Add the master constraints file under `xdc/<board>-Master.xdc`.
3. Verify with `anvil boards`.

See the [Anvil developer docs](https://logismith.github.io/Anvil/contributing/) for
the exact `boards.json` fields and an example.
