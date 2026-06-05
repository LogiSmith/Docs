# Installation — WSL2

!!! note "Stub"
    Skeleton — headings below are the intended structure, to be filled in.

Running the toolchain on Windows via WSL2 (Ubuntu). The toolchain install is the
same as native Ubuntu; the extra work is the WSL setup and forwarding the FPGA
board's USB into the WSL distro.

## 1. Set up WSL2 + Ubuntu

<!-- TODO: wsl --install, choose Ubuntu, update -->

## 2. Install the toolchain

Follow the [Ubuntu (native) guide](ubuntu.md) inside your WSL distro.

<!-- TODO: note any WSL-specific differences (paths, performance, /mnt) -->

## 3. Forward the FPGA board over USB (usbipd-win)

Needed only for `anvil program`. On native Linux this is not required.

<!-- TODO:
     - install usbipd-win on Windows (winget install usbipd)
     - usbipd list / bind / attach --wsl --busid <BUSID>
     - re-attach after replug
     - udev rules inside WSL -->

## Verify

```bash
anvil doctor
```

<!-- TODO: expected output -->
