# Conventie: Database

## Naamgeving

- Tabelnamen: **snake_case, enkelvoud** — `user`, `invoice_line`, `product`
- Kolomnamen: **snake_case** — `first_name`, `created_at`
- Foreign keys: `{tabel}_id` — `user_id`, `invoice_id`
- Booleans: prefix `is_` of `has_` — `is_active`, `has_verified_email`
- Join-tabellen: beide tabelnamen in alfabetische volgorde — `product_tag`

## Primary keys

ULID als primary key, gegenereerd op applicatieniveau:

```typescript
import { ulid } from 'ulid'

id: text('id').primaryKey().$defaultFn(() => ulid())
```

ULID is lexicografisch sorteerbaar op aanmaaktijd, URL-safe en 128-bit.
De collision-kans is verwaarloosbaar klein maar niet nul — acceptabel voor deze use cases.

## Verplichte kolommen

Elke tabel krijgt:

```typescript
created_at: timestamp('created_at').notNull().defaultNow(),
updated_at: timestamp('updated_at').notNull().defaultNow(),
```

`updated_at` wordt **handmatig bijgewerkt** in de applicatielaag (geen database-trigger).
Reden: Cloudflare Workers-compatibiliteit en expliciete controle over wanneer een update telt.

## Soft deletes

Nog te beslissen. Tot die tijd: geen soft deletes — rijen worden hard verwijderd.

## Migraties

- Schema-wijzigingen altijd via Drizzle Kit: `bunx drizzle-kit generate` → `bunx drizzle-kit migrate`
- Nooit direct de database aanpassen zonder bijbehorende migratie in de codebase
- Migraties worden gecommit en zijn onderdeel van de deploymentpipeline
