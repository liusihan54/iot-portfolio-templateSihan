# Module 2

## 🎯 Learning Goals

Through hands-on I²C bus wiring, low-level register parsing, multi-device bus sharing, and logic analyzer debugging across Tasks 5–7 and Task 9 on the ESP32, this module achieves the following learning outcomes:

1. **I²C Bus Wiring & Address Scanning (Task 5):** Configure the ESP32 I²C hardware interface (`SDA = GPIO 21`, `SCL = GPIO 22`) on a breadboard, initialize serial diagnostics (`115200` baud), and execute an I²C bus scanner to verify physical connections and discover slave device addresses (`0x4F`).
2. **Low-Level Register Parsing vs. High-Level Libraries (Task 6):**
   * Communicate directly with the LM75 temperature sensor (`0x4F`) using the native Arduino `Wire` library to read raw bytes from pointer register `0x00`, manually handling byte order and scaling factors.
   * Cross-validate manual register calculations ($22.969\ ^\circ\text{C}$) against a dedicated LM75 library (`Temp0.getTemperature()`, $22.516\ ^\circ\text{C}$) and confirm dynamic sensor response ($22.516\ ^\circ\text{C} \rightarrow 26.266\ ^\circ\text{C}$) through physical heat application.
3. **Multi-Device Bus Sharing & Master-Slave Addressing (Task 7):**
   * Wire multiple I²C slave devices—the LM75 sensor (`0x4F`) and an I²C LCD display (`0x27`)—in parallel onto the same shared `SDA` and `SCL` lines without using extra GPIO pins.
   * Apply Master-Slave addressing principles to read real-time temperature data from one slave and display it on another slave across a single shared bus without data collisions.
4. **Digital Waveform Capture & Hardware Fault Debugging (Task 9):**
   * Deploy and operate a web-based logic analyzer (`hepter/web-logic-analyzer`) to inspect digital bus signals alongside basic pin, PWM, UART, and voltage-divider behavior.
   * Systematically diagnose and resolve breadboard power-distribution faults by identifying and clearing a `3V3`-to-`D19` short circuit that prevented the logic analyzer board from powering up.



### 5. Speed and Maximum Length

```cpp
#include <Wire.h>

void setup() {
  Serial.begin(115200);
  Wire.begin(21, 22);
  delay(1000);

  Serial.println("Scanning I2C bus...");

  for (uint8_t address = 1; address < 127; address++) {
    Wire.beginTransmission(address);

    if (Wire.endTransmission() == 0) {
      Serial.printf("Device at 0x%02X\n", address);
    }
  }

  Serial.println("Scan done.");
}

void loop() {}
```
To see the results, please visit my teammate’s page.
https://github.com/Conger-Y/iot-portfolio-template/blob/main/Module02/pins-and-dividers.md
### 6. Relation to Other Buses
6A：
```cpp
#include <Wire.h>
#include <LM75.h>

LM75 Temp0(0x4F);

void setup() {
  Serial.begin(115200);
  Wire.begin(21, 22);
  Temp0.begin();

  delay(1000);
  Serial.println("Task 6: Library Temperature Read");
}

void loop() {
  float temperature = Temp0.getTemperature();

  Serial.print("Library temperature: ");
  Serial.print(temperature, 3);
  Serial.println(" C");

  delay(1000);
}
```
6B:
```cpp
#include <Wire.h>

const uint8_t SENSOR_ADDR = 0x4F;
const uint8_t TEMP_REG = 0x00;

void setup() {
  Serial.begin(115200);
  Wire.begin(21, 22);
  delay(1000);

  Serial.println("Task 6: Read Temperature Register");
}

void loop() {
  Wire.beginTransmission(SENSOR_ADDR);
  Wire.write(TEMP_REG);

  if (Wire.endTransmission(false) != 0) {
    Serial.println("I2C communication error");
    delay(1000);
    return;
  }

  if (Wire.requestFrom(SENSOR_ADDR, (uint8_t)2) != 2) {
    Serial.println("Read error");
    delay(1000);
    return;
  }

  uint8_t msb = Wire.read();
  uint8_t lsb = Wire.read();

  int16_t raw = (int16_t)(((uint16_t)msb << 8) | lsb);
  float temperature = raw / 256.0f;

  Serial.printf("Raw bytes: 0x%02X 0x%02X\n", msb, lsb);
  Serial.print("Temperature: ");
  Serial.print(temperature, 3);
  Serial.println(" C");

  delay(1000);
}
```
To see the results, please visit my teammate’s page.
https://github.com/Conger-Y/iot-portfolio-template/blob/main/Module02/pins-and-dividers.md
### 7. Typical Uses and Kit Example
```cpp
#include <Wire.h>
#include <LM75.h>
#include <LiquidCrystal_I2C.h>

LM75 Temp0(0x4F);
LiquidCrystal_I2C lcd(0x27, 16, 2);

void setup() {
  Serial.begin(115200);
  Wire.begin(21, 22);

  Temp0.begin();

  lcd.init();
  lcd.backlight();
  lcd.setCursor(0, 0);
  lcd.print("Temperature:");
}

void loop() {
  float temperature = Temp0.getTemperature();

  Serial.print("Temperature: ");
  Serial.print(temperature, 3);
  Serial.println(" C");

  lcd.setCursor(0, 1);
  lcd.print("                ");
  lcd.setCursor(0, 1);
  lcd.print(temperature, 2);
  lcd.print(" C");

  delay(1000);
}
```
To see the results, please visit my teammate’s page.
https://github.com/Conger-Y/iot-portfolio-template/blob/main/Module02/pins-and-dividers.md

## Task 9: Web Logic Analyzer

We used the open-source hepter/web-logic-analyzer firmware to capture digital signals and display waveforms in a web browser.

### Source Code

[Complete Arduino sketch](https://github.com/hepter/web-logic-analyzer/blob/main/web-logic-analyzer.ino)

The firmware includes signal acquisition, a web server, and the browser interface for displaying waveforms.

### Procedure

1. Configure the Wi-Fi credentials and input pins.
2. Upload the firmware to the ESP32.
3. Open the Serial Monitor at 115200 baud.
4. Enter the displayed local IP address in a browser on the same network.
5. Start a capture using the web interface and inspect the resulting waveforms.

Credit: hepter/web-logic-analyzer, licensed under GNU GPL v3.

To see the results, please visit my teammate’s page.
https://github.com/Conger-Y/iot-portfolio-template/blob/main/Module02/pins-and-dividers.md

## Reflection 3
[Reflection 3](/Reflections/ref03.md)


