/* display_74hc595.h
 * Simple driver to output to a 4-digit 7-seg module driven by 74HC595 chain.
 *
 * Assumptions:
 * - Two daisy-chained 74HC595: one controls segments (8 bits), the other controls digit enables (4 bits used).
 * - SPI peripheral is used to clock MOSI/SCLK; RCLK (latch) is controlled with a GPIO.
 *
 * Configure SEGMENT_ACTIVE_HIGH and DIGIT_ACTIVE_HIGH to match your hardware.
 */

#ifndef __DISPLAY_74HC595_H
#define __DISPLAY_74HC595_H

#include "stm32f4xx_hal.h"
#include <stdint.h>

void DISP_Init(SPI_HandleTypeDef *hspi, GPIO_TypeDef *latchPort, uint16_t latchPin);
void DISP_SetDigits(uint8_t d0, uint8_t d1, uint8_t d2, uint8_t d3);
void DISP_RefreshOnce(void); /* call repeatedly to multiplex display (non-blocking for one digit) */
void DISP_SetBrightness(uint8_t percent); /* 0..100, affects per-digit on-time */

#endif /* __DISPLAY_74HC595_H */
