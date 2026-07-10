---
title: "Monitoring Dashboard (Grafana)"
description: "Operační monitoring databáze patogenů — stav dat, pipeline statistiky a přehled záznamů v PostgreSQL."
image: "/images/dashboard-placeholder.svg"
highlight: false
tags: ["monitoring", "databáze", "pipeline", "Grafana"]
data_source: "PostgreSQL — lokální databáze patogenů"
update_freq: "Živě (real-time)"
redirect_url: "/grafana/"
---

**Grafana** je vizualizační nástroj napojený přímo na PostgreSQL databázi portálu.

### Co dashboard zobrazuje

- Počet patogenů v databázi
- Počet datových záznamů (`dashboard_data`)
- Stav pipeline (poslední úspěšný běh)
- Přehled dat per dashboard

### Spuštění

Grafana běží jako součást Docker Compose stacku:

```bash
docker compose up grafana
```

Portál ji zpřístupňuje přes reverzní proxy Apache na **/grafana/**.

<a href="/grafana/" target="_blank" class="btn btn-primary mt-2 mb-3">
  Otevřít Grafana (/grafana/)
</a>

> **Poznámka:** Přihlašovací údaje (`admin` / `portal_dev`) jsou dočasné vývojové heslo — před ostrým provozem je nutné je změnit.

<p class="stat-source">Grafana OSS · Apache 2.0 · napojeno na PostgreSQL 16</p>
