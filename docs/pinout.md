# Pinbelegung (ESP32 DevKit)

| GPIO | Funktion | Verbunden mit |
|------|----------|----------------|
| GPIO18 | PWM-Ausgang Kanal 1 | 220 Ω → Gate IRLZ44N #1 |
| GPIO19 | PWM-Ausgang Kanal 2 | 220 Ω → Gate IRLZ44N #2 |
| GPIO4 | 1-Wire Bus | DS18B20 Data-Pin (+ 4.7 kΩ Pull-up nach VCC) |
| GND | Gemeinsame Masse | MOSFET Source, DS18B20 GND, 20V-Netzteil GND |
| 3.3V | Logikversorgung | DS18B20 VCC, Pull-up-Widerstand |

## Pro Kanal (Dimmer)

```
GPIO(18/19) ──[220 Ω]── Gate (IRLZ44N)
                          │
                        [10 kΩ]
                          │
                         GND ── Source

+20V ── LED+ (L+)
LED- (L−) ── Drain

TVS P6KE33A parallel zur LED:
  Kathode (K) → LED+ (L+)
  Anode  (A) → LED- (L−)
```

## Temperatursensor (DS18B20)

```
VCC (3.3V) ──┬── DS18B20 VCC
             │
           [4.7 kΩ]
             │
GPIO4 ───────┴── DS18B20 Data

GND ── DS18B20 GND
```

## Offene Punkte / noch zu ergänzen

- [ ] Grafisches Schaltbild (Fritzing/KiCad) als PNG/SVG hier ablegen
- [ ] Foto vom realen Aufbau
- [ ] Angabe: LED-Strom pro Kanal, benötigte MOSFET-Kühlung (ja/nein)
- [ ] Sicherung/Überstromschutz auf der 20V-Seite dokumentieren
- [ ] Netzteil-Spezifikation (Sera 60W — welcher Strom max. pro Kanal?)
