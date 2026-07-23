# Destatis — GENESIS-Online (Webservice/API)

Die tief gegliederte Datenbank der amtlichen Statistik. **Wichtigste harte
Quelle** für die deutsche Konjunktur: BIP/VGR, Verbraucherpreise, Produktion,
Auftragseingänge, Einzelhandel, Außenhandel, Arbeitsmarkt (Erwerbstätigkeit).

- **Betreiber:** Statistisches Bundesamt (Destatis)
- **Zugriffsart:** REST/JSON-Webservice; **kostenlose Registrierung** nötig
  (Login oder API-Token)
- **Lizenz:** i. d. R. Datenlizenz Deutschland 2.0 (Namensnennung)

## Enthaltene Daten (relevante Tabellen)

| Indikator | Statistik/Tabellen-Bereich | Frequenz |
|-----------|----------------------------|----------|
| BIP / VGR | Bereich `81000` (z. B. `81000-0001`) | quartals-/jährlich |
| Verbraucherpreisindex (VPI) | `61111` (z. B. `61111-0001`) | monatlich |
| Erzeugerpreise | `61241` | monatlich |
| Produktion produzierendes Gewerbe | `42153` | monatlich |
| Auftragseingang Industrie | `42151` | monatlich |
| Einzelhandelsumsatz | `45212` | monatlich |
| Außenhandel (Aus-/Einfuhr n. Waren) | `51000` | monatlich |
| Erwerbstätigkeit | `13321` | monatlich |

> Tabellen-IDs über `find/find` bzw. den GENESIS-Katalog verifizieren
> (gelegentliche Umnummerierungen).

## Zugriff / API

- **Base URL:** `https://www-genesis.destatis.de/genesisWS/rest/2020/`
- **Auth:** Zugangsdaten (`username`/`password`) bzw. Token im HTTP-Header;
  weitere Parameter im Request-Body (`application/x-www-form-urlencoded`)
- **Wichtige Endpunkte:**
  - `helloworld/logincheck` — Zugang testen
  - `find/find` — Objekte (Tabellen, Statistiken, Reihen) suchen
  - `data/table` — Tabelle abrufen (`name=<Tabellen-ID>`, `area`, `format`)
  - `data/timeseries` — Zeitreihe abrufen
- **Formate:** JSON, CSV (ffcsv)
- **Beispiel (schematisch, VPI-Tabelle):**
  ```
  POST https://www-genesis.destatis.de/genesisWS/rest/2020/data/table
  body: username=…&password=…&name=61111-0001&area=all&format=ffcsv
  ```
- **Doku:** <https://www.destatis.de/DE/Service/OpenData/genesis-api-webservice-oberflaeche.html>
- **OpenAPI (Community):** <https://destatis.api.bund.dev/>
- **Ohne Login (Alternative):** vorgefertigte Konjunktur-CSV unter
  <https://www.destatis.de/DE/Service/OpenData/konjunkturindikatoren-csv.html>

## Aktualisierung

- Frequenz je Tabelle (BIP quartalsweise, Preise/Produktion monatlich).
- Verzug: BIP-Schnellmeldung ~30 Tage; Produktion/Außenhandel ~40 Tage.
- Revisionen üblich (v. a. VGR) → `asof` mitführen.
- Ratenlimits/Fair-Use beachten (großzügig paginieren, nicht parallel hämmern).

## DuckDB-Anbindungsplan

Siehe [`ARCHITECTURE.md`](../../ARCHITECTURE.md). Konkret:

1. **Secrets:** Destatis-Zugangsdaten als GitHub Actions Secret
   (`DESTATIS_USER`, `DESTATIS_PASS`) bzw. Token.
2. **Extract** `pipeline/extract/destatis.py`: je Tabellen-ID aus
   `config/destatis.yml` `data/table` (ffcsv) ziehen →
   `data/raw/destatis/<tabelle>/<asof>.csv`. Retry/Backoff + Ratenlimit über
   `_common.py`.
3. **raw** `read_csv_auto` → `raw_destatis_<tabelle>` (ffcsv ist bereits „flach").
4. **stg** `sql/staging/stg_destatis.sql`: Merkmale/Zeit in Ziel-Schema mappen
   (`source='destatis'`), Perioden (`2026-Q1`, `2026-06`) zu `DATE`.
5. **mart**: in `mart_indicators`.

**Aufwand:** mittel (Auth + Body-Format + ffcsv-Parsing). **Stolperstein:** Login
im Header vs. Body; ffcsv-Struktur je Tabelle unterschiedlich; Tabellen-IDs
gelegentlich geändert → `find/find` zur Verifikation nutzen.

## Quellen

- <https://www-genesis.destatis.de/datenbank/online>
- <https://www.destatis.de/DE/Service/OpenData/_inhalt.html>
