# ADR-0009: Validatie — Zod

## Status
Accepted

## Context
De projecten hebben een validatiebibliotheek nodig voor het valideren van invoer op systeemgrenzen: API-requests, formulierdata, environment variabelen en externe API-responses. De bibliotheek moet TypeScript-native zijn en goed integreren met Hono, Remix en Drizzle.

## Beslissing
Zod wordt gebruikt als schema-validatiebibliotheek. Zod definieert schemas als TypeScript-objecten en genereert daaruit automatisch TypeScript-types, zodat validatie en types altijd in sync zijn.

## Consequenties
**Positief:**
- Schema-first: één definitie levert zowel runtime-validatie als TypeScript-types op.
- Uitstekende integratie met Remix (form-validatie via `zod-form-data`), Hono (middleware) en Drizzle (via `drizzle-zod` voor schema-afleiding).
- Grote community en breed geadopteerd in het TypeScript-ecosysteem.
- Duidelijke foutmeldingen die direct bruikbaar zijn voor gebruikersfeedback.
- Composable: schemas zijn samen te stellen, uit te breiden en te transformeren.
- Geen runtime-afhankelijkheden buiten Zod zelf.

**Negatief:**
- Bundle-grootte is groter dan lichtgewichtere alternatieven zoals Valibot.
- Voor zeer complexe validaties kan de syntax uitgebreid worden.
- Parsing is synchroon — async validaties (bv. database-lookups) vereisen aparte afhandeling.

## Alternatieven overwogen
- **Valibot**: Functioneel vergelijkbaar met Zod maar kleinere bundle door tree-shaking. Jonger en kleinere community. Interessant als bundle-grootte kritiek wordt, maar valt nu af op ecosysteem-integraties.
- **Joi**: Populair maar niet TypeScript-native; types zijn een afterthought. Valt af op TypeScript-integratie.
- **Yup**: Vergelijkbaar met Zod maar langzamer en minder goede TypeScript-ondersteuning. Valt af op DX en performance.
- **TypeBox**: JSON Schema-gebaseerd, uitstekend voor OpenAPI-integratie. Complexer in gebruik voor gewone validatie-use-cases. Valt af op DX.
- **Geen bibliotheek (handmatige validatie)**: Volledige controle maar foutgevoelig, verbose en niet type-safe. Valt af op veiligheid en onderhoud.
