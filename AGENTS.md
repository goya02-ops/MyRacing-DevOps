# AGENTS.md - MyRacing DevOps

Repo de **orquestación y docs**. NO es un monorepo: `MyRacing-Backend/` y
`MyRacing-Frontend/` son clones locales de repos GitHub independientes
(ramas, CI, deploy propios), ignorados por git acá (ver `.gitignore`).

## Setup y verificación

- **pnpm 12, Node 22** (CI fija `node-version: 22` en ambos repos;
  `packageManager` fija el pnpm, corepack lo resuelve solo).
  `pnpm install` dentro de cada carpeta; proyectos pnpm separados, sin
  workspace hoisting desde la raíz.
- pnpm 12 exige `allowBuilds` explícitos en cada `pnpm-workspace.yaml`
  (backend: `bcrypt`/`esbuild` en `true`; frontend: `@swc/core`/`esbuild` en
  `false`, con `onlyBuiltDependencies` para `bcrypt`/`@tailwindcss/oxide`).

| Sub-proyecto | Verificación |
| ------------ | ------------ |
| `MyRacing-Backend` | `pnpm lint && pnpm build && pnpm test` (Vitest; `pnpm test:coverage` solo para gate de `main`) |
| `MyRacing-Frontend` | `pnpm lint && pnpm build && pnpm test` (`lint` es `eslint . --max-warnings 0`; E2E: `pnpm test:e2e`) |

## Base de datos

- Dev MySQL 8: `docker compose -f compose.dev.yaml up -d` → puerto **3307**
  (CI usa **3306** con MySQL efímera vía step `docker run`, no `services:` —
  GitHub prohíbe `secrets` en service containers; no mezclar).
  DB `myracing`, user `admin` / `MiR@cing_2025!`.
- Los jobs de CI consumen los secrets `DB_PASSWORD` y `MYSQL_ROOT_PASSWORD`
  (crear en Settings → Secrets → Actions de cada repo).
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
- E2E de pagos contra sandbox MP real (sin mocks de red): seed login-first
  (registros limitados a 5 req/15min/IP), emails `@test.com` válidos
  (MP rechaza otros dominios como `payer.email`), no tipear iframes de MP,
  `test.setTimeout()` (la opción `timeout` de `test()` la ignora Playwright),
  `trace` + `screenshot` on failure, DB efímera requerida.

## Metodología de trabajo

- Cada issue de GitHub = una tarea chica (1-4 hs), asignada a un integrante.
- Rama por issue: `feature/<id-tema>` → PR a `develop` → PR a `main` (+tag).
  Trabajo sin issue (docs, setup, tooling): `chore/<tema>` → PR a `develop`.
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
- En código (comentarios, títulos/nombres de tests) prohibido referenciar
  issues (`FE-12`, `BE-3`, `#13`): solo en commits, PRs e issues.
- Onboarding: al iniciar un `init` (de skill o de sesión), preguntar a la
  persona si conoce esta metodología; si no, explicarla antes de arrancar.

## Trabajo multi-chat

- **Chat orquestador** (este, parado en DevOps): backlog, prioridades,
  asignaciones, decisiones que cruzan repos y merges. No implementa.
- **Chats por repo**: ejecutan las issues de su repo, parados en su raíz,
  con sus skills. No tocan el otro repo. En cada uno rige el mismo pipeline
  (orquestador local + `tester → programador → juez`, máximo 3 rondas).
- **Escalación**: si un agente detecta algo que afecta al otro repo
  (contrato de API, shape de errores, auth, pagos), lo eleva al orquestador
  con archivo:línea y propuesta, en vez de resolverlo por su cuenta.
- **Derivación**: si el orquestador ve que una tarea pertenece a un repo,
  la deriva a su chat con número de issue.
- **Regla de oro**: lo que cruce repos se acuerda en el orquestador
  **antes** de implementarse en los chats.

## Skills

- Vendoreadas en `<repo>/.agents/skills/` (commiteadas, las usa cualquiera
  sin instalar nada). Si el skill tool no resuelve el ID, leer el
  `SKILL.md` directo (fallback válido).
  - Frontend: `react-best-practices`, `composition-patterns`,
    `playwright-best-practices`, `tdd`, `impeccable`, `security-audit`.
  - Backend: `mysql`, `docker-expert`, `multi-stage-dockerfile`, `tdd`,
    `security-audit`, `typescript-advanced-types`.
- Globales en `~/.agents/skills/` (cada integrante en su máquina):
  `verification-before-completion`, `code-review`, `find-skills`.
- Cargar solo cuando aplique.

## SonarCloud (Quality Gate)

- Análisis en cada push/PR vía la GitHub App `SonarQubeCloud`.
- **No se mergea a `main` si falla el Quality Gate** (Security, Reliability,
  Maintainability, Duplicación). `develop` admite merge aunque falle.
- `SONAR_TOKEN` vive en `.env` de cada repo (ignorado por git); con él se
  puede consultar el QG vía API de SonarCloud.
- Reglas de workflows que ya mordieron: S6505/S8543 (nunca `npx`, usar
  `pnpm exec` con binario lockeado) y S7636 (secrets solo en `env:`,
  nunca expandidos en `run:`).

## MCPs: Mercado Pago, Railway, Vercel (`opencode.json`)

- Servers remotos con OAuth individual: `mercadopago`, `railway`, `vercel`.
  La config se comparte por repo (commiteada, sin secretos); cada integrante
  se autentica con su cuenta vía `/mcps`.
- MP: ver preferencias/pagos en test, usuarios de prueba, historial de
  webhooks. No se usa en tests automatizados (evitan MP por diseño). Usar
  cuenta de **prueba**.
- Railway/Vercel: deploys, logs, variables. El MCP no otorga permisos: para
  ver los mismos deploys hay que agregar a cada uno en el workspace (Railway),
  el team (Vercel) o la cuenta de test (MP).
- `mui` (solo frontend, en su `opencode.json`): server **local** con binario
  lockeado `@mui/mcp` vía `pnpm exec` (sin auth). Docs y generación de
  código MUI para la migración.

## Producto y diseño

- `PRODUCT.md` / `DESIGN.md` (raíz del frontend) son artefactos compartidos
  commiteados: salen de `/impeccable init` y los lee cada comando de diseño.
- Telemetría de Impeccable apagada para todo el proyecto vía
  `"telemetry": false` en `.impeccable/config.json` (solo ese archivo de
  `.impeccable/` se commitea; capturas y sesiones van por `.gitignore`).

## Documentos de detalle

- `MyRacing-Backend/AGENTS.md` — ESM con imports `.js`, entidades
  `*.entity.ts` con imports explícitos en `orm.ts`, RequestContext/EM,
  secretos JWT por defecto inseguros, etc.
- `MyRacing-Frontend/AGENTS.md` — `apiClient.ts`, features/hooks, FE-15/FE-16.
- `VARIABLES-ENTORNO-MERCADOPAGO.md` — pasarela de pagos.
- `docs/politica-git/` — Gitflow simplificado.
