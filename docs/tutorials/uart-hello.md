# Tutorial 3 — UART hello

Time to send data back to your PC. This design transmits `Hello World!` over a
serial port — and introduces the most important workflow feature: **reusing a
module** instead of writing the UART from scratch.

**You'll learn:** the module system (`addmodule` + dependency resolution) ·
instantiating a module · a small sequencer (FSM) · reading serial on your PC.

**Prerequisite:** toolchain [installed](../installation/index.md); previous
tutorials done.

## 1. Start from the example

```bash
mkdir uart && cd uart
anvil init --board Nexys-A7-100T --example uart-hello
```

## 2. Reusing a module

You don't implement UART yourself — you pull in the `uart` module. Look at what
the project uses:

```bash
anvil modules        # lists modules; uart + its dependencies are in this project
```

The example's `config.json` lists:

```json
"modules": ["baud-rate-generator", "uart-tx", "uart-rx", "uart"]
```

You didn't add those four by hand. `uart` **depends on** the other three, so a
single command pulls in the whole tree:

```bash
anvil addmodule uart      # adds uart + baud-rate-generator + uart-tx + uart-rx
```

This is the package-manager part of Anvil: modules declare their dependencies,
and Anvil resolves them. (More in [How it works](../how-it-works.md).)

## 3. Using the module (`top.sv`)

The UART block is *instantiated* like any sub-module — parameters in `#( … )`,
ports wired with `.port(signal)`:

```systemverilog
uart #(
    .TX_LIMIT(TX_LIMIT),       // clock ticks per bit
    .RX_LIMIT(RX_LIMIT)
) u_uart (
    .clk(clk), .resetn(cpu_resetn),
    .tx_data(tx_data), .tx_start(tx_start),
    .tx(uart_tx), .tx_done(tx_done), .tx_busy(tx_busy),
    .rx(1'b1), .rx_data(), .rx_done(), .rx_busy()   // RX unused here
);
```

**`TX_LIMIT = 10415`** sets the baud rate: it's the number of 100 MHz clock ticks
per serial bit — `100 MHz / 9600 ≈ 10417`. The baud-rate generator counts those
ticks; that's how a fast clock becomes a 9600 bit/s signal.

### Sending the message

A small **sequencer** walks through the characters, one per UART transmission,
using the `tx_done` handshake to know when to send the next:

```systemverilog
logic [7:0] msg [0:12];   // "Hello World!\n"
// ...
always_ff @(posedge clk or negedge cpu_resetn) begin
    if (!cpu_resetn) begin idx <= 0; sent <= 0; tx_start <= 0; end
    else begin
        tx_start <= 0;
        if (!sent && (idx == 0 || tx_done)) begin
            if (idx <= 12) begin
                tx_data  <= msg[idx];   // load next char
                tx_start <= 1;          // pulse: send it
                idx      <= idx + 1;
            end else sent <= 1;          // done
        end
    end
end
```

This is a tiny **finite state machine**: load a character → pulse `tx_start` →
wait for `tx_done` → repeat. `cpu_resetn` is the board's reset button (active-low).

## 4. The pins

```tcl
set_property PACKAGE_PIN E3  [get_ports {clk}];         create_clock -period 10.0 [get_ports {clk}]
set_property PACKAGE_PIN D4  [get_ports {uart_tx}]      # USB-serial TX
set_property PACKAGE_PIN C12 [get_ports {cpu_resetn}]   # reset button
# (each also needs IOSTANDARD LVCMOS33)
```

## 5. Build, program, and watch

```bash
anvil build
anvil program
screen /dev/ttyUSB1 9600       # exit: Ctrl+A then K
```

Press the reset button — `Hello World!` appears in the terminal. 🎉

!!! note "On WSL"
    The board's USB must be forwarded into WSL and the FTDI driver loaded for
    `/dev/ttyUSB1` to appear — see the [WSL2 guide](../installation/wsl.md) and
    [Troubleshooting](../troubleshooting.md#programming-serial).

## What you learned

- **Modules** — reuse versioned RTL; `addmodule` resolves dependencies for you.
- **Instantiating** a module with parameters and port connections.
- A **sequencer/FSM** driving a handshake (`tx_start` / `tx_done`).
- Baud rate as a **clock divider** (`TX_LIMIT`).
- Reading serial output on the PC.

## Experiment

- Change the message text in `msg[]`.
- Change the baud rate: set `TX_LIMIT` to `100 MHz / <baud>` (and match it in `screen`).

## Next

→ **RISC-V SoC** — add a CPU and run C firmware on the FPGA. *(Write-up soon; run
it now with `anvil init --board Nexys-A7-100T --example hello-PicoRV`.)*
