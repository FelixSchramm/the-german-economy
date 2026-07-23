---
status: accepted
date: 2026-07-23
decision-makers: Felix Schramm
consulted: —
informed: —
---

# ADR-0004: Deployment auf einem einzelnen VPS, ETL-Orchestrierung über GitHub Actions

## Kontext und Problemstellung

Aus ADR-0001 und ADR-0002 folgt eine Anforderung, die die Hostingauswahl deutlich einschränkt: Die Anwendung benötigt ein **persistentes, gemeinsam genutztes POSIX-Dateisystem**, auf dem die DuckDB-Datei liegt und atomar ausgetauscht werden kann.

Damit scheiden die naheliegenden Serverless-Plattformen für das Backend aus. Auf Vercel Functions, Cloud Run oder AWS Lambda ist das Dateisystem flüchtig und pro Instanz getrennt; horizontale Skalierung würde bedeuten, dass verschiedene Instanzen unterschiedliche Datenstände halten.

Zweitens muss die ETL-Pipeline regelmäßig und zuverlässig laufen, ohne dass ein dauerhaft laufender Scheduler gewartet werden muss.

## Entscheidungstreiber

- Persistentes Dateisystem mit atomarem Rename.
- Kalkulierbare Fixkosten im niedrigen einstelligen Eurobereich pro Monat.
- Wartungsaufwand von wenigen Stunden pro Quartal, nicht pro Woche.
- Reproduzierbare, versionierte ETL-Läufe mit nachvollziehbarem Protokoll.
- Kein Vendor-Lock-in.

## Betrachtete Optionen

1. **Vercel plus Serverless-Backend**
2. **Ein VPS bei Hetzner, Docker Compose**
3. **Fly.io mit persistentem Volume**
4. **Vollständig statisch auf Cloudflare Pages**, ohne Backend

## Entscheidungsergebnis

Gewählt wird eine **geteilte Topologie**:

| Komponente | Ort | Begründung |
|---|---|---|
| Frontend (Next.js) | Cloudflare Pages | zustandslos, global verteilt, kostenlos |
| API (FastAPI) | Hetzner CX22, Docker Compose | benötigt persistentes Dateisystem |
| DuckDB-Datei | Volume auf demselben VPS | muss lokal am API-Prozess liegen |
| Reverse Proxy / TLS | Caddy im selben Compose-Stack | automatische Zertifikate, minimale Konfiguration |
| CDN und Cache | Cloudflare davor | siehe ADR-0003 |
| ETL | GitHub Actions, zeitgesteuert | kein eigener Scheduler nötig |
| Archiv (Parquet) | Cloudflare R2 | portabel, kostenloser Einstieg |

Erwartete Fixkosten: rund 5 Euro pro Monat für den VPS, alles Übrige im kostenlosen Kontingent.

**ETL-Ablauf:** Ein zeitgesteuerter Workflow in GitHub Actions läuft je Quelle in deren Aktualisierungsrhythmus, baut die neue DuckDB-Datei, führt die Qualitätsprüfungen aus ADR-0002 aus, lädt die Parquet-Exporte nach R2, überträgt die Datei per `rsync` über SSH auf den VPS und löst dort den Symlink-Wechsel sowie den Reload aus.

Der Lauf ist damit vollständig im Repository versioniert, protokolliert und ohne lokale Umgebung reproduzierbar. Der VPS führt keine Rechenlast der ETL aus und bleibt entsprechend klein dimensioniert.

### Konsequenzen

**Positiv**

- Die Anforderung aus ADR-0002 wird ohne Umweg erfüllt.
- Fixe, vorhersagbare Kosten ohne nutzungsabhängige Überraschungen.
- Volle Kontrolle über die Laufzeitumgebung; die DuckDB-Version lässt sich exakt pinnen.
- Die ETL läuft auf fremder, kostenloser Rechenleistung; ihre Ausführungszeit belastet den Server nicht.
- Docker Compose ist auf jedem beliebigen Linux-Server reproduzierbar; ein Anbieterwechsel dauert Stunden, nicht Wochen.

**Negativ / einzukalkulieren**

- Der VPS ist ein Single Point of Failure ohne Redundanz. Bei einem Ausfall liefert Cloudflare dank `stale-while-revalidate` zunächst weiter, danach ist das Dashboard nicht erreichbar.
- Betriebsverantwortung liegt vollständig beim Betreiber: Systemaktualisierungen, SSH-Härtung, Firewall, Backups. Kalkuliert sind rund zwei Stunden pro Quartal; unbeaufsichtigte Sicherheitsupdates werden aktiviert.
- Keine automatische Skalierung. Angesichts der Cache-Trefferquote aus ADR-0003 ist das vertretbar.
- GitHub Actions benötigt einen SSH-Deploy-Key mit Schreibrecht auf dem Server. Der Schlüssel wird auf ein eingeschränktes Zielverzeichnis begrenzt und regelmäßig rotiert.
- Der Cron-Zeitplan von GitHub Actions ist nicht garantiert pünktlich und wird bei Inaktivität des Repositories pausiert. Ein externer Monitor prüft deshalb die Datenfrische unabhängig davon.

**Monitoring (verbindlich, nicht optional)**

- `/health` liefert `data_version`, den letzten erfolgreichen Lauf je Quelle und den Verbindungsstatus der Datenbank.
- Ein externer Uptime-Monitor prüft diesen Endpunkt und alarmiert per E-Mail, wenn eine Quelle ihr erwartetes Aktualisierungsintervall um mehr als 100 % überschreitet.
- Ein stillschweigend fehlschlagender Importer ist das wahrscheinlichste Ausfallszenario dieses Projekts: Die Anwendung bleibt erreichbar und zeigt veraltete Zahlen an, ohne dass es jemandem auffällt. Der Freshness-Alarm adressiert genau das.

## Bestätigung

- Ein vollständiger Wiederaufbau des Servers aus dem Repository heraus gelingt in unter 30 Minuten. Dies wird einmal jährlich geübt.
- Verfügbarkeit über 99 % im Monatsmittel, gemessen extern.

## Vor- und Nachteile der Optionen

### Vercel mit Serverless-Backend

- Gut: kein Serverbetrieb, hervorragende Entwicklererfahrung, Frontend-Deployment vorbildlich.
- Schlecht: flüchtiges Dateisystem pro Instanz, damit unvereinbar mit ADR-0001 und ADR-0002. Kalte Starts mit geladener DuckDB-Datei wären zudem spürbar. Für das Backend verworfen, für das Frontend eine gleichwertige Alternative zu Cloudflare Pages.

### Fly.io mit Volume

- Gut: persistentes Volume vorhanden, gute Entwicklererfahrung, regionale Verteilung möglich.
- Schlecht: Volumes sind an eine einzelne Maschine gebunden, wodurch der Vorteil der Verteilung entfällt; das kostenlose Kontingent wurde in der Vergangenheit mehrfach umgestellt. Als gleichwertige Rückfalloption vorgemerkt.

### Vollständig statisch ohne Backend

- Gut: kostenlos, wartungsfrei, unbegrenzt skalierbar.
- Schlecht: schließt interaktive Abfragen aus, siehe ADR-0003. Wird für Standardansichten ergänzend genutzt, ersetzt die API aber nicht.

## Weitere Informationen

- Verwandt: ADR-0001, ADR-0002, ADR-0003
- Offen: Sicherungsstrategie für den VPS selbst. Da der gesamte Datenbestand aus öffentlichen Quellen reproduzierbar ist und Parquet-Snapshots in R2 liegen, hat ein Server-Snapshot niedrige Priorität; die Entscheidung ist dennoch zu dokumentieren.
