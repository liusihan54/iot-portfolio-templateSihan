# Module 0 – Pre-Study

## 1. Sensor–Actuator Chain

## 2. Wokwi Simulation

## 3. Sensor Datasheet

## 4. Accuracy vs. Precision
**Accuracy und Precision beschreiben zwei verschiedene Eigenschaften einer Messung.**

- **Accuracy (Richtigkeit):** Wie nah ist der Messwert am tatsächlichen Wert?
- **Precision (Präzision):** Wie ähnlich sind die Ergebnisse bei mehreren Messungen?

**Beispiel 1: Hohe Präzision, aber geringe Richtigkeit**

Die tatsächliche Wassertemperatur beträgt 25 °C.

Ein Thermometer zeigt bei drei Messungen immer 28 °C an.

Die Ergebnisse sind sehr ähnlich, also ist die Präzision hoch. Aber die Messwerte sind falsch, weil die tatsächliche Temperatur 25 °C beträgt.

**Beispiel 2: Hohe Richtigkeit im Durchschnitt, aber geringe Präzision**

Die tatsächliche Wassertemperatur beträgt wieder 25 °C.

Ein anderes Thermometer zeigt bei drei Messungen 23 °C, 25 °C und 27 °C an.

Der Durchschnitt beträgt 25 °C und ist damit richtig. Die einzelnen Messwerte sind aber unterschiedlich. Deshalb ist die Präzision gering.

## 5. I²C Addressing

## 6. Actuator Choice

## 7. LED Power Calculation

Ein WS2812-LED-Streifen hat 30 LEDs. Jede LED benötigt bei voller weißer Helligkeit maximal ungefähr 60 mA.

**Berechnung:**

30 × 60 mA = 1800 mA = **1,8 A**

Das Netzteil muss also mindestens 1,8 A liefern können. Ein Netzteil mit etwas Reserve ist besser.

Der 5-V-Pin des Mikrocontrollers kann normalerweise nicht so viel Strom liefern. Wenn wir den LED-Streifen direkt darüber versorgen, kann es zu Problemen mit der Stromversorgung kommen. Im schlimmsten Fall kann die Hardware beschädigt werden.

Deshalb sollte der LED-Streifen ein geeignetes externes 5-V-Netzteil verwenden. Der Mikrocontroller steuert nur die LEDs. Beide müssen eine gemeinsame Masse (GND) haben.

## 8. Project Abstract

## Additional Notes

### Wokwi Screenshots

### Datasheet Notes
