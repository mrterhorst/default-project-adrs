# ADR-0001: Runtime — Bun

## Status
Accepted

## Context
De projecten binnen mrterhorst hadden een JavaScript-runtime, bundler en packagemanager nodig. Traditioneel werden Node.js, webpack/Vite en npm/yarn als losse tools gecombineerd. Dit leidt tot complexe configuraties, trage builds en veel tooling-overhead.

## Beslissing
Bun wordt gebruikt als all-in-one runtime, bundler, test-runner en packagemanager. Op de edge (Cloudflare Workers) vervangt de Workers-runtime Bun, maar de rest van de toolchain blijft hetzelfde.

## Consequenties
**Positief:**
- Drastisch snellere installaties, builds en testsuites vergeleken met Node.js + npm.
- Één tool vervangt Node.js, webpack, Babel, npm en Jest.
- Native TypeScript-ondersteuning zonder extra transpilatiestap.
- `bun run`, `bun build`, `bun test` en `bun install` werken out-of-the-box.

**Negatief:**
- Bun is minder volwassen dan Node.js; sommige Node.js-compatibiliteitsedges bestaan nog.
- Deployment naar serveromgevingen vereist Bun-ondersteuning of een Docker-image.
- Op de edge (Cloudflare Workers) is Bun niet de daadwerkelijke runtime — Hono abstraheert dit weg.

## Alternatieven overwogen
- **Node.js + Vite + npm**: Bewezen en breed ondersteund, maar veel meer losse tooling en significant trager. Valt af op complexiteit en overhead.
- **Deno**: Vergelijkbare filosofie als Bun, maar kleinere ecosysteem en minder npm-compatibiliteit. Valt af op ecosysteem volwassenheid.
