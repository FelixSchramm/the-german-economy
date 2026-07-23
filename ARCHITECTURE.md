# Architektur: DuckDB-Warehouse für deutsche Wirtschaftsdaten

Gemeinsames Design für die **Anbindung** aller Datenquellen. Jede Quelle in
[`sources/`](./sources/) folgt diesem Muster; die Anbindungs-Issues verweisen auf
dieses Dokument, damit die Details nicht je Quelle wiederholt werden.

## Zielbild

Ein schlankes, dateibasiertes **DuckDB-Warehouse**, das die wichtigsten
Indikatoren zur deutschen Wirtschaft aus heterogenen Quellen (REST-APIs,
SDMX, XLSX-Downloads) einsammelt, harmonisiert und analysebereit bereitstellt.
Der Build läuft per GitHub Actions; die fertige DuckDB-Datei wird auf einen VPS
ausgeliefert und dort von einer API gelesen. Deployment/Serving sind in
[`docs/adr/0004-vps-deployment-und-github-actions-etl.md`](./docs/adr/0004-vps-deployment-und-github-actions-etl.md)
festgelegt — dieses Dokument beschreibt den **Datenteil** (Extract → Warehouse),
das ADR den **Betriebsteil**.

```
                Extract (Python)              Load/Transform (DuckDB + SQL)
  Quelle  ─────────────────────────►  data/raw/…  ─────────────────────────►  DuckDB
  (API/                                (unveraendert,                          ├─ raw_*      (1:1)
   Datei)                               asof-versioniert)                      ├─ stg_*      (typisiert, harmonisiert)
                                                                              └─ mart_*     (analysebereit, quellenuebergreifend)
                                                                                     │
                                     Deploy (GitHub Actions, siehe ADR-0004)         │
                                     ├─► parquet-Export ─► Cloudflare R2 (Archiv)  ◄─┤
                                     └─► warehouse.duckdb ─► rsync/SSH auf VPS ─► Symlink-Swap + API-Reload
```

## Schichten (Layer)

| Layer | Zweck | Beispiel | Technik |
|-------|-------|----------|---------|
| **raw (Landing)** | Rohantwort unveraendert ablegen, `asof`-versioniert | `data/raw/eurostat/prc_hicp_manr/2026-07-23.json` | Python-Extractor schreibt Datei |
| **raw_\*** (DuckDB) | 1:1-Einlesen der Rohdatei in DuckDB | `raw_eurostat_hicp` | `read_json_auto` / `read_csv_auto` / `read_xlsx` |
| **stg_\*** | typisiert, umbenannt, ein Satz pro Beobachtung, einheitliches Schema | `stg_eurostat_hicp` | SQL (`sqlfluff`-formatiert) |
| **mart_\*** | quellenuebergreifend, analysebereit | `mart_indicators`, `dim_indicator` | SQL |

### Einheitliches Ziel-Schema (`stg_*` → `mart_indicators`)

Alle Beobachtungen laufen in ein **langes** Format zusammen:

| Spalte | Typ | Beschreibung |
|--------|-----|--------------|
| `source` | TEXT | Quelle, z. B. `destatis`, `bundesbank`, `eurostat` |
| `indicator_id` | TEXT | stabiler Schluessel, z. B. `hicp_manr_de` |
| `indicator_name` | TEXT | Klarname |
| `geo` | TEXT | Gebiet (`DE`, Bundesland, `EA` …) |
| `freq` | TEXT | `M`, `Q`, `A` |
| `time_period` | DATE | normalisierter Periodenbeginn |
| `value` | DOUBLE | Wert |
| `unit` | TEXT | Einheit (z. B. `%`, `Index 2020=100`) |
| `adjustment` | TEXT | `nsa` / `sa` / `wda` (saison-/kalenderbereinigt) |
| `asof` | DATE | Abrufdatum (fuer Revisionen) |
| `source_series_key` | TEXT | Originalschluessel/Tabellen-ID der Quelle |

`dim_indicator` haelt die Metadaten (Beschreibung, Einheit, Quelle, Update-Rhythmus)
je `indicator_id`.

## Extract-Muster (Python)

- Ein Modul je Quelle unter `pipeline/extract/<source>.py`.
- Verantwortlich **nur** fuer: Abruf + unveraenderte Ablage unter `data/raw/…`
  inkl. `asof`-Datum. Keine Transformation (Trennung von E und T/L).
- Gemeinsame Helfer (`http_get` mit Retry/Backoff, Ratenlimit, `asof`-Pfad) in
  `pipeline/extract/_common.py`.
- Konfiguration je Quelle deklarativ (welche Tabellen/Reihen), z. B. YAML unter
  `pipeline/config/<source>.yml`, damit neue Reihen ohne Codeaenderung dazukommen.

## Load/Transform (DuckDB + SQL)

- Ein Lauf `pipeline/build.py` oeffnet `warehouse.duckdb`, liest die Rohdateien
  (`read_*_auto`) in `raw_*`, fuehrt dann die SQL-Modelle in Reihenfolge aus:
  `sql/staging/*.sql` → `sql/marts/*.sql`.
- SQL-Dateien sind idempotent (`CREATE OR REPLACE TABLE …`).
- Am Ende steht die fertige `warehouse.duckdb`. Zusätzlich werden die Marts nach
  `data/marts/*.parquet` exportiert — diese Parquet-Snapshots dienen als
  **portables Archiv** und werden nach Cloudflare R2 geladen (siehe ADR-0004),
  nicht ins Repo committet. Die DuckDB-Datei ist das **ausgelieferte Artefakt**
  (rsync auf den VPS), gelesen von der FastAPI-API.

## Revisionen & Idempotenz

- Amtliche Daten werden nachtraeglich revidiert. Deshalb `asof` mitfuehren; die
  Marts zeigen standardmaessig den **letzten** Stand je `(indicator_id, time_period)`,
  die Roh-/Staging-Historie bleibt fuer Vintages erhalten.
- Re-Runs sind gefahrlos: gleiche `asof` ueberschreibt die Rohdatei, `CREATE OR
  REPLACE` baut die Modelle neu.

## Orchestrierung & Deployment (GitHub Actions, ADR-0004)

Verbindliche Referenz: [ADR-0004](./docs/adr/0004-vps-deployment-und-github-actions-etl.md).
Kein eigener Scheduler — GitHub Actions ist Build **und** Deploy.

- Ein Workflow je Update-Rhythmus (monatlich fuer die meisten Reihen;
  quartals-/jaehrlich fuer VGR/Fiskaldaten).
- Ablauf je Lauf:
  1. `uv sync`
  2. `python pipeline/run.py --source <name>` → baut `warehouse.duckdb`
  3. **Qualitaetspruefungen** (vgl. ADR-0002) — Lauf bricht bei Verletzung ab
  4. Parquet-Export → **Cloudflare R2** (Archiv)
  5. `rsync` der `warehouse.duckdb` per SSH auf den **Hetzner-VPS**
  6. auf dem VPS: **Symlink-Wechsel** (atomarer Austausch) + **API-Reload**
- **Serving:** FastAPI liest die DuckDB-Datei lokal vom VPS-Volume; Caddy
  (TLS) + Cloudflare (CDN/Cache) davor; Frontend (Next.js) auf Cloudflare Pages.
- **Secrets** als GitHub Actions Secrets: Quell-Zugaenge (`DESTATIS_*`, `BA_*`),
  `SSH_DEPLOY_KEY` (auf Zielverzeichnis beschraenkt, rotiert), R2-Credentials.

## Monitoring (verbindlich, ADR-0004)

- `/health` der API liefert `data_version`, letzten erfolgreichen Lauf **je
  Quelle** und den DB-Verbindungsstatus.
- Externer Uptime-Monitor prueft `/health` und alarmiert, wenn eine Quelle ihr
  erwartetes Aktualisierungsintervall um > 100 % ueberschreitet
  (**Freshness-Alarm** gegen still fehlschlagende Importer).
- Deshalb fuehrt jede `stg_*`/`mart_*` den `asof`/letzten Stand je Quelle mit,
  damit `/health` die Frische je Quelle berechnen kann.

## Konventionen (aus `CLAUDE.md`)

- **Python**: PEP 8, `black`, `snake_case`, reST-Docstrings (`:param:` / `:return:`),
  je Funktion ein Docstring.
- **SQL**: mit `sqlfluff` formatiert; kompatibel zu Standard-SQL/DuckDB.
- **Doku**: je Quelle eine Markdown-Datei (die `sources/<quelle>/README.md`).
- **Commits/Branches**: Conventional Commits, Feature-Branches.

## Verzeichnis-Layout (Zielzustand)

```
pipeline/
├── config/            # <source>.yml — welche Reihen/Tabellen
├── extract/           # <source>.py + _common.py
├── build.py           # Roh → DuckDB laden, SQL-Modelle ausführen
└── run.py             # CLI-Einstieg (--source ...)
sql/
├── staging/           # stg_*.sql
└── marts/             # mart_*.sql, dim_*.sql
data/
├── raw/               # Landing (asof-versioniert) — gitignored
└── marts/             # exportierte parquet-Marts → Cloudflare R2 (gitignored)
warehouse.duckdb       # gebautes Warehouse, Deploy-Artefakt → VPS (gitignored)
deploy/                # Docker Compose, Caddyfile, FastAPI-App (siehe ADR-0004)
docs/adr/              # Architektur-Entscheidungen (ADR-0004: Deployment)
sources/               # Doku je Quelle (dieses Repo)
```

> `data/raw/`, `data/marts/` und `warehouse.duckdb` liegen in `.gitignore` — die
> Auslieferung erfolgt ueber R2 bzw. rsync (ADR-0004), nicht ueber das Repo.

## Priorisierung der Anbindung (Kern-Set)

Reihenfolge nach Aufwand/Nutzen — API-Quellen zuerst (geringster Aufwand,
hoechste Aktualitaet), Datei-Quellen danach:

1. **Eurostat**, **EZB Data Portal**, **Bundesbank SDMX** — offene REST/SDMX-APIs,
   kein Login, sehr guter Startpunkt.
2. **Destatis GENESIS** — REST/JSON, benoetigt (kostenlosen) Account/Token.
3. **Bundesagentur fuer Arbeit** — neue Statistik-API (seit 12/2025).
4. **ifo**, **ZEW** — XLSX-Download (leichtes Scraping, stabile URL-Muster).

---

*Dieses Dokument ist die verbindliche Referenz fuer die Anbindungs-Issues. Bei
Aenderungen am Zielbild hier zuerst aktualisieren.*
