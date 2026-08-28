# Pinout (ESP32 DevKit)

| GPIO | Function | Connected to |
|------|----------|----------------|
| GPIO18 | PWM output channel 1 | 220 Ω → Gate IRLZ44N #1 |
| GPIO19 | PWM output channel 2 | 220 Ω → Gate IRLZ44N #2 |
| GPIO4 | 1-Wire bus | DS18B20 data pin (+ 4.7 kΩ pull-up to VCC) |
| GND | Common ground | MOSFET source, DS18B20 GND, 20V supply GND |
| 3.3V | Logic supply | DS18B20 VCC, pull-up resistor |

## Per channel (dimmer)

```
GPIO(18/19) ──[220 Ω]── Gate (IRLZ44N)
                          │
                        [10 kΩ]
                          │
                         GND ── Source

+20V ── LED+ (L+)
LED- (L−) ── Drain

TVS P6KE33A across the LED:
  Cathode (K) → LED+ (L+)
  Anode  (A) → LED- (L−)
```

## Temperature sensor (DS18B20)

```
VCC (3.3V) ──┬── DS18B20 VCC
             │
           [4.7 kΩ]
             │
GPIO4 ───────┴── DS18B20 Data

GND ── DS18B20 GND
```

## Open items / still to add

- [ ] Graphical schematic (Fritzing/KiCad) as PNG/SVG
- [ ] Photo of the actual build
- [ ] LED current per channel, whether MOSFET heatsinking is needed
- [ ] Document fuse/overcurrent protection on the 20V side
- [ ] Power supply spec (Sera 60W — max current per channel?)
