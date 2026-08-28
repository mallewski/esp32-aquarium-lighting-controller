# Experimentell / nicht aktiv genutzt

Dateien hier sind bewusst **nicht** über `!include_dir_named` /
`!include_dir_list` in `configuration.yaml` eingebunden. Sie bleiben als
Referenz erhalten, laufen aber nicht.

- `scripts/aq_weather_link_update.yaml`
- `automations/aq_wetterkopplung_automatisch.yaml`

Ursprüngliche Idee: Helligkeit/Wolken-Intensität automatisch an
`weather.*`-Zustände koppeln. Wurde vom Autor als nicht ausgereift und
nicht sinnvoll genug für den produktiven Einsatz eingestuft.

Falls das Konzept später wieder aufgegriffen wird: `input_boolean.aq_weather_link`
existiert weiterhin im Package (`homeassistant/packages/aquarium_package.yaml`)
als Schalter dafür.