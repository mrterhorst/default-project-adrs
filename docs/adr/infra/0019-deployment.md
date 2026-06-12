# ADR-0019: Deployment — Wrangler + Neon

**Scope:** deployment, infra

## Status
Accepted

## Context
De applicatie draait op Cloudflare Workers met Neon als hosted PostgreSQL. Er is een reproduceerbare deployprocedure nodig die past bij de "no fuzz, free-tier-maxxing"-filosofie. Geen CI/CD-overhead voor persoonlijke en vroege-fase projecten.

## Beslissing

### Deploytool: Wrangler (handmatig)
Deployment naar productie gebeurt handmatig vanuit de terminal:

```bash
# 1. Migraties toepassen op Neon (productiedatabase)
bun run db:migrate

# 2. Worker deployen naar Cloudflare
wrangler deploy
```

`bun run db:migrate` is een package.json-script dat `drizzle-kit migrate` aanroept — zie conventions/database.md voor de reden (Bun-websocket-issue bij rechtstreeks `bunx drizzle-kit migrate`).

Geen CI/CD-pipeline, geen preview-omgevingen. De enige omgevingen zijn **lokaal** en **productie**.

### Cloudflare Workers (runtime)
De applicatie draait als Cloudflare Worker. Configuratie in `wrangler.toml`:

```toml
name = "mijnproject"
main = "src/index.ts"
compatibility_date = "2024-01-01"

[[r2_buckets]]
binding = "BUCKET"
bucket_name = "mijnproject-uploads"

[[queues.producers]]
binding = "QUEUE"
queue = "mijnproject-queue"
```

### Neon (PostgreSQL productie)
Neon wordt gebruikt als hosted PostgreSQL. De `DATABASE_URL` wordt ingesteld als Wrangler secret:

```bash
wrangler secret put DATABASE_URL
```

Migraties worden altijd uitgevoerd vóór de Worker-deploy. De Drizzle-migraties in `drizzle/migrations/` zijn de single source of truth voor het databaseschema.

### Free tier limieten (referentie)

| Service | Free tier |
|---|---|
| Cloudflare Workers | 100.000 requests/dag, 10 ms CPU/request |
| Cloudflare R2 | 10 GB opslag, 1M schrijfoperaties/maand |
| Cloudflare Queues | 1M berichten/maand |
| Neon | 0,5 GB opslag, 1 project, 1 branch |
| Sentry | 5.000 events/maand |

## Consequenties
**Positief:**
- Minimale tooling: `wrangler deploy` is één commando en vereist geen externe CI/CD-service.
- Volledige controle over wanneer er gedeployd wordt — geen ongewenste automatische deploys.
- Free tiers van Cloudflare en Neon zijn ruim genoeg voor persoonlijke projecten en vroege-fase producten.
- Reproduceerbaar: dezelfde twee stappen (migrate → deploy) werken altijd.

**Negatief:**
- Handmatig deployen vereist discipline — geen automatische deploy na een commit.
- Geen rollback-mechanisme ingebouwd; terugdraaien is een nieuwe deploy van de vorige versie.
- Bij groei naar een team is handmatig deployen niet houdbaar — dan migreren naar GitHub Actions (eenvoudig toe te voegen).

## Alternatieven overwogen
- **GitHub Actions (automatisch):** Professioneler en foutbestendig voor teams, maar voegt setup-overhead toe die niet nodig is in de beginfase. Makkelijk toe te voegen later.
- **Cloudflare Pages (voor Remix):** Goed voor pure frontend-deploys, maar de stack gebruikt Workers voor zowel API als frontend — één deploy is eenvoudiger.
- **Supabase als database:** Rijkere feature-set maar extra abstractielaag. Valt af ten gunste van Neon (bare PostgreSQL).
