# Contributing to Food Store

¡Gracias por querer contribuir a Food Store! Este documento describe el proceso de desarrollo.

---

## Flujo de Git

### Branch Naming

```
feature/nombre-descriptivo          # Nuevas features
fix/numero-del-issue                 # Bug fixes
refactor/area                        # Refactorings
docs/descripcion                     # Documentación
test/descripcion                     # Tests
```

Ejemplos:
- `feature/user-authentication`
- `fix/cart-calculation-bug`
- `refactor/product-service`
- `docs/api-endpoints`

### Commits — Conventional Commits

Usar el formato: `<type>(<scope>): <subject>`

Types:
- `feat:` Nueva funcionalidad
- `fix:` Bug fix
- `refactor:` Cambio de código sin alterar funcionalidad
- `test:` Agregar o actualizar tests
- `docs:` Cambios en documentación
- `chore:` Tareas de build, dependencies, etc.
- `perf:` Mejoras de rendimiento
- `ci:` Cambios en CI/CD

Ejemplos:
```bash
git commit -m "feat(auth): implement JWT refresh token rotation"
git commit -m "fix(products): correct price calculation for bulk orders"
git commit -m "refactor(cart): simplify state management with Zustand"
git commit -m "test(orders): add unit tests for order FSM"
git commit -m "docs(readme): add architecture diagram"
```

**Reglas:**
- ✅ Commits pequeños y enfocados (una cosa por commit)
- ✅ Mensajes en inglés
- ❌ Commits genéricos ("arreglar código", "cambios varios")
- ❌ Commits muy grandes (refactor + feature + docs a la vez)

---

## Pull Requests

### Template

Cuando abras un PR, incluye:

```markdown
## What does this PR do?
Brief description of the changes.

## Why?
Explain the motivation or problem this solves.

## Related Issue
Closes #123

## How to test?
Steps to verify the changes work correctly.

## Screenshots (if UI changes)
Add before/after screenshots if applicable.

## Checklist
- [ ] Code follows style guidelines
- [ ] I have commented complex logic
- [ ] Tests are added/updated
- [ ] Documentation is updated
- [ ] Linting passes locally
```

### PR Requirements

✅ **Code Style**: Debe pasar linting local  
✅ **Tests**: Agregar tests para nueva funcionalidad  
✅ **Documentation**: Actualizar README/docs si es necesario  
✅ **CI/CD**: GitHub Actions debe pasar  
✅ **Review**: Al menos un revisor debe aprobar  

---

## Code Style

### Backend (Python)

**Formatter**: `black`
```bash
black backend/src --line-length=100
```

**Import Sorter**: `isort`
```bash
isort backend/src
```

**Linter**: `flake8`
```bash
flake8 backend/src --max-line-length=100 --ignore=E203,W503
```

**Run all at once**:
```bash
black backend/src && isort backend/src && flake8 backend/src
```

**Guidelines**:
- Type hints obligatorios en funciones públicas
- Docstrings en functions y classes (formato Google-style)
- Max line length: 100 caracteres
- Imports: stdlib → third-party → local

Example:
```python
def create_user(email: str, password: str) -> User:
    """Create a new user with hashed password.
    
    Args:
        email: User email address
        password: Plain text password (will be hashed)
        
    Returns:
        Created User instance
        
    Raises:
        ValueError: If email already exists
    """
    if user_exists(email):
        raise ValueError(f"User {email} already exists")
    
    hashed_pw = hash_password(password)
    return User(email=email, password_hash=hashed_pw)
```

### Frontend (TypeScript + React)

**Linter**: `eslint`
```bash
npm run lint
```

**Type Checking**: `tsc`
```bash
npm run type-check
```

**Guidelines**:
- Components: PascalCase (`UserCard.tsx`)
- Utilities: camelCase (`formatDate.ts`)
- Constants: UPPER_SNAKE_CASE (`MAX_RETRIES`)
- Props interfaces: `<Component>Props`
- Zustand stores: camelCase (`useAuthStore`)

Example:
```typescript
// components/ProductCard.tsx
interface ProductCardProps {
  id: string;
  name: string;
  price: number;
  onAddToCart: (id: string) => void;
}

export const ProductCard: React.FC<ProductCardProps> = ({
  id,
  name,
  price,
  onAddToCart,
}) => {
  return (
    <div className="card">
      <h2>{name}</h2>
      <p>${price.toFixed(2)}</p>
      <button onClick={() => onAddToCart(id)}>Add to Cart</button>
    </div>
  );
};
```

---

## Testing

### Backend Tests

Use `pytest`:
```bash
pytest backend/tests
pytest backend/tests -v              # Verbose
pytest backend/tests -k "test_name"  # Run specific test
pytest backend/tests --cov           # With coverage
```

Example test:
```python
import pytest
from backend.src.features.users.service import create_user

@pytest.mark.asyncio
async def test_create_user_success():
    """Test successful user creation."""
    user = await create_user(
        email="test@example.com",
        password="securepass123"
    )
    assert user.email == "test@example.com"
    assert user.password_hash is not None

@pytest.mark.asyncio
async def test_create_user_duplicate_email():
    """Test that duplicate emails are rejected."""
    await create_user(email="test@example.com", password="pass1")
    
    with pytest.raises(ValueError, match="already exists"):
        await create_user(email="test@example.com", password="pass2")
```

### Frontend Tests

Use `vitest`:
```bash
npm test
npm test -- --watch      # Watch mode
npm test -- --ui         # UI mode
npm test -- --coverage   # With coverage
```

Example test:
```typescript
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import { ProductCard } from './ProductCard';

describe('ProductCard', () => {
  it('renders product information', () => {
    render(
      <ProductCard
        id="1"
        name="Apple"
        price={2.99}
        onAddToCart={() => {}}
      />
    );
    
    expect(screen.getByText('Apple')).toBeInTheDocument();
    expect(screen.getByText('$2.99')).toBeInTheDocument();
  });

  it('calls onAddToCart when button is clicked', () => {
    const handleAdd = vi.fn();
    const { getByRole } = render(
      <ProductCard
        id="1"
        name="Apple"
        price={2.99}
        onAddToCart={handleAdd}
      />
    );
    
    fireEvent.click(getByRole('button'));
    expect(handleAdd).toHaveBeenCalledWith('1');
  });
});
```

---

## OPSX Workflow

Food Store usa **OPSX** (OpenSpec) para gestionar cambios. Cada feature/fix sigue:

```
1. /opsx:explore       ← (Opcional) Pensar en el problema
2. /opsx:propose       ← Crear propuesta + diseño + tareas
3. /opsx:apply         ← Implementar tareas
4. /opsx:archive       ← Cerrar y sincronizar
```

**Para desarrolladores:**

1. Un change ya fue propuesto y está listo para implement
2. Ejecutar: `/opsx:apply <change-name>`
3. Completar tareas una por una
4. Cuando termines: `/opsx:archive <change-name>` (solo mantainers)

Ver `docs/CHANGES.md` para el roadmap completo.

---

## Development Environment Setup

### Local Setup (One Time)

```bash
# Backend
cd backend
python -m venv .venv
source .venv/bin/activate        # Linux/Mac
.venv\Scripts\activate           # Windows
pip install -r requirements.txt

# Frontend
cd ../frontend
npm install
```

### Running Locally (Daily)

**Terminal 1 — Backend:**
```bash
cd backend
source .venv/bin/activate  # or .venv\Scripts\activate on Windows
uvicorn backend.src.main:app --reload
# API available at http://localhost:8000
```

**Terminal 2 — Frontend:**
```bash
cd frontend
npm run dev
# App available at http://localhost:5173
```

### Pre-commit Hooks (Optional but Recommended)

Install `pre-commit` framework:
```bash
pip install pre-commit
pre-commit install
```

This will automatically run linting before each commit.

---

## Debugging Tips

### Backend
- FastAPI Swagger Docs: `http://localhost:8000/docs`
- Use `print()` or `logging.debug()` to debug
- Check `.env` if variables are not loading

### Frontend
- Browser DevTools (F12)
- React DevTools extension
- TanStack Query DevTools for data debugging
- Check `.env` if API_BASE_URL is incorrect

---

## Getting Help

- **Questions?** Open a discussion in GitHub Discussions
- **Found a bug?** Open an issue with reproduction steps
- **Have an idea?** Open a feature request issue

---

## Recognition

Contributors will be recognized in:
- README.md
- GitHub contributors page
- Release notes (for significant contributions)

Thank you for contributing! 🙏
