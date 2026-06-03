# ADR-0004: CSS Framework — Tailwind CSS

**Scope:** frontend, css

## Status
Accepted

## Context
De projecten hebben een aanpak nodig voor het schrijven van CSS die schaalbaar is, goed werkt met component-gebaseerde UI's en geen grote runtime-overhead introduceert. De keuze moet aansluiten op de component-library keuze (shadcn/ui).

## Beslissing
Tailwind CSS wordt gebruikt als utility-first CSS framework. Tailwind genereert alleen de klassen die daadwerkelijk gebruikt worden, werkt goed met React-componenten en is de basis waarop shadcn/ui gebouwd is.

## Consequenties
**Positief:**
- Utility-first aanpak houdt stijlen dicht bij de markup — geen context-switching naar losse CSS-bestanden.
- Geen ongebruikte CSS in productie (tree-shaking via content-scanning).
- Uitstekende integratie met shadcn/ui (vereist Tailwind).
- Design tokens (kleuren, spacing, typografie) centraal instelbaar via `tailwind.config`.
- Grote community en uitstekende tooling (IntelliSense plugin voor VS Code).

**Negatief:**
- HTML/JSX kan druk worden met veel utility-klassen.
- Designsysteem-kennis vertaalt zich naar Tailwind-syntax, niet naar standaard CSS-concepten.
- Upgrade-paden tussen Tailwind-versies kunnen breaking changes bevatten.

## Alternatieven overwogen
- **CSS Modules**: Goede scoping, dicht bij standaard CSS, maar geen design system out-of-the-box en geen integratie met shadcn/ui. Valt af op ecosysteem-fit.
- **Styled Components / Emotion**: CSS-in-JS met runtime overhead en complexiteit rond SSR. Valt af op performance en complexiteit.
- **UnoCSS**: Vergelijkbaar met Tailwind maar met een pluggable engine. Interessant, maar minder volwassen ecosysteem en shadcn/ui is specifiek op Tailwind gebouwd. Valt af op compatibiliteit.
- **Vanilla CSS**: Maximale controle, maar geen gedeeld design system en veel meer handmatig werk. Valt af op productiviteit.
