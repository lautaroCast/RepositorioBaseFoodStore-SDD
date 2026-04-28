# Food Store — Repositorio Base

Sistema de e-commerce de productos alimenticios desarrollado con **Spec-Driven Development (SDD)** usando OPSX y Claude Code.

---

## 🚀 Quick Start (Elige tu camino)

### ¿Tenés Docker instalado y querés algo rápido?
```bash
git clone <url> food-store && cd food-store
docker-compose up
# Todo arranca en localhost (frontend: 80, backend: 8000)
# ⏳ Espera a que diga "ready"
# ✅ Listo. Abrí http://localhost
```
**Nota:** Docker está disponible desde **Change 06** (check en `docs/CHANGES.md`)

### ¿Preferís setup manual (Python + Node local)?
👉 Saltá a **[Setup del entorno de desarrollo](#setup-del-entorno-de-desarrollo)** más abajo.

**Tiempo estimado:** 15-20 minutos | **Requisitos:** Python 3.11+, Node 18+, PostgreSQL 15+

---

## Tabla de contenidos

- [Documentación del sistema](#documentación-del-sistema)
- [Stack tecnológico](#stack-tecnológico)
- [Arquitectura](#arquitectura)
- [Setup del entorno de desarrollo](#setup-del-entorno-de-desarrollo)
- [Flujo de desarrollo con OPSX](#flujo-de-desarrollo-con-opsx)
- [Variables de entorno](#variables-de-entorno)
- [Convenciones](#convenciones)

---

## Documentación del sistema

Antes de escribir una línea de código, leé los tres documentos en `docs/`:

| Archivo | Contenido |
|---------|-----------|
| `docs/Descripcion.txt` | Visión general, actores del sistema y stack tecnológico |
| `docs/Integrador.txt` | Arquitectura en capas, ERD, API REST y patrones de diseño |
| `docs/Historias_de_usuario.txt` | US-000 a US-076 con criterios de aceptación y reglas de negocio |

Estos documentos son la fuente de verdad del sistema. El agente los lee antes de cada propuesta.

---

## Stack tecnológico

**Backend**: FastAPI · SQLModel · PostgreSQL · Alembic · bcrypt · python-jose · slowapi · MercadoPago SDK  
**Frontend**: React · TypeScript · Vite · TanStack Query · TanStack Form · Zustand · Axios · Tailwind CSS · Recharts

---

## Arquitectura

### Backend — Feature-First Modular Architecture

Cada feature (módulo de negocio) es autónomo y contiene toda su lógica:

```
backend/
├── src/
│   ├── core/                    # Utilidades compartidas
│   │   ├── dependencies/        # Inyección de dependencias FastAPI
│   │   ├── repositories/        # BaseRepository[T], Unit of Work pattern
│   │   └── exceptions/          # Errores comunes (RFC 7807)
│   │
│   ├── features/                # Cada feature es una unidad autónoma
│   │   ├── users/
│   │   │   ├── router.py        # Endpoints HTTP
│   │   │   ├── service.py       # Lógica de negocio
│   │   │   ├── repository.py    # Acceso a datos
│   │   │   ├── model.py         # ORM (SQLModel)
│   │   │   ├── schemas.py       # Pydantic (request/response)
│   │   │   └── tests/           # Tests unitarios
│   │   │
│   │   ├── products/
│   │   ├── orders/
│   │   └── payments/            # Integraciones (MercadoPago, etc.)
│   │
│   └── main.py                  # FastAPI app, routers
│
└── tests/                        # Tests de integración
```

**Ventajas:**
- Cada feature puede desarrollarse en paralelo sin bloqueos
- Fácil de localizar código: "¿dónde está la lógica de users?" → `src/features/users/`
- Escalable: agregar un feature = crear una carpeta nueva
- Facilita migraciones a microservicios después

### Frontend — Feature-Sliced Design (FSD)

Frontend organizado por slices (características) con segmentación clara:

```
frontend/
├── src/
│   ├── features/                # Cada slice es autónomo
│   │   ├── auth/
│   │   │   ├── ui/              # Componentes React
│   │   │   ├── api/             # Llamadas HTTP (Axios)
│   │   │   ├── model/           # Types + Zustand store
│   │   │   ├── tests/           # Tests unitarios
│   │   │   └── index.ts         # Exportación pública
│   │   │
│   │   ├── products/
│   │   ├── cart/
│   │   ├── orders/
│   │   └── payments/
│   │
│   ├── shared/                  # Utilidades cross-slice
│   │   ├── ui/                  # Componentes reutilizables
│   │   ├── api/                 # Configuración HTTP (Axios instance)
│   │   ├── lib/                 # Utility functions
│   │   └── types/               # Tipos globales
│   │
│   ├── App.tsx
│   └── main.tsx
│
└── public/                      # Assets estáticos
```

**Límites de importación:**
- ✅ `features/auth` puede importar de `shared/`
- ❌ `features/auth` NO puede importar de `features/products`
- ❌ `features/products/ui/Button` NO puede ser importado directamente; usar `features/products/ui` en su lugar

---

## Setup del entorno de desarrollo

### Requisitos previos

| Tecnología | Versión | Instalación |
|------------|---------|------------|
| **Git** | Cualquiera | https://git-scm.com/download |
| **Python** | 3.11+ | https://www.python.org/downloads |
| **Node.js** | 18+ | https://nodejs.org |
| **PostgreSQL** | 15+ | https://www.postgresql.org/download |
| **OpenSpec CLI** | Última | `npm install -g @fission-ai/openspec` (opcional) |

**Verificar instalaciones:**
```bash
git --version
python --version
node --version
npm --version
psql --version
```

---

### PASO 1: Clonar el repositorio

```bash
git clone <url-del-repo> food-store
cd food-store
```

**¿Qué ves?**
```
food-store/
├── backend/          ← Python + FastAPI
├── frontend/         ← React + TypeScript
├── docs/             ← Documentación
├── openspec/         ← Cambios (SDD workflow)
├── README.md
└── CONTRIBUTING.md
```

---

### PASO 2: Preparar PostgreSQL

**Opción A: PostgreSQL ya corriendo localmente** (continuá a PASO 3)

**Opción B: PostgreSQL con Docker** (si no lo tenés instalado)
```bash
docker run --name foodstore-postgres \
  -e POSTGRES_USER=foodstore_user \
  -e POSTGRES_PASSWORD=foodstore_password \
  -e POSTGRES_DB=foodstore_db \
  -p 5432:5432 \
  -d postgres:15
```

---

### PASO 3: Backend (FastAPI)

#### 3.1 Entrar a la carpeta backend
```bash
cd backend
```

#### 3.2 Crear archivo `.env` (variables secretas)
```bash
# Linux / Mac
cp .env.example .env

# Windows (PowerShell)
Copy-Item .env.example .env
```

**¿Qué hace?** Copia la plantilla. Ahora edita `.env` y asegúrate de que coincida con tu BD:
```env
DATABASE_URL=postgresql://foodstore_user:foodstore_password@localhost:5432/foodstore_db
JWT_SECRET=my-super-secret-key-change-in-production
JWT_EXPIRY_HOURS=24
CORS_ORIGINS=http://localhost:5173,http://localhost:3000
LOG_LEVEL=info
ENVIRONMENT=development
```

#### 3.3 Crear "virtual environment" (caja aislada para librerías Python)
```bash
# Crear
python -m venv .venv

# Activar (depende de tu OS)
# ========== WINDOWS (PowerShell) ==========
.venv\Scripts\activate

# ========== LINUX / Mac ==========
source .venv/bin/activate
```

**Indicador:** Si la activación funciona, verás `(.venv)` al principio del prompt.

#### 3.4 Instalar dependencias Python
```bash
pip install -r requirements.txt
```

**¿Qué hace?** Lee `requirements.txt` y descarga FastAPI, SQLModel, bcrypt, etc.

#### 3.5 (Opcional) Ejecutar migraciones de BD
```bash
# Crear tablas según modelos
alembic upgrade head

# Poblar datos iniciales (roles, usuarios, estados)
python -m backend.src.seed
```

#### 3.6 Iniciar servidor backend
```bash
uvicorn backend.src.main:app --reload
```

**¿Qué ves?**
```
INFO:     Uvicorn running on http://127.0.0.1:8000
INFO:     Application startup complete
```

**Endpoints:**
- 🔵 **API**: http://localhost:8000
- 📚 **Documentación interactiva (Swagger)**: http://localhost:8000/docs
- ⚙️ **ReDoc**: http://localhost:8000/redoc

---

### PASO 4: Frontend (React + Vite)

#### 4.1 Abrir OTRA terminal (backend sigue corriendo)
```bash
# En una nueva terminal
cd frontend
```

#### 4.2 Crear archivo `.env`
```bash
# Linux / Mac
cp .env.example .env

# Windows (PowerShell)
Copy-Item .env.example .env
```

**Contenido mínimo de `.env`:**
```env
VITE_API_BASE_URL=http://localhost:8000
VITE_LOG_LEVEL=debug
VITE_ENVIRONMENT=development
```

#### 4.3 Instalar dependencias Node.js
```bash
npm install
```

**¿Qué hace?** Lee `package.json` y descarga React, Vite, TypeScript, etc. en `node_modules/`.

#### 4.4 Iniciar servidor frontend
```bash
npm run dev
```

**¿Qué ves?**
```
  VITE v5.0.0  ready in 150 ms

  ➜  Local:   http://localhost:5173/
  ➜  press h to show help
```

**Abre en navegador:** http://localhost:5173

---

### ✅ VERIFICACIÓN FINAL

Si llegaste acá sin errores, tenés:

| Componente | URL | Estado |
|-----------|-----|--------|
| Backend (FastAPI) | http://localhost:8000 | ✅ Corriendo |
| Swagger Docs | http://localhost:8000/docs | ✅ Accesible |
| Frontend (React) | http://localhost:5173 | ✅ Corriendo |
| PostgreSQL | localhost:5432 | ✅ Conectado |

**Todo está listo. Felicidades.** 🎉

---

### 🔧 Terminal Layout Recomendado

**Terminal 1 (Backend):**
```bash
cd backend
source .venv/bin/activate  # (o .venv\Scripts\activate en Windows)
uvicorn backend.src.main:app --reload
```

**Terminal 2 (Frontend):**
```bash
cd frontend
npm run dev
```

**Terminal 3 (Git / comandos):**
```bash
# Aquí haces git commits, cambios, etc.
cd food-store
git status
```

---

### ❌ Troubleshooting

#### Backend no arranca

**Error:** `ModuleNotFoundError: No module named 'fastapi'`
```bash
# Solución: Verifica que virtual environment esté ACTIVADO
# Deberías ver (.venv) al inicio del prompt
pip install -r requirements.txt
```

**Error:** `FATAL: role "foodstore_user" does not exist`
```bash
# Solución: PostgreSQL no está corriendo o credenciales mal
# Verifica DATABASE_URL en .env
# Si usas Docker, asegúrate que el container esté corriendo:
docker ps | grep postgres
```

**Error:** `address already in use (:8000)`
```bash
# Puerto 8000 ocupado. Alternativa:
uvicorn backend.src.main:app --reload --port 8001
```

#### Frontend no arranca

**Error:** `npm: command not found`
```bash
# Node.js no instalado o PATH incorrecto
node --version
npm --version
# Si retorna versión, cierra y abre terminal nuevamente
```

**Error:** `ENOENT: no such file or directory, open '.env'`
```bash
# Solución: .env no existe. Cópialo:
cp .env.example .env
```

**Error:** `Cannot GET http://localhost:5173`
```bash
# Frontend está compilando. Espera 10 segundos y recarga (F5)
```

#### PostgreSQL

**Error:** `psql: error: could not translate host name "localhost" to address`
```bash
# PostgreSQL no está corriendo. Inicia:
# En macOS: brew services start postgresql
# En Windows: búsca "pgAdmin" o "PostgreSQL" en Services
# O usa Docker: docker run -p 5432:5432 ... (ver PASO 2)
```

---

### 📞 Comandos Útiles

```bash
# Backend: instalar nueva librería
pip install nombre-paquete
pip freeze > requirements.txt  # Actualizar requirements.txt

# Frontend: instalar nuevo paquete
npm install nombre-paquete
npm run lint                    # Revisar código
npm run type-check             # TypeScript

# Git
git status                      # Ver cambios
git log --oneline              # Ver commits
git branch                      # Ver ramas
```

---

## Flujo de desarrollo con OPSX

Todo cambio al sistema sigue este ciclo:

```
/opsx:explore   →  pensar antes de comprometerse (opcional)
/opsx:propose   →  generar propuesta + diseño + tareas
/opsx:apply     →  implementar tarea por tarea
/opsx:archive   →  sincronizar specs y cerrar el change
```

Cada change genera artefactos en `openspec/changes/<nombre>/`:
- `proposal.md` — qué y por qué
- `design.md` — cómo (decisiones técnicas)
- `specs/**/*.md` — especificaciones detalladas
- `tasks.md` — checklist de tareas

### Orden de implementación

```
Sprint 0: Infraestructura
  00. project-scaffolding
  01. backend-core-setup
  02. frontend-core-setup
  03. database-seed-and-validation
  04. testing-ci-cd-pipeline
  05. docker-and-local-dev

Sprint 1: Autenticación
  06. user-authentication
  07. authorization-rbac
  08. password-reset-recovery

...y más. Ver docs/CHANGES.md para el roadmap completo.
```

---

## Variables de entorno

### Backend (.env)

```env
# Database
DATABASE_URL=postgresql://foodstore_user:foodstore_password@localhost:5432/foodstore_db

# JWT
JWT_SECRET=your-super-secret-key-change-in-production
JWT_EXPIRY_HOURS=24

# CORS
CORS_ORIGINS=http://localhost:5173,http://localhost:3000

# Logging
LOG_LEVEL=info

# Environment
ENVIRONMENT=development
```

### Frontend (.env)

```env
# API Configuration
VITE_API_BASE_URL=http://localhost:8000

# Logging
VITE_LOG_LEVEL=debug

# Environment
VITE_ENVIRONMENT=development
```

**IMPORTANTE:** `.env` está en `.gitignore` y NUNCA debe subirse al repositorio. Usar `CI/CD secrets` para variables sensibles en producción.

---

## Convenciones

### Git Workflow

**Branch naming:**
```
feature/nombre-del-feature      # Nuevas features
fix/numero-issue                 # Bug fixes
docs/descripcion                 # Documentación
test/descripcion                 # Tests
refactor/descripcion             # Refactors
```

**Commits — Conventional Commits:**
```
feat(modulo): descripción del cambio
fix(modulo): descripción del bug corregido
refactor(modulo): descripción del refactor
test(modulo): descripción de los tests
docs(modulo): descripción del cambio en docs
chore(modulo): tareas de build, deps, etc.
```

Ejemplo:
```bash
git commit -m "feat(auth): implement JWT refresh token rotation"
git commit -m "fix(products): correct price calculation for bulk orders"
```

### Code Style

**Backend (Python):**
- Formateador: `black`
- Import sorter: `isort`
- Linter: `flake8`
- Type hints: obligatorio en funciones públicas

Ejecutar localmente:
```bash
black backend/src
isort backend/src
flake8 backend/src --max-line-length=100
```

**Frontend (TypeScript + React):**
- Linter: `eslint` con soporte TypeScript
- Nombrado de componentes: PascalCase
- Archivos: kebab-case

Ejecutar localmente:
```bash
npm run lint
npm run type-check
```

### Pull Requests

Cada PR debe incluir:
1. **Descripción clara** de qué cambia y por qué
2. **Link al issue** (si aplica)
3. **Pasos para testear** los cambios
4. **Screenshots** (si es UI)
5. **Checklist de verificación**

---

## CI/CD Pipeline

El repositorio tiene configurado GitHub Actions para:

✅ Linting automático (backend y frontend)  
✅ Tests automáticos (cuando existen)  
✅ Type checking (TypeScript)  
✅ Bloqueo de merge si los checks fallan

Pipeline corre automáticamente en:
- Push a cualquier rama
- Pull requests

Ver `.github/workflows/ci.yml` para detalles.

---

## Troubleshooting

**Backend no arranca**
- Verificar: `DATABASE_URL` en `.env` es correcta
- Verificar: PostgreSQL está corriendo
- Limpiar: `rm -rf backend/__pycache__ backend/.pytest_cache`
- Reinstalar: `pip install -r requirements.txt --force-reinstall`

**Frontend no arranca**
- Limpiar: `rm -rf node_modules package-lock.json && npm install`
- Verificar: `VITE_API_BASE_URL` apunta a backend correcto
- Puerto 5173 ocupado: cambiar a otro puerto en `.env`

**GitHub Actions falla**
- Revisar logs en GitHub → Actions tab
- Ejecutar linting localmente: `npm run lint` o `black --check .`
- Verificar sintaxis YAML en `.github/workflows/ci.yml`

---

## Contacto & Contribuciones

Para aportar al proyecto, seguir [CONTRIBUTING.md](./CONTRIBUTING.md).
