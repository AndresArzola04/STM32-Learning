# 07 — SPI EEPROM with Interrupts (Non-Blocking State Machine)

Builds on the blocking SPI EEPROM project by converting the write/read transactions to **interrupt-driven SPI** (`HAL_SPI_Transmit_IT` / `HAL_SPI_Receive_IT`), driven by a simple **finite state machine** in the main loop so the CPU isn't blocked waiting on each SPI transfer to complete.

## What It Does

A 6-state FSM runs continuously in `while(1)`, cycling through:

| State | Action |
|-------|--------|
| 0 | Build a 12-byte buffer (write instruction + address + 10 data bytes), send WREN, then kick off a **non-blocking** `HAL_SPI_Transmit_IT` write to the EEPROM |
| 1 | Wait for the `spi_xmit_flag` to be set by the transmit-complete interrupt callback |
| 2 | Poll the status register (blocking) until the **WIP** (write-in-progress) bit clears, confirming the EEPROM has finished its internal write cycle |
| 3 | Send the READ instruction + address, then kick off a **non-blocking** `HAL_SPI_Receive_IT` to read the 10 bytes back |
| 4 | Wait for the `spi_recv_flag` to be set by the receive-complete interrupt callback |
| 5 | Print the 10 received bytes over UART, delay 1 second, reset to state 0 |

```c
void HAL_SPI_TxCpltCallback(SPI_HandleTypeDef *hspi)
{
  HAL_GPIO_WritePin(GPIOB, GPIO_PIN_6, GPIO_PIN_SET);  // CS high - deselect
  spi_xmit_flag = 1;
}

void HAL_SPI_RxCpltCallback(SPI_HandleTypeDef *hspi)
{
  HAL_GPIO_WritePin(GPIOB, GPIO_PIN_6, GPIO_PIN_SET);
  spi_recv_flag = 1;
}
```

## Peripherals / Concepts

- **Interrupt-driven SPI** — `HAL_SPI_Transmit_IT` / `HAL_SPI_Receive_IT`, and the associated `HAL_SPI_TxCpltCallback` / `HAL_SPI_RxCpltCallback` weak-function overrides that HAL calls automatically when a transfer finishes
- **`volatile` flags for ISR-to-main-loop communication** — `spi_xmit_flag` / `spi_recv_flag` are set inside interrupt context and polled from the main loop; `volatile` prevents the compiler from optimizing away the reads
- **Finite state machine pattern** — a common technique for structuring non-blocking, multi-step peripheral sequences without needing an RTOS
- Chip select (CS) is raised **inside the completion callback** rather than immediately after the `_IT` call returns, since the transfer isn't actually done until the interrupt fires — a common mistake with non-blocking transfers is deselecting the device too early

## Hardware / Wiring

- **NUCLEO-G474RE**
- **25AA040A** SPI EEPROM on SPI1 (SCK/MISO/MOSI), CS on PB6 (software-controlled)
- UART output over USART2 (ST-Link VCP), 115200 baud

## Notes

- The WIP-polling step (state 2) and the final UART print + 1-second delay (state 5) are still blocking calls — only the SPI transmit/receive themselves were converted to interrupt-driven. A fully non-blocking version would also need to make the status-register poll and UART output non-blocking (e.g., timer-based or interrupt-driven UART), which is a natural next step from here.
- `HAL_GPIO_Init` in `MX_GPIO_Init()` in this version configures `LPUART1_RX_Pin`/`LPUART1_TX_Pin` rather than the PB6 CS pin explicitly shown in earlier SPI code — worth double-checking the `.ioc` GPIO assignment for PB6 output mode carried over correctly if this file was based on a modified/renamed pin configuration.
