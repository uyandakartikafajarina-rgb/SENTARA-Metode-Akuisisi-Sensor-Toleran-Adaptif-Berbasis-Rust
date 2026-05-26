# SENTARA — ESP32-S3 + DHT22 + LCD I2C + 3 LED | Rust

**Sensor Tolerant Adaptive Rust-based Acquisition**

> Department of Instrumentation Engineering — Faculty of Vocational Studies — Institut Teknologi Sepuluh Nopember (ITS) 2026

| Author | NRP |
|---|---|
| Vyanda Kartika Fajarina | 2042241030 |
| Aulia Qotrunnada | 2042241031 |

---

## Table of Contents

1. [Project Description & Background](#1-project-description--background)
2. [Block Diagram & Flowchart](#2-block-diagram--flowchart)
3. [Running the Simulation](#3-running-the-simulation)
4. [Simulation Results & Analysis](#4-simulation-results--analysis)
5. [Advantages of the SENTARA Method](#5-advantages-of-the-sentara-method)

---

## 1. Project Description & Background

### What is SENTARA?

**SENTARA** (*Sensor Tolerant Adaptive Rust-based Acquisition*) is a real-time temperature and humidity monitoring system built on the **ESP32-S3** microcontroller and programmed entirely in the **Rust** programming language. The system uses the **DHT22** digital sensor for data acquisition and provides safety-critical feedback through a **16×2 LCD (I2C)** display and a **3-LED indicator array** (green/yellow/red).

### Proposed Method

**Full Title:**
> *"SENTARA: A Tolerant Adaptive Sensor Acquisition Method Based on Rust for Safety-Critical Real-Time Temperature and Humidity Monitoring on ESP32-S3 Using the DHT22 Sensor"*

### Background & Motivation

Previous embedded monitoring systems have been developed predominantly in **C/C++**, which — while performant — exposes the system to memory safety vulnerabilities such as buffer overflows, dangling pointers, and undefined behavior in concurrent contexts. This poses a significant risk in safety-critical applications where data integrity must be guaranteed at all times.

SENTARA is built upon four future-work pillars (FW1–FW4) that collectively address:

- **Memory safety** at the compiler level (Rust's ownership model)
- **Adaptive scheduling** through dynamic TDMA-based sampling
- **On-chip self-calibration** of ADC non-linearity
- **Multivariable fault tolerance** with automatic self-healing recovery

The system introduces a closed-loop control architecture where the DHT22 sensor feeds back environmental data (temperature + humidity) to the ESP32-S3, which computes a calibrated output and drives status indicators in real time.

### Hardware Configuration

| Arduino (C++) Equivalent | Rust GPIO | Function |
|---|---|---|
| `#define DHT_PIN 18` | `gpio18` | DHT22 DATA pin |
| `#define LED_HIJAU 15` | `gpio15` | Green LED — Normal |
| `#define LED_KUNING 16` | `gpio16` | Yellow LED — Warning |
| `#define LED_MERAH 17` | `gpio17` | Red LED — Critical (blinking) |
| `Wire.begin(8, 9)` | `gpio8=SDA, gpio9=SCL` | LCD I2C at address 0x27 |

### Status Logic

| Condition | Status | LED | LCD Display |
|---|---|---|---|
| T < 35 °C AND RH < 80% | NORMAL | 🟢 Green | `Status: NORMAL` |
| T ≥ 35 °C OR RH ≥ 80% | WARNING | 🟡 Yellow | `Status: WARNING` |
| T ≥ 40 °C OR RH ≥ 90% | CRITICAL | 🔴 Red (blink) | `Status: KRITIS!` |
| Sensor failure | ERROR | 🔴 Red (blink) | `Sensor Error!` |

---

## 2. Block Diagram & Flowchart

### 2.1 System Block Diagram

The block diagram below illustrates the closed-loop control architecture of SENTARA. The ESP32-S3 (running Rust embedded firmware) receives a set point, processes sensor data through the ADC & Digital Interface, passes it to the Monitoring System, and receives feedback from the DHT22 Temperature & Humidity Sensor.

<img width="671" height="181" alt="diagram blok pemkon 2 drawio" src="https://github.com/user-attachments/assets/de72129b-0603-4285-bc67-2019bb5268b3" />

*Figure 1 — Closed-loop block diagram: Set Point → ESP32-S3 (Rust) → Data Acquisition Unit (ADC & Digital Interface) → Monitoring Process/System, with DHT22 sensor feedback.*

---

### 2.2 System Flowchart

The flowchart below details the complete firmware execution flow, from system initialization through self-calibration, adaptive sampling, fault detection, memory-safe processing, and real-time serial display.

<img width="533" height="767" alt="diagram blok pemkon drawio (1)" src="https://github.com/user-attachments/assets/86cbb210-c064-402f-9ca8-a51d61a9e4fd" />

*Figure 2 — Firmware flowchart: Initialize ESP32-S3 (Rust runtime) → Initialize DHT22 → Self-Calibration (ADC offset correction) → Read sensors → Data Filtering → Sensor Validation → Fault Detection & Recovery → Adaptive Sampling → Rust Memory-Safe Processing → Display Serial Monitor.*

**Key flowchart stages:**

- **Initialization** — ESP32-S3 and DHT22 are initialized with Rust runtime setup
- **Self-Calibration** — ADC offset correction is performed once at startup
- **Read Loop** — LM35 (analog) and DHT22 (digital) are read simultaneously
- **Data Filtering** — Noise reduction is applied before validation
- **Sensor Valid?** — Branch: if valid, compare sensor data; if not, trigger fault detection
- **Fault Detection & Recovery** — Error logging followed by recovery, then rejoining main loop
- **Significant Change?** — Adaptive scheduling: fast sampling on significant change, normal sampling otherwise
- **Rust Memory-Safe Processing** — All data processed within Rust's ownership-guaranteed environment
- **Display Serial Monitor** — Real-time output to serial for monitoring

---

## 3. Running the Simulation

### 3.1 Prerequisites — Install Rust ESP Toolchain (once only)

```powershell
cargo install espup
espup install
```

### 3.2 Build the Project

```powershell
# In PowerShell — activate ESP environment first
. "$env:USERPROFILE\export-esp.ps1"

# Navigate to project directory
cd SENTARA-FINAL

# Build the firmware
cargo build
```

### 3.3 Run Wokwi Simulator (Visual Simulation)

1. Open **VS Code** and navigate to the `SENTARA-FINAL` folder
2. Open the file `wokwi/diagram.json`
3. Press **F1** → type and select `Wokwi: Start Simulator`
4. LEDs will automatically respond to DHT22 sensor readings in real time

### 3.4 Generate GNUPlot Graphs

#### Install GNUPlot (Windows)

Download the installer from: [http://www.gnuplot.info/download.html](http://www.gnuplot.info/download.html)

#### Run Plot Scripts

```bash
cd gnuplot/
gnuplot plot_temperature.gp   # Temperature graph
gnuplot plot_humidity.gp      # Humidity graph
gnuplot plot_dashboard.gp     # Full 4-panel dashboard
```

#### Output Files

| File | Description |
|---|---|
| `sentara_temperature.png` | Temperature graph — raw, calibrated, moving average, heat index |
| `sentara_humidity.png` | Humidity graph — raw, calibrated, moving average |
| `sentara_dashboard.png` | 4-panel dashboard (temperature + humidity + comparison + heat index) |

### 3.5 CSV Log Format

The system outputs data in the following CSV format (used as GNUPlot input):

```
timestamp_ms, temp_raw, hum_raw, temp_cal, hum_cal, heat_index, avg_temp, avg_hum, reliability
0, 28.5, 65.0, 28.0, 67.0, 28.8, 28.0, 67.0, 100.0
```

### 3.6 Project Structure

```
SENTARA-FINAL/
├── src/main.rs              ← Main Rust source (converted from Arduino)
├── wokwi/diagram.json       ← Wokwi circuit diagram (open in VS Code)
├── gnuplot/
│   ├── plot_temperature.gp  ← Temperature graph script
│   ├── plot_humidity.gp     ← Humidity graph script
│   ├── plot_dashboard.gp    ← 4-panel dashboard script
│   └── run_plots.sh         ← Run all plots at once (Linux/Mac)
├── data/sentara_log.csv     ← Sample data for GNUPlot
├── Cargo.toml
├── wokwi.toml
├── build.rs
└── sdkconfig.defaults
```

---

## 4. Simulation Results & Analysis

All graphs were generated using **GNUPlot** from simulated CSV data captured during a 100-second observation window. The system was tested across the full status range: from **NORMAL** through **WARNING** to **CRITICAL** and back.

---

### 4.1 Real-Time Temperature Monitoring

<img width="1200" height="700" alt="Grafik Suhu" src="https://github.com/user-attachments/assets/5a1b564d-6f95-4f86-9eab-b430e61fcd53" />

*Figure 3 — Real-time temperature monitoring: Raw DHT22 readings (dashed blue), Calibrated Temperature (solid blue), Moving Average N=5 (orange dash-dot), and Heat Index (red dotted). Threshold lines: WARNING at 40 °C (yellow), CRITICAL at 45 °C (red dashed).*

**Analysis:**

- The system begins in the **NORMAL zone** (~28 °C) and gradually rises over 60 seconds, peaking at approximately **45 °C** — crossing both the WARNING (40 °C) and approaching the CRITICAL (45 °C) threshold.
- The **calibrated temperature** tracks the raw reading closely but with ADC offset correction applied, demonstrating the on-chip calibration capability.
- The **Moving Average (N=5)** smooths transient spikes effectively, providing a stable trend signal. Notably, it lags behind the actual temperature rise, confirming its role as a trend indicator rather than an alarm trigger.
- The **Heat Index** exceeds the actual temperature significantly at peak conditions (reaching ~54 °C), reflecting the combined effect of high temperature and elevated humidity — a critical safety parameter for human exposure assessment.
- After t=60s, the system recovers back into the NORMAL zone, validating the adaptive fast-sampling response to significant environmental changes.

---

### 4.2 Real-Time Humidity Monitoring

<img width="1200" height="700" alt="Grafik Kelembaban" src="https://github.com/user-attachments/assets/1bae7c26-c17b-418f-9ed1-8d9798edd988" />

*Figure 4 — Real-time humidity monitoring: Raw DHT22 readings (dashed green), Calibrated Humidity (solid green), Moving Average N=5 (red dash-dot). Threshold lines: WARNING at 85% RH (orange dotted), CRITICAL at 95% RH (red dashed).*

**Analysis:**

- Humidity starts at ~67% RH (NORMAL zone) and rises steadily to a peak of ~83% RH at t=60s — entering the **WARNING zone** (≥80% RH) but not reaching the CRITICAL threshold (95% RH).
- The calibrated humidity follows the raw readings with slight correction, demonstrating sensor linearization applied in firmware.
- The Moving Average reveals a **sustained upward trend** from t=20s onwards, providing early warning signal before the WARNING threshold is crossed.
- The system's dual-variable monitoring (both temperature AND humidity) is crucial: even if temperature alone stays below 40 °C, elevated humidity can still trigger the WARNING state — demonstrating the multivariable nature of SENTARA's safety logic.
- Post-peak recovery at t=80s–100s shows the system correctly transitions back through WARNING and toward NORMAL as conditions improve.

---

### 4.3 Safety-Critical Monitoring Dashboard (4-Panel)

<img width="1400" height="900" alt="Dashboard GNUPlot" src="https://github.com/user-attachments/assets/2f4834c3-5c75-49cb-9a92-dbf760a6bbfd" />

*Figure 5 — Full 4-panel safety-critical monitoring dashboard: (top-left) Temperature Profile with CRITICAL threshold at 45 °C; (top-right) Humidity Profile with WARNING at 85% RH; (bottom-left) Heat Index vs. Actual Temperature with danger zone; (bottom-right) Raw vs. Calibrated comparison with Reliability (%) on secondary axis.*

**Analysis:**

- **Temperature Profile (top-left):** Confirms the peak at ~45 °C at t=60s, with the CRITICAL threshold clearly visible. The moving average demonstrates the lag effect expected of a window-based filter.
- **Humidity Profile (top-right):** Shows humidity reaching ~83% RH, triggering the WARNING state. The WARNING threshold (85% RH) is clearly labeled, and the system correctly identifies the elevated risk period from t=40s–80s.
- **Heat Index / Indeks Panas (bottom-left):** The Heat Index peaks at approximately **69 °C** — far above the "Bahaya" (Danger) threshold of ~54 °C — while the actual temperature peaks at ~45 °C. This panel demonstrates why Heat Index is a superior safety metric in high-humidity environments, as it represents the perceived thermal stress on a human body.
- **Raw vs. Calibrated Comparison + Reliability (bottom-right):** The raw and calibrated temperature traces closely track each other, with the calibrated signal slightly smoothed. The **Reliability** metric (green horizontal line) remains constant at ~100%, indicating zero sensor fault events during this test run — a key indicator of SENTARA's fault-tolerant architecture.

---

## 5. Advantages of the SENTARA Method

Based on simulation results and system design analysis, SENTARA demonstrates four strategic advantages over prior research using C/C++-based architectures:

### 5.1 Absolute Memory Safety Guarantee (*Inherent Memory Safety*)

Unlike C/C++ architectures that are susceptible to buffer overflows, dangling pointers, and race conditions, the use of **Rust guarantees memory isolation at the compiler level**. No runtime garbage collector is needed — memory correctness is enforced at compile time through Rust's ownership and borrow-checker system. This eliminates an entire class of runtime vulnerabilities that would be unacceptable in a safety-critical monitoring system.

### 5.2 Peripheral Power Efficiency via Dynamic Scheduling

SENTARA implements **adaptive sampling based on dynamic TDMA (Time Division Multiple Access) scheduling**. The firmware autonomously adjusts the sensor sampling frequency based on detected environmental change rate:

- **Normal Sampling** when conditions are stable (conserves power)
- **Fast Sampling** when a significant change is detected (maximizes responsiveness)

This is achieved without triggering an increase in chip power consumption, making SENTARA suitable for battery-powered or low-power deployments.

### 5.3 On-Chip Self-Calibration

The system performs **ADC non-linearity correction directly on the ESP32-S3** at startup (and adaptively during operation), without dependency on external calibration instruments. This makes SENTARA deployable in field environments where laboratory calibration equipment is unavailable, while still maintaining measurement accuracy comparable to externally calibrated systems.

### 5.4 Multivariable Fault-Tolerant Mechanism

SENTARA implements a **multivariable fault detection and self-healing architecture**:

- Abnormal readings from either the temperature or humidity channel are independently detected
- The system logs errors and executes an automatic **recovery routine** without halting the main data acquisition loop
- Both variables are monitored simultaneously, ensuring that a failure in one channel triggers appropriate status escalation without corrupting the other channel's data stream

This results in a system with high operational continuity — a critical requirement for safety monitoring applications where downtime or data gaps are unacceptable.

---

## References & Links

- Rust Embedded Documentation: [https://docs.rust-embedded.org](https://docs.rust-embedded.org)
- ESP-IDF Rust Bindings (esp-idf-hal): [https://github.com/esp-rs/esp-idf-hal](https://github.com/esp-rs/esp-idf-hal)
- Wokwi ESP32 Simulator: [https://wokwi.com](https://wokwi.com)
- GNUPlot: [http://www.gnuplot.info](http://www.gnuplot.info)
- DHT22 Datasheet: AOSONG AM2302

---

*SENTARA — Sensor Tolerant Adaptive Rust-based Acquisition | Teknik Instrumentasi ITS 2026*
