# ADR-0006: ORM — Drizzle ORM

**Scope:** database, orm

## Status
Accepted

## Context
De projecten hebben een manier nodig om met de PostgreSQL-database te communiceren vanuit TypeScript. De oplossing moet lichtgewicht zijn, dicht bij SQL blijven, type-safe zijn en goed werken met Bun en Cloudflare Workers.

## Beslissing
Drizzle ORM wordt gebruikt als database-toegangslaag. Drizzle is een TypeScript-first ORM die een thin, type-safe laag over SQL biedt zonder verborgen magie. Queries worden geschreven in een SQL-achtige DSL die direct naar SQL vertaalt.

## Consequenties
**Positief:**
- TypeScript-first: schema-definities en queries zijn volledig type-safe.
- Lichtgewicht en zonder runtime-overhead — geen grote abstractielagen.
- Dicht bij SQL: de DSL is herkenbaar voor iedereen met SQL-kennis; geen verborgen query-generatie.
- Werkt met Bun en Cloudflare Workers (geen Node.js-afhankelijkheden).
- Drizzle Kit biedt schema-migraties op basis van schema-diff en `bunx drizzle-kit studio` als browser-gebaseerde schema- en data-viewer tijdens development.
- Ondersteunt PostgreSQL, MySQL, SQLite en libSQL.

**Negatief:**
- Minder volwassen dan Prisma of TypeORM; sommige geavanceerde features ontbreken nog.
- Kleinere community en minder third-party integraties.
- Complexe queries kunnen verbose worden in de DSL.

## Alternatieven overwogen
- **Prisma**: De meest populaire Node.js ORM, uitstekende DX en grote community. Maar: zware runtime, vereist een query engine binary, niet geschikt voor edge-omgevingen. Valt af op edge-compatibiliteit en bundle-grootte.
- **TypeORM**: Volwassen maar decorator-gebaseerd, complex en met veel impliciete gedrag. Valt af op complexiteit en filosofie (flat over gelaagd).
- **Kysely**: Query builder (geen volledige ORM), type-safe en lichtgewicht. Goede optie, maar geen ingebouwde schema/migratie-tooling. Valt af op productiviteit ten opzichte van Drizzle.
- **Raw SQL (postgres.js / pg)**: Maximale controle en geen overhead, maar geen type-safety op queries. Valt af op DX en onderhoud bij complexere queries.
