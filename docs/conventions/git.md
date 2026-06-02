# Conventie: Git

## Commit messages

Formaat: `<type>: <beschrijving>` — in het Engels, lowercase, geen punt aan het einde.

| Type | Wanneer |
|---|---|
| `feat` | Nieuwe functionaliteit |
| `fix` | Bugfix |
| `chore` | Tooling, dependencies, configuratie |
| `refactor` | Code herstructurering zonder gedragswijziging |
| `docs` | Documentatie, ADRs, conventions |
| `test` | Tests toevoegen of aanpassen |
| `migration` | Database-migratie |

Voorbeelden:
```
feat: add invoice PDF export
fix: correct ULID generation on edge runtime
migration: add is_active column to user table
docs: add ADR-0015 API documentation
```

## Branches

- `main` — altijd deploybaar
- Feature-branches: `feat/<beschrijving>` — `feat/invoice-export`
- Bugfix-branches: `fix/<beschrijving>` — `fix/auth-token-expiry`
- Geen langlevende feature-branches — merge snel, gebruik feature flags indien nodig

## Werkwijze

- Commits zijn klein en afgerond — één logische wijziging per commit
- Geen "WIP" of "temp" commits op main
- Pull voor push op gedeelde branches
