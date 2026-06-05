# Troubleshooting & FAQ

## First thing to try

Most setup problems show up here:

```bash
anvil doctor      # are all external tools installed and found?
```

If a required tool is missing or broken, re-run the installer (idempotent):

```bash
anvil update      # or: curl -fsSL …/install.sh | bash
```

---

## Installation & setup

| Symptom | Cause | Fix |
|---------|-------|-----|
| `symbiflow_synth: not found` | F4PGA / Conda env not set up | `anvil doctor`; re-run the installer |
| `sv2v not found` | sv2v missing | `anvil doctor`; re-run the installer |
| Installer stops after "created conda env" | (old installer) conda ToS prompt ate the script | Update to the current installer (`anvil update`) |
| `AssertionError … fix_xc7_carry.py` | carry-chain patch not applied | Re-run the installer — it applies the patch automatically |

## Synthesis / build

| Symptom | Cause | Fix |
|---------|-------|-----|
| `No top module detected` | empty / port-less top module | Add at least one port to `top` |
| `Duplicate blocks named '…'` | a signal driven twice in `top.sv` | Remove the redundant `assign` |
| Build is very slow the first time | SoC firmware + large design | Normal — simple designs ~20 s, a SoC ~1–2 min |

## Constraints (XDC)

F4PGA needs **plain** constraint syntax, not Vivado's `-dict`:

```tcl
# correct
set_property PACKAGE_PIN H17 [get_ports {led[0]}]
set_property IOSTANDARD LVCMOS33 [get_ports {led[*]}]

# wrong — causes "ValueError: max() arg is an empty sequence"
set_property -dict { PACKAGE_PIN H17 IOSTANDARD LVCMOS33 } [get_ports {led[0]}]
```

| Symptom | Cause | Fix |
|---------|-------|-----|
| `ValueError: max() arg is an empty sequence` | `-dict` XDC syntax, or no active pins | Use plain `set_property`; uncomment the pins you use |
| `Clock name does not correspond to any nets` | `create_clock` but the design has no clock port | Remove `create_clock` from the XDC |

## Programming & serial

| Symptom | Cause | Fix |
|---------|-------|-----|
| `Error: no device found` | board off or USB not attached | Power the board on; on WSL attach USB with usbipd |
| `/dev/ttyUSB*` missing | FTDI driver not loaded | `sudo modprobe ftdi_sio` |
| Serial monitor won't open the port | port held by another process | Use `screen /dev/ttyUSB1 9600` (exit: `Ctrl+A` then `K`) |
| Garbled UART output | baud mismatch / TX too fast | Use 9600 baud |

!!! note "WSL2 — USB to the board"
    On Windows the board's USB must be forwarded into WSL with usbipd-win, and the
    FTDI kernel module loaded. See the [WSL2 install guide](installation/wsl.md).

## Modules

| Symptom | Cause | Fix |
|---------|-------|-----|
| `Unknown module: ''` | `"depends": [""]` instead of `[]` | Use `"depends": []` in `module.json` |
| `Unknown module: '<name>'` | typo or not in the registry | Check `anvil modules` for the exact name |
| `module.json not found` | registry references a missing module dir | Use a name from `anvil modules` |

---

## FAQ

**Do I need Vivado or any vendor tool?**
No — the whole flow is open-source (F4PGA: Yosys + VPR).

**Which boards are supported?**
Run `anvil boards`. Adding a board = a `boards.json` entry + its XDC; see the
[Anvil developer docs](https://logismith.github.io/Anvil/contributing/).

**Does it run on Windows?**
Yes, under WSL2 — see the [WSL2 guide](installation/wsl.md).

**How do I update everything?**
`anvil update` updates the whole toolchain (Anvil + dependencies).

**Where does the bitstream end up?**
`build/<target>/top.bit`.

**Can I use SystemVerilog?**
Yes — `.sv` sources are converted automatically (via sv2v).

**How do I load firmware/memory contents? In Vivado I used a `.coe` / `$readmemh`.**
You don't point at an external hex file. `anvil compile` runs `programator.py`,
which **bakes the firmware directly into a generated `build/firmware/ram.v`** — the
memory contents become explicit `initial` assignments (`mem[i] = 32'h…;`) that get
synthesized into the bitstream. No `.coe`, no Block Memory Generator IP, no
`$readmemh` from a path. (Don't hand-edit `ram.v` — it's regenerated.)

*Why:* the `.coe` / Block-Memory-Generator init flow is **Vivado-specific** and
doesn't exist in F4PGA, and `$readmemh` from an external file is unreliable under
Yosys synthesis (path resolution + BRAM inference). Emitting self-contained
Verilog with inline `initial` values is portable, synthesizes into initialized
BRAM reliably, and behaves identically in simulation and on hardware.
