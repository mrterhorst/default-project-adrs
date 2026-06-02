# ADR-0018: Observability — Sentry + structured logging

## Status
Accepted

## Context
In productie moet zichtbaar zijn wanneer er fouten optreden en wat de context was. De stack draait op Cloudflare Workers, wat beperkingen stelt aan tooling (geen Node.js-native agents, geen langlopende processen).

## Beslissing

### Error tracking: Sentry
Sentry wordt gebruikt voor error tracking via `@sentry/cloudflare`. Sentry vangt onafgehandelde uitzonderingen op, inclusief stacktrace, request-context en gebruikerscontext.

Initialisatie als Hono-middleware:
```typescript
import * as Sentry from '@sentry/cloudflare'

export default Sentry.withSentry(
  (env) => ({ dsn: env.SENTRY_DSN, tracesSampleRate: 0.1 }),
  app.fetch
)
```

Fouten die bewust worden afgehandeld worden handmatig gerapporteerd waar relevant:
```typescript
Sentry.captureException(error, { extra: { userId, action } })
```

### Structured logging: JSON via console
Alle log-output wordt als JSON geschreven via `console.log`. Cloudflare vangt dit op in het Workers-dashboard (realtime) en via Logpush (archief):

```typescript
console.log(JSON.stringify({
  level: 'info',
  message: 'User created',
  userId,
  duration: Date.now() - startTime,
}))
```

Loglevels: `debug` | `info` | `warn` | `error`. In productie alleen `info` en hoger.

Een dunne `lib/logger.ts` wrapper zorgt voor consistente structuur:
```typescript
// lib/logger.ts
export const logger = {
  info: (message: string, data?: object) =>
    console.log(JSON.stringify({ level: 'info', message, ...data })),
  error: (message: string, data?: object) =>
    console.error(JSON.stringify({ level: 'error', message, ...data })),
}
```

### Buiten scope
- Performance monitoring (APM) — niet nodig op dit schaalniveau; Cloudflare Dashboard toont basic metrics.
- Uptime monitoring — Cloudflare biedt dit zelf via Health Checks. Geen extra service.
- Log aggregatie naar externe systemen — optioneel via Cloudflare Logpush naar R2 als dat later nodig blijkt.

## Consequenties
**Positief:**
- Sentry gratis tier (5.000 events/maand) is ruim voor persoonlijke en vroege-fase projecten.
- `@sentry/cloudflare` is officieel ondersteund en werkt zonder aanpassingen op Workers.
- JSON logging via `console.log` is zero-overhead en direct beschikbaar in het Cloudflare-dashboard.
- Geen extra service of infrastructuur nodig voor de basisbehoefte.

**Negatief:**
- Sentry free tier heeft retentielimiet van 30 dagen en geen geavanceerde filters.
- `console.log` logs in Workers zijn vluchtig — zonder Logpush zijn ze niet persistent opgeslagen.
- Bij groei (> 5k errors/maand) moet Sentry worden geüpgraded of vervangen.

## Alternatieven overwogen
- **Cloudflare Workers Observability (beta):** Native Cloudflare oplossing, maar nog beperkt in features vergeleken met Sentry. Kan later het Sentry-abonnement vervangen.
- **Highlight.io:** Open source, goed Workers-support, maar kleinere community. Alternatief als Sentry niet past.
- **Axiom / Logtail:** Krachtige log-aggregatie, maar voegt een betaalde service toe die niet nodig is in de beginfase. Kan via Logpush later worden aangesloten.
