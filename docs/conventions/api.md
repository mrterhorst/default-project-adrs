# Conventie: API (REST)

## URL-structuur

- Kebab-case, enkelvoud resource-naam: `/user`, `/invoice-line`
- Nesting maximaal één niveau diep: `/user/:id/invoice` — niet dieper
- Geen werkwoorden in URLs — gebruik HTTP-methoden

## HTTP-methoden

| Actie | Methode | Voorbeeld |
|---|---|---|
| Lijst ophalen | `GET` | `GET /user` |
| Enkel item | `GET` | `GET /user/:id` |
| Aanmaken | `POST` | `POST /user` |
| Volledig vervangen | `PUT` | `PUT /user/:id` |
| Gedeeltelijk updaten | `PATCH` | `PATCH /user/:id` |
| Verwijderen | `DELETE` | `DELETE /user/:id` |

## Response-formaat

```json
// Succes (enkel item)
{ "data": { ... } }

// Succes (lijst)
{ "data": [ ... ], "meta": { "total": 42 } }

// Fout
{ "error": { "code": "NOT_FOUND", "message": "User not found" } }
```

## Statuscodes

- `200` — succes (GET, PATCH, PUT)
- `201` — aangemaakt (POST)
- `204` — geen inhoud (DELETE)
- `400` — validatiefout (Zod)
- `401` — niet geauthenticeerd
- `403` — geen toegang
- `404` — niet gevonden
- `500` — serverfout

## Validatie

Alle input wordt gevalideerd met Zod via `zValidator` (Hono middleware).
Geen validatie weglaten — ook interne endpoints valideren input.
