## 🎯 Learning Goals

Through hands-on sensor characterization, two-point calibration, repeatability testing, signal filtering, and uncertainty budgeting across Tasks 1–6 on the ESP32, this module achieves the following learning outcomes:

1. **Transducer Transfer Functions & Empirical Curve Fitting (Task 1):**
   * Build a $10\text{ k}\Omega$ voltage divider (`GPIO 15`) to characterize an LDR's resistance-versus-illuminance behavior using an inverse-square distance proxy ($E_r = (15/d)^2$).
   * Fit and compare power-law ($R = 11.307 \cdot E^{-0.159}$, $\text{RMSE} = 0.581\text{ k}\Omega$) and linear ($R = 16.139 - 5.684 \cdot E$, $\text{RMSE} = 0.248\text{ k}\Omega$) models while identifying ambient-light noise floors in unshielded setups.
2. **Digital Sensor Configuration, Thermal Dynamics & Reference Discipline (Task 2):**
   * Interface a 1-Wire DS18B20 temperature sensor ($4.7\text{ k}\Omega$ pull-up) and evaluate how configuring resolution (`9-bit` to `12-bit`) trades quantization step size ($0.5^\circ\text{C}$ down to $0.0625^\circ\text{C}$) against conversion latency ($93.75\text{ ms}$ to $750\text{ ms}$), while recognizing that fine resolution does not overcome factory accuracy limits ($\pm 0.5^\circ\text{C}$).
   * Measure the first-order thermal step response time constant ($\tau$ at the $63.2\%$ threshold) in a cold-to-warm water transition ($12.1^\circ\text{C} \rightarrow 45.3^\circ\text{C}$) and explain why higher convective heat transfer makes water response much faster than still air.
   * Exercise calibration discipline by refraining from applying unverified offset corrections ($+1.43^\circ\text{C}$ difference against an uncalibrated reference thermometer).
3. **Two-Point Calibration & Independent Validation (Task 3):**
   * Calibrate the ESP32 potentiometer–ADC chain (`GPIO 34`, 12-bit, $11\text{ dB}$ attenuation, 100-sample averaging) using two multimeter reference points ($0.53\text{ V}$ and $2.72\text{ V}$), reducing independent validation error at $1.50\text{ V}$ from $-0.156\text{ V}$ to $-0.028\text{ V}$.
   * Perform two-point power-law and linear calibrations on the LDR ($15\text{ cm}$ and $50\text{ cm}$), validate at an independent point ($25\text{ cm}$, $14.265\text{ k}\Omega$), and critically reject unreliable firmware light calibrations when ambient daylight contaminates the distance proxy.
4. **Hysteresis & Mechanical/Sensor Repeatability (Task 4):**
   * Quantify directional sweep differences in an LDR (up to $0.537\text{ V}$ between increasing and decreasing light sweeps) caused by optical hysteresis, CdS response lag, and alignment variance.
   * Separate static electrical stability ($1\text{ mV}$ spread with a fixed shaft) from manual repositioning repeatability ($124\text{ mV}$ spread across 10 return-to-mark trials) on a potentiometer, and compare short-term DS18B20 reading spreads at `9-bit` ($0.0000^\circ\text{C}$) vs. `12-bit` ($0.1250^\circ\text{C}$).
5. **Noise Filtering Trade-Offs & Hysteresis Threshold Control (Task 5):**
   * Evaluate noise attenuation, impulse spike rejection, and phase lag across a 5-sample Moving Average ($\Delta t \approx 47\text{ ms}$), 5-sample Median filter ($\Delta t \approx 43\text{ ms}$, complete rejection of $+500$ count synthetic spikes), and Exponential Moving Average ($\alpha = 0.2$, $\Delta t \approx 85\text{ ms}$).
   * Implement a hysteresis band ($2.0^\circ\text{C}$ to $2.5^\circ\text{C}$) for a frost alarm to eliminate threshold chattering (reducing state switches from 6 down to 2).
6. **Measurement Uncertainty Budgeting & Improvement Prioritization (Task 6):**
   * Construct a quadrature uncertainty budget combining reference multimeter limits ($6.0\text{ mV}$), short-term repeatability ($0.72\text{ mV}$), and validation model discrepancy ($16.2\text{ mV}$) to report a calibrated result with $k = 2$ expanded uncertainty ($V_{\text{pot}} = 1.472 \pm 0.035\text{ V}$).
   * Identify dominant uncertainty contributors ($>85\%$ variance from calibration model discrepancy) to prioritize multi-point calibration over additional software filtering.
## Task 1: LDR Transfer Function

### Arduino: Record ADC and Voltage

Connect the divider output to GPIO15. Record measurements at each lamp distance.

The voltage is measured using `analogReadMilliVolts()`. The raw ADC code and voltage are consecutive measurements.

```cpp
const int LDR_PIN = 15;

void setup() {
  Serial.begin(115200);
  analogReadResolution(12);
  analogSetPinAttenuation(LDR_PIN, ADC_11db);

  Serial.println("time_ms,adc,voltage_V");
}

void loop() {
  int raw = analogRead(LDR_PIN);
  float voltage = analogReadMilliVolts(LDR_PIN) / 1000.0f;

  Serial.print(millis());
  Serial.print(",");
  Serial.print(raw);
  Serial.print(",");
  Serial.println(voltage, 3);

  delay(200);
}
```

### Python: Compare Power-Law and Linear Fits

This added analysis script uses the resistance values listed in the report.

The report contains conflicting descriptions of the divider orientation. The listed resistance values correspond to an LDR connected to 3.3 V and a fixed resistor connected to GND:

`R_LDR = R_fixed × (Vs − Vout) / Vout`

The actual wiring must be verified before accepting that interpretation.

Required Python package: `numpy`.

```python
import numpy as np

distance = np.array([15, 20, 25, 30, 40, 50], dtype=float)
resistance = np.array([
    10.625, 12.603, 14.265,
    14.505, 15.191, 15.984
])

relative_light = (15.0 / distance) ** 2

exponent, log_coefficient = np.polyfit(
    np.log(relative_light),
    np.log(resistance),
    1
)
coefficient = np.exp(log_coefficient)

slope, intercept = np.polyfit(relative_light, resistance, 1)

power_prediction = coefficient * relative_light**exponent
linear_prediction = slope * relative_light + intercept

power_rmse = np.sqrt(np.mean(
    (power_prediction - resistance) ** 2
))
linear_rmse = np.sqrt(np.mean(
    (linear_prediction - resistance) ** 2
))

print(f"Power law: R = {coefficient:.3f} * E^({exponent:.3f})")
print(f"Power-law RMSE: {power_rmse:.3f} kOhm")
print(f"Linear: R = {intercept:.3f} + ({slope:.3f}) * E")
print(f"Linear RMSE: {linear_rmse:.3f} kOhm")
```
To see the results, please visit my teammate’s page. 
https://github.com/Conger-Y/iot-portfolio-template/blob/main/Module03/README.md

## Task 2: DS18B20 Resolution and Response Time
```cpp
#include <OneWire.h>
#include <DallasTemperature.h>

const int DATA_PIN = 4;  // Example: replace with the actual GPIO.
const int RESOLUTION_BITS = 12;
const unsigned long PAUSE_MS = 250;

OneWire oneWire(DATA_PIN);
DallasTemperature sensors(&oneWire);

void setup() {
  Serial.begin(115200);
  sensors.begin();

  if (sensors.getDeviceCount() == 0) {
    Serial.println("No DS18B20 detected.");
    while (true) delay(1000);
  }

  sensors.setResolution(RESOLUTION_BITS);

  Serial.println("time_ms,bits,temperature_C,conversion_ms");
}

void loop() {
  unsigned long start = millis();
  sensors.requestTemperatures();
  unsigned long conversionTime = millis() - start;

  float temperature = sensors.getTempCByIndex(0);

  if (temperature == DEVICE_DISCONNECTED_C) {
    Serial.println("Sensor disconnected.");
    delay(1000);
    return;
  }

  Serial.print(millis());
  Serial.print(",");
  Serial.print(RESOLUTION_BITS);
  Serial.print(",");
  Serial.print(temperature, 4);
  Serial.print(",");
  Serial.println(conversionTime);

  delay(PAUSE_MS);
}
```
To see the results, please visit my teammate’s page. 
https://github.com/Conger-Y/iot-portfolio-template/blob/main/Module03/README.md

## Task 3a: Two-Point Potentiometer–ADC Calibration
```cpp
const int POT_PIN = 34;
const int SAMPLE_COUNT = 100;

const float ADC_LOW = 525.31f;
const float ADC_HIGH = 3180.32f;
const float V_LOW = 0.53f;
const float V_HIGH = 2.72f;

void setup() {
  Serial.begin(115200);
  analogReadResolution(12);
  analogSetPinAttenuation(POT_PIN, ADC_11db);

  Serial.println("mean_adc,simple_V,calibrated_V");
}

void loop() {
  uint32_t sum = 0;

  for (int i = 0; i < SAMPLE_COUNT; i++) {
    sum += analogRead(POT_PIN);
    delay(2);
  }

  float meanADC = sum / float(SAMPLE_COUNT);
  float simpleVoltage = meanADC * 3.3f / 4095.0f;

  float calibratedVoltage =
      V_LOW + (meanADC - ADC_LOW) *
      (V_HIGH - V_LOW) / (ADC_HIGH - ADC_LOW);

  Serial.print(meanADC, 2);
  Serial.print(",");
  Serial.print(simpleVoltage, 3);
  Serial.print(",");
  Serial.println(calibratedVoltage, 3);

  delay(500);
}
```
## Task 3b: LDR Calibration and Independent Validation
```cpp
import math

e1, r1 = 1.0, 10.625
e2, r2 = 0.09, 15.984

e_validation = 0.36
r_validation = 14.265

exponent = math.log(r2 / r1) / math.log(e2 / e1)
coefficient = r1 / e1**exponent

slope = (r2 - r1) / (e2 - e1)
intercept = r1 - slope * e1

predictions = {
    "Power law": coefficient * e_validation**exponent,
    "Linear": intercept + slope * e_validation,
}

for name, prediction in predictions.items():
    error = prediction - r_validation
    relative_error = error / r_validation * 100

    print(
        f"{name}: {prediction:.3f} kOhm, "
        f"error={error:+.3f} kOhm, "
        f"relative error={relative_error:+.2f}%"
    )
```
To see the results, please visit my teammate’s page. 
https://github.com/Conger-Y/iot-portfolio-template/blob/main/Module03/README.md
## Task 4: Hysteresis and Repeatability

### LDR Sweep

```python
from statistics import mean

fixed = [1.527, 1.527, 1.526]

returned = [
    1.559, 1.509, 1.439, 1.563, 1.506,
    1.456, 1.528, 1.563, 1.538, 1.504
]

for label, values in [
    ("Shaft held fixed", fixed),
    ("Returned to mark", returned),
]:
    print(label)
    print(f"Mean: {mean(values):.3f} V")
    print(f"Minimum: {min(values):.3f} V")
    print(f"Maximum: {max(values):.3f} V")
    print(f"Range: {max(values) - min(values):.3f} V")
To see the results, please visit my teammate’s page. 
https://github.com/Conger-Y/iot-portfolio-template/blob/main/Module03/README.md
```
## Task 5a: Noise and Filter Comparison
```cpp
const int INPUT_PIN = 15;
const int WINDOW = 5;
const float ALPHA = 0.2f;
const bool INJECT_SPIKE = false;

int buffer[WINDOW];
int position = 0;
int count = 0;
int sampleNumber = 0;

float ema = 0;
unsigned long lastSample = 0;

void setup() {
  Serial.begin(115200);
  analogReadResolution(12);
  analogSetPinAttenuation(INPUT_PIN, ADC_11db);

  Serial.println("sample,time_ms,raw,input,mean5,median5,ema");
}

void loop() {
  if (sampleNumber >= 500) return;
  if (millis() - lastSample < 20) return;

  lastSample = millis();
  sampleNumber++;

  int raw = analogRead(INPUT_PIN);
  int input = raw;

  if (INJECT_SPIKE && sampleNumber == 470) {
    input += 500;
  }

  buffer[position] = input;
  position = (position + 1) % WINDOW;

  if (count < WINDOW) count++;

  int sorted[WINDOW];
  long sum = 0;

  for (int i = 0; i < count; i++) {
    sorted[i] = buffer[i];
    sum += buffer[i];
  }

  for (int i = 1; i < count; i++) {
    int value = sorted[i];
    int j = i - 1;

    while (j >= 0 && sorted[j] > value) {
      sorted[j + 1] = sorted[j];
      j--;
    }

    sorted[j + 1] = value;
  }

  float movingAverage = sum / float(count);

  float median = (count % 2)
      ? sorted[count / 2]
      : (sorted[count / 2 - 1] + sorted[count / 2]) / 2.0f;

  if (sampleNumber == 1) {
    ema = input;
  } else {
    ema = ALPHA * input + (1.0f - ALPHA) * ema;
  }

  Serial.printf(
      "%d,%lu,%d,%d,%.2f,%.2f,%.2f\n",
      sampleNumber, lastSample, raw, input,
      movingAverage, median, ema
  );
}
```
## Task 5b: Threshold Hysteresis Demonstration
```python
temperatures = [3.0, 1.9, 2.1, 1.8, 2.2, 1.9, 2.6]

single_alarm = False
hysteresis_alarm = False

single_changes = 0
hysteresis_changes = 0

for temperature in temperatures:
    new_single = temperature <= 2.0
    new_hysteresis = hysteresis_alarm

    if temperature <= 2.0:
        new_hysteresis = True
    elif temperature >= 2.5:
        new_hysteresis = False

    single_changes += new_single != single_alarm
    hysteresis_changes += new_hysteresis != hysteresis_alarm

    single_alarm = new_single
    hysteresis_alarm = new_hysteresis

    print(
        f"{temperature:.1f} C: "
        f"single={single_alarm}, "
        f"hysteresis={hysteresis_alarm}"
    )

print("Single-threshold changes:", single_changes)
print("Hysteresis changes:", hysteresis_changes)
```
To see the results, please visit my teammate’s page. 
https://github.com/Conger-Y/iot-portfolio-template/blob/main/Module03/README.md
## Task 6: Uncertainty Budget
```python
import math

voltage = 1.472

u_reference = 6.0
u_repeatability = 0.72
u_model = 16.2

combined_mV = math.sqrt(
    u_reference**2 +
    u_repeatability**2 +
    u_model**2
)

coverage_factor = 2
expanded_V = coverage_factor * combined_mV / 1000

model_share = u_model**2 / combined_mV**2 * 100

print(f"Combined standard uncertainty: {combined_mV:.2f} mV")
print(
    f"Result: {voltage:.3f} +/- {expanded_V:.3f} V "
    f"(k={coverage_factor})"
)
print(f"Model contribution to variance: {model_share:.1f}%")
```
To see the results, please visit my teammate’s page. 
https://github.com/Conger-Y/iot-portfolio-template/blob/main/Module03/README.md
## Self-Reflection – Module 03
https://github.com/liusihan54/iot-portfolio-templateSihan/blob/main/Reflections/ref03.md
