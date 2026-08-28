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
- Extendable dawn/dusk twilight phase: one channel can hold at a low
  brightness for a configurable window before the main ramp continues,
  so you can lengthen observation time without extending the
  photosynthetically relevant light period
- Configurable brightness lead between channels during sunrise/sunset
- Dynamic cloud simulation with randomized intensity and duration
- Adjustable base brightness and siesta (midday dimming)
- Full control via Home Assistant UI
- Fine-grained dimming control using PWM
- Weather-based light adaptation *(experimental, not currently in active use — see `homeassistant/experimental/`)*

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
  - Base brightness (`aq_base_day_pct`)
  - Cloud intensity / duration (`aq_cloud_intensity`, `aq_cloud_duration`)
  - Siesta brightness and duration (`aq_siesta_pct`, `aq_siesta_minutes`)
  - Dawn/dusk twilight brightness (`aq_dawn_twilight_pct`, `aq_dusk_twilight_pct`)

- `input_datetime`:
  - Sunrise / sunset trigger time (`aq_sunrise_time`, `aq_sunset_time`)
  - Siesta start (`aq_siesta_start`)
  - Dawn/dusk twilight end time (`aq_dawn_twilight_until`, `aq_dusk_twilight_until`) —
    when the held channel resumes ramping toward full brightness

- `input_boolean`:
  - Enable clouds (`aq_clouds_enable`)
  - Enable siesta (`aq_siesta_enable`)
  - Enable dawn/dusk twilight phase (`aq_dawn_twilight_enable`, `aq_dusk_twilight_enable`)
  - Enable weather control (`aq_weather_link`) *(experimental, see `homeassistant/experimental/`)*
  - Internal run-state flags, set/cleared by the scripts themselves —
    not meant to be toggled manually (`aq_sunrise_running`, `aq_sunset_running`)

#### Scripts

- `aq_sunrise`:
  - Channel 2 (warm/sunrise) starts first
  - Channel 1 (cool daylight) follows once channel 2's brightness is
    ahead by a configurable lead (`ch2_lead`, default 5 %)
  - Optional dawn twilight phase: channel 2 ramps to a low
    "twilight brightness" and holds there until a configured end time,
    while channel 1 stays off — extends the observable transition
    without extending the main light period
  - Both ramp up gradually to base brightness (`aq_base_day_pct`)

- `aq_sunset`:
  - Channel 1 dims first, unaffected by the twilight option
  - Channel 2 follows once channel 1 has dropped far enough
    (`ch1_lead`, default 5 %)
  - Optional dusk twilight phase: channel 2 holds at a low
    "twilight brightness" until a configured end time before dimming
    the rest of the way to 0

- `aq_clouds_dynamic`:
  - Randomized brightness dips, relative to current base brightness
  - Duration and intensity configurable via `aq_cloud_duration` /
    `aq_cloud_intensity`

- `aq_siesta`:
  - Temporary midday dimming to `aq_siesta_pct` for `aq_siesta_minutes`,
    then back to `aq_base_day_pct`

- `aq_alle_stoppen`:
  - Emergency stop: turns off all lighting scripts and resets the
    running-state helpers

- `aq_weather_link_update` *(experimental, see `homeassistant/experimental/`)*:
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
- Additional environmental simulations (e.g. storms)
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
    aq_clouds_dynamic.yaml
    aq_siesta.yaml
    aq_alle_stoppen.yaml
  automations/
    aq_start_sunrise.yaml
    aq_start_sunset.yaml
    aq_start_siesta.yaml
    aq_wolken_stoppen_bei_sonnenuntergang.yaml
    aq_co2_steuerung.yaml     ← Beispiel: externe Verbraucher an die Lichtlogik koppeln
  experimental/
    README.md                 ← nicht eingebunden, nur Referenz
    scripts/aq_weather_link_update.yaml
    automations/aq_wetterkopplung_automatisch.yaml

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

