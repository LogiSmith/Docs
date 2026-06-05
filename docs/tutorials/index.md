# Tutorials

A learning path that builds up one concept at a time. Each rung is a small,
complete design you build and run on the board.

| # | Tutorial | What you learn |
|---|----------|----------------|
| 1 | **[Blinky](blinky.md)** | The build → program loop, a clock, a counter, an output pin |
| 2 | **[Switches & LEDs](switches-leds.md)** | Inputs and combinational logic |
| 3 | **[UART hello](uart-hello.md)** | Reusing **modules**, clocked protocols, serial output |
| 4 | RISC-V SoC *(soon)* | A CPU + firmware: hardware/software co-design |

!!! note "Rung 4"
    The full write-up is coming. In the meantime it already ships as a runnable
    example — try it now:

    ```bash
    anvil init --board Nexys-A7-100T --example hello-PicoRV
    anvil build && anvil program
    ```

Before starting, make sure the toolchain is [installed](../installation/index.md)
(`anvil doctor`).
