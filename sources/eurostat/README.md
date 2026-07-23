# Eurostat

Statistikamt der EU. Harmonisierte, EU-weit vergleichbare Reihen — für
Deutschland u. a. die für die EZB maßgebliche HVPI-Inflation, BIP im EU-Vergleich,
ILO-Arbeitslosigkeit und Maastricht-Fiskaldaten.

- **Betreiber:** Europäische Kommission (Eurostat)
- **Zugriffsart:** offene REST-API, **kein API-Key nötig**
- **Lizenz:** Eurostat-Copyright-Notice (freie Weiterverwendung mit Quellenangabe)

## Enthaltene Daten (relevante Indikatoren)

| Indikator | Dataset-Code | Frequenz |
|-----------|--------------|----------|
| HVPI, Jahresrate | `prc_hicp_manr` | monatlich |
| HVPI, Index | `prc_hicp_midx` | monatlich |
| BIP (VGR, Volumen) | `namq_10_gdp` | quartalsweise |
| Arbeitslosenquote (ILO) | `une_rt_m` | monatlich |
| Erwerbstätigkeit | `namq_10_a10_e` | quartalsweise |
| Industrieproduktion | `sts_inpr_m` | monatlich |
| Staatsdefizit/-schulden | `gov_10dd_edpt1` | jährlich |
| Außenhandel | `ext_st_ea20sitc` | monatlich |

Filter auf Deutschland über `geo=DE`.

## Zugriff / API

- **Base URL:** `https://ec.europa.eu/eurostat/api/dissemination/statistics/1.0/data/`
- **Muster:** `{base}/{datasetCode}?format=JSON&lang=EN&geo=DE&{weitere Filter}`
- **Formate:** JSON-stat 2.0 (empfohlen), SDMX-CSV, SDMX-ML
- **Beispiel (HVPI Jahresrate, DE):**
  ```
  https://ec.europa.eu/eurostat/api/dissemination/statistics/1.0/data/prc_hicp_manr?format=JSON&geo=DE&coicop=CP00
  ```
- **Auth:** keine
- **Doku:** <https://ec.europa.eu/eurostat/web/user-guides/data-browser/api-data-access/api-getting-started/api>

## Aktualisierung

- Frequenz je Dataset (monatlich/quartalsweise/jährlich).
- Euroraum-Flash zum HVPI sehr früh; nationale Werte folgen.
- Revisionen üblich → `asof` mitführen.

## DuckDB-Anbindungsplan

Siehe [`ARCHITECTURE.md`](../../ARCHITECTURE.md). Konkret:

1. **Extract** `pipeline/extract/eurostat.py`: je Dataset-Code aus
   `config/eurostat.yml` die JSON-stat-Antwort ziehen und unter
   `data/raw/eurostat/<code>/<asof>.json` ablegen.
2. **raw** `read_json_auto` → `raw_eurostat_<code>` (JSON-stat entpacken:
   Dimensionen `geo`, `time`, ggf. `coicop`/`unit`).
3. **stg** `sql/staging/stg_eurostat.sql`: JSON-stat-Würfel zu langem Format
   auffalten, auf Ziel-Schema mappen (`source='eurostat'`, `indicator_id` je
   Dataset+Filter).
4. **mart**: fließt in `mart_indicators` ein.

**Aufwand:** niedrig (offene API, gut dokumentiert). **Stolperstein:** JSON-stat
entpacken (Dimensions-Indexierung) — Helfer schreiben oder `pyjstat` erwägen.

## Quellen

- <https://ec.europa.eu/eurostat/web/main/data/web-services>
- <https://ec.europa.eu/eurostat/data/database>
