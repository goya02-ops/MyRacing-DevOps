# AGENTS.md - MyRacing DevOps

Repo de **orquestación y docs**. NO es un monorepo: `MyRacing-Backend/` y
`MyRacing-Frontend/` son clones locales de repos GitHub independientes
(ramas, CI, deploy propios) y no se versionan acá.

## Verificación por sub-proyecto

| Sub-proyecto | Verificación |
| ------------ | ------------ |
| `MyRacing-Backend` | `pnpm build` (no hay script `test`; CI corre `--if-present`, hoy no-op) |
| `MyRacing-Frontend` | `pnpm lint && pnpm build && pnpm test` |

Ambos usan **pnpm 10** y **Node 20+**. Instalar con `pnpm install` dentro de
cada carpeta (son proyectos pnpm separados, no workspace hoisting desde la
raíz).

## Base de datos

- Dev MySQL 8: `docker compose -f compose.dev.yaml up -d` → puerto **3307**
  (CI usa 3306; no mezclar). DB `myracing`, user `admin` / `MiR@cing_2025!`.
- El backend crea/sincroniza el esquema solo al arrancar (MikroORM
  `syncSchema()`, sin migraciones). No apuntar `pnpm dev`/`start` a una DB
  productiva.

## Frontend

- La API se configura con `VITE_API_BASE_URL` en `.env.local` (default
  `http://localhost:3000/api`). Ojo: el README raiz dice `.env`/`.env.example`,
  el código usa `import.meta.env.VITE_API_BASE_URL`.
- Vitest está configurado dentro de `vite.config.ts` (happy-dom, setup en
  `src/test/setup.ts`, tests colocados como `src/**/*.{test,spec}.*`).
- Scripts: `pnpm test`, `pnpm test:watch`, `pnpm test:coverage`.

## Metodología de trabajo

- Cada issue de GitHub = una tarea chica (1-4 hs), asignada a un integrante.
- Rama por issue: `feature/<id-tema>` → PR a `develop` → PR a `main` (+tag).
- Todo el CI/CD es **automático, sin aprobación humana** (branch protection
  sin reviewers: los gates deciden).
- **Gates en `develop`**: solo se exige que pasen los tests
  (frontend: Lint/Build/Unit tests/E2E; backend: Build/Tests).
  No se exige coverage ni Quality Gate. **No se mergea a `develop`
  si fallan los tests.**
- **Gates en `main`**: se exige además **cobertura TOTAL ≥80%**
  (job `Coverage` con thresholds de vitest, solo corre en PRs/pushes a `main`),
  el check `SonarCloud Code Analysis` y el job **`Quality Gate (proyecto)`**
  (consulta el QG del proyecto completo vía API, no solo el código nuevo del PR).
  Todo como checks requeridos en la branch protection, **sin aprobación humana**.
- **Deploy a producción**: frontend lo publica Vercel automáticamente al mergear
  a `main` (integración Git, sin cambios en el dashboard); como a `main` solo
  entra código con todos los gates verdes, el auto-deploy siempre es seguro.
  Backend lo despliega el workflow `Deploy` a Railway (con su propio guard de QG).
  **Sin gates verdes no hay merge a `main`, y sin merge no hay deploy.**
- Commits atómicos por paso lógico; mensaje convencional
  (`feat:`/`fix:`/`test:`/`refactor:`/`chore:`).

## SonarCloud (Quality Gate)

- El análisis corre en cada push/PR vía la GitHub App `SonarQubeCloud`.
- **No se mergea a `main` si falla el Quality Gate** (Security Rating,
  Reliability, Maintainability, Duplicación). `develop` admite merge
  aunque falle el Quality Gate, para no frenar el trabajo diario.
- El token compartido del equipo está en `.env` de cada repo como
  `SONAR_TOKEN` (archivo ignorado por git). Cada agente puede consultar
  el Quality Gate vía la API de SonarCloud. Recomendado generar uno
  propio cuando se pueda.
- Estado actual (oct 2026): `main` falla el Quality Gate por
  `new_security_rating = 5` (issue pendiente).

## Documentos de detalle

- `MyRacing-Backend/AGENTS.md` — ESM con imports `.js`, entidades
  `*.entity.ts`, RequestContext/EM, secretos JWT por defecto inseguros, etc.
- `VARIABLES-ENTORNO-MERCADOPAGO.md` — pasarela de pagos.
- `docs/politica-git/` — Gitflow simplificado: `feature/*` → PR a `develop`
  → PR a `main` (+tag).
