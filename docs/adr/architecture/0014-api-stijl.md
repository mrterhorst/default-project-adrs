# ADR-0014: API stijl — REST met Hono RPC client

**Scope:** api, frontend

## Status
Accepted

## Context
De Hono API wordt geconsumeerd door meerdere partijen: de eigen Remix frontend, een toekomstige mobile app en externe partijen. Er moet gekozen worden tussen REST en tRPC, en er moet bepaald worden hoe Remix data ophaalt — via directe DB-toegang of via de API.

## Beslissing
De API volgt REST-conventies (HTTP-methoden, statuscodes, JSON). Intern gebruikt de Remix frontend de **Hono RPC client** (`hono/client`) voor type-safe API-aanroepen zonder codegeneratie.

### REST als API-stijl
Hono implementeert de API als een standaard REST-interface. Alle consumers — mobile apps, externe partijen, Remix — spreken dezelfde HTTP/JSON API aan.

### Hono RPC client voor Remix
Hono exporteert het app-type, waarmee de Remix frontend een volledig type-safe client aanmaakt:

```typescript
// server: Hono route met typed input/output
const app = new Hono()
  .get('/users/:id', zValidator('param', userParamSchema), async (c) => {
    const { id } = c.req.valid('param')
    return c.json(await getUserById(id))
  })

export type AppType = typeof app
```

```typescript
// client: Remix loader met type-safe Hono client
import { hc } from 'hono/client'
import type { AppType } from '../server/app'

const client = hc<AppType>('http://localhost:3000')

export async function loader({ params }: LoaderArgs) {
  const res = await client.users[':id'].$get({ param: { id: params.id } })
  return res.json()
}
```

Geen codegeneratie, geen apart schema-formaat — de typen volgen direct uit de Hono-routedefinities.

### Remix leest via Hono, niet direct via Drizzle
Remix loaders en actions roepen de Hono API aan. Drizzle-toegang zit uitsluitend in de Hono-laag. Dit houdt de verantwoordelijkheden gescheiden: Hono is de API, Remix is de web-presentatielaag.

## Consequenties
**Positief:**
- Standaard REST: mobile apps en externe partijen kunnen de API aanspreken zonder speciale client of TypeScript-kennis.
- Hono RPC client geeft dezelfde type-safety als tRPC, maar dan boven op een gewone REST API.
- Eén API-implementatie bedient alle consumers — geen aparte internal/external API-laag.
- Remix loaders blijven dun: ze delegeren business logic aan Hono, niet aan Drizzle rechtstreeks.

**Negatief:**
- Remix loaders maken een HTTP-aanroep naar Hono, ook als beide op dezelfde runtime draaien. Dit is een extra roundtrip ten opzichte van directe DB-toegang. Op Cloudflare Workers (Service Bindings) is dit te vermijden als dat later nodig blijkt.
- De Hono RPC client is strikt gekoppeld aan het Hono app-type; type-safety werkt alleen vanuit TypeScript-consumers.

## Alternatieven overwogen
- **tRPC:** Uitstekende DX en end-to-end type-safety, maar vereist een tRPC-client. Externe partijen en mobile apps kunnen een tRPC API niet eenvoudig consumeren zonder TypeScript. Valt af op universele toegankelijkheid.
- **Remix loaders direct naar Drizzle:** Eenvoudiger voor pure web-apps, maar koppelt de frontend aan de datamodel-laag. Valt af zodra er ook een mobile app of externe API-consumer is.
- **GraphQL:** Flexibel voor complexe data-grafen, maar veel overhead (schema, resolvers, tooling) die niet past bij de "no fuzz"-filosofie. Valt af.
