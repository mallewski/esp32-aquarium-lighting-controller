# Aquarium-Steuerung
ESP32 basierter Dimmer und Temperatursensor für Home Assistant via ESPHome

# Aquarium Lighting Control with ESP32 & Home Assistant

## Overview

This project implements a fully customizable aquarium lighting system using an ESP32, PWM dimming, and Home Assistant automation. The goal is to simulate natural lighting conditions such as sunrise, sunset, cloud cover, and weather-dependent brightness using a cost-effective hardware setup.

The system is designed for Sera LED tubes (20 V DC) but can be adapted to other LED systems.

---

## Features

- Two independent light channels (e.g. warm/red and cool/blue-white)
- Simulated sunrise and sunset with adjustable timing
- Configurable delay between channels (e.g. warm light starts first)
- Dynamic cloud simulation with randomized intensity and duration
- Weather-based light adaptation using Home Assistant weather integration
- Adjustable base brightness and siesta (midday dimming)
- Full control via Home Assistant UI
- Fine-grained dimming control using PWM

---

## Hardware

### Components

- ESP32 DevKit
- IRLZ44N MOSFET (per channel)
- Resistors:
  - 220 Ω (gate resistor)
  - 10 kΩ (pull-down)
- Optional: PC817 optocoupler
- 20 V DC power supply (e.g. Sera 60 W)
- Sera LED tubes (Daylight / Sunrise)

### Wiring (per channel)

- GPIO → 220 Ω → MOSFET Gate
- Gate → 10 kΩ → GND
- Source → GND
- Drain → LED negative (L−)
- LED positive (L+) → +20 V
- Shared ground between ESP32 and power supply

---

## Software Architecture

### ESPHome

- PWM output via `ledc`
- Light entity (`monochromatic`)
- Adjustable frequency (typically 300–1000 Hz)
- Optional gamma correction for improved low-end dimming

### Home Assistant

#### Inputs

- `input_number`:
  - Base brightness
  - Cloud intensity
  - Cloud duration
  - Siesta brightness and duration

- `input_datetime`:
  - Sunrise time
  - Sunset time
  - Siesta start

- `input_boolean`:
  - Enable clouds
  - Enable weather control
  - Enable siesta

#### Scripts

- `aq_sunrise`:
  - Channel 2 (warm) starts first
  - Channel 1 (cool) follows with delay (e.g. 140 seconds)
  - Both ramp up gradually to base brightness

- `aq_sunset`:
  - Channel 1 dims first
  - Channel 2 follows
  - Light becomes warmer during fade-out

- `aq_clouds`:
  - Randomized brightness dips
  - Duration and intensity configurable
  - Relative to current brightness

- `aq_weather_link_update`:
  - Maps weather states to brightness and cloud intensity

---

## Dimming Behavior

The system uses low-side PWM dimming. At very low brightness levels (1–3%), LED behavior becomes non-linear. The project includes:

- Adjustable PWM frequency
- Gamma correction
- Optional signal mapping

The original Sera dimmer shows a more pronounced red tone at low brightness, likely due to internal channel mixing or current control rather than pure PWM.

---

## Measurement and Analysis

A secondary ESP32 can be used as a measurement device:

- Voltage divider (100 kΩ / 10 kΩ)
- ADC sampling on GPIO
- Web-based oscilloscope for PWM signal visualization

This allows reverse engineering of the original dimmer behavior.

---

## Known Limitations

- PWM dimming alone does not change LED spectrum
- Very low brightness range is hardware-dependent
- Home Assistant scripts require careful synchronization when using parallel execution

---

## Future Improvements

- Improved color blending between channels
- More advanced dimming curves
- Better replication of original dimmer behavior
- Additional environmental simulations (e.g. storms, seasonal changes)
- Hardware current control instead of pure PWM

---
## Repository Structure

```
esphome/
  aquarium-steuerung.yaml     ← ESPHome-Kerngerät: PWM-Dimmer, Temp-Sensor
  secrets.yaml.example        ← Vorlage, secrets.yaml selbst NICHT einchecken

homeassistant/
  packages/
    aquarium_package.yaml     ← input_number/input_boolean/input_datetime, Template-Sensoren
  scripts/
    aq_sunrise.yaml
    aq_sunset.yaml
    TODO_missing_scripts.md   ← aq_clouds_dynamic / aq_weather_link_update fehlen noch
  automations/
    aq_start_sunrise.yaml
    aq_start_sunset.yaml
    aq_start_siesta.yaml
    aq_wolken_stoppen_bei_sonnenuntergang.yaml
    aq_wetterkopplung_automatisch.yaml
    aq_co2_steuerung.yaml     ← Beispiel: externe Verbraucher an die Lichtlogik koppeln

docs/
  pinout.md                   ← GPIO-Belegung, Verdrahtungsdetails
  (wiring-diagram.png)        ← TODO: grafisches Schaltbild ergänzen

.gitignore
LICENSE
README.md
```

### Einbindung in Home Assistant

```yaml
# configuration.yaml
homeassistant:
  packages: !include_dir_named packages

script: !include_dir_named scripts
automation: !include_dir_list automations
```

> Die Inhalte unter `homeassistant/` sind **Beispiele/Inspiration**, wie
> das ESPHome-Dimmer-Setup in einer größeren Beleuchtungslogik genutzt
> werden kann — kein Teil der Kernfunktionalität des Projekts.

