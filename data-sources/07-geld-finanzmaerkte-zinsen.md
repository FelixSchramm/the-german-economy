# 07 — Geld, Finanzmärkte & Zinsen

Geldpolitik, Zinsen, Geldmenge und Kreditvergabe — überwiegend Bundesbank/EZB.

## Deutsche Bundesbank — Statistikportal & Zeitreihen

Zentrale Quelle für monetäre und Finanzstatistiken zu Deutschland.

- **Statistikportal:** <https://www.bundesbank.de/de/statistiken>
- **Zeitreihen & Echtzeitdaten:** <https://www.bundesbank.de/de/statistiken/zeitreihen-und-echtzeitdaten>
- **Inhalte:** Zinsen, Geldmengen, Bankstatistik/Kredite, Zahlungsbilanz,
  Wertpapiere, Wechselkurse, Realzins/Umlaufsrenditen

> **Wichtiger Hinweis (Migration):** Die klassische **Zeitreihen-Datenbank**
> wurde zum **30.06.2026 abgeschaltet**. Die Reihen sind ins neue
> **Statistikportal** umgezogen. Der **SDMX-Webservice** (REST-API) bleibt der
> maschinenlesbare Zugangsweg — siehe [`11-apis-und-tools.md`](./11-apis-und-tools.md).

## Leitzinsen & Geldpolitik (EZB)

Für den Euroraum verantwortlich ist die EZB — die Bundesbank setzt sie in
Deutschland um.

- **EZB-Leitzinsen (Hauptrefinanzierung, Einlagesatz, Spitzenrefinanzierung):**
  <https://www.ecb.europa.eu/stats/policy_and_exchange_rates/key_ecb_interest_rates/html/index.en.html>
- **EZB Data Portal (ehem. SDW):** <https://data.ecb.europa.eu/>
- **Geldpolitische Beschlüsse:** <https://www.ecb.europa.eu/press/govcdec/html/index.en.html>

## Wichtige Kennzahlen

| Kennzahl | Beschreibung | Quelle | Frequenz |
|----------|--------------|--------|----------|
| **EZB-Einlagesatz / Hauptrefi-Satz** | geldpolitischer Leitzins | EZB | bei Ratssitzung |
| **Geldmenge M1/M2/M3** | Liquidität im Euroraum | EZB/Bundesbank | monatlich |
| **Bankkredite an Unternehmen/Haushalte** | Kreditdynamik | Bundesbank | monatlich |
| **Umlaufsrendite / Bundesanleihen** | Kapitalmarktzins | Bundesbank | täglich |
| **€STR (Euro Short-Term Rate)** | Referenz-Tagesgeldsatz | EZB | täglich |
| **Bank Lending Survey (BLS)** | Kreditvergabestandards der Banken | EZB/Bundesbank | quartalsweise |

## Aktien-/Kapitalmarkt (Kontext)

- **DAX / Deutsche Börse:** <https://www.deutsche-boerse.com/> (Marktdaten,
  häufig kostenpflichtig für historische Reihen)
- **Renditen/Spreads** über das EZB Data Portal und die Bundesbank verfügbar

---

**Verwandte Dateien:** Inflationsziel & Preisdruck →
[04](./04-preise-inflation.md); internationale Zinsvergleiche →
[10](./10-internationale-quellen.md); API-Zugriff →
[11](./11-apis-und-tools.md).
