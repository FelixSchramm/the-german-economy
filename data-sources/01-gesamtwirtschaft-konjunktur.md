# 01 — Gesamtwirtschaft & Konjunktur (BIP, VGR, Dashboards)

Kennzahlen zur gesamtwirtschaftlichen Leistung und zentrale Aggregatoren.

## Bruttoinlandsprodukt (BIP) / Volkswirtschaftliche Gesamtrechnungen (VGR)

Das BIP ist die zentrale Messgröße der Wirtschaftsleistung. Die VGR liefern
zusätzlich Konsum, Investitionen, Staatsausgaben, Außenbeitrag,
Bruttowertschöpfung nach Branchen u. v. m.

- **Quelle:** Statistisches Bundesamt (Destatis) — VGR
- **Frequenz:** quartalsweise (mit jährlichen Aggregaten)
- **Verzug:** BIP-**Schnellmeldung** ca. 30 Tage nach Quartalsende, danach
  detaillierte Ergebnisse und Revisionen
- **Wichtige Größen:** BIP real (preisbereinigt), saison- und kalenderbereinigt
  (X13-JDemetra), Verwendungs- und Entstehungsseite
- **Link:** <https://www.destatis.de/DE/Themen/Wirtschaft/Volkswirtschaftliche-Gesamtrechnungen-Inlandsprodukt/_inhalt.html>
- **Daten/API:** GENESIS-Online (Themenbereich 81000), siehe
  [`11-apis-und-tools.md`](./11-apis-und-tools.md)

## Konjunkturindikatoren (Destatis Open Data, CSV)

Destatis stellt ein gebündeltes Open-Data-Angebot mit den wichtigsten
Konjunkturindikatoren als **CSV** bereit — ideal für automatisierte Pipelines.

- **Inhalt:** u. a. Produktion, Auftragseingänge, Umsätze, Außenhandel,
  Preise, Arbeitsmarkt als lange Reihen
- **Link:** <https://www.destatis.de/DE/Service/OpenData/konjunkturindikatoren-csv.html>
- **Konjunkturindikatoren-Themenseite:** <https://www.destatis.de/DE/Themen/Wirtschaft/Konjunkturindikatoren/_inhalt.html>

## Dashboard Deutschland (Destatis)

Aggregiertes Portal, das amtliche und weitere Daten (Konjunktur, Preise,
Arbeitsmarkt, Energie, Mobilität) in einem Dashboard zusammenführt — inkl.
experimenteller/Echtzeit-Indikatoren.

- **Web:** <https://www.dashboard-deutschland.de/>
- **API:** offene Schnittstelle, dokumentiert über die bundesAPI-Community:
  <https://github.com/bundesAPI/dashboard-deutschland-api>
- **Nutzung:** guter Startpunkt für einen schnellen Gesamtüberblick und für
  Indikatoren, die sonst über viele Quellen verteilt sind

## Konjunkturprognosen des Bundes

- **Jahreswirtschaftsbericht / Projektion der Bundesregierung** (BMWK):
  <https://www.bmwk.de/> (jährlich, mit Frühjahrs-/Herbstprojektion)
- Prognosen der Institute: siehe
  [`09-forschungsinstitute-prognosen.md`](./09-forschungsinstitute-prognosen.md)

---

**Verwandte Dateien:** Frühindikatoren → [02](./02-fruehindikatoren-stimmung.md),
Produktion/Aufträge → [05](./05-produktion-industrie-handel.md).
