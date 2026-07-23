# ZEW-Konjunkturerwartungen (ZEW-Finanzmarkttest)

Vorlaufender Stimmungsindikator aus der Befragung von bis zu ~350
Finanzmarktexpertinnen und -experten zur erwarteten Konjunktur der nächsten
sechs Monate. **Läuft dem ifo tendenziell ~1 Monat voraus.** Erhoben seit
Dezember 1991.

- **Betreiber:** ZEW — Leibniz-Zentrum für Europäische Wirtschaftsforschung, Mannheim
- **Zugriffsart:** **kein offenes API** — Veröffentlichung auf der Website,
  historische Reihen als XLSX/CSV (teils über Spiegel wie IWH)
- **Lizenz:** ZEW-Nutzungsbedingungen (Quellenangabe; Weiterverwendung prüfen)

## Enthaltene Daten (relevante Reihen)

| Reihe | Beschreibung | Frequenz |
|-------|--------------|----------|
| ZEW-Konjunkturerwartungen | Saldo (positiv minus negativ), 6-Monats-Ausblick | monatlich |
| Lageeinschätzung | Einschätzung der aktuellen Lage | monatlich |
| Inflations-/Zinserwartungen | weitere Salden aus dem Finanzmarkttest | monatlich |
| Euroraum-Erwartungen | ZEW-Konjunkturerwartungen für den Euroraum | monatlich |

## Zugriff / Download

- **Themenseite:** <https://www.zew.de/das-zew/aktuelles/zew-konjunkturerwartungen>
- **Formate:** XLSX/CSV (Download von der ZEW-Seite; genaue Datei-URL je
  Veröffentlichung prüfen)
- **Alternative Spiegel/lange Reihen:** IWH-Konjunkturdaten —
  <https://www.iwh-halle.de/en/research/data-and-analysis/macroeconomic-reports/macro-data-download>
- **Hinweis:** Kein stabiles, dokumentiertes URL-Muster wie bei ifo — der
  aktuelle Datei-Link muss i. d. R. von der Themenseite gescrapt werden.

## Aktualisierung

- Monatlich, Veröffentlichung meist zur Monatsmitte.
- Salden werden i. d. R. nicht revidiert (Umfragewerte), aber Ausgangswerte prüfen.

## DuckDB-Anbindungsplan

Siehe [`ARCHITECTURE.md`](../../ARCHITECTURE.md). Konkret:

1. **Extract** `pipeline/extract/zew.py`: Themenseite laden, den aktuellen
   XLSX/CSV-Link extrahieren (leichtes HTML-Scraping), Datei nach
   `data/raw/zew/konjunkturerwartungen/<asof>.xlsx` ablegen. Robuste
   Fehlerbehandlung, falls sich das Seitenlayout ändert.
2. **raw** `read_xlsx`/`read_csv_auto` → `raw_zew_konjunktur`.
3. **stg** `sql/staging/stg_zew.sql`: Wide→Long, `indicator_id`
   (`zew_erwartungen`, `zew_lage`), `source='zew'`, Perioden zu `DATE`.
4. **mart**: in `mart_indicators`.

**Aufwand:** mittel-hoch (kein API, kein stabiles URL-Muster → Scraping des
Datei-Links nötig, wartungsintensiver). **Stolperstein:** Link-Extraktion bricht
bei Website-Umbau; als Absicherung IWH-Spiegel als sekundäre Quelle vorsehen und
Datei-Layout versionieren.

## Quellen

- <https://www.zew.de/das-zew/aktuelles/zew-konjunkturerwartungen>
- <https://www.iwh-halle.de/en/research/data-and-analysis/macroeconomic-reports/macro-data-download>
