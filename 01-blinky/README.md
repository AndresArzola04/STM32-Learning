# 01 — Blinky

First STM32 project: toggling the onboard LED (LD2, connected to **PA5**) on the NUCLEO-G474RE using the STM32 HAL library.

Simple in concept, but getting from "empty project" to "blinking LED" ended up being a good crash course in how the STM32CubeMX / STM32CubeIDE toolchain actually fits together.

## What It Does

Configures **PA5** as a GPIO output and toggles it every 1 second using `HAL_GPIO_TogglePin()` and `HAL_Delay()`, blinking the Nucleo board's onboard LED.

```c
while (1)
{
    HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
    HAL_Delay(1000);
}
```

## Peripherals / Concepts

- GPIO configuration via STM32CubeMX
- STM32 HAL driver (`HAL_GPIO_TogglePin`, `HAL_Delay`, `HAL_Init`)
- System clock configuration (`SystemClock_Config`)
- STM32CubeIDE project build system (Makefile-based, GNU ARM toolchain)

## Build & Flash

1. Open `G474RE-Blinky.ioc` in STM32CubeMX (or import the project directly into STM32CubeIDE)
2. Build (Project → Build Project)
3. Flash to the Nucleo board over USB (Run → Debug or Run As → STM32 C/C++ Application)

## Debugging Notes

This project surfaced a few real toolchain issues worth documenting, since diagnosing them was most of the actual learning:

**1. HAL functions "undeclared" despite following the tutorial exactly**
The original project was created as an **Empty STM32 Project**, which only sets up the compiler and startup files — it doesn't pull in the HAL library at all. No `main.h`, no `stm32g4xx_hal.h`, no HAL source files in the build. Fix: generate the project from **STM32CubeMX** instead, which produces a full project with HAL drivers linked in and clock/GPIO init already scaffolded.

**2. CubeMX wizard missing from STM32CubeIDE's New Project menu**
As of STM32CubeIDE v2.0.0, the integrated CubeMX pin/clock configurator was removed from the IDE entirely and now only exists as a **standalone CubeMX application**. Tutorials made on older IDE versions assume an integrated "STM32 Project" wizard that no longer exists. Fix: install standalone STM32CubeMX, configure pins/clocks there, generate the project (with Toolchain/IDE set to STM32CubeIDE), then import the generated folder into CubeIDE via File → Import → Existing Projects into Workspace.

**3. Build and Run greyed out after import**
The imported project wasn't recognized as a proper C/C++ project (Properties showed only generic "Resource" settings, no "C/C++ Build" section). Root cause traced further to `make`/`gcc`/`g++` not being found in the build environment PATH, even though the ARM GCC toolchain was installed. Fix: in Preferences → C/C++ → STM32Cube → **Toolchain Manager**, the installed GNU Tools for STM32 build wasn't set as the workspace default. Setting it as default, restarting CubeIDE, and doing a clean re-import of a properly CubeMX-generated project resolved it.

**Takeaway:** most of the friction here was toolchain/environment configuration rather than the firmware code itself — a reminder that a good chunk of embedded work is making sure the build system and hardware are actually talking to each other before the application logic even matters.

## Hardware

- NUCLEO-G474RE (STM32G474RETx)
- Onboard LED LD2 (PA5) — no external components required
