# Installation — WSL2

Running the toolchain on Windows via WSL2. The toolchain itself is the same as on
native Ubuntu — the extra work is on the Windows side: a WSL distro with the right
kernel, and forwarding the FPGA board's USB into it.

A helper script does all of that and then runs the normal Ubuntu installer inside
the distro, so there is only one command to run.

!!! warning "Update WSL before you start"
    The board flow needs two kernel modules — `vhci-hcd` (USB/IP) and `ftdi_sio`
    (FTDI serial) — that only recent WSL kernels ship. Without them the board
    cannot be attached to WSL at all: no `anvil program`, no serial console.

    From an **Administrator** PowerShell:

    ```powershell
    wsl --update
    wsl --shutdown
    ```

    If WSL is not installed yet, `wsl --install` (also Administrator) installs it;
    reboot afterwards.

---

## Requirements

| Component | Minimum |
|-----------|---------|
| Windows | 10 build 19041+ (2004) or Windows 11 |
| WSL | 2.7.13.0 or newer |
| WSL kernel | 6.18.33.2 or newer (ships `vhci-hcd` + `ftdi_sio`) |
| Disk | ~25 GB free |
| usbipd-win | only to program a board — `winget install --exact dorssel.usbipd-win` |

Virtualization (VT-x / AMD-V) must be enabled in the BIOS/UEFI.

---

## Automatic installation

The [`toolchain-setup`](https://github.com/LogiSmith/toolchain-setup) helper
prepares the WSL side and then runs the Linux installer inside the new distro.
From a **normal (non-Administrator)** PowerShell:

```powershell
irm https://raw.githubusercontent.com/LogiSmith/toolchain-setup/main/wsl-setup.ps1 -OutFile "$env:TEMP\wsl-setup.ps1"; powershell -ExecutionPolicy Bypass -File "$env:TEMP\wsl-setup.ps1"
```

!!! note "Why not `irm ... | iex`?"
    Piping a script straight into `iex` is the usual Windows one-liner, but this
    one is interactive: `exit` inside `iex` ends the **whole PowerShell session**,
    so a failed check would close your window along with the message explaining
    how to fix it. Saving the file first keeps the output on screen — and lets you
    pass [options](#options).

Prefer to read it before running (good practice for any remote script):

```powershell
git clone https://github.com/LogiSmith/toolchain-setup.git
cd toolchain-setup
notepad .\wsl-setup.ps1        # review
.\wsl-setup.ps1
```

If PowerShell refuses to run it (execution policy):

```powershell
powershell -ExecutionPolicy Bypass -File .\wsl-setup.ps1
```

It is safe to re-run: an existing distro is reused, never overwritten, and the
Linux installer it calls is idempotent.

### What it checks first

Before creating anything, it verifies the Windows side and stops with an
explanation if something is missing:

```
==> 1. WSL present
  [ok] Windows build 26200
  [ok] WSL is installed

==> 2. WSL and kernel versions
  [ok] WSL 2.7.13.0 (tested 2.7.13.0)
  [ok] kernel 6.18.33.2 (tested 6.18.33.2)
  [ok] .wslconfig present, no custom kernel configured
  [ok] 1193.2 GB free on C:
  [ok] usbipd-win present (USB passthrough to WSL)
```

| Check | Why it matters |
|-------|----------------|
| Windows build | WSL 2 needs 19041+ |
| WSL version | older releases lack `wsl --install --name` |
| WSL kernel | older kernels have no `vhci-hcd` / `ftdi_sio` |
| `.wslconfig` | a custom `kernel=` line is allowed, but reported — a hand-built kernel may lack the modules |
| Free disk | Conda + F4PGA architecture definitions + Verilator need room |
| usbipd-win | required later to attach the board |

An outdated WSL stops the run right here, with the `wsl --update` instructions —
rather than failing an hour into the install.

### Creating the distro

It creates a distro named **`anvil`** (use `-Name` for something else). If that
name already exists you are asked whether to reuse it or pick another one —
nothing is ever deleted.

WSL then asks for the Linux username and password itself:

```
==> 4. WSL distro 'anvil'
  creating 'anvil' from image 'Ubuntu-24.04' (this downloads a few hundred MB) ...
Downloading: Ubuntu 24.04 LTS
Installing: Ubuntu 24.04 LTS
Distribution successfully installed. It can be launched via 'wsl.exe -d anvil'
  [ok] created 'anvil'

  Now create your Linux user account.
  WSL asks for the username and password itself; this script neither sets
  nor reads them. It continues on its own when you are done.

Provisioning the new WSL instance anvil
This might take a while...
Create a default Unix user account: geek
New password:
Retype new password:
passwd: password updated successfully
```

Pick any username you like — it is your Linux account, unrelated to your Windows
login. Once the password is set the script carries on by itself; there is nothing
to close.

!!! info "About the password prompts"
    You are asked for a password **more than once**, by two different programs:

    1. **WSL**, when creating the account above (`New password` / `Retype`).
    2. **`sudo`**, once the Linux installer starts — it needs root to install
       packages, and can ask again if the install runs long.

    Neither prompt comes from the installer itself, and **nothing stores your
    password**. It goes straight into WSL and `sudo`; the scripts never read it,
    never write it to a file, and never send it anywhere. Nothing is kept beyond
    the normal `sudo` timeout inside that distro.

### Options

Pass these after the script path, e.g.

```powershell
powershell -ExecutionPolicy Bypass -File "$env:TEMP\wsl-setup.ps1" -Name my-distro -InstallArgs '--no-test'
```

| Flag | Effect |
|------|--------|
| `-Name <name>` | Distro name (default `anvil`) |
| `-Image <image>` | Image from `wsl --list --online` (default `Ubuntu-24.04`) |
| `-Reuse` | Reuse the distro if it exists, without asking |
| `-SkipAnvil` | Only prepare and check WSL; do not install the toolchain |
| `-SkipDriverCheck` | Skip the kernel-module check (no board programming) |
| `-AllowOlder` | Accept a WSL/kernel older than the tested versions |
| `-InstallArgs '<flags>'` | Passed through to the Linux installer, e.g. `'--no-test'` |

### How to know it worked

The helper is **fail-fast**: it stops at the first check that fails, before
changing anything further. Two green markers tell you it went through.

First, the WSL side is ready and the toolchain install starts:

```
WSL distro 'anvil' is ready. (user: geek, kernel: 6.18.33.2)
  starting the Anvil toolchain install in 5 ...
```

Then the Linux installer runs — 20–40 minutes for the Conda environment, F4PGA
architecture definitions and the Verilator build — and ends with its own banner:

```
✓ Toolchain installed and all tests passed
```

followed by the helper's closing summary:

```
OK — toolchain installed in WSL distro 'anvil'.

  Enter it with:            wsl -d anvil
  Make it your default:     wsl --set-default anvil
  Attach the board first:   usbipd list  ->  usbipd attach --wsl --busid <BUSID>
  Then, inside WSL:         anvil doctor
```

If you see those, you are done. A red `[ERROR]` line instead always names the step
that failed and what to do about it — fix that and re-run.

### After it finishes

```powershell
wsl -d anvil
```

and inside the distro:

```bash
anvil doctor
```

---

## Manual setup

If you would rather do the Windows side yourself.

### 1. WSL2 + Ubuntu

From an Administrator PowerShell:

```powershell
wsl --install                       # first time only, then reboot
wsl --update                        # make sure the kernel is current
wsl --shutdown
wsl --install --distribution Ubuntu-24.04 --name anvil
```

Confirm the distro is version 2 and the kernel is new enough:

```powershell
wsl --list --verbose
wsl --version
```

### 2. Check the kernel modules

Inside the distro:

```bash
sudo modprobe vhci-hcd && sudo modprobe ftdi_sio && lsmod | grep -E "vhci|ftdi"
```

Both `vhci_hcd` and `ftdi_sio` must appear. If they do not, either update WSL
(`wsl --update`, then `wsl --shutdown`), or build a kernel with
`CONFIG_USBIP_VHCI_HCD` and `CONFIG_USB_SERIAL_FTDI_SIO` enabled and point
`%USERPROFILE%\.wslconfig` at it — see
[Microsoft's custom kernel guide](https://learn.microsoft.com/windows/wsl/use-custom-kernel).

To load them on every boot:

```bash
printf 'vhci-hcd\nftdi_sio\n' | sudo tee /etc/modules-load.d/anvil-usbip.conf
```

### 3. Install the toolchain

Follow the [Ubuntu (native) guide](ubuntu.md) inside your WSL distro — the steps
are identical.

---

## Forward the FPGA board over USB (usbipd-win)

Needed only for `anvil program`. On native Linux this is not required.

Install it once, on the **Windows** side:

```powershell
winget install --exact dorssel.usbipd-win
```

Then, each time you plug the board in (Administrator PowerShell):

```powershell
usbipd list                                 # find the board's BUSID
usbipd attach --wsl --busid <BUSID>
```

Inside WSL, confirm it arrived:

```bash
lsusb                 # should list Future Technology Devices (0403:6010)
ls /dev/ttyUSB*       # serial console device
```

!!! note "Re-attach after replugging"
    Unplugging the board (or running `wsl --shutdown`) detaches it. Run
    `usbipd attach --wsl --busid <BUSID>` again — the kernel modules stay loaded.

---

## Verify

Inside the distro:

```bash
anvil doctor
```

All required rows (`Python`, `sv2v`, `F4PGA / Conda`) should report `OK`. Then
build a real example end to end:

```bash
mkdir -p ~/projects/uart-hello && cd ~/projects/uart-hello
anvil init --board Nexys-A7-100T --example uart-hello
anvil build
```

A bitstream under `build/<target>/top.bit` means the toolchain works.

---

## Troubleshooting

**"Required kernel modules not found"** — the running kernel has no `vhci-hcd` /
`ftdi_sio`. Note that `wsl --version` reports the kernel WSL *ships*, while a
running WSL keeps the kernel it booted with until every distro stops. So run
`wsl --shutdown` first and try again; if it persists, `wsl --update`.

**"detected custom (non-Microsoft) kernel"** — there is a `kernel=` line in
`%USERPROFILE%\.wslconfig`. That is allowed, but a hand-built kernel only works
where its modules are installed. Comment the line out and run `wsl --shutdown` to
return to the stock kernel.

**The distro starts as `root`** — the Linux installer refuses to run as root. WSL's
default user is a setting separate from the account, so set it explicitly:

```powershell
wsl --manage anvil --set-default-user <username>
```

**`anvil` not found after installing** — open a new shell, or run
`source ~/.bashrc`, so the alias and Conda are active.
