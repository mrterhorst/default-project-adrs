# ADR-0003: Frontend Framework — Remix + React

## Status
Accepted

## Context
De projecten hebben een full-stack web framework nodig dat server-side rendering ondersteunt, dicht bij web standards werkt en goed integreert met de rest van de stack (Bun, Hono, TypeScript). Het framework moet de kloof tussen server en client overbruggen zonder onnodige abstracties.

## Beslissing
Remix (met React) wordt gebruikt als full-stack frontend framework. Remix werkt met standaard Web API's (Request, Response, FormData), heeft een duidelijk data-laagmodel via loaders en actions, en integreert goed met Bun en Cloudflare Workers.

## Consequenties
**Positief:**
- Gebouwd op Web Standards: forms, fetch, HTTP — kennis is overdraagbaar.
- Server-side rendering en streaming out-of-the-box.
- Loaders en actions geven een helder model voor data-fetching en mutaties zonder aparte state-management libraries.
- Goede foutafhandeling en nested routing.
- Kan worden gedeployd op Cloudflare Workers (via de Cloudflare adapter).
- TypeScript-first.

**Negatief:**
- Leercurve voor developers die gewend zijn aan client-side SPA-patronen.
- React als UI-laag brengt de volledige React-overhead mee.
- Minder flexibel dan een puur client-side aanpak als de UI volledig dynamisch moet zijn.
- Community kleiner dan Next.js.

## Alternatieven overwogen
- **Next.js**: Marktleider, groot ecosysteem, maar sterker gebonden aan Vercel. Server Components zijn complex en App Router voegt cognitieve last toe. Valt af op complexiteit en vendor-nabijheid.
- **SvelteKit**: Eleganter en lichter dan React, maar kleiner ecosysteem en minder component-libraries beschikbaar. Valt af op ecosysteem en beschikbaarheid van shadcn/ui (React-only).
- **Astro**: Uitstekend voor content-sites, maar minder geschikt als full-stack applicatieframework met rijke interactiviteit. Valt af op use case mismatch.
- **TanStack Start**: Veelbelovend maar nog in vroeg stadium. Valt af op volwassenheid.
