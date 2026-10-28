# Module 2

We are going to show here notes for Task 1, 2, 3, 4, and 5 to prove

## Task 1

### Voltage Divider Experiment

We built a voltage divider using three 10 kΩ resistors connected in series.

- **Input voltage (Vin):** Nominally 5 V (ESP32 VIN)
- **R1:** 10 kΩ
- **R2 + R3:** 20 kΩ
- **Calculated output voltage:** 3.333 V
- **Measured output voltage:** 3.487 V
- **Difference:** 0.154 V
- **Percentage difference:** Approximately 4.62%

**Calculation:**

Vout = Vin × R2 / (R1 + R2)

Vout = 5 × 20 / (10 + 20) = 3.333 V


**Result:**

The measured output voltage was 3.487 V, slightly higher than the calculated value of 3.333 V.

This difference may be caused by variations in the USB supply voltage, resistor tolerances, or measurement accuracy.

**Conclusion:**

The experiment demonstrates how a voltage divider reduces a higher voltage using resistors. The measured result was reasonably close to the theoretical value.

## Task 2

# Bus Profile: I²C / I³C

**Pod 5 — Bus Systems Survey**

### 1. History and Background
I²C (Inter-Integrated Circuit) was introduced by Philips in 1982 to simplify communication between chips in electronic devices. I³C (Improved Inter-Integrated Circuit) was introduced by the MIPI Alliance in 2017 as a faster, more efficient successor to I²C.

### 2. Wires and Connectors
Both buses use two main signal lines:
- **SDA:** Serial Data
- **SCL:** Serial Clock

Power and GND connections are also required. Common I²C sensor modules have VCC, GND, SDA and SCL pins.

### 3. Voltage, Signaling and Drive Style
I²C commonly uses 3.3 V or 5 V logic, depending on the devices. Standard I²C uses open-drain outputs with pull-up resistors.

I³C uses a combination of open-drain and push-pull signaling to improve communication efficiency. Both are serial communication buses.

### 4. Topology and Addressing
Multiple devices share the SDA and SCL lines.

I²C commonly uses static 7-bit addresses and supports multiple controllers with arbitration.

I³C additionally supports dynamic address assignment, allowing the controller to manage device addresses automatically.

### 5. Speed and Maximum Length
- **I²C:** Common modes are 100 kbit/s and 400 kbit/s. Higher-speed modes also exist.
- **I³C:** Up to 12.5 Mbit/s raw bitrate in standard SDR mode, with optional faster modes.
- **Distance:** Both are mainly designed for short connections between chips or on circuit boards. There is no single universal maximum cable length; capacitance, speed and signal quality are important.

### 6. Relation to Other Buses
Compared with SPI, I²C uses fewer signal wires and supports device addressing, but common I²C modes are slower.

Compared with RS-485 and CAN, I²C and I³C are primarily designed for short-distance communication inside devices rather than long industrial or automotive cables.

I³C combines the low pin count of I²C with improved speed, efficiency and device management.

### 7. Typical Uses and Kit Example
I²C is commonly used for OLED displays, temperature sensors, motion sensors and real-time clocks.

For example, an ESP32 can connect to several I²C sensors using the same SDA and SCL lines.

I³C is used in advanced sensor and embedded systems with compatible controllers and devices.

### 8. Curiosities / Remarkable Features
1. **Open-drain signaling:** I²C devices normally pull a line LOW rather than actively driving it HIGH, allowing devices to share the same bus.
2. **Dynamic addressing:** I³C can assign addresses to compatible devices automatically, reducing address-management problems.
3. **In-Band Interrupts:** I³C devices can request attention over the shared bus without requiring a separate interrupt wire.

### 9. What Can Go Wrong / Debugging
A common I²C problem is a missing device during address scanning.

Possible causes include incorrect wiring, missing or unsuitable pull-up resistors, voltage incompatibility or address conflicts.

We can debug this by checking power and GND, scanning I²C addresses, and measuring SDA/SCL signals with a logic analyzer.

A device holding SDA LOW can prevent the whole bus from working.

### 10. Sources and Verification
- NXP Semiconductors: *UM10204 — I²C-bus Specification and User Manual*, Rev. 7.0, 2021.
- MIPI Alliance: *MIPI I3C and I3C Basic — Specification Overview and FAQ*.
- Device datasheets and experimental measurements should be used to verify voltage levels, operating speed and address configuration.

**Verification status:** Protocol features checked against official documentation. Our particular ESP32 and sensor connections, bus voltage, operating speed and signal quality have not yet been measured for this task.

## Task ...

...

If the module is longer than 1 week, you might have several reflections within it. If not, delete this and the following.

## Reflection 3
[Reflection 3](/Reflections/ref03.md)


