# Food Store — Repositorio Base

Sistema de e-commerce de productos alimenticios desarrollado con **Spec-Driven Development (SDD)** usando OPSX y Claude Code.

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

- Python 3.11+
- Node.js 18+
- PostgreSQL 15+
- Git
- (Opcional) OpenSpec CLI: `npm install -g @fission-ai/openspec`

### 1. Clonar e inicializar

```bash
git clone <url-del-repo> food-store
cd food-store
```

### 2. Backend

```bash
cd backend

# Copiar template de variables de entorno
cp .env.example .env

# Crear virtual environment
python -m venv .venv
source .venv/bin/activate      # Linux/Mac
.venv\Scripts\activate         # Windows (PowerShell)

# Instalar dependencias
pip install -r requirements.txt

# (Opcional) Ejecutar migraciones
# alembic upgrade head
# python -m backend.src.seed

# Iniciar servidor
uvicorn backend.src.main:app --reload
```

**Endpoint:** `http://localhost:8000`  
**Swagger Docs:** `http://localhost:8000/docs`

### 3. Frontend

```bash
cd frontend

# Instalar dependencias
npm install

# Copiar template de variables de entorno
cp .env.example .env

# Iniciar servidor de desarrollo
npm run dev
```

**App:** `http://localhost:5173`

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
