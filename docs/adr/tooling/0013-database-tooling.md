# ADR-0013: Database tooling — Drizzle Studio + Adminer

## Status
Accepted

## Context
Tijdens development is een GUI nodig om de database te inspecteren: schema bekijken en data browsen. De database draait in Docker in WSL2 en is bereikbaar op `localhost:5432` vanuit Windows (Docker Desktop's WSL2-integratie exposed de poort automatisch).

Een Windows-native GUI-installatie is niet gewenst; de voorkeur gaat uit naar browser-gebaseerde tools die draaien als onderdeel van de development-setup.

## Beslissing
Twee gratis, browser-gebaseerde tools worden gebruikt, elk met een eigen rol:

### Drizzle Studio (schema-inzicht)
Drizzle Studio wordt gestart via:
```bash
bunx drizzle-kit studio
```
Het opent automatisch in de browser en toont het schema exact zoals gedefinieerd in de Drizzle-schemadefinities. Geen installatie, geen configuratie — het leest de bestaande Drizzle-config. Primair gebruik: schema inspecteren en snel data browsen via de Drizzle-laag.

### Adminer (data browsen en SQL)
Adminer draait als extra service in `compose.yml` naast PostgreSQL:
```yaml
services:
  db:
    image: postgres:17
    # ... (zie ADR-0012)

  adminer:
    image: adminer
    ports:
      - "8080:8080"
    depends_on:
      - db
```
Bereikbaar op `http://localhost:8080`. Inloggen met de credentials uit `.env`. Primair gebruik: ad-hoc SQL uitvoeren, data aanpassen, tabellen browsen zonder Drizzle-context.

### Rolverdeling
| Behoefte | Tool |
|---|---|
| Schema bekijken (Drizzle-aware) | Drizzle Studio |
| Data browsen en filteren | Adminer of Drizzle Studio |
| Ad-hoc SQL uitvoeren | Adminer |
| Migraties uitvoeren | `bunx drizzle-kit migrate` (terminal) |

## Consequenties
**Positief:**
- Geen Windows-installaties: beide tools draaien in de browser.
- Adminer in `compose.yml` is automatisch beschikbaar zodra de development-omgeving opstart — geen aparte stap.
- Drizzle Studio is Drizzle-aware: toont relaties en types zoals ze in code zijn gedefinieerd, niet alleen de ruwe PostgreSQL-structuur.
- Beide tools zijn gratis en open source.

**Negatief:**
- Adminer is functioneel maar sober; minder features dan DBeaver of TablePlus voor complexe query-analyse.
- Drizzle Studio is een development-helper, geen volwaardige database-beheer-tool.
- Adminer draait altijd mee als `compose.yml` up is; wil je het niet, verwijder de service of start selectief met `docker compose up db`.

## Alternatieven overwogen
- **DBeaver (Windows-app):** Gratis, krachtig en vertrouwd. Verbindt via `localhost:5432`. Valt niet volledig af, maar is zwaarder dan nodig voor data-inzicht en schema-browsen. Bruikbaar als terugvaloptie voor complexe queries.
- **TablePlus (Windows-app):** Modern en licht, maar betaald. Valt af op kosten.
- **pgAdmin 4 (Docker-container):** Officiële PostgreSQL GUI, gratis. Functioneel maar zwaar en minder intuïtief dan Adminer voor eenvoudige taken.
