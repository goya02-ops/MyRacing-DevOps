# Variables de entorno — MyRacing y Mercado Pago

Este documento lista las variables necesarias para levantar **backend** y **frontend** con el flujo de pagos (Checkout Pro, tarjeta vía SDK, webhooks). Los archivos `.env` no se versionan: copiá estos nombres en `MyRacing-Backend/.env` y `MyRacing-Frontend/.env`.

---

## Backend (`MyRacing-Backend/.env`)

| Variable                   | Obligatoria              | Descripción                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| -------------------------- | ------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `MERCADOPAGO_ACCESS_TOKEN` | **Sí** (para pagos)      | **Access Token** de la cuenta de Mercado Pago que **cobrará** (vendedor). Es la credencial privada de la API; en el panel aparece como token de prueba o producción. El servidor no arranca el módulo de pagos si falta (el mensaje de error puede mencionar `MP_ACCESS_TOKEN`, pero la variable que lee el código es `MERCADOPAGO_ACCESS_TOKEN`).                                                                                                                            |
| `URL_FRONTEND`             | Recomendada              | URL base del front (sin barra final). Ej.: `http://localhost:5173` o `https://app.tudominio.com`. Se usa en `back_urls` de la preferencia (redirección tras el pago).                                                                                                                                                                                                                                                                                                         |
| `URL_BACKEND`              | Recomendada              | URL base del API (sin `/api` al final). Ej.: `http://localhost:3000` o `https://api.tudominio.com`. Fallback en código: `http://localhost:3000`.                                                                                                                                                                                                                                                                                                                              |
| `URL_WEBHOOK_MP`           | Recomendada (producción) | URL **absoluta** y **pública** del webhook: debe apuntar al endpoint `POST` de notificaciones. En este proyecto la ruta es **`{URL_BACKEND implícita en tu dominio}/api/payment/wh-mp`**. Ej.: `https://tu-servidor.com/api/payment/wh-mp`. Usá **HTTPS**; `localhost` suele ser rechazado por Mercado Pago: en local conviene un túnel (ngrok, etc.) u omitir el webhook y probar con redirecciones y consultas al API. Sin espacios alrededor del `=` en el archivo `.env`. |
| `JWT_SECRET`               | Sí (API)                 | Secret para firmar tokens de acceso. Las rutas de pago exigen usuario autenticado.                                                                                                                                                                                                                                                                                                                                                                                            |
| `JWT_REFRESH_SECRET`       | Sí (API)                 | Secret para refresh tokens.                                                                                                                                                                                                                                                                                                                                                                                                                                                   |

**Coherencia de credenciales:** El Access Token del backend y la Public Key del frontend deben ser del **mismo entorno** de Mercado Pago (ambos de prueba o ambos de producción).

---

## Frontend (`MyRacing-Frontend/.env`)

Vite solo expone al navegador las variables cuyo nombre empieza con **`VITE_`**.

| Variable             | Obligatoria                                 | Descripción                                                                                                                                   |
| -------------------- | ------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `VITE_MP_PUBLIC_KEY` | **Sí** (checkout / tarjeta en el navegador) | **Public Key** de la misma aplicación de Mercado Pago que el Access Token del backend. Ej.: clave que empieza con `TEST-...` o `APP_USR-...`. |
| `VITE_API_BASE_URL`  | Recomendada                                 | Base URL del API. Ej.: `http://localhost:3000/api`. Fallback en código: `http://localhost:3000/api`.                                          |

**Formato del `.env`:** una variable por línea, sin espacios alrededor del `=`, por ejemplo:

```env
VITE_MP_PUBLIC_KEY=TEST-xxxx-xxxx-xxxx-xxxx
VITE_API_BASE_URL=http://localhost:3000/api
```

Tras cambiar el `.env`, reiniciá el servidor de desarrollo de Vite.

**Nota sobre otras páginas:** cualquier pantalla que inicialice el SDK de Mercado Pago (`initMercadoPago`) debe leer la misma variable `import.meta.env.VITE_MP_PUBLIC_KEY`, no nombres sin prefijo `VITE_` (en el navegador quedarían `undefined`).

---

## Resumen rápido (solo Mercado Pago)

| Dónde    | Variable                       | Qué es en Mercado Pago                            |
| -------- | ------------------------------ | ------------------------------------------------- |
| Backend  | `MERCADOPAGO_ACCESS_TOKEN`     | Access Token (privado)                            |
| Frontend | `VITE_MP_PUBLIC_KEY`           | Public Key (pública)                              |
| Backend  | `URL_WEBHOOK_MP`               | URL de tu servidor para `POST /api/payment/wh-mp` |
| Backend  | `URL_FRONTEND` / `URL_BACKEND` | Redirecciones y construcción de URLs correctas    |

---

## Base de datos y resto del backend

El proyecto usa MikroORM con MySQL; la cadena de conexión está en código (`MyRacing-Backend/src/shared/orm.ts`). Para un entorno distinto (Docker, otro host) tendrás que ajustar esa configuración o externalizarla: no forma parte de las variables de `config.ts` hoy.

Si necesitás solo verificar el flujo de MP, además del `.env` asegurate de tener el servicio de base de datos accesible según esa configuración.
