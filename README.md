# STM32 Learning

A hands-on log of embedded firmware projects built while learning the STM32 ecosystem on an **STM32G474RE Nucleo board**. Each subfolder is a self-contained STM32CubeIDE project covering a specific peripheral or concept, built from scratch using STM32CubeMX for configuration and the STM32 HAL library.

This repo exists to document practical, hardware-verified experience with the STM32 platform — every project here has been built, flashed, and tested on physical hardware, not just simulated.

## Hardware

- **Board:** NUCLEO-G474RE
- **MCU:** STM32G474RETx (Arm Cortex-M4, FPU, 170 MHz)
- **Toolchain:** STM32CubeIDE 2.2.0 + STM32CubeMX + GNU Tools for STM32 (arm-none-eabi-gcc)

## Projects

| # | Project | Peripherals / Concepts | Status |
|---|---------|------------------------|--------|
| 01 | [Blinky](./01-blinky) | GPIO output, HAL_Delay, clock configuration | ✅ Complete |
| 02 | [I2C Temp Sensor over UART](./02-i2c-uart-tmp102) | I2C master read/write, UART transmit, TMP102 sensor, bit manipulation | ✅ Complete |
| 03 | [FreeRTOS Multi-Threaded Blink](./03-freertos-blink) | FreeRTOS/CMSIS-OS v2, thread creation, task priorities, hardware timer as RTOS tick | ✅ Complete |
| 04 | [UART Transmit via DMA](./04-uart-dma) | UART + DMA, direct register control, DMA callback registration | ✅ Complete |
| 05 | [ADC Continuous Sampling with DMA](./05-adc-dma) | ADC continuous mode, DMA peripheral-to-memory, half/full transfer callbacks | ✅ Complete |

_(Table will grow as more tutorial modules are completed.)_

## What Each Project Folder Contains

Every numbered project directory includes:
- Full STM32CubeIDE project (`Core/`, `Drivers/`, `.project`, `.cproject`)
- The `.ioc` file used to generate the peripheral/clock configuration in CubeMX
- A project-level `README.md` with what the project demonstrates, any issues hit along the way, and how they were resolved

## Skills Demonstrated

- STM32CubeMX peripheral and clock configuration
- STM32 HAL driver usage (C)
- DMA configuration for both peripheral-to-memory (ADC) and memory-to-peripheral (UART) transfers
- FreeRTOS/CMSIS-OS task creation and scheduling
- STM32CubeIDE project setup, build system, and toolchain troubleshooting
- Hardware bring-up and on-target debugging (SWD/ST-Link)
- Peripheral/BSP conflict diagnosis (isolating and resolving competing HAL initialization calls)

## Roadmap

Following along with Digi-Key's STM32 getting-started series, with each video's topic becoming its own numbered project folder here. Upcoming topics include timers/PWM and interrupt-driven I/O.

## About

Documented by Andres Arzola while building embedded systems experience during a firmware/embedded engineering job search. See individual project READMEs for detailed notes, including debugging process for real issues encountered (toolchain configuration, project import, HAL setup).