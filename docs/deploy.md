# Deploy: Vercel (frontend) + Railway (backend + MySQL)

## Plataforma

- **Frontend** (`MyRacing-Frontend`): Vercel. Al conectar el repo a Vercel,
  construye con `pnpm build` y publica `dist/`. HTTPS automático.
- **Backend** (`MyRacing-Backend`): Railway. Servicio Node con HTTPS automático
  en `https://<app>.up.railway.app`.
- **MySQL**: Railway (o servicio externo). Ver `compose.dev.yaml` para dev.

## Variables de entorno

### Backend (Railway)

| Variable | Descripción |
| --- | --- |
| `DB_HOST` / `DB_PORT` / `DB_NAME` / `DB_USER` / `DB_PASSWORD` | Conexión a MySQL |
| `JWT_SECRET` / `JWT_REFRESH_SECRET` | Secrets JWT |
| `MERCADOPAGO_ACCESS_TOKEN` | Token de Mercado Pago |
| `URL_BACKEND` | `https://<app>.up.railway.app` |
| `URL_FRONTEND` | `https://<app>.vercel.app` |
| `URL_WEBHOOK_MP` | `https://<app>.up.railway.app/api/payment/wh-mp` |
| `BREVO_API_KEY` / `FROM_EMAIL` | Email |

### Frontend (Vercel)

| Variable | Descripción |
| --- | --- |
| `VITE_API_BASE_URL` | `https://<app>.up.railway.app/api` |
| `VITE_MP_PUBLIC_KEY` | Public key de Mercado Pago |

## Mercado Pago (HTTPS)

- Railway provee HTTPS automático, por lo que el requisito de MP queda cumplido.
- En Mercado Pago Developers → Webhooks, registrar:
  `https://<app>.up.railway.app/api/payment/wh-mp`
- En Railway setear `URL_WEBHOOK_MP` con el mismo valor.

## Flujo de entrega (100% automático, sin aprobación humana)

- Push/PR a `develop`: CI corre Lint/Build/tests. Gate = **tests verdes**
  (no se exige coverage ni Quality Gate). Merge a `develop` bloqueado si fallan.
- PR `develop` → `main`: CI exige además **Coverage total ≥80%**
  (job `Coverage`, solo corre en PRs/pushes a `main`) y el check
  `SonarCloud Code Analysis`. Merge a `main` bloqueado sin gates verdes.
- Al mergear a `main`, el workflow `Deploy` corre solo si el CI pasó,
  re-chequea el Quality Gate del proyecto vía API (aborta si no es `OK`)
  y despliega: frontend a Vercel (`vercel --prod`), backend a Railway.
- Estado actual: los release PRs `develop → main` están **bloqueados por diseño**
  (coverage ~0% + QG en ERROR) hasta escribir los tests (BE-2..BE-11, FE-1..FE-9).

## Secrets de GitHub requeridos

| Secret | Repo | Uso |
| --- | --- | --- |
| `SONAR_TOKEN` | ambos | Guard del Quality Gate en el workflow `Deploy` (ya configurado) |
| `VERCEL_TOKEN` / `VERCEL_ORG_ID` / `VERCEL_PROJECT_ID` | frontend | Deploy a producción por CLI |
| `RAILWAY_TOKEN` (+ var `RAILWAY_SERVICE`) | backend | Deploy a producción por CLI (pendiente crear el servicio) |

Sin estos secrets, el workflow `Deploy` **aborta explícitamente**: no hay deploy silencioso.

> Nota Vercel: la integración Git auto-despliega `main` a producción.
> Cuando `VERCEL_TOKEN` esté configurado, cambiar en el dashboard
> (Project → Settings → Git) la rama de producción a una rama sin uso,
> para que producción solo se despliegue vía el workflow `Deploy` con gates.
