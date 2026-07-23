# 11 — APIs & Tools (maschinenlesbarer Zugriff)

Für automatisierte Pipelines/Dashboards: die wichtigsten programmatischen
Zugänge zu deutschen Wirtschaftsdaten.

## Destatis GENESIS-Online — Webservice (REST/JSON)

Tiefe amtliche Statistik-Datenbank mit API-Zugang.

- **Weboberfläche:** <https://www-genesis.destatis.de/datenbank/online>
- **API-/Webservice-Doku:** <https://www.destatis.de/DE/Service/OpenData/genesis-api-webservice-oberflaeche.html>
- **OpenAPI-Spezifikation (Community, bund.dev):** <https://destatis.api.bund.dev/>
- **Zugang:** kostenlose Registrierung; Abruf per Tabellen-ID (z. B. Außenhandel
  `51000`, VGR `81000`); Formate JSON/CSV
- **Hinweis:** Ratenlimits/Token beachten; für große Reihen paginieren

## Destatis Open-Data-CSV (ohne Registrierung)

- **Konjunkturindikatoren als CSV:** <https://www.destatis.de/DE/Service/OpenData/konjunkturindikatoren-csv.html>
- **Vorteil:** direkter Datei-Download, ideal für einfache ETL-Jobs

## Dashboard Deutschland — API

- **Doku (bundesAPI):** <https://github.com/bundesAPI/dashboard-deutschland-api>
- **Web:** <https://www.dashboard-deutschland.de/>
- **Inhalt:** aggregierte Indikatoren aus vielen Quellen über eine Schnittstelle

## Deutsche Bundesbank — SDMX-Webservice (REST)

- **Hilfe/Doku:** <https://www.bundesbank.de/de/statistiken/zeitreihen-datenbanken/hilfe-zu-sdmx-webservice>
- **Formate:** SDMX-ML 2.1 und SDMX-CSV; ausschließlich REST über HTTPS
- **Abruf:** je Dataflow (z. B. `BBEX3` für Wechselkurse) bzw. je Zeitreihen-Key
- **Wichtig:** Die alte Zeitreihen-Datenbank wurde **30.06.2026** abgeschaltet;
  Reihen liegen nun im **Statistikportal**, der SDMX-Webservice bleibt bestehen

## EZB — Data Portal API (SDMX-REST)

- **Doku:** <https://data.ecb.europa.eu/help/api/overview>
- **Formate:** SDMX-ML, SDMX-CSV, JSON
- **Inhalt:** Zinsen, Geldmengen, Wechselkurse, Finanzstatistiken Euroraum

## Eurostat — REST-API

- **Doku:** <https://ec.europa.eu/eurostat/web/main/data/web-services>
- **Formate:** JSON-stat, SDMX
- **Inhalt:** harmonisierte EU-Reihen (HVPI, BIP, Arbeitslosigkeit, Defizit)

## bundesAPI — Community-Sammlung deutscher Behörden-APIs

- **Übersicht:** <https://bund.dev/> und <https://github.com/bundesAPI>
- **Nutzen:** OpenAPI-Specs und Beispielclients für Destatis, Dashboard
  Deutschland, KBA u. v. m.

## Praktische Hinweise

- **Caching:** Reihen ändern sich meist nur monatlich/quartalsweise — lokal
  cachen (vgl. Muster im Schwester-Repo, z. B. `@st.cache_data(ttl=3600)`).
- **Revisionen beachten:** amtliche Daten werden nachträglich revidiert; API-Pulls
  idealerweise mit Abrufdatum versionieren.
- **Lizenz:** meist „Datenlizenz Deutschland 2.0" (Namensnennung) — vor
  Weiterverwendung prüfen.

---

**Verwandte Dateien:** die inhaltlichen Quellen in
[01](./01-gesamtwirtschaft-konjunktur.md)–[10](./10-internationale-quellen.md).
