# 05 — ADC Continuous Sampling with DMA

Continuously samples an analog input via **ADC1** using **DMA** to transfer conversions into a circular buffer in the background, with no CPU intervention needed per-sample. GPIO toggling on the half-complete/complete DMA callbacks gives a visual/scope-measurable indicator of how long each half of the buffer takes to fill.

Builds on the DMA concepts from the UART project, this time using DMA in **peripheral-to-memory** direction instead of memory-to-peripheral.

## What It Does

- ADC1 is configured for **continuous conversion mode**, sampling a single channel (ADC_CHANNEL_1) at 12-bit resolution
- `HAL_ADC_Start_DMA()` kicks off continuous ADC conversions that are automatically streamed into a 4096-entry buffer (`adc_buf`) via DMA — the ADC never has to wait on the CPU to move each sample
- Two HAL callback functions fire automatically as the DMA fills the buffer:
  - `HAL_ADC_ConvHalfCpltCallback()` — fires when the **first half** of the buffer is full; sets PA5 high
  - `HAL_ADC_ConvCpltCallback()` — fires when the **entire buffer** is full (and DMA wraps back to the start, since `DMAContinuousRequests` is enabled); clears PA5 low
- This produces a square-wave-like toggle on PA5 whose period corresponds to how long it takes to fill half the ADC buffer — useful for scoping/verifying actual sample throughput

```c
void HAL_ADC_ConvHalfCpltCallback(ADC_HandleTypeDef* hadc) {
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_SET);
}

void HAL_ADC_ConvCpltCallback(ADC_HandleTypeDef* hadc) {
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_RESET);
}
```

## Peripherals / Concepts

- **ADC continuous conversion mode** — free-running sampling without needing to manually re-trigger each conversion
- **DMA in peripheral-to-memory direction** — moving ADC conversion results into RAM automatically, freeing the CPU entirely during sampling
- **DMA half-transfer and full-transfer interrupts** — using both callback points to measure/observe timing, not just completion
- **Circular buffering pattern** — `DMAContinuousRequests = ENABLE` combined with the ADC's continuous mode keeps the buffer perpetually refilling
- GPIO used as a debug/timing signal (toggled from within an ISR-context callback) rather than for its own sake

## Hardware

- NUCLEO-G474RE
- Analog signal source wired to the pin mapped to ADC1 Channel 1 (check your specific pin mapping in the `.ioc` — commonly PA0 or PC0 depending on channel-to-pin assignment on this MCU)
- PA5 (onboard LD2) used as the visual/scope output for buffer-fill timing
- UART4 and a GPIO output on PA10 are configured in this project but not yet wired into the main loop — left over from earlier experimentation, worth removing or repurposing for actually streaming the ADC data out over serial in a follow-up.

## Notes

- `ADC_BUF_LED` (4096 samples) is a fairly large buffer — worth experimenting with smaller sizes to see how it changes the half/complete callback toggle rate on the scope, as a way to build intuition for DMA transfer timing versus ADC sample rate.
- Since the callbacks run in interrupt context, keep any future additions to them short — avoid blocking calls like `HAL_Delay()` inside `HAL_ADC_ConvHalfCpltCallback`/`HAL_ADC_ConvCpltCallback`.
