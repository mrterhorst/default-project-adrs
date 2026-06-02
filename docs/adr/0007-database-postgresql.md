# ADR-0007: Database — PostgreSQL

## Status
Accepted

## Context
De projecten hebben een relationele database nodig voor persistente opslag. De database moet betrouwbaar, open source en breed ondersteund zijn, en goed werken met Drizzle ORM.

## Beslissing
PostgreSQL wordt gebruikt als primaire relationele database. PostgreSQL is een volwassen, open source RDBMS met uitstekende SQL-compliance, krachtige features (JSONB, full-text search, CTEs, window functions) en brede hostingondersteuning.

## Consequenties
**Positief:**
- Meest volwassen open source relationele database met uitstekende SQL-compliance.
- Rijke feature-set: JSONB, arrays, full-text search, window functions, CTEs.
- Brede hostingondersteuning: Supabase, Neon, Railway, Render, zelf-gehost.
- Uitstekende integratie met Drizzle ORM.
- ACID-compliant en battle-tested voor productie-workloads.
- Sterke community en documentatie.

**Negatief:**
- Vereist een managed database-service of eigen infrastructuur; niet serverless by default.
- Voor edge-deployments (Cloudflare Workers) is directe TCP-verbinding niet mogelijk — vereist een HTTP-proxy (bv. Neon serverless driver of Supabase).
- Zwaarder dan SQLite voor lokale development; vereist een draaiende database-server.

## Alternatieven overwogen
- **MySQL / MariaDB**: Vergelijkbaar, maar minder SQL-compliant en minder rijke feature-set. Valt af op feature-pariteit.
- **SQLite (via Turso/libSQL)**: Serverless-vriendelijk en edge-compatibel, maar minder geschikt voor multi-user productie-workloads. Valt af op schaalbaarheid en productie-fit.
- **PlanetScale (MySQL)**: Interessante branching-feature, maar MySQL-basis en niet open source hosted. Valt af op filosofie en vendor lock-in.
- **MongoDB**: Document-gebaseerd, flexibel schema, maar relationele queries zijn complex zonder SQL. Valt af op SQL-kennis hergebruik en datamodellering.
