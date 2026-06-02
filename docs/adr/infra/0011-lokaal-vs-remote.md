# ADR-0011: Lokaal vs remote — wat draait waar

## Status
Accepted

## Context
De stack draait uiteindelijk op Cloudflare Workers met bijbehorende services (R2, Queues) en een hosted PostgreSQL-database. Tijdens development moet duidelijk zijn wat lokaal emuleerbaar is, wat een echte externe service vereist, en hoe de overgang van lokaal naar preview/productie eruit ziet.

## Beslissing
Er zijn drie omgevingen met een duidelijke scheiding:

### Lokaal (development)
| Laag | Wat draait lokaal |
|---|---|
| App / API | `wrangler dev` — emuleert de Cloudflare Workers runtime via Miniflare |
| Database | PostgreSQL in Docker (zie ADR-0012) |
| File storage | Wrangler local R2 — opgeslagen in een lokale map |
| Queues | Wrangler local Queues — in-memory verwerking |

`wrangler dev` wordt verkozen boven `bun dev` omdat de stack Cloudflare-specifieke bindings gebruikt (R2, Queues). Wrangler emuleert deze lokaal zonder externe verbinding. Hono werkt identiek op beide runtimes — geen codewijzigingen nodig.

### Preview (staging)
Cloudflare Workers met echte CF-services (R2, Queues) en een aparte Neon-branch of -database als preview-database. Ingezet via Wrangler of CI/CD (zie ADR-0019).

### Productie
Cloudflare Workers + Cloudflare R2 + Cloudflare Queues + Neon (PostgreSQL). Neon wordt gebruikt voor hosted PostgreSQL; geen Supabase.

## Consequenties
**Positief:**
- Volledig offline ontwikkelen is mogelijk: alle lokale lagen draaien zonder externe verbinding.
- `wrangler dev` vangt runtime-discrepanties vroeg op die `bun dev` zou missen (Workers-specifieke API's, binding-gedrag).
- Neon biedt database-branches, waardoor preview-omgevingen een geïsoleerde kopie van de schema kunnen krijgen.
- Duidelijke grens: wat lokaal werkt, werkt op preview en productie — geen omgevingsspecifieke code.

**Negatief:**
- Wrangler-emulatie is niet 100% identiek aan de productie Workers-runtime; edge cases kunnen pas op preview zichtbaar worden.
- Wrangler local R2 en Queues zijn vereenvoudigde emulaties — gedrag bij fouten of limieten kan afwijken van productie.
- Neon vereist een externe verbinding voor preview en productie; geen lokale Neon-alternatief.

## Alternatieven overwogen
- **`bun dev` als primaire dev-server:** Sneller opstarten, maar mist Workers-runtime emulatie en CF-bindings. Valt af zodra R2 of Queues in gebruik zijn.
- **Altijd echte Cloudflare-services (ook lokaal):** Geen emulatie-afwijkingen, maar vereist netwerkverbinding en risico op vervuiling van echte data. Valt af op ontwikkelcomfort.
- **Supabase als hosted PostgreSQL:** Feature-rijker dan Neon, maar voegt een eigen auth-laag en extra abstractie toe die niet nodig is. Neon is lichter en dicht bij bare PostgreSQL. Valt af.
