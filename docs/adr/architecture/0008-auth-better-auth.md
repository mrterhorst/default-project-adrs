# ADR-0008: Authenticatie — Better Auth

## Status
Accepted

## Context
De projecten hebben een authenticatieoplossing nodig die framework-agnostisch is, goed integreert met Hono en Remix, en sessie- en OAuth-gebaseerde auth ondersteunt. De oplossing moet self-hosted zijn om volledige controle over gebruikersdata te behouden.

## Beslissing
Better Auth wordt gebruikt als authenticatiebibliotheek. Better Auth is een framework-agnostische, TypeScript-first auth-bibliotheek die werkt met elke HTTP-handler (inclusief Hono) en eigen adapters heeft voor Drizzle ORM.

## Consequenties
**Positief:**
- Framework-agnostisch: werkt met Hono, Next.js, Express en anderen via een standaard Request/Response interface.
- Drizzle-adapter beschikbaar — schema en sessies integreren direct met de bestaande database-laag.
- Ondersteunt email/wachtwoord, OAuth (GitHub, Google, etc.) en magic links out-of-the-box.
- Self-hosted: gebruikersdata blijft in eigen PostgreSQL-database.
- TypeScript-first met goede type-inferentie voor sessies en gebruikers.
- Actieve ontwikkeling en groeiende community.

**Negatief:**
- Jonger en minder bewezen dan Auth.js of Lucia in productieomgevingen.
- Documentatie is nog in ontwikkeling; sommige edge-cases zijn minder goed gedocumenteerd.
- Self-hosted betekent zelf verantwoordelijk voor security-updates en patches.

## Alternatieven overwogen
- **Auth.js (NextAuth)**: Breed gebruikt, maar nauw verweven met Next.js. Hono-integratie is mogelijk maar niet first-class. Valt af op framework-fit.
- **Lucia**: Lichtgewicht en framework-agnostisch, maar discontinued (project is gestopt). Valt af op continuïteit.
- **Clerk**: Uitstekende DX en UI-componenten, maar SaaS-model (data bij derde partij, kosten bij schaal). Valt af op data-eigenaarschap en kosten.
- **Supabase Auth**: Goed, maar koppelt authenticatie aan Supabase als hosting-platform. Valt af op vendor lock-in.
- **Zelf implementeren**: Maximale controle maar veiligheidsrisico's en veel onderhoud. Valt af op veiligheid en onderhoud.
