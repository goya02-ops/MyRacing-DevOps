# Política de Git — MyRacing

Los 3 repositorios del proyecto (`MyRacing-Backend`, `MyRacing-Frontend` y
`MyRacing-DevOps`) siguen la misma política: **Gitflow simplificado**.

## Ramas

| Rama | Propósito | Regla |
|---|---|---|
| `main` | Estable / release (defensa) | Solo PR desde `develop`. Encaja con tag de release (`vX.Y.Z`). |
| `develop` | Integración diaria | Rama de trabajo base. Todo `feature/*` la mergea. |
| `feature/<tema>` | Una tarea por rama | Se crea desde `develop`, se mergea vía PR a `develop`. |

## Flujo de trabajo

```
develop  ──►  feature/<tema>  ──►  PR a develop  ──►  (CI ✓) ──►  merge a develop
                                                                        │
                                                                        ▼
release: PR develop ──► main  (+ tag vX.Y.Z)  ──►  (CI ✓) ──►  deploy
```

1. Crear la feature desde `develop`: `git checkout -b feature/<tema> develop`.
2. Commits atómicos y descriptivos siguiendo [Conventional Commits](https://www.conventionalcommits.org/)
   (`feat:`, `fix:`, `refactor:`, `chore:`, `ci:`, `docs:`, `test:`).
3. Abrir PR a `develop`. El pipeline de CI (lint, build, tests) **debe pasar** para mergear.
4. Para un release: PR `develop` → `main` y crear tag de release.

## CI/CD

- Cada sub-repo tiene `.github/workflows/ci.yml` y `.github/workflows/deploy.yml`.
- El `deploy.yml` se dispara solo cuando el `ci.yml` **terminó con éxito** sobre `main`
  (no se despliega nada si falla algún test o build).
- El **E2E full-stack** vive en el pipeline del frontend, que levanta el backend
  real + MySQL de forma efímera en CI.

## Estándares por repo

- **Backend**: ramas `main` + `develop`, módulos `src/<feature>/`, tests con Vitest.
- **Frontend**: ramas `main` + `develop`, features en `src/features/`, Vitest + Playwright.
- **DevOps**: solo orquestación y docs; no se despliega.

## Convención de nombres de features

- `fix/<descripcion>` — bugfix
- `feat/<descripcion>` — nueva funcionalidad
- `refactor/<descripcion>` — refactor
- `chore/<descripcion>` — mantenimiento/limpieza
- `test/<descripcion>` — tests