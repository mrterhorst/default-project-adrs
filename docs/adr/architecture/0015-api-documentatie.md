# ADR-0015: API documentatie — OpenAPI via @hono/zod-openapi + Scalar

**Scope:** api

## Status
Accepted

## Context
De API heeft externe consumers en een toekomstige mobile app (zie ADR-0014). Die consumers hebben documentatie nodig. Handmatig bijgehouden API-documentatie raakt snel verouderd; documentatie gegenereerd vanuit de code niet.

## Beslissing
OpenAPI 3.x documentatie wordt automatisch gegenereerd vanuit de Hono-routedefinities via `@hono/zod-openapi`. De documentatie is publiek beschikbaar op `/docs` via Scalar UI.

### Hoe het werkt

Routes worden gedefinieerd met `createRoute` in plaats van de standaard `.get()` / `.post()`:

```typescript
import { createRoute, OpenAPIHono, z } from '@hono/zod-openapi'

const app = new OpenAPIHono()

const getUserRoute = createRoute({
  method: 'get',
  path: '/user/{id}',
  request: {
    params: z.object({ id: z.string() }),
  },
  responses: {
    200: {
      content: { 'application/json': { schema: UserSchema } },
      description: 'User gevonden',
    },
    404: { description: 'Niet gevonden' },
  },
})

app.openapi(getUserRoute, async (c) => {
  const { id } = c.req.valid('param')
  // handler
})
```

De OpenAPI-spec en de Scalar UI worden toegevoegd via:

```typescript
app.doc('/openapi.json', { openapi: '3.0.0', info: { title: 'API', version: '1.0.0' } })
app.get('/docs', scalarReference({ url: '/openapi.json' }))
```

### Zod-schema's als single source of truth
Dezelfde Zod-schema's die runtime-validatie doen (zie ADR-0009) genereren de OpenAPI-spec. Er is geen aparte spec-file of codegenerator — de documentatie volgt automatisch de code.

### Hono RPC client
De Hono RPC client (zie ADR-0014) werkt ongewijzigd naast `@hono/zod-openapi`. Interne TypeScript-consumers gebruiken de typed client; externe consumers gebruiken de OpenAPI-spec.

## Consequenties
**Positief:**
- Documentatie is altijd in sync met de implementatie — geen handmatig onderhoud.
- Scalar UI is modern, snel en gratis. Geen extra infrastructuur nodig.
- Geen overhead voor Cloudflare Workers: Scalar UI laadt via CDN, spec-generatie is lichtgewicht.
- Externe consumers en mobile-ontwikkelaars hebben direct een bruikbare referentie.

**Negatief:**
- Routes definiëren via `createRoute` is iets meer boilerplate dan standaard Hono-routes.
- `OpenAPIHono` vervangt de standaard `Hono` instantie — kleine aanpassing in de app-setup.

## Alternatieven overwogen
- **Handmatige OpenAPI-spec (YAML/JSON):** Volledig flexibel, maar raakt onvermijdelijk verouderd. Valt af.
- **Geen documentatie:** Niet houdbaar zodra externe consumers of een mobile app aanhaken. Valt af.
- **Swagger UI:** Functioneel, maar Scalar is moderner, sneller en beter leesbaar. Valt af op DX.
- **tRPC met typed client only:** Geen OpenAPI-output, sluit externe consumers buiten. Valt af (zie ADR-0014).
