# TWI Scanner for nRF52840 DK

This is a simple TWI (I2C) scanner example program for the **nRF52840 Development Kit** using **nRF5 SDK v17.1.0**. It scans all I2C addresses from 0x01 to 0x7F and logs any connected device addresses.

---

## 📁 Project Folder Structure

Create your project in the following path:

C:\nrf52sdk\nRF5_SDK_17.1.0_ddde560\examples\MY PROJECTS\twi_scanner\pca10056\blank\ses

ruby
Copy
Edit

⚠️ **Note:** The directory path should be exact. Otherwise, Segger Embedded Studio may throw errors like:

directory or file does not exist

yaml
Copy
Edit

---

## 📌 Key Features

- Scans all 127 possible TWI/I2C addresses.
- Uses TWI0 or TWI1 based on `sdk_config.h`.
- Logs detected addresses over RTT (via `nrf_log`).
- Uses GPIO pins **SCL = P0.27** and **SDA = P0.26**.
- Frequency: 400 kHz.

---

## 🔧 Hardware Requirements

- Nordic nRF52840 DK (PCA10056)
- One or more I2C/TWI devices (sensors, EEPROMs, etc.)
- Segger Embedded Studio
- J-Link debugger

---

## 📦 SDK Dependencies

Ensure the following SDK modules are enabled in `sdk_config.h`:

```c
#define TWI0_ENABLED 1
#define NRF_LOG_ENABLED 1
#define NRF_LOG_BACKEND_RTT_ENABLED 1
#define APP_ERROR_CHECK(...) // Already provided in app_error.h
🧠 Code Overview
c
Copy
Edit
#include <stdio.h>
#include "boards.h"
#include "app_util_platform.h"
#include "app_error.h"
#include "nrf_drv_twi.h"
#include "nrf_gpio.h"
#include "nrf_log.h"
#include "nrf_log_ctrl.h"
#include "nrf_log_default_backends.h"
These headers include essential functions and macros for TWI, GPIO, logging, and error handling.

🔁 TWI Instance Setup
c
Copy
Edit
#if TWI0_ENABLED
#define TWI_INSTANCE_ID     0
#elif TWI1_ENABLED
#define TWI_INSTANCE_ID     1
#endif

static const nrf_drv_twi_t m_twi = NRF_DRV_TWI_INSTANCE(TWI_INSTANCE_ID);
Determines whether to use TWI0 or TWI1 based on sdk_config.h.

⚙️ TWI Initialization
c
Copy
Edit
void twi_init(void)
{
    const nrf_drv_twi_config_t twi_config = {
       .scl = 27,
       .sda = 26,
       .frequency = NRF_DRV_TWI_FREQ_400K,
       .interrupt_priority = APP_IRQ_PRIORITY_HIGH,
       .clear_bus_init = false
    };
    
    ret_code_t err_code = nrf_drv_twi_init(&m_twi, &twi_config, NULL, NULL);
    APP_ERROR_CHECK(err_code);

    nrf_drv_twi_enable(&m_twi);
}
Configures the TWI with SCL on P0.27 and SDA on P0.26 at 400kHz.

🚀 Main Logic
c
Copy
Edit
int main(void)
{
    uint8_t address;
    uint8_t sample_data;
    bool detected_device = false;

    APP_ERROR_CHECK(NRF_LOG_INIT(NULL));
    NRF_LOG_DEFAULT_BACKENDS_INIT();

    NRF_LOG_INFO("TWI scanner started.");
    NRF_LOG_FLUSH();

    twi_init();

    for (address = 1; address <= 127; address++)
    {
        if (nrf_drv_twi_rx(&m_twi, address, &sample_data, sizeof(sample_data)) == NRF_SUCCESS)
        {
            detected_device = true;
            NRF_LOG_INFO("TWI device detected at address 0x%x.", address);
        }
        NRF_LOG_FLUSH();
    }

    if (!detected_device)
    {
        NRF_LOG_INFO("No device was found.");
        NRF_LOG_FLUSH();
    }

    while (true) { /* Loop forever */ }
}
Scans all I2C addresses and prints which ones acknowledge communication.

📝 License
text
Copy
Edit
Copyright (c) 2016 - 2021,
Nordic Semiconductor ASA. All rights reserved.

Redistribution and use in source and binary forms, with or without modification,
are permitted provided that the following conditions are met:
(Full Nordic SDK license terms included in source file comments.)
✅ Output Example (via RTT Viewer)
css
Copy
Edit
TWI scanner started.
TWI device detected at address 0x68.
TWI device detected at address 0x3C.
🛠️ Debugging Tips
Ensure correct I2C wiring (SCL, SDA, GND).

If no device is found, double-check pull-ups and power.

Make sure sdk_config.h has correct logging and TWI settings.

Use RTT Viewer to see logs in real time.

🧩 Customization
To use different GPIOs, modify .scl and .sda in twi_config.

Change scan range or bus frequency as needed.

You can integrate this with sensors like MPU6050, BMP280, etc
