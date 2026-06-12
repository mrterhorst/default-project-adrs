# Conventie: Developer workflow

## Nieuw project starten

Bij het opstarten van een nieuw project leest Claude altijd eerst:

1. `github.com/mrterhorst/default-project-adrs` via GitHub MCP
   - Alle ADRs in `docs/adr/` (architecture, infra, tooling)
   - Alle conventions in `docs/conventions/`
2. Op basis daarvan maakt Claude een `CLAUDE.md` aan in de projectroot (zie onderaan).

Daarna:
- `compose.yml` aanmaken (zie ADR-0012)
- `wrangler.toml` aanmaken (zie ADR-0019)
- `.env` en `.env.example` aanmaken
- Drizzle configureren en eerste schema opstellen

## Dagelijkse werkwijze

### Opstarten
```bash
docker compose up -d          # PostgreSQL + Adminer starten
wrangler dev                  # App starten (Workers runtime lokaal)
```

### Database-first
Marc schetst het schema. Claude werkt het uit in Drizzle-schemadefinities.
Daarna: `bunx drizzle-kit generate` → `bun run db:migrate`.
Nooit code schrijven die een nog niet-bestaande tabel veronderstelt.

### Werkverdeling
- **Marc:** richting, datamodel, beslissingen
- **Claude:** uitvoering, code, ADRs schrijven, migraties uitwerken

### Code vs GitHub MCP
- **Code:** altijd lokaal wijzigen, nooit via GitHub MCP
- **ADRs en docs:** via GitHub MCP schrijven, daarna lokaal `git pull`
- **Conventions:** via GitHub MCP schrijven, daarna lokaal `git pull`

### Deployen
```bash
bun run db:migrate             # Migraties toepassen op Neon
wrangler deploy                # Worker deployen
```

---

## CLAUDE.md template voor nieuwe projecten

Maak bij elk nieuw project een `CLAUDE.md` aan met deze structuur:

```markdown
# [Projectnaam]

## Stack
Volgt de standaard mrterhorst-stack.
Zie github.com/mrterhorst/default-project-adrs voor alle ADRs en conventions.

## Projectspecifieke context
[Wat doet dit project? Wat is het domein?]

## Datamodel
[Korte beschrijving van de kernentiteiten en hun relaties]

## Afwijkingen van de standaardstack
[Alleen invullen als dit project bewust afwijkt van een ADR — inclusief reden]

## Lokaal opstarten
\`\`\`bash
docker compose up -d
wrangler dev
\`\`\`

## Deployen
\`\`\`bash
bun run db:migrate
wrangler deploy
\`\`\`
```
