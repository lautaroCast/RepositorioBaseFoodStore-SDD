## ADDED Requirements

### Requirement: Backend folder structure SHALL follow feature-first modular architecture
The backend SHALL organize code by feature, with each feature owning its router, service, repository, and model files. This enables parallel development and clear encapsulation of feature logic.

#### Scenario: Developer adds a new feature
- **WHEN** a developer adds a feature named "users"
- **THEN** they create `backend/src/features/users/` with subfolders: `router.py`, `service.py`, `repository.py`, `model.py`, `schemas.py`, `tests/`

#### Scenario: Feature directory structure
- **WHEN** examining the backend directory
- **THEN** the structure is `backend/src/features/{feature_name}/` with all related code in one place
- **THEN** shared utilities live in `backend/src/core/` (e.g., BaseRepository, dependency injection helpers)

### Requirement: Frontend folder structure SHALL follow Feature-Sliced Design (FSD)
The frontend SHALL organize code by feature slice, with each slice containing UI, API, and model concerns. Import boundaries SHALL prevent circular dependencies and accidental coupling.

#### Scenario: Developer adds a new feature to frontend
- **WHEN** a developer adds a feature named "product-catalog"
- **THEN** they create `frontend/src/features/product-catalog/` with subfolders: `ui/` (React components), `api/` (API client functions), `model/` (types and stores), `tests/`

#### Scenario: Import boundaries are enforced
- **WHEN** code in one slice attempts to import from another slice
- **THEN** the import is allowed only if the target is a public export from that slice's `index.ts`
- **THEN** deeply nested imports (e.g., `features/user/ui/components/Button`) are prohibited; use `features/user/ui` instead

#### Scenario: Shared layer for cross-slice utilities
- **WHEN** code is needed across multiple slices (e.g., HTTP interceptors, utility functions)
- **THEN** it lives in `frontend/src/shared/` with subfolders: `ui/` (shared components), `api/` (shared client config), `lib/` (utility functions), `types/` (global types)

### Requirement: Version control files (.gitignore) SHALL exclude build artifacts, dependencies, and environment secrets
The .gitignore file SHALL prevent accidental commits of generated files, node_modules, Python cache, and .env files.

#### Scenario: Backend generated files are ignored
- **WHEN** a developer runs backend migrations or builds
- **THEN** generated files like `__pycache__/`, `*.pyc`, `.pytest_cache/`, `venv/` are automatically excluded from git

#### Scenario: Frontend generated files are ignored
- **WHEN** a developer runs npm install or build
- **THEN** generated files like `node_modules/`, `dist/`, `.cache/` are automatically excluded from git

#### Scenario: Environment files are never committed
- **WHEN** a developer creates a `.env` file locally
- **THEN** git refuses to track it; it never appears in commits or history

### Requirement: Documentation files SHALL provide architecture overview and setup instructions
README.md and related docs SHALL guide developers through local setup, architecture, and conventions.

#### Scenario: New developer clones the repo
- **WHEN** a new developer clones the repository
- **THEN** they read README.md and understand: project purpose, tech stack, folder structure, how to start backend and frontend locally

#### Scenario: Developer understands code organization
- **WHEN** a developer reads README.md
- **THEN** they see a diagram or table explaining backend feature structure and frontend FSD layout
- **THEN** they understand where to add a new feature for backend vs. frontend

#### Scenario: Setup instructions are clear
- **WHEN** following README.md
- **THEN** the developer can run backend (e.g., `python -m uvicorn backend.src.main:app`) and frontend (e.g., `npm run dev`) with 5 minutes of effort
- **THEN** both services are accessible and health-check endpoints respond
