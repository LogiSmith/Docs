# Installation — Ubuntu (native)

!!! note "Stub"
    Skeleton — headings below are the intended structure, to be filled in.
    Tested on Ubuntu <!-- TODO: version, e.g. 22.04 LTS -->.

## Prerequisites

<!-- TODO: supported Ubuntu versions, apt update, build-essential, git, python3 -->

## 1. Anvil

<!-- TODO: clone repo into ~/opt/anvil, add the `anvil` alias to ~/.bashrc -->

## 2. F4PGA + Conda

<!-- TODO: install Miniconda, create the xc7 env, F4PGA install dir -->

## 3. sv2v

<!-- TODO: download release into ~/opt/sv2v/ (or apt/PATH) -->

## 4. Icarus Verilog *(optional — `anvil test`)*

<!-- TODO: apt install iverilog -->

## 5. openFPGALoader *(optional — `anvil program`)*

<!-- TODO: build from source / package, udev rules for board access -->

## 6. RISC-V toolchain *(optional — SoC firmware)*

<!-- TODO: install riscv64-unknown-elf-gcc/g++ -->

## Verify

```bash
anvil doctor
```

<!-- TODO: expected output, all-green example -->
