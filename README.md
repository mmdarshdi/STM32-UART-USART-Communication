# STM32 UART/USART Communication

![MCU](https://img.shields.io/badge/MCU-STM32F103C8T6-03234B)
![Language](https://img.shields.io/badge/language-C-555555)
![Framework](https://img.shields.io/badge/framework-STM32%20HAL-03234B)
![IDE](https://img.shields.io/badge/IDE-Keil%20%C2%B5Vision-2e7d32)
![License](https://img.shields.io/badge/license-MIT-yellow)

Embedded C firmware for the **STM32F103C8T6** demonstrating bidirectional serial communication with a PC terminal over **USART1**, using both blocking (polling) and interrupt-driven transfers from the STM32 HAL.

## Overview

The firmware receives a line of text from a PC serial terminal, echoes it back, and then sends a one-byte counter using interrupt-driven transmission. It shows the two main HAL transfer styles side by side: `HAL_UART_Receive` / `HAL_UART_Transmit` (blocking, with timeout) and `HAL_UART_Transmit_IT` (non-blocking).

## Features

- Bidirectional communication with a PC terminal at 115200 baud (8N1)
- Blocking receive with a 5 s timeout, up to 12 bytes per cycle
- Line echo: everything received before the carriage return (`\r`) is sent back
- Interrupt-driven transmit of a one-byte counter value
- `HAL_UART_RxCpltCallback` toggles the on-board LED (PC13) when an interrupt-driven receive completes
- Configured through STM32CubeMX (`USART.ioc` included)

## Hardware

| Component | Details |
|-----------|---------|
| MCU | STM32F103C8T6 (ARM Cortex-M3) |
| PC link | USB-to-TTL serial adapter (3.3 V logic) |
| Programmer / debugger | ST-Link over SWD |

## Pin Mapping

| Function | Pin | Notes |
|----------|-----|-------|
| USART1 TX | PA9 | Connect to the adapter's RX |
| USART1 RX | PA10 | Connect to the adapter's TX |
| On-board LED | PC13 | GPIO output, toggled in the RX-complete callback |
| SWDIO / SWCLK | PA13 / PA14 | Programming and debug |
| HSE crystal | PD0 / PD1 | System clock source |

Connect the grounds of the STM32 board and the USB-to-TTL adapter.

## Serial Settings

| Parameter | Value |
|-----------|-------|
| Peripheral | USART1 (asynchronous mode) |
| Baud rate | 115200 |
| Data bits | 8 |
| Parity | None |
| Stop bits | 1 |
| Flow control | None |
| Oversampling | 16 |

## System Configuration

| Item | Value |
|------|-------|
| System clock | HSE with PLL ×9, 72 MHz (8 MHz crystal) |
| APB1 / APB2 | 36 MHz / 72 MHz |
| Flash latency | 2 wait states |

## How It Works

Each pass through the main loop does the following:

1. **Receive:** `HAL_UART_Receive()` blocks until 12 bytes arrive or the 5 s timeout expires.
2. **Find end of line:** the buffer is scanned for the carriage return (`\r`, ASCII 13) that terminates the typed line.
3. **Echo:** only the bytes before the `\r` are sent back with `HAL_UART_Transmit()`.
4. **Interrupt transmit:** after a short delay, the first byte of the `data2` array is sent with `HAL_UART_Transmit_IT()`. The call returns immediately and the transfer completes in the background through the USART1 interrupt.
5. **Update:** after a second delay, `data2[0]` is loaded with the running counter `i`, which is then incremented, so the next cycle transmits the new value.

## Project Structure

```
.
├── Core/              Application sources (main.c) and peripheral setup
├── Drivers/           STM32 HAL and CMSIS drivers
├── MDK-ARM/           Keil µVision project
├── USART.ioc          STM32CubeMX configuration
├── LICENSE
└── README.md
```

## Build & Flash

**Requirements:** Keil µVision 5 (with the `Keil.STM32F1xx_DFP` pack) and an ST-Link programmer. STM32CubeMX is optional.

1. Clone the repository:
   ```bash
   git clone https://github.com/mmdarshdi/STM32-UART-USART-Communication.git
   ```
2. Open the `.uvprojx` file inside `MDK-ARM/` in Keil µVision.
3. Build the project (**F7**) and flash it to the board (**F8**).

To change the configuration, open `USART.ioc` in STM32CubeMX and regenerate the code. Your code inside the `USER CODE` blocks is preserved.

## Usage

1. Wire the USB-to-TTL adapter: adapter **TX → PA10**, adapter **RX → PA9**, **GND → GND**.
2. Open a serial terminal (PuTTY, Tera Term, etc.) at **115200, 8N1** and set Enter to send a carriage return (CR).
3. Type a short line (up to 12 characters including Enter) and press **Enter**. The board echoes the text back.
4. The counter byte is sent as a raw binary value, so it appears as a control character in a text terminal. Use the terminal's hex view to see its value.

## Possible Improvements

- Bounded line parsing, with the buffer index reset on every cycle
- Interrupt-driven reception (`HAL_UART_Receive_IT`) with a ring buffer
- DMA transfers for continuous streams
- A simple command parser (for example LED on/off or reading sensor values)

## Related Projects

- [STM32-Keypad-lcd-interface](https://github.com/mmdarshdi/STM32-Keypad-lcd-interface)
- [STM32-Timer-Start-Stop-Control](https://github.com/mmdarshdi/STM32-Timer-Start-Stop-Control)
- [STM32-IR-Pulse-Counter](https://github.com/mmdarshdi/STM32-IR-Pulse-Counter)

## License

Released under the MIT License. See [LICENSE](LICENSE) for details.

## Author

**Mohammadrahim Arshadi**, Biomedical Engineer focused on medical embedded systems
[GitHub](https://github.com/mmdarshdi) · [LinkedIn](https://www.linkedin.com/in/mmdarshdi)
