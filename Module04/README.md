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
