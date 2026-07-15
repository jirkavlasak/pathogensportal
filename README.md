# Pathogen Portal CZ

Český portál pro sledování dat o patogenech (COVID-19, chřipka, ...). Statický web
postavený na [Hugo](https://gohugo.io/), s doplňkovým pipeline stackem pro sběr dat
(PostgreSQL + Grafana + Python scrapery).

Produkčně běží na **https://pathogens.vm.cesnet.cz/** (Apache, Let's Encrypt certifikát).

## Co tu je

```
site/                    Hugo web (téma jako git submodule v site/themes/hugo-pathogens-portal)
site/content/            obsah stránek (dashboardy, news, events, ...)
site/layouts/            site-level přepisy šablon z tématu (viz níže "Přepisy šablon tématu")
site/static/             statické soubory, vč. site/static/data/charts/*.json pro grafy
site/static/vendor/      lokálně hostované Bootstrap, jQuery, DataTables, Chart.js atd.
                         (viz "Proč jsou knihovny lokálně, ne z CDN")
db/init.sql              schéma PostgreSQL databáze (tabulky pathogens, dashboard_data)
grafana/                 provisioning + předpřipravené dashboardy pro Grafanu
scripts/                 Python pipeline — stahování a zpracování dat
scripts/scrapers/        jednotlivé scrapery (MZČR, ECDC, SZÚ, ÚZIS ISIN)
scripts/requirements.txt Python závislosti (requests, pandas, openpyxl, pdfplumber)
nextstrain/               profil pro fylogenetické analýzy (Nextstrain)
data/                     stažená/zpracovaná data — NENÍ v gitu (viz .gitignore)
docker-compose.yml        lokální dev prostředí (hugo, postgres, grafana, pipeline, nextstrain)
```

## Přepisy šablon tématu

Téma `hugo-pathogens-portal` je git submodule (cizí repo), takže se v něm needitujeme
napřímo. Hugo umí přepsat libovolnou šablonu tématu stejnojmenným souborem ve vlastním
`site/layouts/`. Používáme to pro:

- `site/layouts/partials/head.html`, `footer.html` — natažení lokálních (ne CDN)
  knihoven, viz níže
- `site/layouts/partials/navbar.html` — oprava odkazu na ELIXIR logo (vede na
  `elixir-europe.org`, ne zpátky na hlavní stránku)
- `site/layouts/dashboards/single.html` — lokální Chart.js místo CDN

Při update tématu (submodulu) zkontrolovat, jestli tyhle přepisy pořád dávají smysl.

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

## Proč jsou knihovny lokálně, ne z CDN

Šablona tématu původně natahovala Bootstrap, Bootstrap Icons, DataTables, jQuery a
Chart.js z externích CDN (`cdn.jsdelivr.net`, `cdn.datatables.net`, `code.jquery.com`).
Na sítích, které tyhle domény blokují nebo filtrují (běžné na akademických/firemních
sítích), se CSS/JS vůbec nenačetlo a stránka se zobrazila bez stylů. Řešení: všechny
tyhle knihovny jsou stažené a hostované lokálně v `site/static/vendor/`, šablony na ně
odkazují **relativní** cestou (`/vendor/...`), takže se vždy natáhnou ze stejné domény
a stejného protokolu (http/https) jako zbytek stránky.

**Důležité:** cokoliv v `site/layouts/` (přepisy šablon) musí odkazovat na vlastní
zdroje relativní cestou nebo přes Hugo `.RelPermalink`, nikdy přes `.Permalink`
(ten generuje absolutní URL podle `baseURL`, tedy natvrdo `https://`) — jinak se
při návštěvě přes `http://` prohlížeč pokusí o CORS spojení jinam a při
nedůvěryhodném certifikátu to spadne (přesně tohle se stalo s `theme.min...css`).

## Jaká máme data

Zdroje (stahují se scriptem `scripts/run_all.py` do `data/`, viz `scripts/scrapers/`):

- **MZČR** (`data/mzcr/`) — COVID-19 data Ministerstva zdravotnictví ČR
- **ECDC** (`data/ecdc/`) — evropská COVID-19 data pro ČR
- **SZÚ** (`data/szu/`) — historická data chřipky od Státního zdravotního ústavu
- **ÚZIS ISIN** (`data/isin/`) — data o infekčních nemocech (PDF/CSV, parsuje se přes
  `pdfplumber`)

Zpracovaná data (CSV, SQLite `covid.db`, PDF podklady, vektorové úložiště pro
vyhledávání) se ukládají do `data/` — tato složka se necommituje (obsahuje velké
a často se měnící soubory), viz `.gitignore`.

## Jak to aktualizovat

Mimo Docker (přímo na produkčním serveru) používá pipeline virtuální prostředí
`scripts/.venv` (`requirements.txt` má napevno přišpendlené verze, systémový `pip`
na Ubuntu 24.04 je navíc zamčený přes PEP 668). Skript `generate_json.py` navíc
potřebuje systémový `sqlite3` CLI (`apt install sqlite3`).

1. **Stáhnout čerstvá data:**
   ```bash
   cd scripts && .venv/bin/python run_all.py
   ```
2. **Vygenerovat JSON podklady pro grafy** (čte `data/mzcr`, `data/ecdc`, `data/covid.db`,
   píše do `site/static/data/charts/*.json`, které Hugo šablony vykreslují přes Chart.js):
   ```bash
   .venv/bin/python generate_json.py
   ```
3. **Přebuildovat statický web:**
   ```bash
   cd site && hugo --minify
   ```
4. **Nasadit na produkci** — na `pathogens.vm.cesnet.cz` Apache servíruje přímo
   `site/public/` na obou vhostech (`/etc/apache2/sites-available/000-default.conf`
   pro `:80`, `pathogens-ssl.conf` pro `:443`), takže po přebuildu stačí mít nový
   obsah v `site/public/` a není potřeba nic restartovat.

## HTTPS / certifikát

Produkce běží na certifikátu od Let's Encrypt (**RSA klíč**, ne ECDSA — RSA se
řetězí přes univerzálně důvěryhodný `ISRG Root X1`, zatímco výchozí ECDSA větev jde
přes novější `ISRG Root X2`, který nemusí být v trust store starších/firemně
spravovaných systémů). Certifikát je v `/etc/letsencrypt/live/pathogens-rsa/`,
`certbot` má nastavenou automatickou obnovu přes systemd timer (`certbot.timer`).
