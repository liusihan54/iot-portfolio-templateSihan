# Module 05 — LEDs & Light as Output
## 🎯 Learning Goals

Through hands-on PWM dimming, RGB channel characterization, addressable LED strip control, and power budgeting across Tasks 1–5 on the MH-ET LIVE ESP32 MiniKit, this module achieves the following learning outcomes:

1. **Single-Channel PWM Dimming & Duty Cycle Control (Task 1):**
   * Configure a digital output (`GPIO 16`) with `analogWrite()` to sweep 8-bit PWM duty cycles (`0–255`), demonstrating how modulating the ratio of on-time to period controls average LED current and perceived brightness.
2. **RGB Color Mixing, Serial Command Parsing & Forward-Voltage Physics (Task 2):**
   * Wire a common-cathode RGB LED module (`COMMON_ANODE = false`) across `GPIO 16`, `GPIO 17`, and `GPIO 18`, and implement a validated `115200` baud serial REPL (`R G B` parsing via `sscanf`, value clamping to `0–255`, and rejection of malformed inputs such as `abc`).
   * Measure voltage drops across identical $150\ \Omega$ onboard resistors at full duty cycle (`PWM = 255`) and calculate channel currents via Ohm's law (**Red =** $7.13\text{ mA}$ at $1.07\text{ V}$, **Green =** $4.20\text{ mA}$ at $0.63\text{ V}$, **Blue =** $3.33\text{ mA}$ at $0.50\text{ V}$), explaining why higher LED forward voltages ($V_f$) in green and blue diodes reduce resistor voltage drop and current.
3. **Hardware-Offloaded Addressable LED Control (`NeoPixelBusLg`, Task 3):**
   * Drive an 8-pixel WS2812B strip on `GPIO 16` using `NeoPixelBusLg` (`NeoGrbFeature`, `NeoEsp32Rmt0Ws2812xMethod`), offloading the strict $800\text{ kHz}$ single-wire NRZ timing waveform to ESP32 hardware RMT Channel 0 so Wi-Fi/BLE interrupts do not corrupt frames.
   * Implement bounds-checked serial pixel control (`index R G B` for indices `0–7` and `clear`) while enforcing a firmware luminance cap (`SetLuminance(80)`, $\approx 31.4\%$).
4. **Empirical Strip Current Measurement & Power Budget Scaling (Task 3):**
   * Measure series supply current on the $5\text{ V}$ rail (`10 A` DC multimeter range) at `80 / 255` luminance, verifying that all-white illumination ($\approx 0.09\text{ A}$ / $0.45\text{ W}$) draws roughly three times the current of all-red ($\approx 0.03\text{ A}$ / $0.15\text{ W}$).
   * Scale power budgets from `8` to `144` pixels ($0.48\text{ A}$ up to $8.64\text{ A}$ worst-case full white at $60\text{ mA/pixel}$), determining when USB 2.0 ($500\text{ mA}$ limit) suffices versus when dedicated external $5\text{ V}$ supplies and dual-end power injection are required.
5. **AI Code Verification & System-Level Output Architecture (Tasks 4 & 5):**
   * Critically verify and debug AI-scaffolded firmware against library APIs and hardware behavior (replacing base `NeoPixelBus` with `NeoPixelBusLg` to support `SetLuminance(80)`).
   * Contrast simple GPIO-driven indicator LEDs against addressable strips across power distribution, $800\text{ kHz}$ NRZ timing constraints, and $3.3\text{ V}$-to-$5\text{ V}$ logic-level thresholds.
**Board:** MH-ET LIVE ESP32 MiniKit (`mhetesp32minikit`)  
**Framework & Toolchain:** Arduino framework via PlatformIO CLI (`pio`) / Arduino IDE, Serial monitor at `115200` baud

## Repository Structure

```text
Module05/
├── README.md          # Lab report, measurements, power budget, and study log
├── task1-led/         # Task 1: Single-channel PWM fade on GPIO 16
├── task2-rgb/         # Task 2: Common-cathode RGB LED serial REPL (GPIO 16, 17, 18)
└── task3-strip/       # Task 3: 8-pixel WS2812B addressable strip via NeoPixelBusLg (GPIO 16)
```

---
