# Conventie: TypeScript

## Naamgeving

- Variabelen en functies: **camelCase** — `userId`, `getInvoiceById`
- Types en interfaces: **PascalCase** — `UserProfile`, `InvoiceLineItem`
- Constanten (compile-time vast): **SCREAMING_SNAKE_CASE** — `MAX_RETRY_COUNT`
- Bestanden: **kebab-case** — `user-service.ts`, `invoice-line.ts`
- Zod-schema's: suffix `Schema` — `userSchema`, `createInvoiceSchema`
- Drizzle-tabellen: camelCase, enkelvoud — `user`, `invoiceLine`

## Stijl

- Altijd expliciete return types op geëxporteerde functies
- Geen `any` — gebruik `unknown` als type onbekend is en vereng daarna
- Prefer `type` boven `interface` tenzij declaration merging nodig is
- Geen barrel-exports (`index.ts` die alles herexporteert) — importeer direct
- `async/await` boven `.then()` chains

## Bestandsstructuur

Plat over gelaagd: geen diepe mappenstructuur voor de applicatielaag.
Groepeer op feature, niet op technische laag (`/user/` boven `/controllers/`, `/services/`).
