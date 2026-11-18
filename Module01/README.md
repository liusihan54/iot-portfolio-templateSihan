# Module 1

## 🎯 Learning Goals

Through hands-on prototyping and hardware characterization across Tasks 1–4 on the ESP32, this module achieves the following learning outcomes:

1. **Basic Electronics & Digital I/O (Task 1):** Understand breadboard internal bus connections and power distribution, safely drive an LED through a current-limiting resistor, and wire a push button using pull-up and pull-down configurations to control digital GPIO states.
2. **Development Environment & Serial Data Visualization (Task 2):** Configure ESP32 USB drivers and the Arduino IDE toolchain, establish reliable serial communication (`115200` baud), and use the Serial Monitor and Serial Plotter to inspect and visualize real-time signals.
3. **ADC Quantization, Resolution vs. Accuracy & Non-Linearity (Task 3):**
   * Quantify theoretical vs. observed step sizes across `12-bit` ($4096$ steps, $\approx 0.806\text{ mV/step}$) and `10-bit` ($1024$ steps, $\approx 3.223\text{ mV/step}$) SAR ADC modes using `analogReadResolution()`.
   * Distinguish **quantization resolution** from **hardware accuracy** by cross-checking ESP32 readings against a digital multimeter.
   * Identify and document the classic ESP32 ADC non-linear regions—the **low-end dead zone** ($0\text{ V} - 0.15\text{ V}$) and **high-end saturation zone** ($> 2.90\text{ V} - 3.15\text{ V}$)—and restrict reliable measurements to the linear $0.15\text{ V} - 2.90\text{ V}$ window.
4. **Analog Sensor Conditioning & Physical Conversion (Task 4):**
   * Design and build a voltage divider ($3.3\text{ V} \rightarrow \text{LDR} \rightarrow \text{GPIO 33} \rightarrow 10\text{ k}\Omega \rightarrow \text{GND}$) to convert resistance changes into a measurable voltage positively correlated with light intensity.
   * Derive and calculate physical sensor resistance ($R_{LDR} = R_{fixed} \cdot \frac{V_{ref} - V_{out}}{V_{out}}$) across dark ($\approx 93.13\text{ k}\Omega$), ambient room light ($\approx 23.40\text{ k}\Omega$), and bright flashlight ($\approx 4.67\text{ k}\Omega$) conditions.
   * Evaluate practical transducer limitations, including CdS response latency ($200\text{ ms} - 500\text{ ms}$), non-linear logarithmic sensitivity ($R \propto \text{Lux}^{-\gamma}$), thermal drift, and repeatability variance.

## Task Inspirations

- Inspiration 1
- Inspiration 2

## Task Personal Profile

## Task What is IoT

Domains and included areas:

-
-


Commonly used (data) protocols:

-
-

Typical devices (appliance or micro controller):

-
-

Challenges:

-
-

## Reflection 1
[Reflection 1](/Reflections/ref01.md)
