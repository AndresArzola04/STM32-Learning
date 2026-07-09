# 02 — I2C Temperature Sensor over UART (TMP102)

Reads temperature from a **TMP102 I2C temperature sensor** and transmits the reading over **UART (USART1)** at 115200 baud, printed as a formatted string every 500ms.

Builds on the GPIO/HAL basics from `01-blinky` by adding two more common embedded peripherals — I2C for sensor communication and UART for serial output/debugging.

## What It Does

1. Sends a register-select byte over I2C to tell the TMP102 which register to read from (the temperature register, `0x00`)
2. Reads back 2 bytes containing the raw temperature data
3. Combines and sign-extends the bytes into a 12-bit two's-complement value
4. Converts to a Celsius float using the TMP102's 0.0625°C/LSB resolution
5. Formats the result as a string and transmits it over UART
6. Repeats every 500ms

```c
buf[0] = REG_TEMP;
HAL_I2C_Master_Transmit(&hi2c1, TMP102_ADDR, buf, 1, HAL_MAX_DELAY);
HAL_I2C_Master_Receive(&hi2c1, TMP102_ADDR, buf, 2, HAL_MAX_DELAY);

val = ((int16_t)buf[0] << 4) | (buf[1] >> 4);
if (val > 0x7FF) { val |= 0xF000; }   // sign-extend for negative temps

temp_c = val * 0.0625;
sprintf((char*)buf, "%u.%02u C\r\n", ...);
HAL_UART_Transmit(&huart1, buf, strlen((char*)buf), HAL_MAX_DELAY);
```

## Peripherals / Concepts

- **I2C (Master mode)** — `HAL_I2C_Master_Transmit` / `HAL_I2C_Master_Receive`, register-pointer read pattern
- **UART** — `HAL_UART_Transmit`, 115200 baud 8N1, TX/RX FIFO threshold configuration
- **Bit manipulation** — combining two raw bytes into a 12-bit signed value, two's-complement sign extension
- **BSP helpers** — `BSP_LED_Init`, `BSP_PB_Init` (user button configured for EXTI interrupt), `BSP_COM_Init` for the Nucleo's onboard ST-LINK virtual COM port
- System clock configured via PLL from HSI (boosted voltage scaling for higher clock speed)

## Hardware / Wiring

- **NUCLEO-G474RE**
- **TMP102** temperature sensor breakout, wired to the I2C1 pins:
  - SDA → I2C1_SDA
  - SCL → I2C1_SCL
  - TMP102 7-bit address `0x48` (shifted left by 1 for the 8-bit HAL address format)
- UART output viewable over the Nucleo's onboard ST-LINK VCOM port (no external USB-serial adapter needed) — open a serial terminal (PuTTY, Tera Term, etc.) at 115200 baud to see live readings

## Notes

- Error handling is included on both the I2C transmit and receive calls — if either fails, an error string is sent over UART instead of a temperature reading, which is useful for catching wiring/addressing issues without a debugger attached.
- The 7-bit-to-8-bit address shift (`0x48 << 1`) is a common gotcha with I2C on STM32 HAL — the HAL API expects the address pre-shifted, unlike some other platforms.
