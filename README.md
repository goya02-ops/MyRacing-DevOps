# MyRacing - DevOps Stack

Repositorio de orquestación que coordina los dos sub-proyectos de MyRacing:
la API REST (backend) y la SPA (frontend), junto con la documentación del
trabajo práctico.

## Estructura del Proyecto

El proyecto está dividido en **3 repositorios Git separados**, cada uno con su
propio ciclo de vida (ramas, CI/CD, deploy):

```
MyRacing-DevOps/                  ← este repo (orquestación y docs)
├── compose.dev.yaml              # Opcional: solo MySQL 8 para desarrollo
├── docs/                         # Diagramas, política de git y minutas del TP
├── Enunciado.md                  # Consigna del trabajo práctico
└── README.md
├── MyRacing-Backend/             → goya02-ops/MyRacing-Backend (clon local, NO versionado acá)
│   Express 5 + TypeScript + MikroORM + MySQL
└── MyRacing-Frontend/            → goya02-ops/MyRacing-Frontend (clon local, NO versionado acá)
    React 19 + Vite + TypeScript + Tailwind/Tremor
```

Los sub-proyectos **no se versionan en este repo**: cada uno vive y se despliega
desde su propio repositorio en GitHub.

---

## Requisitos Previos

- **Node.js 20+** y **pnpm** (`npm install -g pnpm`)
- **MySQL 8** accesible (nativo, cloud o vía Docker opcional, ver abajo)

---

## Desarrollo Local (sin Docker — recomendado)

### 1. Base de datos

El backend necesita un MySQL 8 con una base `myracing`. Opciones:

- **MySQL nativo/cloud**: creá la base y el usuario una única vez:
  ```sql
  CREATE DATABASE myracing;
  CREATE USER 'admin'@'localhost' IDENTIFIED BY 'MiR@cing_2025!';
  GRANT ALL PRIVILEGES ON myracing.* TO 'admin'@'localhost';
  ```
- **Docker opcional** (solo para la DB): `docker compose -f compose.dev.yaml up -d`

> Las tablas se crean automáticamente al arrancar el backend (MikroORM
> sincroniza el esquema en desarrollo).

### 2. Configurar variables de entorno

```bash
cp MyRacing-Backend/.env.example MyRacing-Backend/.env
cp MyRacing-Frontend/.env.example MyRacing-Frontend/.env   # si existe; el frontend usa VITE_*
```

### 3. Instalar dependencias

```bash
cd MyRacing-Backend && pnpm install
cd ../MyRacing-Frontend && pnpm install
```

### 4. Ejecutar

```bash
# Terminal 1 - Backend (tsx con hot reload)
cd MyRacing-Backend && pnpm dev

# Terminal 2 - Frontend (Vite)
cd MyRacing-Frontend && pnpm dev
```

**Acceso:**
- Frontend: http://localhost:5173
- Backend: http://localhost:3000

---

## Scripts Disponibles

### Backend

```bash
pnpm dev      # Desarrollo (tsx con hot reload)
pnpm build    # Compilar TypeScript
pnpm start    # Producción
pnpm test     # Tests (Vitest)
```

### Frontend

```bash
pnpm dev      # Desarrollo (Vite)
pnpm build    # Build producción
pnpm lint     # Linting
pnpm test     # Tests (Vitest)
```

---

## CI/CD y Deploy

- **CI**: cada sub-repo tiene su pipeline en GitHub Actions (lint, build,
  unit, integración y e2e) que corre en cada PR/push.
- **E2E** vive en el pipeline del **frontend** y levanta el backend real + MySQL
  de forma efímera en CI.
- **Deploy**: por **buildpack** (Railway/Render) desde cada sub-repo. **Docker
  NO es requisito**: el deploy no usa imágenes; el contenedor de MySQL
  (`compose.dev.yaml`) es solo una comodidad opcional para desarrollo.
- El deploy está **bloqueado si fallan los tests** del repo correspondiente.

---

## Configuración

### Variables de Entorno (Backend)

Ver `MyRacing-Backend/.env.example` y heredar `MyRacing-DevOps/VARIABLES-ENTORNO-MERCADOPAGO.md`
para el flujo de pagos.

| Variable             | Descripción              | Valor por defecto     |
| -------------------- | ------------------------ | --------------------- |
| `DB_HOST`            | Host MySQL               | `localhost`           |
| `DB_PORT`            | Puerto MySQL             | `3307`                |
| `DB_NAME`            | Base de datos            | `myracing`            |
| `DB_USER`            | Usuario MySQL            | `admin`               |
| `DB_PASSWORD`        | Contraseña MySQL         | `MiR@cing_2025!`      |
| `JWT_SECRET`         | Secret JWT access        | -                     |
| `JWT_REFRESH_SECRET` | Secret JWT refresh       | -                     |
| `URL_FRONTEND`       | URL frontend             | `http://localhost:5173` |
| `URL_BACKEND`        | URL backend              | `http://localhost:3000` |

---

## API Endpoints

```
/api/auth          Autenticación
/api/users         Usuarios
/api/categories    Categorías
/api/circuits      Circuitos
/api/simulators   Simuladores
/api/combinations  Combinaciones
/api/membership    Membresías
/api/races         Carreras
/api/race-users    Inscripciones
/api/payment       Pagos (Mercado Pago)
```

---

## Política de Ramas

Cada sub-repo sigue **Gitflow simplificado** (ver `docs/politica-git/`):
- `main` = estable / release (defensa).
- `develop` = integración.
- `feature/<tema>` → PR a `develop`.
- Release: PR `develop` → `main` (+ tag).

El seguimiento del equipo se registra en `docs/minutas/`.