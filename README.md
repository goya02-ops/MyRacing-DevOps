# MyRacing - DevOps Stack

Entorno de orquestación que unifica Frontend, Backend y Base de Datos para desarrollo y producción.

## Estructura del Proyecto

```
MyRacing-DevOps/
├── compose.yaml       # Producción (Full stack en Docker)
├── compose.dev.yaml   # Desarrollo (solo MySQL en Docker)
├── README.md          # Este archivo
├── MyRacing-Backend/ # API REST (Express + TypeScript + MikroORM)
└── MyRacing-Frontend/ # SPA (React + Vite + TypeScript)
```

---

## Requisitos Previos

- **Docker Desktop** (o Docker Engine)
- **pnpm** instalado: `npm install -g pnpm`

---

## Desarrollo Local (Recomendado)

### 1. Levantar MySQL

```bash
docker compose -f compose.dev.yaml up -d
```

### 2. Configurar Variables de Entorno

```bash
cp MyRacing-Backend/.env.example MyRacing-Backend/.env
```

### 3. Instalar Dependencias

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

## Producción (Docker)

Levanta todo el stack en contenedores:

```bash
docker compose up --build
```

---

## Scripts Disponibles

### Backend

```bash
pnpm dev      # Desarrollo (tsx con hot reload)
pnpm build    # Compilar TypeScript
pnpm start    # Producción
```

### Frontend

```bash
pnpm dev      # Desarrollo (Vite)
pnpm build    # Build producción
pnpm lint     # Linting
pnpm test     # Tests
```

---

## Configuración

### Variables de Entorno (Backend)

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
