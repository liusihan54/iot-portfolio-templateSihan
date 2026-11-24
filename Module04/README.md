# Module 4 – Liquid Level Measurement: Freshwater, Seawater, Diesel & Wastewater Tank Analysis
## 🎯 Learning Goals

Through multi-fluid experimental characterization, calibration modeling, and comparative sensor evaluation across Freshwater, Seawater, Diesel Fuel, and Wastewater tanks on the ESP32 and M5StickS3, this module achieves the following learning outcomes:

1. **Resistive Liquid Sensing, Pulsed Excitation & Electrode Drift (Freshwater & Seawater):**
   * Wire a resistive needle probe (`HD-38`) using pulsed GPIO power (`GPIO25`) to mitigate electrolysis and compare 2-point vs. 4-point inverse calibration models ($\text{ADC} = A + B/\text{level pct}$).
   * Quantify severe downward signal drift (`67%` drop under constant `100%` submersion) caused by electrode polarization, verify dry-baseline cycle repeatability (`~1.2%` variance), and explain why high ionic conductivity in seawater causes rapid saturation and accelerated trace corrosion.
2. **Non-Contact Ultrasonic Sensing & Sloshing Dynamics (Diesel Tank):**
   * Recognize why non-conductive hydrocarbons (diesel) require non-contact sensing and compare direct liquid-surface ultrasonic reflections against buoyant cork-float targets.
   * Evaluate dynamic tank rocking effects (`simulate_rocking.py`) and physical edge cases (unguided float tilting/sticking and near-field ultrasonic dead-zone saturation around `18–20 cm`).
3. **Optical Time-of-Flight (ToF) Scattering vs. Transparency Limits (`VL53L0X`):**
   * Compare `VL53L0X` laser ranging in clear water versus turbid simulated wastewater, demonstrating how infrared beam penetration and specular refraction cause `89%` invalid readings in clear water at full depth, whereas suspended particulates in turbid wastewater scatter the beam cleanly at the surface (`100%` valid readings, `7 mm` spread).
4. **Gravimetric Load Cell Sensing & Vessel Geometry Effects (`HX711`):**
   * Calibrate a strain-gauge load cell (`HX711`) to infer fluid mass and volume without chemical, acoustic, or optical contact, applying median filtering to reject mechanical placement outliers (`353.3 g`).
   * Account for non-cylindrical container geometry, verifying that half liquid depth in a tapered vessel corresponds to `40.3%` of full volume rather than `50%`.
5. **Engineering Trade-Offs & Maritime/Industrial Sensor Selection:**
   * Synthesize fluid properties (conductivity, turbidity, bio-fouling risk) and environmental dynamics (sloshing) into a structured selection matrix, justifying industrial best practices such as perforated stilling pipes (baffle tubes) for ultrasonic/radar tank gauges and external load cells for fouling-prone process vessels.
<img width="1280" height="1706" alt="aec9b1edacb80062622853c64adb15bc" src="https://github.com/user-attachments/assets/4b8b4100-4f5e-44a6-9d75-900e3cc77ad5" />

## Overview

This repository documents the experimental evaluation, calibration, and comparative analysis of low-cost liquid level sensing technologies (**Resistive Needle Probe HD-38 / HW-038**, **Ultrasonic Distance Sensor HC-SR04 / JSN-SR04T**, **Time-of-Flight Laser Sensor VL53L0X**, and **HX711 Load Cell**) across four different fluid mediums: **Freshwater**, **Seawater**, **Diesel Fuel**, and **Wastewater**.

The goal is to assess sensor linearity, signal drift, repeatability, environmental interference (sloshing/rocking), turbidity/optical scattering, and medium compatibility for maritime and industrial tank monitoring applications.

---

# Module 04 – Task 3: Wastewater Holding Tank Analysis

This task compares three measurement methods on simulated wastewater:

1. Ultrasonic distance measurement
2. VL53L0X optical time-of-flight measurement
3. HX711 load-cell measurement

The following sketches were reconstructed from the experiment record. They are not recovered original source files. Upload each sketch separately to its corresponding board.

## 3.1 Ultrasonic Distance Measurement

Board: ESP32 Dev Module.

| Connection | ESP32 pin |
|---|---|
| TRIG | GPIO25 |
| ECHO | GPIO26 |
| Ground | GND |

Use the supply voltage specified for the actual ultrasonic module. If its ECHO output is 5 V, reduce it to an ESP32-compatible voltage before connecting GPIO26.

This sketch collects five valid measurements and uses their median. Invalid bursts produce an error message instead of a water-level classification.

The original alarm thresholds were not recorded. The example values below must be checked against the actual mounting height and measured high-water position.

```cpp
const int TRIG_PIN = 25;
const int ECHO_PIN = 26;
const int SAMPLE_COUNT = 5;

// Example thresholds, not recovered experimental settings.
const float ALARM_ON_CM = 15.0f;
const float ALARM_OFF_CM = 16.0f;

bool highWater = false;

float readDistanceCm() {
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);

  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);

  unsigned long duration = pulseIn(ECHO_PIN, HIGH, 30000);

  if (duration == 0) return -1.0f;

  return duration * 0.0343f / 2.0f;
}

void setup() {
  Serial.begin(115200);
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);
  digitalWrite(TRIG_PIN, LOW);
}

void loop() {
  float distances[SAMPLE_COUNT];
  int valid = 0;

  for (int attempt = 0;
       attempt < 10 && valid < SAMPLE_COUNT;
       attempt++) {
    float distance = readDistanceCm();

    if (distance > 0) {
      distances[valid++] = distance;
    }

    delay(70);
  }

  if (valid < SAMPLE_COUNT) {
    Serial.println("READ ERROR | Water state unknown");
    delay(500);
    return;
  }

  for (int i = 1; i < SAMPLE_COUNT; i++) {
    float value = distances[i];
    int j = i - 1;

    while (j >= 0 && distances[j] > value) {
      distances[j + 1] = distances[j];
      j--;
    }

    distances[j + 1] = value;
  }

  float distanceCm = distances[SAMPLE_COUNT / 2];

  if (distanceCm <= ALARM_ON_CM) {
    highWater = true;
  } else if (distanceCm >= ALARM_OFF_CM) {
    highWater = false;
  }

  Serial.print("Distance (cm): ");
  Serial.print(distanceCm, 2);

  if (highWater) {
    Serial.println(" | HIGH WATER");
  } else {
    Serial.println(" | NORMAL");
  }

  delay(300);
}
```

The distance decreases as the water surface rises, provided the sensor is measuring the intended surface.

Keep the sensor fixed between fill conditions. Measurements can be collected after adding water and allowing it to settle; continuous pouring is not required.

A median filter reduces isolated outliers but does not correct sidewall reflections or measurements inside the sensor's minimum operating distance.

## 3.2 VL53L0X ToF Measurement

Board: M5StickS3.  
Required library: Adafruit VL53L0X.

The original experiment sketch used:

| Connection | Pin / value |
|---|---|
| SDA | GPIO9 |
| SCL | GPIO10 |
| I²C address | 0x29 |
| I²C clock | 100 kHz |

The sensor was mounted approximately 10 cm above the cup rim. Its position remained fixed when comparing water levels.

```cpp
#include <Wire.h>
#include <Adafruit_VL53L0X.h>

Adafruit_VL53L0X tof;
bool sensorReady = false;

void setup() {
  Serial.begin(115200);
  delay(1000);

  Wire.begin(9, 10, 100000);
  sensorReady = tof.begin(0x29, false, &Wire);
}

void loop() {
  if (!sensorReady) {
    Serial.println("VL53L0X init failed. Check wiring.");
    delay(1000);
    return;
  }

  VL53L0X_RangingMeasurementData_t reading = {};
  const auto result = tof.rangingTest(&reading, false);

  if (result != VL53L0X_ERROR_NONE) {
    Serial.printf(
        "READ ERROR, code=%d\n",
        static_cast<int>(result)
    );
  } else if (reading.RangeStatus != 0) {
    Serial.printf(
        "INVALID, status=%u\n",
        static_cast<unsigned>(reading.RangeStatus)
    );
  } else {
    Serial.printf(
        "Distance: %u mm\n",
        static_cast<unsigned>(reading.RangeMilliMeter)
    );
  }

  delay(500);
}
```

Invalid readings are retained in the log rather than converted to zero or included in distance statistics.

Cardboard across the cup rim was used as a solid-target control. Its readings represent the cardboard position, not the liquid surface.

### Python: Summarize the Recorded ToF Results

This added script uses the last five numeric readings transcribed in the report. Valid-output counts refer to the larger recorded output windows.

```python
from statistics import median

tests = {
    "Empty cup": [240, 234, 240, 239, 238],
    "Cardboard control": [106, 103, 104, 105, 102],
    "Clear water, half": [237, 235, 242, 206, 226],
    "Wastewater, half": [226, 229, 219, 226, 230],
    "Wastewater, full": [135, 141, 138, 134, 137],
}

for name, values in tests.items():
    print(
        f"{name}: median={median(values):.0f} mm, "
        f"span={max(values) - min(values)} mm"
    )

for name, valid, total in [
    ("Clear water, full", 2, 19),
    ("Wastewater, half", 16, 19),
    ("Wastewater, full", 19, 19),
]:
    print(
        f"{name}: {valid}/{total} valid "
        f"({100 * valid / total:.1f}%)"
    )
```

These observations describe the tested samples and setup. They do not establish reliable performance for every wastewater composition.

## 3.3 HX711 Load-Cell Measurement

Board: ESP32 Dev Module.  
Required library: HX711 by bogde.

GPIO32 and GPIO33 below must be checked against the actual wiring because the linked record does not specify the HX711 pins.

The original calibration factor was not recorded. This reconstructed sketch calculates a new factor using the documented 20 g reference mass.

### Calibration Procedure

1. Leave the weighing platform unloaded.
2. Send `t` in the Serial Monitor to tare.
3. Place the 20 g reference mass on the platform.
4. Wait for it to settle, then send `c`.
5. Remove the reference mass and measure the empty cup.
6. Measure the cup at the selected fill conditions.

Do not tare with the empty cup if the intention is to reproduce gross cup-plus-water readings. Subtract the measured empty-cup baseline afterward.

```cpp
#include <HX711.h>

const int DOUT_PIN = 32;  // Verify actual wiring.
const int SCK_PIN = 33;   // Verify actual wiring.
const float REFERENCE_MASS_G = 20.0f;

HX711 scale;

bool tared = false;
bool calibrated = false;

void setup() {
  Serial.begin(115200);

  scale.begin(DOUT_PIN, SCK_PIN);
  scale.set_scale(1.0f);

  Serial.println("Unload platform; send t to tare.");
  Serial.println("Then place 20 g reference; send c.");
}

void loop() {
  if (!scale.wait_ready_timeout(1000)) {
    Serial.println("HX711 not ready.");
    return;
  }

  if (Serial.available()) {
    char command = Serial.read();

    if (command == 't') {
      scale.tare(10);
      tared = true;
      Serial.println("Tare complete.");
    }

    if (command == 'c') {
      if (!tared) {
        Serial.println("Tare the unloaded platform first.");
      } else {
        double referenceCounts = scale.get_value(10);

        if (referenceCounts == 0) {
          Serial.println("Calibration failed: zero response.");
        } else {
          float factor = referenceCounts / REFERENCE_MASS_G;
          scale.set_scale(factor);
          calibrated = true;

          Serial.print("Calibration factor: ");
          Serial.println(factor, 6);
        }
      }
    }
  }

  if (tared && calibrated) {
    Serial.print("Mass (g): ");
    Serial.println(scale.get_units(5), 1);
  }

  delay(500);
}
```

The calibration is stored in memory only and must be repeated after a restart.

The sketch averages five readings. The report's median values were calculated across the recorded readings afterward.

Calibration with a 20 g reference is not sufficient to establish accuracy near the approximately 400 g full-cup load. Additional reference masses closer to the working range would improve validation.

### Python: Calculate Net Water Mass and Volume

This added script reproduces the calculations from the recorded measurements.

```python
from statistics import mean, median

empty = [7.7, 7.7, 8.2, 8.3]
half = [161.9, 161.2]
full = [388.4, 353.3, 389.5, 390.7]

empty_mass = median(empty)
half_gross = mean(half)
full_gross = median(full)

half_net = half_gross - empty_mass
full_net = full_gross - empty_mass

# Approximation used in the report.
density_g_per_ml = 1.0

half_volume = half_net / density_g_per_ml
full_volume = full_net / density_g_per_ml
volume_fraction = half_volume / full_volume * 100

print(f"Empty-cup baseline: {empty_mass:.2f} g")
print(f"Half-depth net mass: {half_net:.2f} g")
print(f"Full-cup net mass: {full_net:.2f} g")
print(f"Half-depth estimated volume: {half_volume:.1f} mL")
print(f"Full-cup estimated volume: {full_volume:.1f} mL")
print(f"Half-depth fraction of full volume: {volume_fraction:.1f}%")
```

The alarm activates when the measured distance is at or below 15 cm and clears when the distance reaches or exceeds 16 cm. Between these thresholds, the previous alarm state is retained to reduce repeated switching.

## Self-Reflection
https://github.com/liusihan54/iot-portfolio-templateSihan/blob/main/Reflections/ref04.md
