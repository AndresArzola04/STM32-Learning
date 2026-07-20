# 06 — SPI EEPROM Read/Write (25AA040A)

Communicates with a **25AA040A SPI EEPROM** over SPI1, exercising the chip's write-enable, status register, byte-write, and byte-read instructions, with results printed over UART2 for verification.

## What It Does

1. Sends a startup message over UART to confirm the board is alive
2. Sets the **Write Enable Latch (WREN)** on the EEPROM — required before any write operation
3. Reads back the **status register (RDSR)** and prints it, to confirm the write-enable bit is set
4. Writes 3 test bytes (`0xAB 0xCD 0xEF`) to address `0x05` using the **WRITE** instruction
5. Polls the status register in a loop, masking out the **Write-In-Progress (WIP)** bit, to wait until the EEPROM finishes its internal write cycle before continuing
6. Reads the same 3 bytes back using the **READ** instruction and prints them over UART — confirming the write actually landed
7. Prints the final status register value

Chip select (CS, on PB6) is manually toggled low/high around each SPI transaction, since NSS is configured in software mode (`SPI_NSS_SOFT`) rather than hardware-managed.

```c
HAL_GPIO_WritePin(GPIOB, GPIO_PIN_6, GPIO_PIN_RESET);   // CS low - select EEPROM
HAL_SPI_Transmit(&hspi1, (uint8_t *)&EEPROM_WRITE, 1, 100);
HAL_SPI_Transmit(&hspi1, (uint8_t *)&addr, 1, 100);
HAL_SPI_Transmit(&hspi1, (uint8_t *)spi_buf, 3, 100);
HAL_GPIO_WritePin(GPIOB, GPIO_PIN_6, GPIO_PIN_SET);     // CS high - deselect
```

## Peripherals / Concepts

- **SPI (Master mode)** — `HAL_SPI_Transmit` / `HAL_SPI_Receive`, software-controlled chip select, MSB-first, mode 0 (CPOL=0, CPHA=0)
- **EEPROM command set** — WREN (write enable), RDSR (read status register), WRITE, READ instructions per the 25AA040A datasheet
- **Status polling** — waiting on the WIP (write-in-progress) bit rather than a fixed delay, since EEPROM write cycle time can vary
- **UART for debug/verification output** — printing raw register values and read-back data to confirm the SPI transaction actually worked at the byte level, not just that the calls returned successfully

## Hardware / Wiring

- **NUCLEO-G474RE**
- **25AA040A** SPI EEPROM, wired to SPI1:
  - SCK → SPI1_SCK
  - MISO → SPI1_MISO
  - MOSI → SPI1_MOSI
  - CS → PB6 (software-controlled GPIO output, not a hardware NSS pin)
- UART output viewable over the onboard ST-Link VCOM port (USART2, PA2/PA3) at 115200 baud

## Notes

- SPI clock is set to a fairly conservative baud rate prescaler (`SPI_BAUDRATEPRESCALER_128`) — worth experimenting with faster prescalers once basic read/write is confirmed working, to see how fast the EEPROM can reliably be driven.
- This project also calls `BSP_COM_Init(COM1, ...)` in addition to the manual `MX_USART2_UART_Init()` — the same pattern that caused silent UART failures in an earlier project (`04-uart-dma`), since `COM1` maps to the same physical USART2 peripheral being configured by hand. In this project the SPI/UART test sequence runs *before* `BSP_COM_Init()` is called, so it happens to work, but the redundant call should still be removed to avoid confusion or a re-init conflict if code is added later in the main loop.
