# ADR & Conventions Repository

Dit is de centrale kennisrepo voor alle projecten van mrterhorst.
Lees deze repo altijd via GitHub MCP voordat je architectuur- of technologiekeuzes maakt.

## Structuur

```
docs/
  adr/
    architecture/   →  welke technologie en waarom (frameworks, ORM, auth, API-stijl)
    infra/          →  hoe en waar dingen draaien (runtime, deployment, lokale setup)
    tooling/        →  developer workflow en tools
  conventions/      →  bindende regels voor code en samenwerking
```

## ADR-formaat (Nygard, Nederlands)

```markdown
# ADR-XXXX: Onderwerp

## Status
Accepted

## Context
Waarom moest er een beslissing gemaakt worden?

## Beslissing
Wat is er besloten?

## Consequenties
Wat zijn de gevolgen, zowel positief als negatief?

## Alternatieven overwogen
Welke alternatieven zijn afgevallen en waarom?
```

- Genummerd: `XXXX-onderwerp.md` (viercijferig)
- Status: `Proposed` | `Accepted` | `Deprecated` | `Superseded`
- Taal: Nederlands

## Conventions (bindend)

De bestanden in `docs/conventions/` zijn bindende regels — geen discussie, geen uitzonderingen zonder een nieuw ADR.

- `database.md` — naamgeving, primary keys, verplichte kolommen, migraties
- `typescript.md` — naamgeving, stijl, bestandsstructuur
- `api.md` — REST-conventies, response-formaat, statuscodes
- `git.md` — commit messages, branches, werkwijze

## Werkwijze

- Lees altijd relevante ADRs en conventions vóórdat je een technologiekeuze maakt of code schrijft
- Geen schema-wijzigingen zonder Drizzle-migratie
- Database-first: schema wordt eerst ontworpen, daarna de code
- ADRs schrijven via GitHub MCP, daarna lokaal `git pull`
- Code altijd lokaal wijzigen, nooit direct via MCP

## Stack (samenvatting)

| Laag | Keuze | ADR |
|---|---|---|
| Runtime | Bun | 0001 |
| Backend API | Hono | 0002 |
| Frontend | Remix + React | 0003 |
| CSS | Tailwind | 0004 |
| Components | shadcn/ui | 0005 |
| ORM | Drizzle | 0006 |
| Database | PostgreSQL | 0007 |
| Auth | Better Auth | 0008 |
| Validatie | Zod | 0009 |
| Dev-omgeving | WSL2 + Wrangler | 0010 |
| API-stijl | REST + Hono RPC | 0014 |
