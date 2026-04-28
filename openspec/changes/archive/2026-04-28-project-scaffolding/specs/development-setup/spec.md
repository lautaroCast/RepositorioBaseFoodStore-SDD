## ADDED Requirements

### Requirement: Environment configuration SHALL use .env files with a .env.example template
Developers SHALL configure backend and frontend via environment variables loaded from .env files. .env.example SHALL document all required variables.

#### Scenario: Backend environment variables
- **WHEN** backend starts
- **THEN** it reads from `.env` for: `DATABASE_URL`, `JWT_SECRET`, `JWT_EXPIRY`, `CORS_ORIGINS`, `LOG_LEVEL`, `ENVIRONMENT` (local/staging/prod)
- **THEN** missing required variables cause startup failure with clear error message

#### Scenario: Frontend environment variables
- **WHEN** frontend builds
- **THEN** it reads from `.env` for: `VITE_API_BASE_URL`, `VITE_LOG_LEVEL`, `VITE_ENVIRONMENT`
- **THEN** builds with clear configuration for API endpoint and logging

#### Scenario: .env.example serves as documentation
- **WHEN** a new developer clones the repo
- **THEN** they see `.env.example` with all environment variables and example values
- **THEN** they can copy `.env.example` to `.env` and start locally with sensible defaults
- **THEN** `.env.example` is tracked in git; `.env` is ignored

#### Scenario: Production-ready environment docs
- **WHEN** deploying to production
- **THEN** deployment pipeline or documentation specifies which `.env` variables must be set for production
- **THEN** sensitive values (JWT_SECRET, DATABASE_URL) are never committed; they come from CI/CD secrets

### Requirement: Development conventions SHALL be documented (commit messages, branch naming, PR process)
README.md or CONTRIBUTING.md SHALL define coding standards and collaboration workflows.

#### Scenario: Developer creates a feature branch
- **WHEN** starting work on a feature
- **THEN** they follow branch naming: `feature/{feature-name}` or `fix/{issue-number}`
- **THEN** their commits follow conventional commit format: `feat:`, `fix:`, `docs:`, `test:`, etc.
- **THEN** this is documented so all team members follow the same pattern

#### Scenario: Pull request process is clear
- **WHEN** a developer opens a PR
- **THEN** PR template (if present) guides them to fill title, description, linked issue, and checklist
- **THEN** reviewers know what to look for (code quality, tests, documentation)

#### Scenario: Code style consistency
- **WHEN** backend code is written
- **THEN** it follows Python conventions documented in README (e.g., black formatting, isort import order, type hints)
- **WHEN** frontend code is written
- **THEN** it follows TypeScript/React conventions (e.g., ESLint config, naming patterns)

### Requirement: Local development environment SHALL be reproducible
Setup instructions SHALL allow any developer to run both backend and frontend locally with minimal configuration.

#### Scenario: Backend local setup
- **WHEN** developer follows backend setup steps
- **THEN** they can install dependencies (`pip install -r requirements.txt` or equivalent)
- **THEN** they can run database migrations
- **THEN** they can start the backend server on localhost:8000

#### Scenario: Frontend local setup
- **WHEN** developer follows frontend setup steps
- **THEN** they can install dependencies (`npm install`)
- **THEN** they can start the dev server on localhost:5173 (or similar)
- **THEN** the frontend connects to the local backend API

#### Scenario: Services are independently runnable
- **WHEN** either backend or frontend is running
- **THEN** the other is not required to be running (though some features may degrade gracefully)
- **THEN** developers can work on backend or frontend independently
