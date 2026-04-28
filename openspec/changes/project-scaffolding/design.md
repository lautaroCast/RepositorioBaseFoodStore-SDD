## Context

Food Store is building a full-stack e-commerce platform with distinct backend (FastAPI) and frontend (React) teams. Both will need to work in parallel; without shared conventions now, integration becomes painful. The project needs:
- Clear separation of concerns: backend code lives separately from frontend
- Consistent import/export patterns within each side
- Environment configuration that works for local development, testing, and production
- Automated tooling to catch basic errors before code review

The team wants to follow proven patterns: **feature-first modular architecture** for backend (every feature owns its routes, services, repositories) and **Feature-Sliced Design (FSD)** for frontend (slices, segments, shared layer).

## Goals / Non-Goals

**Goals:**
- Establish repository structure that supports parallel frontend/backend development
- Define import boundaries and module organization conventions
- Setup CI/CD foundation for linting and tests
- Document development workflow and conventions
- Create templates (.env.example, README) that accelerate onboarding

**Non-Goals:**
- Implement actual API endpoints or React components (those are later changes)
- Setup production infrastructure or deployment pipelines (separate scope)
- Configure advanced build optimizations or bundler tweaks
- Establish security policies or audit logging (separate changes handle that)

## Decisions

### Decision 1: Feature-First Backend Architecture
**Choice**: Each backend feature (users, products, orders, etc.) owns its own folder with router.py, service.py, repository.py, model.py, and tests.
**Rationale**: 
- Encapsulates feature logic, reducing cross-feature dependencies
- Makes it obvious where to add new code for a feature
- Teams can work on features in parallel with minimal merge conflicts
- Easy to move a feature to a separate service later
**Alternatives considered**:
- Layered architecture (all routers together, all services together) → harder to locate feature logic, more cross-cutting concerns
- Domain-Driven Design with aggregates → adds complexity early; feature-first is simpler and can evolve to DDD later

### Decision 2: Feature-Sliced Design for Frontend
**Choice**: Organize by slice (user, product, cart, etc.), each slice has segments (ui/, api/, model/), with shared layer for cross-slice utilities.
**Rationale**:
- Scales better than flat folder structure as feature count grows
- Import boundaries prevent accidental circular dependencies
- Teams own slices end-to-end (UI, logic, API calls)
- Reduces coupling between unrelated features
**Alternatives considered**:
- Flat/component-based (all components in one folder) → becomes unmanageable at 100+ components
- Layered (all pages together, all components together) → violates single responsibility, hard to modify one feature

### Decision 3: Environment Configuration via .env Files
**Choice**: Use .env.example as a template. Developers copy to .env (git-ignored). Both frontend and backend use environment variables for config.
**Rationale**:
- Standard practice, familiar to all developers
- Keeps secrets out of version control
- Same pattern works for local, staging, production (just different .env file)
**Alternatives considered**:
- Config files in YAML/JSON → harder to keep secrets out, more complex in multi-language project
- Environment variables only (no .env) → requires manual setup for each developer

### Decision 4: GitHub Actions for CI/CD
**Choice**: Basic workflow that runs on every push: lint backend (black, isort, flake8), lint frontend (eslint), run tests if they exist.
**Rationale**:
- Catches basic errors before code review
- Free tier on GitHub is sufficient for this project size
- Runs locally (pre-commit) or in CI—same checks everywhere
- Foundation for later: coverage, security scanning, deployment pipelines
**Alternatives considered**:
- No CI/CD → errors slip to production; slower feedback
- Complex multi-stage pipeline → overkill at this stage, adds maintenance burden

## Risks / Trade-offs

**[Risk] Developers ignore import boundaries in frontend**
→ Mitigation: Document boundaries in README and validate with ESLint import plugin during setup. Code review catches violations.

**[Risk] .env.example gets out of sync with actual environment variables**
→ Mitigation: Add a pre-commit hook (or CI check) to validate that .env has all keys from .env.example. Document update process in README.

**[Risk] Feature-first backend leads to code duplication across features**
→ Mitigation: Extract shared logic to a `core/` or `shared/` folder early. Establish pattern in design: when to extract vs. when to tolerate duplication.

**[Trade-off] Strict folder structure might feel rigid initially**
→ Mitigation: Revisit after 3-4 features are implemented. Be willing to refactor if the structure doesn't serve the team.

## Migration Plan

1. Create directory structure (no code changes yet—just folders)
2. Move existing README.md to new location and update with architecture diagram
3. Create .env.example with all backend and frontend variables
4. Create GitHub Actions workflow file
5. Update .gitignore to include environment files, node_modules, __pycache__, etc.
6. Commit and document in a SETUP.md guide for developers

No rollback needed—this is infrastructure, not data.

## Open Questions

- Should we add a Makefile or Docker Compose for local development? (Out of scope for this change, but good to decide now)
- Should pre-commit hooks be mandatory or optional? (Suggest optional at first, enforce later)
- Do we need a separate docs/ folder for architecture diagrams, API specs, DB ERD? (Plan for next change—keep this one focused)
