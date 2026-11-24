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

## Task 1: Single LED PWM Dimming

Connect GPIO16 to the LED through a current-limiting resistor, with the LED cathode connected to GND.

The program gradually increases and decreases the PWM value. Each fade takes approximately five seconds.

```cpp
#include <Arduino.h>

const int LED_PIN = 16;

void setup() {
  pinMode(LED_PIN, OUTPUT);
}

void loop() {
  for (int duty = 0; duty <= 255; duty++) {
    analogWrite(LED_PIN, duty);
    delay(20);
  }

  delay(1000);

  for (int duty = 255; duty >= 0; duty--) {
    analogWrite(LED_PIN, duty);
    delay(20);
  }

  delay(1000);
}
```
To see the results, please visit my teammate’s page
https://github.com/Conger-Y/iot-portfolio-template/edit/main/Module05/module_05_lab_report%20(1).md
## Task 2: RGB Mixing and Input Validation

The tested RGB module has a common cathode and onboard 150 Ω resistors marked `151`.

| Module connection | ESP32 connection |
|---|---|
| R | GPIO16 |
| G | GPIO17 |
| B | GPIO18 |
| Common cathode / − | GND |

Commands use three integers followed by a newline:

```text
255 0 0
0 255 0
0 0 255
255 255 255
0 0 0
```

Values are clamped to 0–255. Malformed commands leave the previous color unchanged.

```cpp
#include <Arduino.h>
#include <stdio.h>

const int RED_PIN = 16;
const int GREEN_PIN = 17;
const int BLUE_PIN = 18;
const bool COMMON_ANODE = false;

char input[64];
size_t length = 0;
bool overflow = false;

void applyColor(int red, int green, int blue) {
  analogWrite(RED_PIN, COMMON_ANODE ? 255 - red : red);
  analogWrite(GREEN_PIN, COMMON_ANODE ? 255 - green : green);
  analogWrite(BLUE_PIN, COMMON_ANODE ? 255 - blue : blue);
}

void processCommand(const char* command) {
  int red, green, blue;
  char extra;

  if (sscanf(command, "%d %d %d %c",
             &red, &green, &blue, &extra) != 3) {
    Serial.println(
        "ERROR: Enter three integers, e.g. 255 0 0"
    );
    return;
  }

  red = constrain(red, 0, 255);
  green = constrain(green, 0, 255);
  blue = constrain(blue, 0, 255);

  applyColor(red, green, blue);

  Serial.printf(
      "Applied RGB: %d %d %d\n",
      red, green, blue
  );
}

void setup() {
  Serial.begin(115200);

  pinMode(RED_PIN, OUTPUT);
  pinMode(GREEN_PIN, OUTPUT);
  pinMode(BLUE_PIN, OUTPUT);

  applyColor(0, 0, 0);
  Serial.println("Enter R G B, e.g. 255 0 0");
}

void loop() {
  while (Serial.available()) {
    char c = Serial.read();

    if (c == '\r') continue;

    if (c == '\n') {
      if (overflow) {
        Serial.println("ERROR: Command too long");
      } else if (length > 0) {
        input[length] = '\0';
        processCommand(input);
      }

      length = 0;
      overflow = false;
    } else if (!overflow) {
      if (length < sizeof(input) - 1) {
        input[length++] = c;
      } else {
        overflow = true;
      }
    }
  }
}
```

### Error-Handling Test

Send:

```text
abc
```

Expected response:

```text
ERROR: Enter three integers, e.g. 255 0 0
```

The existing LED color should remain unchanged.

### Recorded Channel Measurements

Each channel was activated separately at PWM 255. Voltage was measured across its onboard resistor.

| Channel | Resistance | Resistor voltage | Calculated current |
|---|---:|---:|---:|
| Red | 150 Ω | 1.07 V | 7.13 mA |
| Green | 150 Ω | 0.63 V | 4.20 mA |
| Blue | 150 Ω | 0.50 V | 3.33 mA |

Current was calculated using `I = U / R`. No additional measurement program is required.
To see the results, please visit my teammate’s page
https://github.com/Conger-Y/iot-portfolio-template/edit/main/Module05/module_05_lab_report%20(1).md

## Task 3: Addressable WS2812B Strip

Required library: NeoPixelBus by Makuna.

| Strip connection | ESP32 connection |
|---|---|
| DIN | GPIO16 |
| 5 V power input | Verified USB-powered VIN / 5 V rail |
| GND | GND |

Connect the controller data wire to the end marked `DIN`, not `DOUT`.

The sketch uses `NeoPixelBusLg` so that `SetLuminance(80)` is supported.

Commands:

```text
index R G B
clear
```

Pixel indices range from 0 to 7. Each pixel command updates only the selected pixel and retains the other pixels' colors.

```cpp
#include <Arduino.h>
#include <NeoPixelBusLg.h>
#include <stdio.h>
#include <string.h>

const uint16_t PIXEL_COUNT = 8;
const uint8_t DATA_PIN = 16;

NeoPixelBusLg<
    NeoGrbFeature,
    NeoEsp32Rmt0Ws2812xMethod,
    NeoGammaNullMethod
> strip(PIXEL_COUNT, DATA_PIN);

char input[64];
size_t length = 0;
bool overflow = false;

void processCommand(const char* command) {
  if (strcmp(command, "clear") == 0) {
    strip.ClearTo(RgbColor(0, 0, 0));
    strip.Show();
    Serial.println("OK: All LEDs off");
    return;
  }

  int index, red, green, blue;
  char extra;

  if (sscanf(command, "%d %d %d %d %c",
             &index, &red, &green, &blue, &extra) != 4) {
    Serial.println(
        "ERROR: Enter index R G B, or clear"
    );
    return;
  }

  if (index < 0 || index >= PIXEL_COUNT) {
    Serial.println("ERROR: LED index must be 0 to 7");
    return;
  }

  if (red < 0 || red > 255 ||
      green < 0 || green > 255 ||
      blue < 0 || blue > 255) {
    Serial.println("ERROR: RGB values must be 0 to 255");
    return;
  }

  strip.SetPixelColor(index, RgbColor(red, green, blue));
  strip.Show();

  Serial.printf(
      "OK: LED %d = %d %d %d\n",
      index, red, green, blue
  );
}

void setup() {
  Serial.begin(115200);

  strip.Begin();
  strip.SetLuminance(80);
  strip.ClearTo(RgbColor(0, 0, 0));
  strip.Show();

  Serial.println("Enter index R G B, or clear");
}

void loop() {
  while (Serial.available()) {
    char c = Serial.read();

    if (c == '\r') continue;

    if (c == '\n') {
      if (overflow) {
        Serial.println("ERROR: Command too long");
      } else if (length > 0) {
        input[length] = '\0';
        processCommand(input);
      }

      length = 0;
      overflow = false;
    } else if (!overflow) {
      if (length < sizeof(input) - 1) {
        input[length++] = c;
      } else {
        overflow = true;
      }
    }
  }
}
```
To see the results, please visit my teammate’s page
https://github.com/Conger-Y/iot-portfolio-template/edit/main/Module05/module_05_lab_report%20(1).md

## Self-Reflection
