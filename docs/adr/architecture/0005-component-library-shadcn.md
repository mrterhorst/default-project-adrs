# ADR-0005: Component Library — shadcn/ui

## Status
Accepted

## Context
De projecten hebben herbruikbare UI-componenten nodig (buttons, dialogs, forms, etc.) die toegankelijk zijn, stylen via Tailwind en volledig in eigen beheer zijn. De keuze moet passen bij React en Tailwind CSS.

## Beslissing
shadcn/ui wordt gebruikt als component-basis. shadcn/ui is geen npm-pakket maar een collectie componenten die via de CLI naar de eigen codebase worden gekopieerd. De componenten zijn gebouwd op Radix UI (accessibility primitives) en gestyled met Tailwind CSS.

## Consequenties
**Positief:**
- Volledige eigenaarschap: componenten leven in de eigen repo en zijn volledig aanpasbaar.
- Geen versie-lock op een externe component-library; updates zijn opt-in per component.
- Gebouwd op Radix UI — solide accessibility-basis (keyboard navigatie, ARIA, focus management).
- Tailwind-native — past naadloos in de bestaande CSS-aanpak.
- Grote en actieve community; veel voorbeelden en extensies beschikbaar.

**Negatief:**
- Componenten moeten handmatig geüpdatet worden als shadcn/ui wijzigingen uitbrengt.
- De initiële setup kost meer tijd dan het installeren van een kant-en-klare component-library.
- Aanpassingen in gekopieerde componenten kunnen conflicteren met toekomstige upstream-wijzigingen.

## Alternatieven overwogen
- **MUI (Material UI)**: Compleet en volwassen, maar opinionated Material Design-stijl en grote bundle. Moeilijk te restylen richting eigen design. Valt af op flexibiliteit en bundle-grootte.
- **Chakra UI**: Goede DX en toegankelijkheid, maar CSS-in-JS runtime en minder Tailwind-integratie. Valt af op CSS-aanpak mismatch.
- **Radix UI (primitives only)**: Volledig unstyled — maximale flexibiliteit maar meer eigen styling-werk. shadcn/ui is feitelijk een laag bovenop Radix; directe Radix-gebruik valt af op productiviteit.
- **Headless UI (Tailwind Labs)**: Vergelijkbaar concept maar minder componenten en minder actieve ontwikkeling. Valt af op completheid.
