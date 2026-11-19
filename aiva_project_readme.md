# AIVA – Agentic Intelligent Voice Assistant

![Status](https://img.shields.io/badge/Status-Work%20in%20Progress-yellow)
![Platform](https://img.shields.io/badge/Platform-ESP32--S3-blue)
![Framework](https://img.shields.io/badge/Framework-ESP--IDF-orange)

**Developer:** Atharva Nakod  
**Project Type:** ESP32-S3 Based Hybrid AI Voice Assistant  

AIVA (Agentic Intelligent Voice Assistant) is a personal embedded systems project focused on building a compact, hands-free voice assistant powered by the ESP32-S3 microcontroller. The system combines local offline command processing with cloud-based conversational AI, providing real-time audio feedback and visual status updates via an OLED display.

---

## 📑 Table of Contents

- [Project Overview](#project-overview)
- [Project Objectives](#project-objectives)
- [Key Features](#key-features)
- [System Architecture](#system-architecture)
- [Hardware Components](#hardware-components)
- [GPIO Pin Configuration](#gpio-pin-configuration)
- [Software and Development Environment](#software-and-development-environment)
- [Firmware Foundation and Upstream Attribution](#firmware-foundation-and-upstream-attribution)
- [Build and Flashing Instructions](#build-and-flashing-instructions)
- [Current Development Status](#current-development-status)
- [Safety and Hardware Notes](#safety-and-hardware-notes)
- [Future Improvements](#future-improvements)
- [Acknowledgements](#acknowledgements)

---

## 📖 Project Overview

Embedded voice assistants typically rely entirely on continuous cloud connectivity, creating latency and rendering the device unusable when offline. AIVA is designed as a hybrid solution built around the **ESP32-S3-WROOM-1 (N16R8)** module. 

My goal with this project is to integrate digital I2S audio input/output hardware with an I2C OLED interface, adapting an open-source ESP-IDF voice framework to run reliably on custom breadboard/prototype hardware. The device captures voice commands through an omnidirectional I2S microphone, processes intent either locally or via cloud APIs over Wi-Fi, and outputs synthesized speech through a Class-D amplifier and speaker.

---

## 🎯 Project Objectives

- Build a functional, compact hardware prototype for hands-free voice interaction using the ESP32-S3.
- Implement reliable full-duplex or half-duplex I2S audio capture (INMP441) and playback (MAX98357A).
- Provide clear visual system feedback (listening, processing, speaking, network status) on a 0.96-inch SSD1306 OLED screen.
- Establish a dual-mode architecture capable of handling basic hardware actions offline while routing complex queries to cloud LLMs when connected to Wi-Fi.
- Customize and tune the underlying ESP-IDF firmware specifically for the AIVA pinout and acoustic enclosure.

---

## ✨ Key Features

To maintain an accurate representation of the project, features are separated into current development targets and planned capabilities:

### In Progress / Hardware Integration
- **Digital Audio Capture:** Interfacing the INMP441 I2S MEMS microphone with the ESP32-S3.
- **Audio Output Pipeline:** Driving a 3W speaker using the MAX98357A I2S Class-D amplifier.
- **Visual Status Display:** Rendering system states and text feedback on a 128×64 SSD1306 I2C OLED display.
- **Custom Board Definition:** Configuring ESP-IDF board support files to match custom GPIO mappings.

### Planned Functionality
- **Offline Command Mode:** Local execution of basic device commands (such as LED control) without requiring an active internet connection.
- **Online Conversational Mode:** Wi-Fi connectivity to stream audio/text to cloud-based AI/LLM services for complex question answering and natural voice responses.
- **Acoustic Tuning:** Gain adjustment and noise floor calibration for improved speech recognition accuracy.

---

## 🏗️ System Architecture

AIVA operates around a hybrid dual-mode execution flow:

```
                   +-----------------------+
                   |   INMP441 I2S Mic     |
                   +-----------+-----------+
                               | (I2S Audio Input)
                               v
+-------------+    +-----------------------+    +-----------------------+
| SSD1306     |<---|   ESP32-S3-WROOM-1    |--->| MAX98357A I2S Amp     |
| OLED (I2C)  |    |       (N16R8)         |    +-----------+-----------+
+-------------+    +-----------+-----------+                |
                               |                            v
                 +-------------+-------------+    +-----------------------+
                 |                           |    |   4Ω 3W Speaker       |
                 v                           v    +-----------------------+
      [ Offline Mode ]               [ Online Mode ]
      - Local Command Parsing        - Wi-Fi Connection
      - Direct GPIO/LED Control      - Cloud AI / LLM Processing
      - No Internet Required         - Conversational Audio Reply
```

1. **Offline Mode:** Designed for low-latency, local execution. The microcontroller processes selected basic voice commands directly on-device to trigger simple hardware actions (such as LED-related actions) without relying on external servers.
2. **Online Mode:** When connected to Wi-Fi, the system leverages cloud-based AI/LLM services to handle complex queries, generate conversational responses, and stream the resulting audio back through the MAX98357A amplifier.

---

## 🛠️ Hardware Components

The prototype uses the following exact hardware configuration:

| Component | Specification |
| :--- | :--- |
| **Microcontroller** | ESP32-S3-WROOM-1, N16R8 (16MB Flash, 8MB PSRAM) |
| **Microphone** | INMP441 I2S Digital Microphone |
| **Audio Amplifier** | MAX98357A I2S Class-D Amplifier |
| **Speaker** | 2-inch, 4 Ohm, 3 Watt |
| **Display** | 0.96-inch SSD1306 OLED, 128×64 pixels (I2C) |
| **Connectivity** | Wi-Fi (2.4 GHz via ESP32-S3) |
| **Development Framework** | ESP-IDF |

---

## 🔌 GPIO Pin Configuration

All peripheral connections are mapped to the ESP32-S3 as detailed below.

### INMP441 Microphone (I2S Input)

| Microphone Pin | ESP32-S3 GPIO | Function |
| :--- | :--- | :--- |
| **SCK / BCLK** | `GPIO 5` | Serial Clock / Bit Clock |
| **WS / LRCLK** | `GPIO 4` | Word Select / Left-Right Clock |
| **SD** | `GPIO 6` | Serial Data Out |
| **L/R** | `GND` | Channel Select (Tied to GND for Left Channel) |

### MAX98357A Audio Amplifier (I2S Output)

| Amplifier Pin | ESP32-S3 GPIO | Function |
| :--- | :--- | :--- |
| **BCLK** | `GPIO 15` | Bit Clock |
| **LRC** | `GPIO 16` | Left-Right Clock |
| **DIN** | `GPIO 7` | Digital Audio Data In |
| **VIN** | `5V` | Power Supply Input |
| **GND** | `GND` | Ground |

> **Note on Speaker Wiring:** The 4Ω 3W speaker connects directly to the `OUT+` and `OUT−` terminals on the MAX98357A. Because this is a Bridge-Tied Load (BTL) Class-D amplifier, **neither speaker terminal should ever be connected to GND**.

### SSD1306 OLED Display (I2C)

| OLED Pin | ESP32-S3 GPIO | Function |
| :--- | :--- | :--- |
| **SDA** | `GPIO 41` | I2C Data Line |
| **SCL** | `GPIO 42` | I2C Clock Line |
| **VCC** | `3.3V / 5V` | According to specific OLED module requirements |
| **GND** | `GND` | Ground |

---

## 💻 Software and Development Environment

To build and flash the firmware, set up the following development environment:

- **ESP-IDF:** Espressif IoT Development Framework (required for ESP32-S3 target support and build tools).
- **Python:** Required by ESP-IDF build scripts and dependency management.
- **Git:** Version control for cloning the repository and managing submodules.
- **Visual Studio Code (Recommended):** Configured with the official Espressif IDF extension for code navigation, building, and serial debugging.

---

## 🔗 Firmware Foundation and Upstream Attribution

The firmware architecture for AIVA is built upon the open-source **Xiaozhi ESP32** voice assistant project:
- **Upstream Repository:** [https://github.com/78/xiaozhi-esp32](https://github.com/78/xiaozhi-esp32)

The upstream project provides the core voice assistant framework, audio pipeline structure, and cloud communication protocols. I did not independently develop the original upstream framework; rather, my engineering work focuses on adapting and customizing it for my specific hardware setup:

- Creating a custom ESP32-S3 board configuration profile.
- Mapping and verifying custom GPIO assignments for I2S and I2C peripherals.
- Integrating the INMP441 microphone and MAX98357A amplifier into the audio subsystem.
- Integrating and configuring the 0.96-inch SSD1306 OLED display.
- Performing hardware-specific firmware customization, audio gain tuning, and stability testing.

*Please consult the upstream repository directly for its applicable software license and terms of use.*

---

## 🚀 Build and Flashing Instructions

Once your ESP-IDF environment is exported and active in your terminal, follow these steps to compile and flash the firmware to the ESP32-S3:

1. **Set the target chip to ESP32-S3:**
   ```bash
   idf.py set-target esp32s3
   ```

2. **Configure project settings (optional):**
   If you need to verify custom pin definitions, Wi-Fi credentials, or PSRAM settings (Octal PSRAM for N16R8):
   ```bash
   idf.py menuconfig
   ```

3. **Build the firmware:**
   ```bash
   idf.py build
   ```

4. **Flash the firmware to the board:**
   Replace `COMx` with your system's actual serial port (e.g., `COM3` on Windows, or `/dev/ttyACM0` / `/dev/ttyUSB0` on Linux):
   ```bash
   idf.py -p COMx flash
   ```

5. **Monitor serial output:**
   View boot logs and runtime diagnostics via the serial monitor (replace `COMx` with your port):
   ```bash
   idf.py -p COMx monitor
   ```
   *(Tip: You can combine flashing and monitoring into a single step using `idf.py -p COMx flash monitor`. Press `Ctrl+]` to exit the monitor.)*

---

## 📊 Current Development Status

**Status:** `Work in Progress`

- [x] Hardware component selection and breadboard wiring design.
- [x] GPIO pinout mapping for I2S audio and I2C display peripherals.
- [ ] Custom ESP-IDF board configuration and firmware adaptation.
- [ ] End-to-end verification of INMP441 audio capture and MAX98357A playback.
- [ ] SSD1306 OLED UI state integration.
- [ ] Implementation and testing of local offline commands (LED control).
- [ ] Full online conversational AI testing and audio tuning.

---

## ⚠️ Safety and Hardware Notes

- **Bridge-Tied Load (BTL) Amplifier Warning:** The MAX98357A outputs a differential PWM signal across `OUT+` and `OUT−`. Do **not** connect `OUT−` to `GND` or tie it to another amplifier's output, as this can permanently damage the IC.
- **Power Supply Stability:** Driving a 3W speaker at high volume causes brief current spikes on the `5V` rail. Ensure your USB port or external 5V power supply can deliver sufficient current to prevent ESP32-S3 brownout resets.
- **OLED Voltage Compatibility:** Check the onboard voltage regulator of your specific 0.96-inch SSD1306 module before wiring `VCC`. While most breakout boards support both `3.3V` and `5V`, verify your module's silkscreen markings to avoid damaging the display driver.
- **PSRAM Configuration:** Because the hardware uses the **ESP32-S3-WROOM-1 N16R8** module, ensure Octal SPI PSRAM is enabled in `menuconfig` so the audio buffers and network stack have adequate memory.

---

## 🔮 Future Improvements

- **Custom PCB & Enclosure:** Transition from breadboard prototyping to a compact custom PCB and a 3D-printed acoustic enclosure.
- **Wake-Word Optimization:** Fine-tune local wake-word detection sensitivity to minimize false triggers in noisy environments.
- **Expanded Offline Command Set:** Add more local voice commands for controlling additional peripherals without cloud latency.
- **Battery Management:** Integrate a LiPo battery charging and fuel-gauge circuit for portable, untethered operation.

---

## 🙏 Acknowledgements

- **Xiaozhi ESP32 Contributors:** Special thanks to [78/xiaozhi-esp32](https://github.com/78/xiaozhi-esp32) for developing and sharing the open-source ESP32 voice assistant firmware foundation that makes this project possible.
- **Espressif Systems:** For the comprehensive ESP-IDF framework and ESP32-S3 technical documentation.