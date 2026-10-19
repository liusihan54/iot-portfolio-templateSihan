# Module 0 – Pre-Study

## 1. Sensor–Actuator Chain
**Beispiel: Bluetooth-Kopfhörer mit aktiver Geräuschunterdrückung (ANC)**

Ich benutze Bluetooth-Kopfhörer mit aktiver Geräuschunterdrückung.

**Signalkette:**

Schallwellen → Mikrofon → Signalaufbereitung → ADC → DSP/MCU → Lautsprecher

**Erklärung der einzelnen Schritte:**

1. **Schallwellen:** Geräusche aus der Umgebung treffen auf das Mikrofon.
2. **Mikrofon (Sensor):** Das Mikrofon wandelt die Schallwellen in ein elektrisches Signal um.
3. **Signalaufbereitung:** Das Signal wird verstärkt und gefiltert.
4. **ADC:** Das analoge Signal wird in digitale Werte umgewandelt.
5. **DSP/MCU:** Der Prozessor analysiert die Geräusche und berechnet ein Gegensignal.
6. **Lautsprecher (Aktor):** Der Lautsprecher gibt das Gegensignal aus, um die Umgebungsgeräusche zu reduzieren.

So arbeiten Sensor, Signalverarbeitung und Aktor zusammen.

## 2. Wokwi Simulation

## 3. Sensor Datasheet
**Gewählter Sensor: VL53L0X (Abstandssensor)**

Der VL53L0X ist ein Abstandssensor. Er misst die Entfernung zu einem Objekt mit der Time-of-Flight-Methode (ToF). Dabei verwendet er Infrarotlicht.

**Schnittstelle:** I²C

**I²C-Adresse:** 0x29 (7-Bit-Adresse)

**Genauigkeit:** Laut Datenblatt beträgt die typische Genauigkeit im High-Accuracy-Modus bei einer Entfernung von 1,2 m ungefähr ±3 %.

Die Genauigkeit hängt von den Messbedingungen ab, zum Beispiel vom Umgebungslicht, der Entfernung und der Oberfläche des Objekts. Bei starkem Umgebungslicht oder einer schlecht reflektierenden Oberfläche können größere Messfehler auftreten.

**Quelle:** STMicroelectronics – VL53L0X Datasheet

https://www.st.com/resource/en/datasheet/vl53l0x.pdf

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
Bei I²C können mehrere Sensoren dieselben zwei Leitungen benutzen: SDA (Datenleitung) und SCL (Taktleitung).

Das Problem ist, dass der Mikrocontroller die Sensoren unterscheiden muss. Wenn zwei Sensoren dieselbe I²C-Adresse haben, kann es zu einem Adresskonflikt kommen.

Deshalb hat jeder Sensor auf dem Bus normalerweise eine eigene Adresse. Der Mikrocontroller verwendet diese Adresse, um einen bestimmten Sensor anzusprechen und Daten mit ihm auszutauschen.

Wenn zwei Sensoren dieselbe Adresse haben, kann man zum Beispiel die Adresse eines Sensors ändern oder einen I²C-Multiplexer verwenden.

## 6. Actuator Choice
**Ich würde für einen kleinen Türverschluss einen Servomotor wählen.**

Ein Servomotor kann sich gezielt auf einen bestimmten Winkel bewegen. Dadurch kann er den Türverschluss öffnen und schließen.

**Die Vorteile sind:**

- **Drehmoment:** Ein passender Servomotor hat genug Kraft, um einen kleinen Türverschluss zu bewegen.
- **Steuerung:** Die Steuerung ist einfach, weil man den gewünschten Winkel vorgeben kann.
- **Stromverbrauch:** Der Motor muss nur alle paar Minuten arbeiten. Wenn der Verschluss mechanisch in seiner Position bleibt, kann der Servo zwischen den Bewegungen abgeschaltet werden.

**Ich würde keinen normalen Gleichstrommotor ohne zusätzliche Steuerung verwenden.**

Ein normaler Gleichstrommotor dreht sich kontinuierlich und kann nicht automatisch an einer bestimmten Position stoppen. Dafür braucht man zusätzliche Sensoren oder eine andere Steuerung.

Deshalb ist ein Servomotor für diese Aufgabe einfacher und praktischer.

## 7. LED Power Calculation

Ein WS2812-LED-Streifen hat 30 LEDs. Jede LED benötigt bei voller weißer Helligkeit maximal ungefähr 60 mA.

**Berechnung:**

30 × 60 mA = 1800 mA = **1,8 A**

Das Netzteil muss also mindestens 1,8 A liefern können. Ein Netzteil mit etwas Reserve ist besser.

Der 5-V-Pin des Mikrocontrollers kann normalerweise nicht so viel Strom liefern. Wenn wir den LED-Streifen direkt darüber versorgen, kann es zu Problemen mit der Stromversorgung kommen. Im schlimmsten Fall kann die Hardware beschädigt werden.

Deshalb sollte der LED-Streifen ein geeignetes externes 5-V-Netzteil verwenden. Der Mikrocontroller steuert nur die LEDs. Beide müssen eine gemeinsame Masse (GND) haben.

## 8. Project Abstract

### Wokwi Screenshots

### Datasheet Notes

#### 1. VL53L0X – Abstandssensor

Der VL53L0X misst Entfernungen mit Infrarotlicht. Er verwendet die Time-of-Flight-Methode (ToF).

**Technische Daten:**

- **Messgröße:** Abstand
- **Messprinzip:** Time of Flight (ToF)
- **Betriebsspannung:** 2,6–3,5 V
- **Schnittstelle:** I²C
- **I²C-Adresse:** 0x29 (7 Bit)
- **Messbereich:** Bis zu etwa 2 m unter günstigen Bedingungen
- **Auflösung:** Entfernungsausgabe in Millimetern
- **Genauigkeit:** Im High-Accuracy-Modus bei 1,2 m typischerweise besser als ±3 %
- **Messzeit:** Je nach Modus etwa 20–200 ms

**Bedingungen und Einschränkungen:**

Die Messung hängt vom Umgebungslicht, von der Entfernung und von der Oberfläche des Objekts ab. Besonders bei starkem Licht oder dunklen Oberflächen können größere Messfehler auftreten. Auch Temperaturänderungen können die Messung beeinflussen.

**Quelle:** [VL53L0X Datenblatt – STMicroelectronics](https://www.st.com/resource/en/datasheet/vl53l0x.pdf)

---

#### 2. MPR121 – Kapazitiver Berührungssensor

Der MPR121 erkennt Berührungen durch Änderungen der elektrischen Kapazität. Er kann bis zu 12 Elektroden überwachen.

**Technische Daten:**

- **Messgröße:** Elektrische Kapazität bzw. Kapazitätsänderung
- **Messprinzip:** Kapazitive Messung
- **Betriebsspannung:** 1,71–3,6 V
- **Schnittstelle:** I²C
- **I²C-Adresse:** 0x5A bis 0x5D, abhängig von der Beschaltung
- **Messbereich:** Ungefähr 10 pF bis über 2000 pF
- **Auflösung:** Bis zu 0,01 pF
- **Reaktionszeit:** Beispielsweise 64 ms bei einer Abtastrate von 16 ms
- **Genauigkeit:** Keine allgemeine Genauigkeit in Prozent angegeben. Die Berührungserkennung hängt von der Anwendung und den Einstellungen ab.

**Bedingungen und Einschränkungen:**

Die Messung hängt von der Größe und Form der Elektroden sowie von den Einstellungen ab. Elektrische Störungen und Veränderungen in der Umgebung können die Berührungserkennung beeinflussen.

**Quelle:** [MPR121 Datenblatt – NXP](https://www.nxp.com/docs/en/data-sheet/MPR121.pdf)
