# ADC_READ_NUCLEO_F411RE_I2C_SPI_USART

this GitHub repository contains:
Convert ADC value and get that value to display in different display module.
The NUCLEO-F411RE platform integrates a high-performance 12-bit SAR ADC subsystem capable of up to 2.4 MSPS conversion speed, supporting 16 external channels with programmable sampling cycles (3 to 480 cycles). Its reference voltage framework supports Vref+ = 3.3 V, enabling resolution scaling of 0.8 mV/LSB at full range. Sensor or process variables are acquired via GPIO-configured analog input pins (e.g., PA0–PA5), with DMA or polling modes ensuring deterministic sampling.

Captured ADC samples are formatted into scaled engineering units and multiplexed for visualization across three industrial-grade display mechanisms. A 4-digit LED module (74HC595-based) operates via SPI @ 1–4 MHz, enabling numeric visualization such as voltage or temperature with a multiplexing refresh rate of 1 ms per digit to ensure persistence of vision. A 128×64 SSD1306 OLED interface via I2C @ 400 kHz fast mode provides graphical viewing capability for trend, icon, and text output. Additionally, USART2 @ 115200 bps streams calibrated ADC data packets to a PC serial terminal for logging, traceability, and SCADA uplink compatibility.

Firmware architecture includes HAL-based ADC acquisition, ring-buffer processing, scaling transfer functions, and protocol drivers. This industrial approach enables sensor monitoring, process diagnostics, and edge-level HMI applications using standard communication buses (SPI, I2C, USART) with deterministic real-time response suitable for embedded instrumentation and automation domains.
