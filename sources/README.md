# sources/ — Datenquellen mit Anbindungsplan

Ein Unterordner je Datenquelle. Jede `README.md` beschreibt **welche Daten** die
Quelle enthält, **wie man zugreift** und den **DuckDB-Anbindungsplan**. Die
gemeinsame Zielarchitektur steht in [`../ARCHITECTURE.md`](../ARCHITECTURE.md).

> Thematischer Überblick nach Indikatoren (BIP, Inflation, …) →
> [`../data-sources/`](../data-sources/). Dieser Ordner ist quellenzentriert und
> auf die technische Anbindung ausgerichtet.

## Kern-Set (Runde 1)

| Quelle | Ordner | Zugriff | Auth | Aufwand |
|--------|--------|---------|------|---------|
| Eurostat | [`eurostat/`](./eurostat/) | REST (JSON-stat/SDMX-CSV) | keine | niedrig |
| EZB Data Portal | [`ezb-data-portal/`](./ezb-data-portal/) | SDMX-REST | keine | niedrig |
| Deutsche Bundesbank | [`bundesbank-sdmx/`](./bundesbank-sdmx/) | SDMX-REST | keine | niedrig |
| Destatis GENESIS | [`destatis-genesis/`](./destatis-genesis/) | REST/JSON | Login/Token | mittel |
| Bundesagentur für Arbeit | [`bundesagentur-fuer-arbeit/`](./bundesagentur-fuer-arbeit/) | API (seit 12/2025) | ggf. Key | mittel |
| ifo Geschäftsklima | [`ifo-geschaeftsklima/`](./ifo-geschaeftsklima/) | XLSX-Download | keine | mittel |
| ZEW-Konjunkturerwartungen | [`zew-konjunkturerwartungen/`](./zew-konjunkturerwartungen/) | XLSX/CSV (Scraping) | keine | mittel-hoch |

## Empfohlene Anbindungsreihenfolge

Erst die offenen APIs (geringster Aufwand, höchste Aktualität), dann Auth-Quellen,
zuletzt die dateibasierten:

1. Eurostat → 2. EZB → 3. Bundesbank → 4. Destatis GENESIS →
5. Bundesagentur für Arbeit → 6. ifo → 7. ZEW

## Schema je Quellen-README

Damit alle Quellen einheitlich dokumentiert sind:

- **Kurzbeschreibung** + Betreiber, Zugriffsart, Lizenz
- **Enthaltene Daten** (Tabelle relevanter Reihen/IDs)
- **Zugriff / API** (Base-URL, Auth, Formate, Beispiel-Request)
- **Aktualisierung** (Frequenz, Verzug, Revisionen)
- **DuckDB-Anbindungsplan** (Extract → raw → stg → mart, Aufwand, Stolpersteine)
- **Quellen** (Links)

## Weitere Quellen (spätere Runden)

Kandidaten aus [`../data-sources/`](../data-sources/), noch nicht angebunden:
Kraftfahrt-Bundesamt (KBA), OECD, IWF/WEO, Destatis-Staatsfinanzen, BMF, sowie
Stimmungsindikatoren (HCOB/S&P Global PMI, GfK-Konsumklima, sentix).
