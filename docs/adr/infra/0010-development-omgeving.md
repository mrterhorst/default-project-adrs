# ADR-0010: Development omgeving

**Scope:** tooling, infra

## Status
Accepted

## Context
Projecten worden ontwikkeld op een Windows-machine. Voor moderne webdevelopment met Unix-tools, Bun en Docker is een geschikte lokale omgeving nodig die aansluit bij de productie-runtime (Linux) zonder elke keer te deployen om iets te testen.

## Beslissing
De development omgeving bestaat uit de volgende lagen:

**OS-laag:** WSL2 (Windows Subsystem for Linux) met een Ubuntu-distributie als primaire development-omgeving. Alle projectcode, Git-repositories en tooling leven volledig in het WSL-bestandssysteem (`/home/marc/...`).

**Editor:** VS Code op Windows met de Remote - WSL extensie. VS Code draait op Windows maar verbindt via de extensie direct met de WSL-omgeving; bestanden, terminal en extensies werken alsof je op Linux zit.

**Runtime:** Bun geïnstalleerd in WSL. Zie ADR-0001.

**Dev-server:** De applicatie draait lokaal via `bun dev` met hot-reload. Geen container voor de app zelf — directe Bun-uitvoering geeft de snelste feedback loop.

**Database:** PostgreSQL draait in een Docker-container voor isolatie. Geen lokale PostgreSQL-installatie — Docker houdt de host schoon en maakt het eenvoudig de database te resetten of te vervangen. Uitgewerkt in ADR-0012.

**Standaard VS Code extensies per project:**
- ESLint + Prettier — linting en formatteren
- Tailwind CSS IntelliSense — autocomplete voor Tailwind classes
- Drizzle-gerelateerde tooling — schema-inzicht en migraties

## Consequenties
**Positief:**
- Code in WSL-bestandssysteem geeft optimale I/O-performance voor Bun en Git; `/mnt/c/` cross-filesystem toegang is significant trager.
- Remote - WSL geeft een native Linux-ervaring in VS Code zonder een aparte VM of dual-boot.
- `bun dev` met hot-reload is aanzienlijk sneller dan een containerized app-setup.
- Docker voor de database isoleert dependencies; geen conflicterende PostgreSQL-versies op de host.
- Dicht bij productie: WSL2 + Linux-runtime in Docker elimineert "werkt op mijn machine"-discrepanties.

**Negatief:**
- WSL2 heeft een eigen netwerkstack; poorten zijn beschikbaar via `localhost` maar soms zijn extra stappen nodig voor externe toegang vanaf het netwerk.
- Twee bestandssystemen (Windows + WSL) kunnen verwarring geven; bewust kiezen voor WSL-kant voorkomt dit.
- Docker Desktop op Windows verbruikt extra geheugen; WSL-backend is efficiënter dan Hyper-V-backend.

## Alternatieven overwogen
- **Volledig native Windows:** Geen goede Unix-tooling, afwijkende padconventies, Bun-ondersteuning op Windows is secundair. Valt af.
- **Code op Windows-kant (`/mnt/c/...`):** Toegankelijk vanuit Windows Explorer en WSL, maar bestandssysteem-crossing maakt I/O tot 10× trager voor operaties zoals `bun install`. Valt af.
- **Volledige Docker Compose stack (app + DB):** Maximale isolatie, maar hot-reload via volume-mounts in Docker is merkbaar trager dan native Bun. Past niet bij de snelle iteratie-filosofie van de stack. Valt af voor de app-laag.
