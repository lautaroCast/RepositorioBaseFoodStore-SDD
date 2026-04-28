## Why

Food Store is a complex multi-tier system requiring coordination between frontend, backend, and infrastructure. Without proper project structure from the start, development becomes chaotic—duplicate code, inconsistent patterns, and difficult onboarding for new developers. This change establishes the foundation: repository structure, CI/CD setup, and conventions that all 15 subsequent changes will depend on.

## What Changes

- Initialize Git repository with proper .gitignore for Node.js + Python + environment files
- Create backend folder structure following **feature-first modular architecture** (each feature owns its router, service, repository, model)
- Create frontend folder structure following **Feature-Sliced Design (FSD)** with import boundaries and shared modules
- Add README.md with project overview, setup instructions, and architecture diagram
- Add .env.example with template variables for backend (FastAPI, PostgreSQL, JWT secret) and frontend (API base URL)
- Document repository conventions: commit messages, branch naming, PR requirements
- Setup GitHub Actions CI/CD pipeline for basic linting and test runs

## Capabilities

### New Capabilities
- `project-structure`: Defines repository layout, folder organization, and import boundaries for backend and frontend
- `development-setup`: Establishes environment files, configuration templates, and local development guidelines
- `ci-cd-foundation`: GitHub Actions workflow for automated linting and testing on every push

### Modified Capabilities
- (None—this is the initial change)

## Impact

- **Code**: All future code must follow the structure and conventions defined here
- **Teams**: Developers working on frontend and backend can develop in parallel immediately
- **Onboarding**: New team members clone the repo and have a clear, documented structure to follow
- **Testing & CI**: Every change from here on benefits from automated testing and linting
