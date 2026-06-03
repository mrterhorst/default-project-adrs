# ADR-0017: Vendor abstractiepatroon en queues

**Scope:** api, infra, queues

## Status
Accepted

## Context
Een aantal diensten in de stack zijn vendor-specifiek en gevoelig voor lock-in: object storage (R2), e-mailverzending en message queues. Als de rest van de codebase direct tegen vendor-APIs praat, is wisselen van provider een ingrijpende operatie. Tegelijk is Cloudflare Queues de logische keuze als queue-implementatie binnen het Cloudflare Workers-ecosysteem.

## Beslissing

### Abstractielaag voor vendor-gevoelige diensten
Elke dienst waarbij de implementatie kan wisselen, krijgt een dunne abstractielaag in `lib/`:

```
lib/
  queue.ts    →  publish(queue, payload), consume(handler)
  storage.ts  →  upload(key, data), download(key), delete(key)
  email.ts    →  send({ to, subject, body })
```

De rest van de applicatiecode importeert uitsluitend uit `lib/` — nooit direct vanuit een vendor-SDK (`cloudflare:workers`, `@aws-sdk/...`, e.d.). De implementatie achter `lib/` is inwisselbaar zonder aanpassing aan de rest van de code.

### Queue-implementatie: Cloudflare Queues
De queue-implementatie in `lib/queue.ts` gebruikt Cloudflare Queues. Queues worden gebruikt voor **interne asynchrone verwerking**:
- E-mail verzenden na een gebruikersactie
- Achtergrondtaken (PDF-generatie, imports)
- Snelle acceptatie van inkomende webhooks, vertraagde verwerking

### Buiten de basisstack
Pub/sub-systemen, workflow-engines en message buses zijn altijd project-specifiek. Ze worden niet in deze basisstack opgenomen en worden beschreven in project-specifieke ADRs wanneer ze nodig zijn.

## Consequenties
**Positief:**
- De applicatiecode kent geen vendor-details — `lib/storage.ts` werkt of het nu R2, S3 of lokaal bestandssysteem onder zit.
- Wisselen van provider (bijv. R2 → S3) is een wijziging van één bestand, niet een refactor door de hele codebase.
- Testbaarheid: de `lib/`-interface is eenvoudig te mocken in tests.
- Duidelijke grens: wat in `lib/` zit is vendor-detail, wat erbuiten zit is applicatielogica.

**Negatief:**
- De abstractielaag is een dunne wrapper — te veel abstractie vermijden. De interface modelleert de gemeenschappelijke deler, niet de rijkste feature-set van één vendor.
- Vendor-specifieke features (bijv. R2 multipart upload) moeten bewust worden blootgesteld of omzeild.

## Alternatieven overwogen
- **Direct tegen vendor-API praten:** Eenvoudiger op korte termijn, maar elke provider-wissel is een grote refactor. Valt af op onderhoudbaarheid.
- **Zware abstractiebibliotheek (bijv. BullMQ, AWS SDK wrappers):** Voegt afhankelijkheden toe die niet passen bij de edge-runtime. Valt af op bundle-grootte en compatibiliteit.
