# EZB Data Portal

Datenportal der Europäischen Zentralbank (Nachfolger der Statistical Data
Warehouse / SDW). Maßgeblich für Geldpolitik und Finanzstatistik des Euroraums —
Leitzinsen, Geldmengen, Wechselkurse, Referenzzinsen.

- **Betreiber:** Europäische Zentralbank (EZB)
- **Zugriffsart:** offene SDMX-REST-API, **keine Authentifizierung**
- **Lizenz:** frei nutzbar mit Quellenangabe

## Enthaltene Daten (relevante Indikatoren)

| Indikator | Dataflow | Beispiel-Key | Frequenz |
|-----------|----------|--------------|----------|
| EZB-Leitzins (Hauptrefi) | `FM` | `D.U2.EUR.4F.KR.MRR_FR.LEV` | bei Ratssitzung |
| EZB-Einlagesatz | `FM` | `D.U2.EUR.4F.KR.DFR.LEV` | bei Ratssitzung |
| €STR (Tagesgeld) | `EST` | `B.EU000A2X2A25.WT` | täglich |
| Geldmenge M3 (Euroraum) | `BSI` | `M.U2.Y.V.M30.X.I.U2.2300.Z01.A` | monatlich |
| Wechselkurs EUR/USD | `EXR` | `D.USD.EUR.SP00.A` | täglich |
| Bank-Kreditzinsen | `MIR` | (diverse) | monatlich |

> Geldmengen/Kredite sind Euroraum-Aggregate; für rein deutsche Schnitte ist die
> Bundesbank die passendere Quelle (siehe `../bundesbank-sdmx/`).

## Zugriff / API

- **Base URL:** `https://data-api.ecb.europa.eu/service/`
- **Muster:** `{base}/data/{flowRef}/{key}?format=csvdata`
- **Formate:** SDMX-ML 2.1, **SDMX-CSV** (`format=csvdata`), JSON (`format=jsondata`)
- **Beispiel (EUR/USD, monatlich):**
  ```
  https://data-api.ecb.europa.eu/service/data/EXR/M.USD.EUR.SP00.A?format=csvdata
  ```
- **Key-Syntax:** punktseparierte Dimensionswerte; `+` für ODER, leere Position = alle
- **Zeitfilter:** `?startPeriod=2015-01&endPeriod=2026-12`
- **Doku:** <https://data.ecb.europa.eu/help/api/data>

## Aktualisierung

- Zinsen bei EZB-Ratssitzungen, Marktdaten täglich, Aggregate monatlich.
- CSV enthält `KEY`, `TIME_PERIOD`, `OBS_VALUE` u. a.

## DuckDB-Anbindungsplan

Siehe [`ARCHITECTURE.md`](../../ARCHITECTURE.md). Konkret:

1. **Extract** `pipeline/extract/ezb.py`: je `(flowRef, key)` aus `config/ezb.yml`
   die SDMX-CSV ziehen → `data/raw/ezb/<flow>_<key>/<asof>.csv`.
2. **raw** `read_csv_auto` → `raw_ezb_<flow>`.
3. **stg** `sql/staging/stg_ezb.sql`: `TIME_PERIOD` zu `DATE` normalisieren
   (D/M/Q gemischt), auf Ziel-Schema mappen (`source='ecb'`).
4. **mart**: in `mart_indicators`.

**Aufwand:** niedrig. **Stolperstein:** gemischte Frequenzen (täglich vs. monatlich)
sauber auf `time_period`/`freq` abbilden; die richtigen Series-Keys finden
(Data-Explorer der EZB nutzen).

## Quellen

- <https://data.ecb.europa.eu/>
- <https://data.ecb.europa.eu/help/api/overview>
