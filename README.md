# MyRacing - Entorno de Desarrollo Local (DevOps Stack)

Este repositorio es la **capa de orquestación** que utiliza Docker Compose para unificar los servicios del proyecto (Frontend, Backend y DB).

**NO contiene el código fuente.** Los repositorios de código (`MyRacing-Frontend` y `MyRacing-Backend`) deben ser clonados dentro de esta carpeta.

## 🛠️ Requisitos Previos

Asegúrese de tener instalados:

1.  **Git**
2.  **Docker Desktop** (o Docker Engine)
3.  **pnpm** (instalado globalmente)

## 🚀 Pasos de Instalación y Arranque

Para que el entorno funcione, siga estos pasos **en el orden exacto**:

### 1. Clonar el Stack (Este Repositorio)

```bash
git clone git@github.com:goya02-ops/MyRacing-DevOps.git
cd MyRacing-DevOps
```

### 2. Clonar los Repositorios de Código

# Clonar el Frontend

```bash
git clone git@github.com:goya02-ops/MyRacing-Frontend.git
```

# Clonar el Backend

```bash
git clone git@github.com:goya02-ops/MyRacing-Backend.git
```

### 3. Levantar el Entorno

# Usar '--build' es NECESARIO la primera vez en cada máquina

```bash
docker compose up --build
```
