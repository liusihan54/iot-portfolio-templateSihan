# Module 0 – Pre-Study

## 1. Sensor–Actuator Chain

## 2. Wokwi Simulation

## 3. Sensor Datasheet

## 4. Accuracy vs. Precision

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
