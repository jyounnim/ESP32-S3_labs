# ESP32-S3

**Language:** **English** | [한국어](./README_kr.md)

---

# 00. Development Environment Setup — VS Code + PlatformIO (ESP32-S3)

This is the starting document for the entire ESP32-S3 lab series, based on a non-OS (bare-metal, no RTOS — `setup()`/`loop()` structure) approach. Once you've set up your development environment using this document, proceed through the labs in order.

## Prerequisites

- A PC (Windows / macOS / Linux)
- An ESP32-S3 development board (this series is written for the **ESP32-S3-DevKitC-1, N16R8** — 16MB Quad Flash + 8MB Octal PSRAM)
- A USB cable that supports data transfer (not a charge-only cable)

---

## Step 1. Install VS Code

Install the version for your OS from [code.visualstudio.com](https://code.visualstudio.com). Skip this step if you already have it installed.

## Step 2. Install the PlatformIO IDE Extension

1. Open VS Code Extensions (`Ctrl+Shift+X`) → search for `PlatformIO IDE` → install
2. Restart VS Code after installation (the first load may take a few minutes)
3. Installation is complete once the PlatformIO ant icon appears in the left Activity Bar

## Step 3. Create a New Project

1. PlatformIO Home → **New Project**
2. Board: search for and select `Espressif ESP32-S3-DevKitC-1`
3. Framework: **Arduino**
4. Once created, `platformio.ini` and `src/main.cpp` are generated automatically

## Step 4. Configure N16R8 Flash/PSRAM Settings

The default board definition targets N8 (8MB Flash, no PSRAM). If you're using an N16R8 (16MB Quad Flash + 8MB Octal PSRAM) board, add the following settings to your `platformio.ini`.

```ini
[env:esp32-s3-devkitc-1]
platform = espressif32
board = esp32-s3-devkitc-1
framework = arduino
monitor_speed = 115200

; ---- N16R8 (16MB Quad Flash + 8MB Octal PSRAM) settings ----
board_build.flash_mode = qio
board_upload.flash_size = 16MB
board_build.partitions = default_16MB.csv
board_build.arduino.memory_type = qio_opi
build_flags =
    -DBOARD_HAS_PSRAM
```

> Since Flash is Quad (`qio`) and PSRAM is Octal (`opi`), we use `memory_type = qio_opi`. If you're using a different module variant (N16R8V, N32R8V, etc.), the Flash/PSRAM configuration may differ, so check the markings on the physical chip.

## Step 5. Hello World

```cpp
#include <Arduino.h>

void setup() {
  Serial.begin(115200);
  delay(1000);
  Serial.println("Hello, ESP32-S3!");
}

void loop() {
  Serial.println("Running...");
  delay(1000);
}
```

Verify by Build (checkmark icon in the bottom status bar) → Upload (arrow icon) → Serial Monitor (plug icon).

## Step 6. Check Your Core Version (Important)

Whether you should use the newer PWM API (`ledcAttach`) or the older one (`ledcSetup` + `ledcAttachPin`) depends on **the installed Arduino-ESP32 core version**. Check it with the code below.

```cpp
void setup() {
  Serial.begin(115200);
  delay(1000);
#if defined(ESP_ARDUINO_VERSION_MAJOR)
  Serial.printf("Arduino-ESP32 core: %d.%d.%d\n", ESP_ARDUINO_VERSION_MAJOR, ESP_ARDUINO_VERSION_MINOR, ESP_ARDUINO_VERSION_PATCH);
#endif
}
void loop() {}
```

- **core 2.x**: PWM uses `ledcSetup(channel, freq, res)` + `ledcAttachPin(pin, channel)` + `ledcWrite(channel, duty)`
- **core 3.x and above**: PWM uses `ledcAttach(pin, freq, res)` + `ledcWrite(pin, duty)`

The PWM-related examples in this series are **written for core 2.x (the legacy API)** — if you're on 3.x, switch to the newer API as you go (each example shows both approaches side by side).

## Setup Checklist

- [ ] VS Code + PlatformIO IDE installed
- [ ] Project created and N16R8 Flash/PSRAM settings applied
- [ ] Hello World build/upload/serial output verified
- [ ] Arduino-ESP32 core version checked

---
