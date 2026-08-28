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
