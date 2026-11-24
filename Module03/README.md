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
