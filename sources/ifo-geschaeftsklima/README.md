# ifo Geschäftsklimaindex

Wichtigster deutscher **Frühindikator**. Monatliche Befragung von ~9.000
Unternehmen zur aktuellen Geschäftslage und den Erwartungen für die nächsten
sechs Monate. Bildet die BIP-Entwicklung eng ab.

- **Betreiber:** ifo Institut, München
- **Zugriffsart:** **kein API** — XLSX-Zeitreihen zum Download (stabiles URL-Muster)
- **Lizenz:** ifo-Nutzungsbedingungen (Quellenangabe; Weiterverwendung prüfen)

## Enthaltene Daten (relevante Reihen)

| Reihe | Beschreibung | Frequenz |
|-------|--------------|----------|
| Geschäftsklima | Hauptindex (Saldo/Index) | monatlich |
| Geschäftslage | Teilindex aktuelle Lage | monatlich |
| Geschäftserwartungen | Teilindex Erwartungen | monatlich |
| Branchen | Verarbeitendes Gewerbe, Handel, Bau, Dienstleistungen | monatlich |
| ifo Exportklima / Exporterwartungen | separate XLSX | monatlich |

## Zugriff / Download

- **Themenseite:** <https://www.ifo.de/umfragen/ifo-geschaeftsklima-deutschland>
- **XLSX-Muster (Deutschland):**
  `https://www.ifo.de/sites/default/files/secure/timeseries/gsk-d-<YYYYMM>.xlsx`
  (Beispiel: `.../gsk-d-202503.xlsx`)
- **Exportklima-Muster:**
  `https://www.ifo.de/sites/default/files/secure/timeseries/imklima-d-<YYYYMM>.xlsx`
- **Historische lange Reihen:** EconStor —
  <https://www.econstor.eu/handle/10419/165792>
- **Formate:** XLSX

## Aktualisierung

- Monatlich, Veröffentlichung meist gegen Monatsende (des Berichtsmonats).
- Rückwirkende Anpassungen (Saisonbereinigung) möglich → `asof` mitführen.

## DuckDB-Anbindungsplan

Siehe [`ARCHITECTURE.md`](../../ARCHITECTURE.md). Konkret:

1. **Extract** `pipeline/extract/ifo.py`: aktuelle XLSX über das Datums-Muster
   (`gsk-d-<YYYYMM>.xlsx`) laden; aktuellen Monat bestimmen, bei 404 auf Vormonat
   zurückfallen. Ablage `data/raw/ifo/geschaeftsklima/<asof>.xlsx`.
2. **raw** DuckDB `read_xlsx` (Extension `spatial`/`excel`) bzw. in Python mit
   `pandas.read_excel` einlesen → `raw_ifo_geschaeftsklima`.
3. **stg** `sql/staging/stg_ifo.sql`: Wide→Long (Monatsspalten/-zeilen auffalten),
   Teilindizes als `indicator_id` (`ifo_klima`, `ifo_lage`, `ifo_erwartung`),
   `source='ifo'`.
4. **mart**: in `mart_indicators`.

**Aufwand:** mittel (kein API, XLSX-Layout kann sich ändern). **Stolperstein:**
XLSX-Struktur (Mehrfach-Header, mehrere Blätter je Branche) robust parsen;
URL-Datumslogik + 404-Fallback; ggf. HTML der Themenseite nach dem aktuellen
Datei-Link scrapen statt Datum zu raten.

## Quellen

- <https://www.ifo.de/umfragen>
- <https://www.econstor.eu/handle/10419/165792>
