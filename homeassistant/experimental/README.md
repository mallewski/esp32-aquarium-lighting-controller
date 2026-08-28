# Experimental / not actively used

Files here are intentionally **not** wired into `configuration.yaml` via
`!include_dir_named` / `!include_dir_list`. They're kept as reference,
but don't run.

- `scripts/aq_weather_link_update.yaml`
- `automations/aq_wetterkopplung_automatisch.yaml`

Original idea: automatically couple brightness/cloud intensity to
`weather.*` states. Deemed not mature enough and not useful enough for
production use by the author.

If this concept gets revisited later: `input_boolean.aq_weather_link`
still exists in the package
(`homeassistant/packages/aquarium_package.yaml`) as the toggle for it.
