# ADR-0012: Database lokaal — PostgreSQL via Docker Compose

**Scope:** database, infra

## Status
Accepted

## Context
Tijdens development draait PostgreSQL lokaal (zie ADR-0011). Er moet een keuze gemaakt worden over hoe de container wordt gestart, hoe data wordt bewaard en of projecten een gedeelde of eigen database-instantie krijgen.

## Beslissing
Elk project krijgt een eigen `compose.yml` in de root van de repository. PostgreSQL draait als een Docker-container met een named volume voor persistentie.

### Standaard `compose.yml`
```yaml
services:
  db:
    image: postgres:17
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### Omgevingsvariabelen
De database-credentials staan in `.env` (niet in Git):
```
DB_NAME=mijnproject
DB_USER=mijnproject
DB_PASSWORD=dev
DATABASE_URL=postgresql://mijnproject:dev@localhost:5432/mijnproject
```

`DATABASE_URL` wordt direct door Drizzle gebruikt. De `compose.yml` zelf wordt wél gecommit; `.env` niet.

### Veelgebruikte commando's
| Actie | Commando |
|---|---|
| Start DB (achtergrond) | `docker compose up -d` |
| Stop DB | `docker compose stop` |
| Reset (data wissen) | `docker compose down -v` |
| Logs bekijken | `docker compose logs db` |

Na een reset: Drizzle-migraties opnieuw uitvoeren met `bunx drizzle-kit migrate`.

## Consequenties
**Positief:**
- `compose.yml` is versie-beheerd en staat in de repo — nieuwe teamleden of een nieuwe machine zijn direct operationeel na `docker compose up -d`.
- Named volume bewaart data tussen restarts; expliciete reset (`down -v`) geeft een schone lei wanneer nodig.
- Eén container per project geeft volledige isolatie: geen risico van schema-conflicten tussen projecten.
- Docker Compose is uitbreidbaar: een tweede service (bijv. mailhog, redis) toevoegen is één blok in hetzelfde bestand.
- Dicht bij productie: zelfde PostgreSQL-versie (17) als de Neon-hosted omgeving.

**Negatief:**
- Elk project heeft een eigen container; bij meerdere gelijktijdige projecten draaien meerdere PostgreSQL-instanties. Geheugengebruik is beperkt (~50–100 MB per container), maar is het vermelden waard.
- Poortnummer 5432 kan conflicteren als twee projecten tegelijk draaien; per project een afwijkende poort instellen (bijv. `5433:5432`) lost dit op.

## Alternatieven overwogen
- **`docker run` (los commando):** Werkt, maar is niet gedocumenteerd in de repo en moeilijker te herhalen. Valt af op reproduceerbaarheid.
- **Lokale PostgreSQL-installatie:** Geen Docker nodig, maar vervuilt de host met een systeembrede installatie en maakt versiewissels lastig. Valt af.
- **Gedeelde lokale PostgreSQL (één container, meerdere databases):** Lichter, maar schema-migraties van het ene project kunnen de verbinding van het andere verstoren. Valt af op isolatie.
