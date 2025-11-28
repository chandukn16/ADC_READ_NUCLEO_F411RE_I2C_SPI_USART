/*
 * display_74hc595.c
 *
 *  Created on: Nov 18, 2025
 *      Author: Admin
 */

/* display_74hc595.c
 * Minimal driver implementation. Uses HAL SPI transmit and a GPIO latch.
 *
 * Usage:
 * - call DISP_Init(&hspi1, LATCH_GPIO_Port, LATCH_Pin);
 * - call DISP_SetDigits(1,2,3,4); // set the decimal digits to show
 * - in main loop call DISP_RefreshOnce() frequently (it will update one digit per call)
 */

#include "display_system.h"

/* --- CONFIG: change these to match your module if needed --- */
/* If segments are active HIGH (1 turns LED segment ON) set to 1, otherwise 0 */
#define SEGMENT_ACTIVE_HIGH 0
/* If digit enable line is active HIGH (1 = enable that digit) set to 1, otherwise 0 */
#define DIGIT_ACTIVE_HIGH 1
/* Total digits on your module */
#define NUM_DIGITS 4

/* --- internal state --- */
static SPI_HandleTypeDef *hspi_ptr = NULL;
static GPIO_TypeDef *latch_port = NULL;
static uint16_t latch_pin = 0;
static uint8_t digits_buf[NUM_DIGITS] = {0,0,0,0};
static uint8_t current_digit = 0;
static uint32_t last_refresh_time = 0;
static uint16_t refresh_interval_ms = 2; /* 2ms per digit = 125Hz refresh rate */

/* 7-segment font for common-cathode (segments active HIGH)
   bit mapping assumed: bit0 = segment A, bit1 = B, bit2 = C, bit3 = D, bit4 = E, bit5 = F, bit6 = G, bit7 = DP
   Change mapping if your module uses different bit order.
*/
static const uint8_t seg_font[16] = {
    0x3F, /* 0: 0b0011_1111 */
    0x06, /* 1: 0b0000_0110 */
    0x5B, /* 2 */
    0x4F, /* 3 */
    0x66, /* 4 */
    0x6D, /* 5 */
    0x7D, /* 6 */
    0x07, /* 7 */
    0x7F, /* 8 */
    0x6F, /* 9 */
    0x77, /* A */
    0x7C, /* b */
    0x58, /* C */
    0x5E, /* d */
    0x79, /* E */
    0x71  /* F */
};

void DISP_Init(SPI_HandleTypeDef *hspi, GPIO_TypeDef *latchPort, uint16_t latchPin)
{
    hspi_ptr = hspi;
    latch_port = latchPort;
    latch_pin = latchPin;

    /* Reset display buffer */
    for (int i=0;i<NUM_DIGITS;i++) digits_buf[i]=0;
    current_digit = 0;
    last_refresh_time = 0;
}

void DISP_SetDigits(uint8_t d0, uint8_t d1, uint8_t d2, uint8_t d3)
{
    digits_buf[0] = d0;
    digits_buf[1] = d1;
    digits_buf[2] = d2;
    digits_buf[3] = d3;
}

void DISP_SetBrightness(uint8_t percent)
{
    if (percent == 0) refresh_interval_ms = 0;
    else {
        if (percent > 100) percent = 100;
        /* map 0..100% to 1..5 ms per digit (lower = dimmer, higher = brighter) */
        refresh_interval_ms = 1 + (uint16_t)((4 * percent) / 100);
    }
}

/* Compose the two bytes to send to 74HC595s:
   tx[0] = segments byte (8 bits)
   tx[1] = digit select byte (only low 4 bits used)
   NOTE: If your module requires the other order (digit byte first), swap before transmit.
*/
static void send2bytes(uint8_t segByte, uint8_t digitByte)
{
    uint8_t tx[2];
    /* Many PCBs expect the byte sent first to become the last shift register.
       If segments end up on digit outputs or vice versa, swap tx[0] and tx[1]. */
    tx[0] = segByte;
    tx[1] = digitByte;

    /* Latch low -> shift -> latch high to update outputs */
    HAL_GPIO_WritePin(latch_port, latch_pin, GPIO_PIN_RESET);
    HAL_SPI_Transmit(hspi_ptr, tx, 2, 10);
    HAL_GPIO_WritePin(latch_port, latch_pin, GPIO_PIN_SET);
}

void DISP_RefreshOnce(void)
{
    if (hspi_ptr == NULL) return;

    /* Non-blocking timing control */
    uint32_t current_time = HAL_GetTick();
    if (current_time - last_refresh_time < refresh_interval_ms) {
        return;
    }

    last_refresh_time = current_time;

    /* Build segment byte from font */
    uint8_t val = digits_buf[current_digit];
    uint8_t seg = 0;
    if (val < 16) seg = seg_font[val];
    else seg = 0x00; /* blank if out of range */

    if (!SEGMENT_ACTIVE_HIGH) seg = ~seg; /* invert if segments are active low */

    /* Build digit selection byte:
       assume digits are on lower 4 bits (bit0 = digit0, bit1 = digit1, ...).
    */
    uint8_t digitByte = (1 << current_digit);
    if (!DIGIT_ACTIVE_HIGH) digitByte = ~digitByte;

    /* If hardware maps digits to other bits, modify digitByte here. */

    send2bytes(seg, digitByte);

    /* Advance to next digit */
    current_digit++;
    if (current_digit >= NUM_DIGITS) current_digit = 0;
}
