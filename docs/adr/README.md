# Architecture Decision Records (ADR)

Architekturentscheidungen im [MADR](https://adr.github.io/madr/)-Format. Jede
Datei dokumentiert **eine** Entscheidung mit Kontext, betrachteten Optionen und
Konsequenzen.

## Index

| ADR | Titel | Status |
|-----|-------|--------|
| 0001 | *(referenziert, noch nicht abgelegt)* | — |
| 0002 | *(referenziert, noch nicht abgelegt)* | — |
| 0003 | *(referenziert, noch nicht abgelegt)* | — |
| [0004](./0004-vps-deployment-und-github-actions-etl.md) | Deployment auf einem einzelnen VPS, ETL-Orchestrierung über GitHub Actions | accepted |

> **Hinweis:** ADR-0004 verweist auf ADR-0001–0003 (persistentes Dateisystem /
> DuckDB, Qualitätsprüfungen, Caching/CDN). Diese sind hier noch **nicht**
> abgelegt — bitte nachreichen, damit die Querverweise auflösen.

## Konventionen

- Dateiname: `NNNN-kurzer-slug.md` (fortlaufende Nummer).
- Neue ADRs starten im Status `proposed`, dann `accepted`/`rejected`/`superseded`.
- Eine Entscheidung wird nicht editiert, sondern durch ein neues ADR abgelöst
  (`superseded by ADR-XXXX`).
