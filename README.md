# Pathogen Portal CZ

Český portál pro sledování dat o patogenech (COVID-19, chřipka, ...). Statický web
postavený na [Hugo](https://gohugo.io/), s doplňkovým pipeline stackem pro sběr dat
(PostgreSQL + Grafana + Python scrapery).

Produkčně běží na **http://pathogens.vm.cesnet.cz/**.

## Co tu je

```
site/               Hugo web (téma jako git submodule v site/themes/hugo-pathogens-portal)
site/content/       obsah stránek (dashboardy, news, events, ...)
site/static/        statické soubory, vč. site/static/data/charts/*.json pro grafy
db/init.sql         schéma PostgreSQL databáze (tabulky pathogens, dashboard_data)
grafana/            provisioning + předpřipravené dashboardy pro Grafanu
scripts/            Python pipeline — stahování a zpracování dat
scripts/scrapers/   jednotlivé scrapery (MZČR, ECDC, SZÚ)
nextstrain/         profil pro fylogenetické analýzy (Nextstrain)
data/               stažená/zpracovaná data — NENÍ v gitu (viz .gitignore)
docker-compose.yml  lokální dev prostředí (hugo, postgres, grafana, pipeline, nextstrain)
```

## Jak to spustit lokálně

Vyžaduje Docker + Docker Compose.

```bash
docker compose up
```

- Hugo dev server: http://localhost:1313/ (live reload)
- PostgreSQL: localhost:5432 (`portal` / `portal_dev`, DB `pathogens`)
- Grafana: http://localhost:3000/ (`admin` / `portal_dev`)

Datová pipeline a Nextstrain běží jen na vyžádání (profily):

```bash
docker compose --profile pipeline run --rm pipeline
docker compose --profile nextstrain run --rm nextstrain
```

## Jaká máme data

Zdroje (stahují se scriptem `scripts/run_all.py` do `data/`, viz `scripts/scrapers/`):

- **MZČR** (`data/mzcr/`) — COVID-19 data Ministerstva zdravotnictví ČR
- **ECDC** (`data/ecdc/`) — evropská COVID-19 data pro ČR
- **SZÚ** (`data/szu/`) — historická data chřipky od Státního zdravotního ústavu

Zpracovaná data (CSV, SQLite `covid.db`, PDF podklady, vektorové úložiště pro
vyhledávání) se ukládají do `data/` — tato složka se necommituje (obsahuje velké
a často se měnící soubory), viz `.gitignore`.

## Jak to aktualizovat

1. **Stáhnout čerstvá data:**
   ```bash
   python scripts/run_all.py
   ```
2. **Vygenerovat JSON podklady pro grafy** (čte `data/mzcr`, `data/ecdc`, píše do
   `site/static/data/charts/*.json`, které Hugo šablony vykreslují přes Chart.js):
   ```bash
   python scripts/generate_json.py
   ```
3. **Přebuildovat statický web:**
   ```bash
   cd site && hugo --minify
   ```
4. **Nasadit na produkci** — na `pathogens.vm.cesnet.cz` Apache servíruje přímo
   `site/public/` (`DocumentRoot` v `/etc/apache2/sites-available/000-default.conf`),
   takže po přebuildu stačí mít nový obsah v `site/public/` a není potřeba nic
   restartovat.

Poznámka: produkční `baseURL` v `site/hugo.toml` je zatím `http://` — na serveru
ještě není nastavené HTTPS.
