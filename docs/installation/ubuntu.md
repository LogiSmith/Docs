# Installation — Ubuntu (native)

Setting up the full toolchain on native Ubuntu (bare-metal or VM). Tested on
**Ubuntu 22.04 LTS** (x86_64).

!!! tip "On Windows?"
    Use the [WSL2 guide](wsl.md) instead — it reuses these same steps and adds
    the Windows-side USB forwarding needed to program a board.

There are two ways to install:

- **[Automatic](#automatic-installation)** — one command, recommended for most people.
- **[Manual](#manual-installation)** — step by step, if you want to understand or
  customise each part.

Both install the same tools at the fixed locations Anvil expects (`~/opt/anvil`,
`~/opt/sv2v/sv2v`, `~/miniconda3` with the `xc7` env, `~/opt/f4pga`).

---

## Automatic installation

The [`toolchain-setup`](https://github.com/LogiSmith/toolchain-setup) script
installs everything and runs an end-to-end build to verify it works.

```bash
curl -fsSL https://raw.githubusercontent.com/LogiSmith/toolchain-setup/main/install.sh | bash
```

Prefer to read it before running (good practice for any `curl | bash`):

```bash
git clone https://github.com/LogiSmith/toolchain-setup.git
cd toolchain-setup
less install.sh        # review
./install.sh
```

The script is **idempotent** — re-running it skips anything already installed.

### Options

| Flag | Effect |
|------|--------|
| `--minimal` | Skip optional tools (RISC-V toolchain + openFPGALoader/board) |
| `--no-board` | Skip only openFPGALoader + udev (no board programming) |
| `--no-test` | Skip the end-to-end build self-test |
| `--skip-apt` | Skip the apt steps (deps already present) |
| `-h`, `--help` | Show help |

Example — minimal install without board tooling (e.g. a headless test box):

```bash
./install.sh --minimal
```

### After it finishes

Open a new shell (or `source ~/.bashrc`) so the `anvil` alias and Conda are
active, then confirm:

```bash
anvil doctor
```

That's it — skip the manual section below. It remains as a reference for what the
script does and for troubleshooting.

---

## Manual installation

Follow these if you prefer to install step by step, or to understand what the
automatic script does. Keep the paths as-is unless you also change them in
`anvil.py`.

### 1. Build dependencies

```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get install -y build-essential flex bison libssl-dev \
    libelf-dev bc python3 pahole cmake pkg-config \
    libusb-1.0-0-dev libudev-dev git g++ gcc \
    libftdi1-dev libhidapi-dev zlib1g-dev unzip wget
```

Simulator + waveform viewer (needed for `anvil test`):

```bash
sudo apt install -y iverilog gtkwave
```

### 2. Anvil CLI

```bash
git clone https://github.com/LogiSmith/Anvil.git ~/opt/anvil
chmod +x ~/opt/anvil/anvil.py
echo 'alias anvil="python3 ~/opt/anvil/anvil.py"' >> ~/.bashrc
source ~/.bashrc
anvil --help
```

### 3. sv2v (SystemVerilog → Verilog)

Anvil converts every `.sv` source with `sv2v`. Install the release binary to
`~/opt/sv2v/sv2v`:

```bash
wget https://github.com/zachjs/sv2v/releases/download/v0.0.13/sv2v-Linux.zip -O /tmp/sv2v.zip
unzip -o /tmp/sv2v.zip -d /tmp
mkdir -p ~/opt/sv2v
cp /tmp/sv2v-Linux/sv2v ~/opt/sv2v/sv2v
chmod +x ~/opt/sv2v/sv2v
~/opt/sv2v/sv2v --version
```

### 4. F4PGA (synthesis & place/route)

#### 4.1 Miniconda

```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O /tmp/miniconda.sh
bash /tmp/miniconda.sh -b -p ~/miniconda3
source ~/miniconda3/etc/profile.d/conda.sh
echo "source ~/miniconda3/etc/profile.d/conda.sh" >> ~/.bashrc
```

#### 4.2 Conda environment

```bash
git clone https://github.com/chipsalliance/f4pga-examples ~/f4pga-examples
cd ~/f4pga-examples
conda env create -f environment.yml
conda activate xc7
```

> If `environment.yml` is not found, try `conda env create -f xc7/environment.yml`.

#### 4.3 Architecture definitions (Artix-7)

```bash
export F4PGA_INSTALL_DIR=~/opt/f4pga
export FPGA_FAM=xc7
mkdir -p $F4PGA_INSTALL_DIR/$FPGA_FAM

F4PGA_TIMESTAMP='20220907-210059'
F4PGA_HASH='66a976d'

wget -qO- https://storage.googleapis.com/symbiflow-arch-defs/artifacts/prod/foss-fpga-tools/symbiflow-arch-defs/continuous/install/${F4PGA_TIMESTAMP}/symbiflow-arch-defs-install-xc7-${F4PGA_HASH}.tar.xz | tar -xJC $F4PGA_INSTALL_DIR/${FPGA_FAM}

wget -qO- https://storage.googleapis.com/symbiflow-arch-defs/artifacts/prod/foss-fpga-tools/symbiflow-arch-defs/continuous/install/${F4PGA_TIMESTAMP}/symbiflow-arch-defs-xc7a100t_test-${F4PGA_HASH}.tar.xz | tar -xJC $F4PGA_INSTALL_DIR/${FPGA_FAM}
```

#### 4.4 Carry-chain patch

Complex designs (e.g. a PicoRV32 SoC) hit a known assertion bug in F4PGA's
carry-chain fixer. Apply this one-time patch:

```bash
sed -i 's/assert list_of_cells\[0\] is None, (bit, list_of_cells\[0\], cellname)/if list_of_cells[0] is not None: continue/' \
    ~/miniconda3/envs/xc7/lib/python3.7/site-packages/f4pga/utils/xc7/fix_xc7_carry.py
```

It relaxes an overly strict assertion that fails on multi-driver nets; the tool
continues correctly afterwards.

### 5. RISC-V toolchain *(optional — SoC firmware)*

Needed only for `anvil compile` (SoC projects with C/C++ firmware):

```bash
sudo apt install -y gcc-riscv64-unknown-elf
```

### 6. openFPGALoader *(optional — programming a board)*

> The apt/conda version is too old — build from source.

```bash
git clone https://github.com/trabucayre/openFPGALoader /tmp/ofl
cd /tmp/ofl && mkdir build && cd build
cmake .. && make -j$(nproc) && sudo make install
/usr/local/bin/openFPGALoader --Version
```

USB permissions for the board (Nexys A7 = FTDI `0403:6010`):

```bash
echo 'SUBSYSTEM=="usb", ATTR{idVendor}=="0403", ATTR{idProduct}=="6010", MODE="0666", GROUP="plugdev"' \
    | sudo tee /etc/udev/rules.d/99-openfpgaloader.rules
sudo usermod -aG plugdev $USER
sudo udevadm control --reload-rules && sudo udevadm trigger
```

### Verify

```bash
anvil doctor
```

All required tools (`Python`, `sv2v`, `F4PGA / Conda`) should report `OK`.
Optional rows may be `WARN` if you skipped sections 5–6.

Then build a real example end to end:

```bash
mkdir -p ~/projects/uart-hello && cd ~/projects/uart-hello
anvil init --board Nexys-A7-100T --example uart-hello
anvil build
```

A bitstream under `build/<target>/top.bit` means the toolchain works.
