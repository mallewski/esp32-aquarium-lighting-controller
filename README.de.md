# Aquarium-Beleuchtungssteuerung mit ESP32 & Home Assistant

[English version](README.md)

## Überblick

Dieses Projekt implementiert ein vollständig anpassbares Aquarium-Beleuchtungssystem mit einem ESP32, PWM-Dimmung und Home-Assistant-Automation. Ziel ist es, natürliche Lichtverhältnisse wie Sonnenauf-/-untergang, Wolkendecke und wetterabhängige Helligkeit mit kostengünstiger Hardware zu simulieren.

Das System ist für Sera-LED-Röhren (20 V DC) ausgelegt, lässt sich aber an andere LED-Systeme anpassen.

---

## Features

- Zwei unabhängige Lichtkanäle (z.B. warm/rot und kühl/blau-weiß)
- Simulierter Sonnenauf- und -untergang mit einstellbarem Timing
- Erweiterbare Dämmerungsphase (Dawn/Dusk Twilight): Ein Kanal kann für ein konfigurierbares Zeitfenster auf niedriger Helligkeit gehalten werden, bevor die eigentliche Rampe weiterläuft – so lässt sich die Beobachtungszeit verlängern, ohne die photosynthetisch relevante Lichtphase zu strecken
- Konfigurierbarer Helligkeits-Vorsprung zwischen den Kanälen bei Sonnenauf-/-untergang
- Einstellbare Rampen-Geschwindigkeit (Sekunden pro 1%-Helligkeitsschritt), live über die Home-Assistant-Oberfläche änderbar
- Konfigurierbare Rampen-Geschwindigkeit (Sekunden pro 1%-Helligkeitsschritt), gilt für Haupt- und Dämmerungs-Rampen gleichermaßen
- Dynamische Wolkensimulation mit randomisierter Intensität und Dauer
- Einstellbare Basishelligkeit und Siesta (Mittagsabsenkung)
- Volle Steuerung über die Home-Assistant-Oberfläche
- Feingranulare Dimmung über PWM
- Wassertemperatur-Überwachung via DS18B20-Sensor
- Wetterabhängige Lichtanpassung *(experimentell, aktuell nicht im aktiven Einsatz — siehe `homeassistant/experimental/`)*

---

## Hardware

### Komponenten

- ESP32 DevKit
- IRLZ44N MOSFET (pro Kanal)
- Widerstände:
  - 220 Ω (Gate-Widerstand)
  - 10 kΩ (Pull-down)
- TVS-Diode P6KE33A (pro Kanal, parallel zur LED — Überspannungsschutz)
- Optional: PC817 Optokoppler
- 20 V DC Netzteil (z.B. Sera 60 W)
- Sera LED-Röhren (Daylight / Sunrise)
- DS18B20 Wassertemperatursensor + 4,7 kΩ Pull-up-Widerstand

### Verdrahtung (pro Kanal)

- GPIO → 220 Ω → MOSFET Gate
- Gate → 10 kΩ → GND
- Source → GND
- Drain → LED minus (L−)
- LED plus (L+) → +20 V
- TVS-Diode parallel zur LED (K an L+, A an L−)
- Gemeinsame Masse zwischen ESP32 und Netzteil

Die vollständige GPIO-Tabelle und die Sensor-Verdrahtung stehen in `docs/pinout.md`. Ein grafisches Schaltbild und Fotos vom Aufbau fehlen noch — Beiträge willkommen.

---

## Software-Architektur

### ESPHome

- PWM-Ausgang über `ledc`
- Light-Entity (`monochromatic`)
- Einstellbare Frequenz (typisch 300–1000 Hz)
- Optionale Gamma-Korrektur für bessere Dimmung im unteren Bereich

### Home Assistant

#### Inputs

- `input_number`:
  - Basishelligkeit (`aq_base_day_pct`)
  - Wolken-Intensität / -Dauer (`aq_cloud_intensity`, `aq_cloud_duration`)
  - Siesta-Helligkeit und -Dauer (`aq_siesta_pct`, `aq_siesta_minutes`)
  - Dämmerungshelligkeit Sonnenauf-/-untergang (`aq_dawn_twilight_pct`, `aq_dusk_twilight_pct`)
  - Rampen-Geschwindigkeit, Sekunden pro 1%-Schritt (`aq_ramp_step_delay`)
  - Rampen-Geschwindigkeit, Sekunden pro 1%-Schritt (`aq_ramp_step_delay`)

- `input_datetime`:
  - Trigger-Zeit Sonnenauf-/-untergang (`aq_sunrise_time`, `aq_sunset_time`)
  - Siesta-Start (`aq_siesta_start`)
  - Dämmerungsende Sonnenauf-/-untergang (`aq_dawn_twilight_until`, `aq_dusk_twilight_until`) —
    Zeitpunkt, an dem der gehaltene Kanal weiter Richtung Volllicht rampt

- `input_boolean`:
  - Wolken aktiv (`aq_clouds_enable`)
  - Siesta aktiv (`aq_siesta_enable`)
  - Dämmerungsphase Sonnenauf-/-untergang aktiv (`aq_dawn_twilight_enable`, `aq_dusk_twilight_enable`)
  - Wetterkopplung aktiv (`aq_weather_link`) *(experimentell, siehe `homeassistant/experimental/`)*
  - Interne Laufstatus-Flags, werden von den Scripts selbst gesetzt/zurückgesetzt —
    nicht zur manuellen Bedienung gedacht (`aq_sunrise_running`, `aq_sunset_running`)

#### Scripts

- `aq_sunrise`:
  - Kanal 2 (warm/Sunrise) startet zuerst
  - Kanal 1 (kühles Tageslicht) folgt, sobald Kanal 2 um einen konfigurierbaren
    Vorsprung voraus ist (`ch2_lead`, Standard 5 %)
  - Optionale Dämmerungsphase: Kanal 2 fährt auf eine niedrige
    „Dämmerungshelligkeit" hoch und hält dort bis zu einer konfigurierten
    Endzeit, während Kanal 1 aus bleibt — verlängert die beobachtbare
    Übergangsphase, ohne die Hauptlichtphase zu strecken
  - Beide fahren anschließend graduell auf die Basishelligkeit (`aq_base_day_pct`) hoch

- `aq_sunset`:
  - Kanal 1 dimmt zuerst, unbeeinflusst von der Dämmerungsoption
  - Kanal 2 folgt, sobald Kanal 1 weit genug abgesunken ist
    (`ch1_lead`, Standard 5 %)
  - Optionale Dämmerungsphase: Kanal 2 hält auf niedriger
    „Dämmerungshelligkeit" bis zu einer konfigurierten Endzeit, bevor
    er den Rest bis auf 0 abdimmt

- `aq_clouds_dynamic`:
  - Randomisierte Helligkeits-Einbrüche, relativ zur aktuellen Basishelligkeit
  - Dauer und Intensität konfigurierbar über `aq_cloud_duration` /
    `aq_cloud_intensity`

- `aq_siesta`:
  - Temporäre Mittagsabsenkung auf `aq_siesta_pct` für `aq_siesta_minutes`,
    danach zurück auf `aq_base_day_pct`

- `aq_alle_stoppen`:
  - Notaus: schaltet alle Beleuchtungs-Scripts aus und setzt die
    Laufstatus-Helper zurück

- `aq_weather_link_update` *(experimentell, siehe `homeassistant/experimental/`)*:
  - Mappt Wetterzustände auf Helligkeit und Wolken-Intensität

---

## Dimmverhalten

Das System nutzt Low-Side-PWM-Dimmung. Bei sehr niedrigen Helligkeiten (1–3%) wird das LED-Verhalten nichtlinear. Das Projekt enthält dafür:

- Einstellbare PWM-Frequenz
- Gamma-Korrektur
- Optionales Signal-Mapping

Der originale Sera-Dimmer zeigt bei niedriger Helligkeit einen stärkeren Rotstich, vermutlich durch interne Kanalmischung oder Stromregelung statt reinem PWM.

---

## Messung und Analyse

Ein zweiter ESP32 kann als Messgerät dienen:

- Spannungsteiler (100 kΩ / 10 kΩ)
- ADC-Abtastung an GPIO
- Web-basiertes Oszilloskop zur PWM-Signal-Visualisierung

Damit lässt sich das Verhalten des Original-Dimmers reverse-engineeren.

---

## Plattform erweitern

Der ESP32 hat noch reichlich freie GPIOs, I2C und Rechenleistung übrig, ist also ein naheliegender Hub für mehr als nur die Beleuchtung. Der bereits umgesetzte Wassertemperatursensor (DS18B20) in `esphome/aquarium-steuerung.yaml` (siehe die `one_wire:`- und `sensor:`-Blöcke) ist dafür eine funktionierende Vorlage — ein weiterer Sensor lässt sich größtenteils nach demselben Muster ergänzen. Ein paar Ideen dazu, grob nach Umsetzungsaufwand sortiert:

- **Lichtmessung (einfach, günstig):** Ein BH1750 I2C-Lux-Sensor (~2€, natives ESPHome-Component) liefert ein Echtzeit-Signal dazu, was tatsächlich im Becken ankommt — praktisch zum Gegenchecken der Dimmkurve oder um eine ausfallende LED-Röhre frühzeitig zu erkennen. Wichtig: Das misst Lux, nicht PAR/µmol — ein echter PAR-Quantensensor ist eine andere (und deutlich teurere) Geräteklasse, aber Lux reicht gut als relativer Indikator für Drift-Erkennung.

- **Futterautomat (moderat):** Ein Schrittmotor- oder servogesteuerter Futterautomat ist ein gängiges DIY-Muster, ESPHome hat native `stepper`/`servo`-Components — lässt sich sauber in den bestehenden zeitbasierten Automatisierungsansatz der Beleuchtung integrieren.

- **Wasserstand/Leck-Erkennung (moderat):** Einfache Schwimmerschalter oder kapazitive Sensoren, unkompliziert als `binary_sensor`.

- **CO2-Messung (schwierig, oft nicht lohnend):** Günstige NDIR-Sensoren messen CO2 in **Luft**, nicht gelöstes CO2 im Wasser — hier direkt nicht nutzbar. Echte Aquawasser-CO2-Sonden sind teures Labor-/Aquakultur-Equipment. Der übliche DIY-Workaround ist die Schätzung von gelöstem CO2 aus pH + KH (Karbonathärte) über die Standardformel — braucht aber eine hinreichend genaue, gut kalibrierte pH-Sonde. Günstige analoge pH-Module driften erfahrungsgemäß und brauchen häufige Nachkalibrierung — eher ein größeres Projekt als ein Quick-Add-on.

Keine der Ideen oben ist aktuell im Repo umgesetzt — hier nur als Ausgangspunkt für alle, die die Plattform erweitern wollen, mit dem Temperatursensor als Referenzmuster.

---

## Bekannte Einschränkungen

- PWM-Dimmung allein ändert das LED-Spektrum nicht
- Der sehr niedrige Helligkeitsbereich ist hardwareabhängig
- Home-Assistant-Scripts benötigen sorgfältige Synchronisation bei paralleler Ausführung

---

## Geplante Verbesserungen

- Verbesserte Farbmischung zwischen den Kanälen
- Fortgeschrittenere Dimmkurven
- Bessere Nachbildung des Original-Dimmer-Verhaltens
- Weitere Umgebungssimulationen (z.B. Gewitter)
- Hardware-Stromregelung statt reinem PWM
- Grafisches Schaltbild und Pinout-Tabelle (siehe `docs/pinout.md` — aktuell nur Text, Schaltplan/Foto fehlen noch)

---

## Repository-Struktur

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
README.de.md
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