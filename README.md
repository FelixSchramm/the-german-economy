# The German Economy — Datenquellen-Kompass

Eine kuratierte, laufend erweiterbare Sammlung der wichtigsten **Datenquellen zur
deutschen Wirtschaft**. Ziel: einen schnellen, strukturierten Überblick über den
**Stand der Konjunktur** zu bekommen — welche Zahl kommt woher, wie aktuell ist
sie, und wie greift man sie (idealerweise per API) ab.

> Dieses Repo ist bewusst als **Quellen- und Wissenssammlung** aufgebaut, nicht
> als Datenspeicher. Es verlinkt die amtlichen/offiziellen Primärquellen und
> beschreibt Zugriffswege, Aktualisierungsrhythmen und Kennzahlen.

## Wozu das Ganze?

Wer den „Stand der Wirtschaft" beurteilen will, braucht ein kleines Set an
Indikatoren aus verschiedenen Bereichen — harte amtliche Statistiken (BIP,
Inflation, Arbeitsmarkt, Außenhandel) **und** vorlaufende Stimmungsindikatoren
(ifo, ZEW, Einkaufsmanagerindizes). Dieses Repo bündelt sie an einem Ort.

## Aufbau

Alle Inhalte liegen in [`data-sources/`](./data-sources/):

| Datei | Thema |
|-------|-------|
| [`00-uebersicht.md`](./data-sources/00-uebersicht.md) | Schnellüberblick: Kern-Indikatoren + Quellen auf einen Blick |
| [`01-gesamtwirtschaft-konjunktur.md`](./data-sources/01-gesamtwirtschaft-konjunktur.md) | BIP / VGR, Konjunkturindikatoren, Dashboard Deutschland |
| [`02-fruehindikatoren-stimmung.md`](./data-sources/02-fruehindikatoren-stimmung.md) | ifo, ZEW, HCOB/S&P Global PMI, GfK/HDE, sentix |
| [`03-arbeitsmarkt.md`](./data-sources/03-arbeitsmarkt.md) | Arbeitslosigkeit, Erwerbstätigkeit, offene Stellen, Kurzarbeit |
| [`04-preise-inflation.md`](./data-sources/04-preise-inflation.md) | VPI/HVPI, Erzeuger-, Einfuhr- und Großhandelspreise |
| [`05-produktion-industrie-handel.md`](./data-sources/05-produktion-industrie-handel.md) | Industrieproduktion, Auftragseingänge, Einzelhandel |
| [`06-aussenhandel.md`](./data-sources/06-aussenhandel.md) | Exporte / Importe, Handelsbilanz, Leistungsbilanz |
| [`07-geld-finanzmaerkte-zinsen.md`](./data-sources/07-geld-finanzmaerkte-zinsen.md) | Bundesbank, EZB, Zinsen, Geldmenge, Kredite |
| [`08-oeffentliche-finanzen.md`](./data-sources/08-oeffentliche-finanzen.md) | Staatsdefizit/-schulden, Steuern, Haushalt |
| [`09-forschungsinstitute-prognosen.md`](./data-sources/09-forschungsinstitute-prognosen.md) | SVR, Gemeinschaftsdiagnose, DIW/ifo/IW/IWH/RWI |
| [`10-internationale-quellen.md`](./data-sources/10-internationale-quellen.md) | Eurostat, OECD, IWF, Weltbank, EZB (SDW) |
| [`11-apis-und-tools.md`](./data-sources/11-apis-und-tools.md) | GENESIS-API, Bundesbank-SDMX, Dashboard-Deutschland-API, bundesAPI |

## Anbindung / Warehouse

Neben der thematischen Quellenübersicht in `data-sources/` gibt es eine
**quellenzentrierte Anbindungs-Ebene**:

- [`ARCHITECTURE.md`](./ARCHITECTURE.md) — Zielarchitektur: ein
  **DuckDB-Warehouse** (raw → staging → mart) für die wichtigsten Indikatoren.
- [`sources/`](./sources/) — je Datenquelle ein Ordner mit enthaltenen Daten,
  Zugriffsweg und konkretem DuckDB-Anbindungsplan.

Die Umsetzung wird über GitHub-Issues (ein Issue je Quelle) gesteuert.

## Prinzipien

- **Primärquellen zuerst** — amtliche Statistik (Destatis, Bundesbank, BA)
  vor aggregierten Portalen (Statista, finanzen.net).
- **Aktualität dokumentieren** — je Quelle: Frequenz + typischer
  Veröffentlichungsverzug.
- **Maschinenlesbar bevorzugen** — wo es eine API / Open-Data-CSV gibt, ist sie
  vermerkt.

## Mitmachen

Neue Quelle gefunden? Einfach die passende Datei in `data-sources/` ergänzen
(gleiches Tabellen-/Abschnittsschema beibehalten) und einen PR öffnen.

## Lizenz

Siehe [`LICENSE`](./LICENSE). Die verlinkten Datenquellen unterliegen jeweils
**eigenen Nutzungsbedingungen** (häufig „Datenlizenz Deutschland 2.0" bzw.
Eurostat-/OECD-Lizenzen) — bitte vor Weiterverwendung prüfen.

---

*Stand der letzten Recherche: Juli 2026. Links und Veröffentlichungstermine
können sich ändern — im Zweifel die jeweilige Primärquelle prüfen.*
