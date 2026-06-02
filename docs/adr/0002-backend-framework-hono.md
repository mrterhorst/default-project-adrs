# ADR-0002: Backend Framework — Hono

## Status
Accepted

## Context
De projecten hebben een HTTP-framework nodig voor het bouwen van API's. Het framework moet aansluiten op de filosofie van "no fuzz, dicht bij web standards" en zowel lokaal met Bun als op Cloudflare Workers kunnen draaien zonder aanpassingen.

## Beslissing
Hono wordt gebruikt als backend HTTP-framework. Hono is een minimaal, snel framework dat de standaard Web API's (Request/Response) gebruikt en multi-runtime is: het draait op Bun, Cloudflare Workers, Deno en Node.js.

## Consequenties
**Positief:**
- Multi-runtime: dezelfde code draait lokaal op Bun en op de edge via Cloudflare Workers.
- Gebouwd op Web Standards (Fetch API, Request, Response) — geen vendor lock-in op framework-abstracties.
- Zeer lichtgewicht en snel; minimale overhead.
- Goede TypeScript-ondersteuning en type-safe routing met RPC-modus.
- Middleware-ecosysteem voor veelgebruikte patronen (CORS, auth, logging).

**Negatief:**
- Kleiner ecosysteem dan Express of Fastify; minder community plugins.
- Minder "batteries included" dan NestJS of AdonisJS — bewust, maar vereist meer eigen keuzes.
- Relatief jong framework; API kan nog evolueren.

## Alternatieven overwogen
- **Express**: Groot ecosysteem, maar oud design, geen native TypeScript, niet multi-runtime. Valt af op moderniteit en edge-compatibiliteit.
- **Fastify**: Sneller dan Express, goede TypeScript-support, maar niet bedoeld voor edge-deployment. Valt af op multi-runtime vereiste.
- **NestJS**: Volledig opinionated framework met decorators en DI-container. Te zwaar en te ver van web standards. Valt af op complexiteit en filosofie.
- **Elysia**: Bun-native en snel, maar alleen voor Bun — geen Cloudflare Workers-support. Valt af op multi-runtime vereiste.
