# 03 — FreeRTOS Multi-Threaded Blink

Introduces **FreeRTOS** (via CMSIS-OS v2) on the STM32G474RE, running two separate threads that each toggle the same onboard LED (PA5) at different rates, to explore basic RTOS task creation and scheduling.

## What It Does

- `osKernelInitialize()` sets up the RTOS kernel before any peripherals-dependent code runs
- Two threads are created with `osThreadNew()`:
  - **blink01** — Normal priority, toggles PA5 and delays 500ms
  - **blink02** — Below-normal priority, toggles PA5 and delays 600ms
- `osKernelStart()` hands control over to the scheduler, which then runs both threads according to their priority and delay timings
- TIM6 is used as the RTOS time base, incrementing the HAL tick via `HAL_IncTick()` in `HAL_TIM_PeriodElapsedCallback()`

```c
void StartBlink01(void *argument)
{
  for(;;)
  {
    HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
    osDelay(500);
    osThreadTerminate(NULL);
  }
}
```

## Peripherals / Concepts

- **FreeRTOS / CMSIS-OS v2** — kernel initialization, thread creation (`osThreadNew`), thread attributes (priority, stack size), `osDelay` for non-blocking task delays
- **Task priorities** — `osPriorityNormal` vs `osPriorityBelowNormal`, and how the scheduler interleaves tasks of different priority
- **Hardware timer as RTOS tick source** — TIM6 configured as the FreeRTOS time base instead of SysTick (STM32CubeMX default when using FreeRTOS, since SysTick is often reserved for HAL timing)
- GPIO, clock configuration, BSP LED/button/COM init (carried over from earlier projects)

## Hardware

- NUCLEO-G474RE, onboard LED LD2 (PA5) — no external components required

## Notes

- Both threads currently target the **same GPIO pin (PA5)**, so the visible blink pattern is the combined toggle effect of both tasks rather than two independently observable LEDs — worth revisiting with a second LED/pin per thread to make the scheduling behavior visually distinguishable.
- Each thread body calls `osThreadTerminate(NULL)` **inside** the `for(;;)` loop rather than after it — as written, this means each thread actually terminates itself after its first delay/toggle cycle rather than running indefinitely, despite the `for(;;)` structure suggesting continuous execution. Something to be aware of / fix if the goal is a continuously-running periodic task.
