# ADR-0021: E-mailprovider Resend

**Scope:** infra, e-mail

## Status
Accepted

## Context
ADR-0017 legt vast dát e-mailverzending achter een abstractie in `lib/email.ts` zit, maar niet wélke provider eronder hangt. De stack draait op edge-runtimes (Cloudflare Workers), dus de provider moet aanspreekbaar zijn zonder Node-specifieke SDK's (geen nodemailer/SMTP-sockets). Battuto gebruikt e-mail al voor auth-flows (e-mail-OTP) en loopt met `lib/email.ts` op deze beslissing vooruit.

## Beslissing

### Provider: Resend
E-mailverzending gaat via de Resend HTTP-API.

- **Kale `fetch`, geen SDK:** de API is één POST naar `https://api.resend.com/emails` met een Bearer-token. Dat werkt overal waar `fetch` bestaat, ook op Cloudflare Workers, en houdt de dependency-boom leeg.
- **Configuratie via env:** `RESEND_API_KEY` (verplicht voor echt verzenden) en `EMAIL_FROM` (afzender; zonder eigen domein is Resends `onboarding@resend.dev` bruikbaar in dev).
- **Dev-transport zonder key:** ontbreekt `RESEND_API_KEY`, dan logt `lib/email.ts` de volledige mail naar de console in plaats van te verzenden. Flows als e-mail-OTP zijn daarmee lokaal testbaar zonder account of netwerk.
- **Geen response-body in foutmeldingen:** bij een mislukte call wordt alleen de statuscode doorgegeven — de body kan adresgegevens bevatten.

### Interface blijft ADR-0017
Feature-code importeert uitsluitend `sendEmail({ to, subject, body })` uit `lib/email.ts` en kent geen Resend-details. Wisselen van provider is een wijziging van één bestand.

## Consequenties
**Positief:**
- Edge-compatibel zonder polyfills of SDK; één fetch-call.
- Lokaal ontwikkelen vereist geen mailaccount — het dev-transport logt.
- Ruime gratis tier (3.000 mails/maand) dekt hobby- en dev-gebruik.

**Negatief:**
- Plain-text only in de huidige interface; HTML-templates zijn een bewuste uitbreiding van de abstractie wanneer nodig.
- Afzenderdomein moet bij Resend geverifieerd worden voor productie-gebruik.

## Alternatieven overwogen
- **SMTP (nodemailer):** werkt niet op edge-runtimes (TCP-sockets). Valt af.
- **AWS SES:** goedkoopst bij volume, maar vereist SDK of request-signing (SigV4) — onnodig zwaar voor deze schaal. Valt af op eenvoud.
- **Postmark / Mailgun:** vergelijkbare HTTP-API's, maar krappere gratis tiers en geen voordeel boven Resend. Vallen af.
