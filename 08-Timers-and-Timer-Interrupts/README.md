# 08 — Timer Interrupt (TIM16)

Uses **TIM16** as a periodic interrupt source to toggle the onboard LED (PA5), rather than relying on `HAL_Delay()` in the main loop — a step toward timing that doesn't block the CPU.

## What It Does

- TIM16 is configured with a prescaler of 17000 and a period (auto-reload value) of 9999, producing a **1-second update interrupt** (170 MHz timer clock ÷ 17000 prescaler = 10 kHz tick rate; 10,000 ticks per period = 1 second)
- `HAL_TIM_Base_Start_IT()` starts the timer in interrupt mode
- Every time the timer rolls over (reaches its period and resets), HAL automatically calls `HAL_TIM_PeriodElapsedCallback()`, where the LED is toggled:

```c
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
{
  if (htim == &htim16)
  {
    HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
  }
}
```

- The `while(1)` loop itself is empty — all the actual work happens in the timer ISR, freeing the main loop entirely

## Peripherals / Concepts

- **Hardware timer configuration** — prescaler and auto-reload period calculation to derive a specific interrupt interval from the timer's input clock
- **Timer interrupt mode** — `HAL_TIM_Base_Start_IT()` vs. polling the counter manually
- **`HAL_TIM_PeriodElapsedCallback`** — the shared weak-function callback HAL uses for any timer's update event, requiring an `htim` instance check to identify which timer fired
- Moving time-based logic out of the main loop and into an ISR, as a step toward more responsive, non-blocking firmware design

## Hardware

- NUCLEO-G474RE, onboard LED LD2 (PA5) — no external components required

## Notes

- The file still contains a **commented-out alternate version** of this project that used `__HAL_TIM_GET_COUNTER()` to manually measure elapsed microseconds around a blocking `HAL_Delay(50)` call, printing the result over UART. That version demonstrates using a timer purely as a free-running counter/stopwatch (polling mode) rather than as an interrupt source — worth revisiting as a separate comparison point between polling-based and interrupt-based timing.
- UART2 and its FIFO threshold setup are still initialized in this project but currently unused in the active code path (the UART print statements are commented out alongside the polling-mode logic) — a leftover from the polling version.
