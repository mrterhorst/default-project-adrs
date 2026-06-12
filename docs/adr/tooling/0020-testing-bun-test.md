# ADR-0020: Testing met bun:test

**Scope:** tooling, testing

## Status
Accepted

## Context
Bun is de vastgelegde runtime (ADR-0001). Er was nog geen beslissing over een testrunner; de eerste tests (Zod-schema's, promptcompositie, pure helpers) zijn in Battuto geschreven en vroegen om een keuze. Bun levert een ingebouwde testrunner (`bun test`) die tsconfig-paths leest (path-aliases werken dus ook in tests) en geen configuratie vereist.

## Beslissing

### Testrunner: `bun:test`
Unit tests gebruiken de ingebouwde runner van Bun. Geen Jest, geen Vitest, geen extra dependencies of config.

- Imports uit `bun:test` (`describe`, `test`, `expect`).
- Standaardcommando: `bun run test` (script wijst naar `bun test`).
- Alleen de basis-API gebruiken — geen runner-specifieke features, zodat een eventuele migratie naar Vitest triviaal blijft.

### Colocated testbestanden
Testbestanden staan naast de bron (`system-prompt.test.ts` naast `system-prompt.ts`), niet in een aparte `tests/`-map. De test hoort bij de module en verhuist mee. `bun test` vindt `*.test.ts` overal behalve `node_modules`.

### Echte input boven gecaste objecten
Tests bouwen de echte inputvorm op die de code parst (bijv. `FormData` voor zfd-schema's) in plaats van objecten te casten — de test dekt dan ook de parselaag zelf.

### Buiten deze beslissing
Route-/integratietests, AI-call-mocks, E2E (Playwright) en coverage-eisen zijn projectspecifieke vervolgbeslissingen. Zodra route- of externe-call-tests nodig zijn, is dát het moment om een mock-strategie te kiezen.

## Consequenties
**Positief:**
- Nul extra dependencies en nul configuratie; de runner komt mee met de al gekozen runtime.
- Snel: Bun draait tests zonder aparte transpile-stap.
- Colocatie houdt tests vindbaar en mee-verhuizend bij refactors.

**Negatief:**
- `bun:test` wijkt op details af van Jest/Vitest (mock-API, snapshot-gedrag). Mitigatie: alleen de basis-API gebruiken.
- Ecosysteem rond bun:test (plugins, reporters) is kleiner dan dat van Vitest.

## Alternatieven overwogen
- **Vitest:** rijkere mock- en plugin-API, maar voegt een dependency-boom en configuratie toe die voor pure unit tests niets oplevert. Valt af op eenvoud.
- **Jest:** de facto standaard, maar traag onder ESM/TypeScript en past niet bij de Bun-runtime. Valt af.
