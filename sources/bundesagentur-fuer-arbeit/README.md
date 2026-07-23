# Bundesagentur für Arbeit — Statistik

Die aktuellsten Arbeitsmarktdaten Deutschlands (nationale Definition):
Arbeitslosigkeit/-quote, Unterbeschäftigung, gemeldete Stellen (BA-X),
sozialversicherungspflichtige Beschäftigung, Kurzarbeit.

- **Betreiber:** Statistik der Bundesagentur für Arbeit (BA)
- **Zugriffsart:** **seit Dezember 2025 offizielle API** für automatisierte
  Datenabfragen (ergänzend zu XLSX-Downloads)
- **Lizenz:** Nutzungsbedingungen der BA-Statistik (Namensnennung)

## Enthaltene Daten (relevante Reihen)

| Indikator | API-Produkt | Frequenz |
|-----------|-------------|----------|
| Eckwerte Arbeitslosigkeit | „Eckwerte Arbeitslosigkeit" (ALO) | monatlich |
| Arbeitslosigkeit Zeitreihen | ALO-Zeitreihe | monatlich |
| Eckwerte Beschäftigung | „Eckwerte Beschäftigung" (BST) | monatlich (Verzug ~2 M) |
| Beschäftigung Zeitreihen | BST-Zeitreihe | monatlich |
| Gemeldete Arbeitsstellen (BA-X) | STEA-Zeitreihe | monatlich |
| Ausbildungsmarkt | BB-Zeitreihe | monatlich/jährlich |

## Zugriff / API

- **API-Übersicht:** <https://statistik.arbeitsagentur.de/DE/Navigation/Service/API/API-Start-Nav.html>
- **Beispiel-Endpunkte (Doku-Seiten mit Schnittstellenbeschreibung):**
  - Arbeitslosigkeit: <https://statistik.arbeitsagentur.de/DE/Statischer-Content/Service/API/API-ALO.html>
  - Beschäftigung: <https://statistik.arbeitsagentur.de/DE/Statischer-Content/Service/API/API-BST.html>
  - Beschäftigung Zeitreihe: <https://statistik.arbeitsagentur.de/DE/Statischer-Content/Service/API/API-BST-Zeitreihe.html>
  - Gemeldete Stellen Zeitreihe: <https://statistik.arbeitsagentur.de/DE/Statischer-Content/Service/API/API-STEA-Zeitreihe.html>
- **Auth:** je API-Produkt prüfen (i. d. R. Client-/Key-basiert über das
  BA-Serviceportal); genaue Vorgaben auf den Doku-Seiten oben.
- **Formate:** maschinenlesbar (JSON/CSV) laut API-Doku.
- **Fallback ohne API:** Zeitreihen-XLSX unter
  <https://statistik.arbeitsagentur.de/DE/Navigation/Statistiken/Interaktive-Statistiken/Zeitreihen/Lange-Zeitreihen-Nav.html>

## Aktualisierung

- Arbeitslosigkeit: sehr aktuell, am Ende des Berichtsmonats (eine der ersten
  Konjunkturzahlen).
- Beschäftigung: größerer Verzug (~2 Monate wegen Meldeverfahren).

## DuckDB-Anbindungsplan

Siehe [`ARCHITECTURE.md`](../../ARCHITECTURE.md). Konkret:

1. **Zugang klären:** API-Doku-Seiten prüfen, ggf. Key/Client im BA-Serviceportal
   anlegen → als GitHub Actions Secret.
2. **Extract** `pipeline/extract/ba.py`: je Produkt aus `config/ba.yml` abrufen →
   `data/raw/ba/<produkt>/<asof>.json` (bzw. `.csv`).
3. **raw** `read_json_auto`/`read_csv_auto` → `raw_ba_<produkt>`.
4. **stg** `sql/staging/stg_ba.sql`: auf Ziel-Schema mappen (`source='ba'`,
   `geo` für Bund/Länder/Kreise), Perioden zu `DATE`.
5. **mart**: in `mart_indicators`.

**Aufwand:** mittel (API noch jung, Auth-Verfahren je Produkt verifizieren).
**Stolperstein:** die API ist neu — Feld-/Antwortstruktur zuerst an einem
Produkt (ALO) prototypen, bevor alle Produkte angebunden werden; XLSX-Fallback
bereithalten.

## Quellen

- <https://statistik.arbeitsagentur.de/>
- <https://statistik.arbeitsagentur.de/DE/Navigation/Service/API/API-Start-Nav.html>
