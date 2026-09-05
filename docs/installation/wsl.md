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

Four steps. Run them in order from a **normal (non-Administrator)** PowerShell.

### Step 1 — Run the installer

```powershell
irm https://raw.githubusercontent.com/LogiSmith/toolchain-setup/main/wsl-setup.ps1 -OutFile "$env:TEMP\wsl-setup.ps1"; powershell -ExecutionPolicy Bypass -File "$env:TEMP\wsl-setup.ps1"
```

It first checks the Windows side (see [What it checks](#what-it-checks)), then
creates a WSL distro called **`anvil`** and downloads Ubuntu into it — a few
hundred MB, so give it a minute.

!!! failure "If it stops with a red `[ERROR]`"
    The message names the check that failed and how to fix it. By far the most
    common one is an outdated WSL, fixed from an **Administrator** PowerShell:

    ```powershell
    wsl --update
    wsl --shutdown
    ```

    Then run Step 1 again. If `irm` itself cannot download (proxy or TLS issues),
    fetch `wsl-setup.ps1` from
    [GitHub](https://github.com/LogiSmith/toolchain-setup) by hand and run
    `powershell -ExecutionPolicy Bypass -File .\wsl-setup.ps1`.

### Step 2 — Choose a username and password

WSL asks for these itself, and waits for you:

```
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

Pick anything for the username — this is your Linux account, unrelated to your
Windows login. **Remember the password: you need it in Step 3.** Nothing is echoed
while you type it, which is normal.

Once it says `password updated successfully`, the script carries on by itself —
there is no window to close and nothing to type.

### Step 3 — Enter that password again when `sudo` asks

The toolchain install starts automatically after a five-second countdown:

```
WSL distro 'anvil' is ready. (user: geek, kernel: 6.18.33.2)
  starting the Anvil toolchain install in 5 ...
```

It installs system packages, so `sudo` asks for **the same password you just
created** — and may ask again if the install runs long:

```
[sudo] password for geek:
```

!!! info "Your password is never stored"
    Both prompts come from WSL and `sudo`, not from the installer. The scripts
    never read your password, never write it to a file, and never send it
    anywhere. Nothing is kept beyond the normal `sudo` timeout inside the distro.

### Step 4 — Wait for the finish message

This part takes **20–40 minutes** (Conda environment, F4PGA architecture
definitions, Verilator build). You are done when you see both of these:

```
✓ Toolchain installed and all tests passed
```

```
OK — toolchain installed in WSL distro 'anvil'.

  Enter it with:            wsl -d anvil
  Make it your default:     wsl --set-default anvil
  Attach the board first:   usbipd list  ->  usbipd attach --wsl --busid <BUSID>
  Then, inside WSL:         anvil doctor
```

The first line is the Linux installer confirming its own end-to-end test passed;
the second is the helper's summary. Anything else — in particular a red
`[ERROR]` or `✗ Install failed during step: …` — means it did not finish; the
message says which step and what to do. Fix that and run Step 1 again, it is safe
to re-run.

### After it finishes

```powershell
wsl -d anvil
```

and inside the distro:

```bash
anvil doctor
```

### What it checks

Before creating anything, Step 1 verifies the Windows side and stops with an
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

If the distro name `anvil` is already taken you are asked whether to reuse it or
pick another one — nothing is ever deleted, and the whole script is safe to re-run.

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

---

## Manual setup

If you would rather read the helper before running it, or do the Windows side by
hand.

```powershell
git clone https://github.com/LogiSmith/toolchain-setup.git
cd toolchain-setup
notepad .\wsl-setup.ps1        # review it, then run it if you like
.\wsl-setup.ps1
```

The rest of this section is what that script does, step by step, if you prefer to
do it yourself.


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

Click an entry to expand it.

??? failure "`[ERROR] Required kernel modules not found`"

    The running kernel has no `vhci-hcd` / `ftdi_sio`, so the board cannot be
    attached to WSL.

    Careful with versions here: `wsl --version` reports the kernel WSL *ships*,
    but a running WSL keeps whatever kernel it booted with until **every** distro
    stops. The two can disagree.

    ```powershell
    wsl --shutdown          # boot onto the shipped kernel, then re-run
    wsl --update            # if it still fails (Administrator)
    ```

    The message also tells you whether `/lib/modules/<running kernel>` exists in
    the distro. If it says `MISSING`, the kernel and the distro's modules do not
    match — see the next entry.

??? warning "`[warn] detected custom (non-Microsoft) kernel`"

    There is an active `kernel=` line in `%USERPROFILE%\.wslconfig`.

    A custom kernel is allowed, but it only works where its modules are
    installed — a distro created later will not have them. To go back to the
    stock kernel, comment the line out and restart the VM:

    ```powershell
    notepad "$env:USERPROFILE\.wslconfig"    # put # in front of the kernel= line
    wsl --shutdown
    ```

??? failure "The distro starts as `root`"

    The Linux installer refuses to run as root. WSL's default user is a setting
    separate from the account itself, so a distro can hold a perfectly good
    account and still start as root.

    ```powershell
    wsl --manage anvil --set-default-user <username>
    ```

??? failure "`anvil: command not found` after installing"

    The `anvil` alias and Conda are set up in `~/.bashrc`, which an already-open
    shell has not read.

    ```bash
    source ~/.bashrc        # or just open a new shell
    ```

??? question "Can I start over?"

    Yes — the distro is disposable and nothing outside it is touched. This
    permanently deletes it and everything inside:

    ```powershell
    wsl --unregister anvil
    ```

    Then run Step 1 again. To keep the old one instead, install alongside it with
    a different name: `-Name anvil2`.
