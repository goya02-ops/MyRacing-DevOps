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

## Flujo de entrega

- Push a `develop`: CI corre tests (gate de `develop`). SonarCloud avisa pero no bloquea.
- PR `develop` → `main`: exige tests, coverage ≥80% (vitest thresholds) y Quality Gate SonarCloud. Deploy a Vercel/Railway al mergear.
- No se deploya nada en `develop`.
