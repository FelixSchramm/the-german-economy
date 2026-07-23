# Deutsche Bundesbank — SDMX-Webservice

Zentrale Quelle für **deutschlandspezifische** monetäre und Finanzstatistiken:
Zinsen, Geldmengen, Bankkredite, Zahlungsbilanz/Leistungsbilanz, Wertpapiere,
Wechselkurse.

- **Betreiber:** Deutsche Bundesbank
- **Zugriffsart:** offene SDMX-REST-API, **keine Authentifizierung**
- **Lizenz:** frei nutzbar mit Quellenangabe

> **Hinweis:** Die alte Zeitreihen-Datenbank wurde zum **30.06.2026**
> abgeschaltet; die Reihen liegen im neuen Statistikportal. Der **SDMX-Webservice
> bleibt** der maschinenlesbare Zugang.

## Enthaltene Daten (relevante Reihen)

| Bereich | Dataflow | Beispiel-Key | Frequenz |
|---------|----------|--------------|----------|
| Wechselkurse | `BBEX3` | `D.USD.EUR.BB.AC.000` | täglich |
| Zinsen/Renditen (u. a. Umlaufsrendite) | `BBSIS` | (diverse) | täglich/monatlich |
| Bankkredite an Unternehmen/Haushalte | `BBK01` / MFI-Statistik | (diverse) | monatlich |
| Zahlungs-/Leistungsbilanz | `BBDB2` | (diverse) | monatlich |
| Geldmengenaggregate (dt. Beitrag) | MFI-Bilanzstatistik | (diverse) | monatlich |

Konkrete Keys über das Statistikportal / die Metadaten-Endpunkte ermitteln.

## Zugriff / API

- **Base URL:** `https://api.statistiken.bundesbank.de/rest/`
- **Muster:** `{base}/data/{flowRef}/{key}?format=csv`
- **Formate:** SDMX-ML 2.1, **SDMX-CSV** (`format=csv`)
- **Beispiel (EUR/USD-Referenzkurs, täglich):**
  ```
  https://api.statistiken.bundesbank.de/rest/data/BBEX3/D.USD.EUR.BB.AC.000?format=csv
  ```
- **Wildcards:** leere Key-Position = alle (z. B. `D..EUR.BB.AC.000`); `+` = ODER
- **Metadaten:** `/metadata/...` für Strukturen/Codelisten
- **Doku:** <https://www.bundesbank.de/de/statistiken/zeitreihen-datenbanken/hilfe-zu-sdmx-webservice>

## Aktualisierung

- Marktdaten täglich, Aggregate/Bilanzstatistik monatlich.
- Revisionen bei Statistikreihen möglich → `asof` mitführen.

## DuckDB-Anbindungsplan

Siehe [`ARCHITECTURE.md`](../../ARCHITECTURE.md). Konkret:

1. **Extract** `pipeline/extract/bundesbank.py`: je `(flowRef, key)` aus
   `config/bundesbank.yml` SDMX-CSV ziehen → `data/raw/bundesbank/<flow>_<key>/<asof>.csv`.
2. **raw** `read_csv_auto` → `raw_bundesbank_<flow>`.
3. **stg** `sql/staging/stg_bundesbank.sql`: `TIME_PERIOD` normalisieren, Ziel-Schema
   (`source='bundesbank'`).
4. **mart**: in `mart_indicators`.

**Aufwand:** niedrig (API wie EZB, gleiches SDMX-Muster → gemeinsamer Helfer mit
`../ezb-data-portal/` möglich). **Stolperstein:** passende Keys finden; CSV-Spalten
je Dataflow leicht unterschiedlich benannt.

## Quellen

- <https://www.bundesbank.de/de/statistiken>
- <https://statistiken.bundesbank.de/content/991208>
