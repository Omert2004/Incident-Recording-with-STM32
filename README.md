# 🚨 Incident Recording with STM32

[![Platform](https://img.shields.io/badge/platform-STM32G071RB-blue.svg)]()
[![Language](https://img.shields.io/badge/language-C-green.svg)]()
[![License](https://img.shields.io/badge/license-Educational-lightgrey.svg)]()
[![Framework](https://img.shields.io/badge/framework-STM32CubeIDE-orange.svg)]()
[![Hardware](https://img.shields.io/badge/IMU-MPU6050-red.svg)]()

---

## 📘 Project Overview

**Incident Recording with STM32** is an embedded system designed to **record IMU sensor data before and after an event trigger** (button press).  
Developed on the **Nucleo-G071RB** using **STM32 HAL**, the system stores samples directly into the MCU’s **Flash memory** while maintaining precise 10 ms sampling intervals.

The project was implemented during a **20-day internship at TEİ TUSAŞ Engine Industries, Inc. (TEI)** in the **Software Development Department**.

---

## ⚙️ Key Features

✅ **10 ms periodic IMU sampling (TIM3 interrupt)**  
✅ **Event-based recording** — 100 samples before + 150 samples after a button press  
✅ **Circular buffer** — ensures newest 100 samples always available  
✅ **RTC timestamps** stored with each record  
✅ **Flash logging** — uses internal Flash pages 54–63 (128 KB MCU Flash)  
✅ **Compliance:** MISRA-C (2012) & DO-178C aviation standards  
✅ **Fully interrupt-driven architecture (no polling)**  

---

## 🧠 System Architecture
### 🧩 Data Flow

```text
          ┌────────────────────────┐
          │     MPU6050 (GY-521)   │
          │  (Acceleration + Gyro) │
          └──────────┬─────────────┘
                     │  I²C Communication
                     ▼
          ┌────────────────────────┐
          │   STM32G071RB MCU      │
          │                        │
          │ ┌────────────────────┐ │
          │ │ TIM3 (10 ms Timer) │ │  → Triggers periodic IMU sampling
          │ └────────────────────┘ │
          │            │
          │            ▼
          │ ┌────────────────────┐ │
          │ │ Circular Buffer    │ │  → Stores last 100 samples
          │ └────────────────────┘ │
          │            │
          │   ┌────────┴────────┐
          │   │  User Button    │ (EXTI Interrupt)
          │   └────────┬────────┘
          │            ▼
          │  Event Trigger → Write Data to Flash
          │
          │ ┌────────────────────┐
          │ │ Flash Memory       │ │  → Logs 100 past + 150 future samples
          │ └────────────────────┘
          │
          └────────────────────────┘
```
---

### ⚙️ Task Breakdown
| Module    | Function                             |
|:-----------|:-------------------------------------|
| **I²C**    | Communication with MPU6050           |
| **TIM3**   | Periodic 10 ms interrupts            |
| **EXTI**   | Button press detection               |
| **RTC**    | Timestamping (hh:mm:ss)              |
| **FLASH**  | Logging 250 samples per event        |
| **Buffer** | FIFO circular buffer (100 entries)   |

---

## 🧩 Hardware Connections

| STM32 Pin   | GY-521 Pin  | Function        |
|:-------------|:------------:|:----------------|
| 3V3          | VCC          | Power           |
| GND          | GND          | Ground          |
| PB9 (D14)    | SDA          | I²C Data        |
| PB8 (D15)    | SCL          | I²C Clock       |
| PC13         | User Button  | Trigger input   |

- **I²C address:** `0x68` (AD0 = GND)  
- **Sampling period:** 10 ms  
- **Flash page capacity:** 2 KB (256 records × 8 bytes)  

---

## 💾 Data Structure

Each Flash record is **8 bytes**:

| Field      |    Type    | Size (bytes) | Description           |
|:-----------|:-----------:|:-------------:|:----------------------|
| `Pitch`    | int16_t     | 2             | Tilt around X-axis    |
| `Roll`     | int16_t     | 2             | Tilt around Y-axis    |
| `Hours`    | uint8_t     | 1             | RTC hour              |
| `Minutes`  | uint8_t     | 1             | RTC minute            |
| `Seconds`  | uint8_t     | 1             | RTC second            |
| `Padding`  | uint8_t     | 1             | Structure alignment   |

---

## 📊 Flash Memory Map

| Flash Page | Start Address | Usage |
|-------------|---------------|--------|
| Page 54 | `0x0801B000` | 1st Event |
| Page 55 | `0x0801B800` | 2nd Event |
| Page 56-63 | `...` | Up to 10 events |

Each event = **250 samples × 8 bytes = 2000 bytes ≈ 2 KB per page.**

---

## 🧰 Development Environment

| Component          | Tool                         |
|:-------------------|:-----------------------------:|
| **MCU**            | STM32G071RB (Nucleo-G071RB)  |
| **Sensor**         | MPU6050 (GY-521)             |
| **IDE**            | STM32CubeIDE                 |
| **Configurator**   | STM32CubeMX                  |
| **Language**       | C (HAL Library)              |
| **OS**             | Windows / Ubuntu             |
| **Debugger**       | ST-Link                      |
| **Version Control**| GitHub                       |

---

## 🧪 Verification

- Verified Flash writes using **STM32CubeProgrammer**
- Checked address increments and record alignment (`0x0801B320` → 100 samples OK)
- Confirmed correct circular-buffer flushing and post-event sampling
- Observed process timing (~7 s for full write; optimization planned)

---

## 🚀 Future Improvements

- 🧵 Buffer Flash writes into **50-sample packets** to reduce erase cycles  
- 💾 Integrate **SD-card logging** for extended storage  
- 🧮 Implement **checksum or CRC validation**  
- ⚡ Introduce **DMA-based I²C acquisition** to improve performance  

---

## 🧑‍💻 Author

**Oğuz Mert Coşkun**  
📧 [oguzmertcoskun@gmail.com](mailto:oguzmertcoskun@gmail.com)  
🎓 Electrical & Electronics Engineering — Özyeğin University  
---

## 📄 License

This repository is provided for **educational and research purposes only.**  
Feel free to use or extend it with proper credit to the author.

---
## 📂 Repository Structure

```text
Incident-Recording-with-STM32/
│
├── Timer_Integration/   # ✅ Final project source code
│   ├── Core/
│   ├── Drivers/
│   ├── STM32CubeMX files
│   └── main.c
│
├── Documentation/
│   └── CS400_Internship_Report.pdf
│
├── README.md
└── LICENSE
```

### 🧩 Keywords
`STM32` `IMU` `MPU6050` `I2C` `FlashMemory` `Timers` `Interrupts`  
`CircularBuffer` `IncidentRecording` `RealTimeSystems` `MISRA-C` `DO-178C`




