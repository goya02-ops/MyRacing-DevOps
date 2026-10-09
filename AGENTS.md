# AGENTS.md - MyRacing DevOps

Repo de **orquestación y docs**. NO es un monorepo: `MyRacing-Backend/` y
`MyRacing-Frontend/` son clones locales de repos GitHub independientes
(ramas, CI, deploy propios), ignorados por git acá (ver `.gitignore`).

## Setup y verificación

- **pnpm 10, Node 22** (CI fija `node-version: 22` en ambos repos).
  `pnpm install` dentro de cada carpeta; proyectos pnpm separados, sin
  workspace hoisting desde la raíz.

| Sub-proyecto | Verificación |
| ------------ | ------------ |
| `MyRacing-Backend` | `pnpm build && pnpm test` (Vitest; `pnpm test:coverage` solo para gate de `main`) |
| `MyRacing-Frontend` | `pnpm lint && pnpm build && pnpm test` (`lint` es `eslint . --max-warnings 0`; E2E: `pnpm test:e2e`) |

## Base de datos

- Dev MySQL 8: `docker compose -f compose.dev.yaml up -d` → puerto **3307**
  (CI usa **3306** con service container; no mezclar). DB `myracing`,
  user `admin` / `MiR@cing_2025!`.
- El backend crea/sincroniza el esquema solo al arrancar (MikroORM
  `syncSchema()`, sin migraciones). No apuntar `pnpm dev`/`start` a una DB
  productiva. Los tests del backend requieren MySQL alcanzable
  (`src/shared/orm.ts` conecta al importar).

## Frontend

- La API se configura con `VITE_API_BASE_URL` en `.env.local` (default
  `http://localhost:3000/api`). El código usa
  `import.meta.env.VITE_API_BASE_URL`.
- Vitest vive en `vite.config.ts` (happy-dom, setup en `src/test/setup.ts`,
  tests en `src/**/*.{test,spec}.*`, coverage v8 con thresholds 80).
- E2E (Playwright, job full stack con backend real): instalar navegadores con
  `pnpm exec playwright install --with-deps chromium` — **nunca `npx`**
  (Sonar S6505/S8543: `npx` instala on-demand sin versión fijada; `pnpm exec`
  usa el binario lockeado). `playwright.config.ts` usa `reuseExistingServer`.

## Metodología de trabajo

- Cada issue de GitHub = una tarea chica (1-4 hs), asignada a un integrante.
- Rama por issue: `feature/<id-tema>` → PR a `develop` → PR a `main` (+tag).
- CI/CD **automático, sin aprobación humana** (branch protection sin
  reviewers: los gates deciden).
- **Gates en `develop`**: solo que pasen los tests
  (frontend: Lint/Build/Unit/E2E; backend: Build/Tests).
  **No se mergea a `develop` si fallan.**
- **Gates en `main`**: además **cobertura TOTAL ≥80%**
  (job `Coverage`, thresholds de vitest, solo corre en PRs/pushes a `main`),
  check `SonarCloud Code Analysis` y job **`Quality Gate (proyecto)`**
  (QG del proyecto completo vía API, no solo el diff del PR).
- **Deploy**: frontend en Vercel (auto al mergear a `main`); backend vía
  workflow `Deploy` a Railway (con guard de QG propio).
  Sin gates verdes no hay merge a `main`, y sin merge no hay deploy.
- Commits atómicos por paso lógico; mensaje convencional
  (`feat:`/`fix:`/`test:`/`refactor:`/`chore:`).

## Agentes y subagentes (TDD)

Siempre: test en rojo → código mínimo en verde → refactor → todo verde.
En código existente: tests de caracterización primero, después refactor.

- Roles (valen para OpenCode o cualquier otra herramienta):
  - `tester`: escribe **primero** el test que falla (en bugs, el test debe
    exponer el bug). Verifica **ejecutando**, no afirmando.
  - `programador`: código **mínimo** para pasar a verde (sin gold-plating),
    después refactoriza con los tests cuidándolo. Commitea atómico.
  - `juez`: **solo lectura, no modifica código**. Veta si falla la rúbrica
    senior: estructura según el `AGENTS.md` del repo, legibilidad,
    cero hardcodeo, errores explícitos, sin código muerto/duplicación,
    tipado estricto, cambio cubierto por tests, gates verdes.
- Tests verdes + código ilegible = CAMBIOS igual: la legibilidad es
  criterio de veto (un compañero lo entiende en frío).
- En React, detectar prop drilling es parte del refactor (no opcional):
  props que cruzan 3+ niveles sin usarse → context, composición o query.
- Pipeline por issue: `tester → programador → tester (re-verifica) → juez`.
  Si el juez pide cambios: **máximo 3 rondas**, después escala al humano.
- El orquestador parte el issue, lanza en orden, pushea y abre el PR.
  Los subagentes **no pushean ni crean PRs**.

## SonarCloud (Quality Gate)

- Análisis en cada push/PR vía la GitHub App `SonarQubeCloud`.
- **No se mergea a `main` si falla el Quality Gate** (Security, Reliability,
  Maintainability, Duplicación). `develop` admite merge aunque falle.
- `SONAR_TOKEN` vive en `.env` de cada repo (ignorado por git); con él se
  puede consultar el QG vía API de SonarCloud.

## Documentos de detalle

- `MyRacing-Backend/AGENTS.md` — ESM con imports `.js`, entidades
  `*.entity.ts` con imports explícitos en `orm.ts`, RequestContext/EM,
  secretos JWT por defecto inseguros, etc.
- `MyRacing-Frontend/AGENTS.md` — `apiClient.ts`, features/hooks, FE-15/FE-16.
- `VARIABLES-ENTORNO-MERCADOPAGO.md` — pasarela de pagos.
- `docs/politica-git/` — Gitflow simplificado.
