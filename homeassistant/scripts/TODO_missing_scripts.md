# Fehlende Scripts

Diese zwei Scripts werden von Automationen in diesem Repo referenziert,
ihr tatsächlicher Inhalt wurde im Rahmen dieser Konversation aber nie
geteilt. Ich wollte hier keinen Code erfinden, der eventuell nicht zu
deiner echten Logik passt — bitte selbst ergänzen:

- `aq_clouds_dynamic.yaml`
  Referenziert von: `automations/aq_start_sunset.yaml`,
  `automations/aq_wolken_stoppen_bei_sonnenuntergang.yaml`
  Laut README: randomisierte Helligkeits-Einbrüche ("Wolken"),
  Dauer/Intensität über `input_number.aq_cloud_duration` /
  `input_number.aq_cloud_intensity` konfigurierbar, relativ zur
  aktuellen Helligkeit.

- `aq_weather_link_update.yaml`
  Referenziert von: `automations/aq_wetterkopplung_automatisch.yaml`
  Laut README: mappt `weather.*`-Zustände auf Helligkeit und
  Wolken-Intensität.

Zusätzlich existiert laut deiner Zustände-Ansicht eine Entity
`script.aq_siesta`, deren Inhalt ebenfalls nicht geteilt wurde — aktuell
liegt die Siesta-Logik stattdessen direkt inline in
`automations/aq_start_siesta.yaml`. Ggf. bei Gelegenheit vereinheitlichen.
