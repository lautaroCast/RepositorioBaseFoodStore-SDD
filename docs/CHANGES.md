# Mapa Completo de Changes — Food Store

## Resumen Ejecutivo

Food Store se desarrolla a través de **16 changes incrementales**, organizados en bloques lógicos:

- **SPRINT 0** (6 changes): Infraestructura, setup y patrones base
- **SPRINT 1** (3 changes): Autenticación, autorización y seguridad
- **SPRINT 2** (4 changes): Catálogo de productos, ingredientes, categorías
- **SPRINT 3** (2 changes): Carrito, pedidos y máquina de estados
- **SPRINT 4** (1 change): Integración MercadoPago y pagos

Cada change es una unidad atómica: proposal.md (qué), design.md (cómo), tasks.md (checklist).

---

## SPRINT 0 — Infraestructura y Patrones Base

### Change 00: `project-scaffolding`

**¿Qué cubre?**
- Inicialización del repositorio Git
- Estructura de carpetas backend (feature-first) y frontend (FSD)
- .gitignore, README.md base, .env.example

**Historias de usuario implementadas:**
- US-000: Inicialización del repositorio y estructura del proyecto

**¿De qué depende?**
- Ninguna (entrada punto 1)

**Relación con otras historias de usuario:**
- Todas las demás dependen de esto: es la fundación del proyecto.

---

### Change 01: `backend-core-setup`

**¿Qué cubre?**
- Configuración de FastAPI, dependencias core
- Setup de PostgreSQL y Alembic
- Variables de entorno, CORS middleware, rate limiting middleware
- Modelo de datos inicial (ERD v5 completo)
- Implementación de BaseRepository[T] y Unit of Work pattern
- Implementación de get_current_user y require_role

**Historias de usuario implementadas:**
- US-000a: Configuración del entorno backend (FastAPI + dependencias)
- US-000b: Configuración de PostgreSQL, migraciones y seed data
- US-000d: Implementación de patrones base (BaseRepository, UoW, dependencias FastAPI)

**¿De qué depende?**
- Change 00 (estructura de carpetas existente)

**Relación con otras historias de usuario:**
- Base para todos los cambios backend posteriores
- Establece las capas: Router → Service → UoW → Repository → Model
- Define la convención de errores RFC 7807

---

### Change 02: `frontend-core-setup`

**¿Qué cubre?**
- Configuración de React + TypeScript + Vite
- Setup de TanStack Query, Zustand, Axios
- Configuración de Tailwind CSS
- Axios interceptor con JWT refresh automático
- Los 4 stores de Zustand: authStore, cartStore, paymentStore, uiStore
- Estructura FSD con límites de importación

**Historias de usuario implementadas:**
- US-000c: Configuración del entorno frontend
- US-000e: Configuración de los stores de Zustand

**¿De qué depende?**
- Change 00 (estructura de carpetas existente)

**Relación con otras historias de usuario:**
- Base para todos los cambios frontend posteriores
- Establece la separación Zustand (estado del cliente) vs TanStack Query (estado del servidor)

---

### Change 03: `database-seed-and-validation`

**¿Qué cubre?**
- Script de seed data para roles, estados de pedido, formas de pago, usuario admin
- Migraciones Alembic generadas desde modelos SQLModel
- Validaciones de integridad referencial
- Soft delete y campos de auditoría en todas las entidades
- Snapshot pattern en Pedido y DetallePedido

**Historias de usuario implementadas:**
- Continuación de US-000b (completar seed data)

**¿De qué depende?**
- Change 01 (backend core con BaseRepository y UoW)

**Relación con otras historias de usuario:**
- Prepara la BD para que TODAS las operaciones funcionen correctamente
- Sin esto, los roles no existen y RBAC no funciona

---

### Change 04: `error-handling-and-validation`

**¿Qué cubre?**
- Middleware global de manejo de excepciones
- Formato de error consistente RFC 7807
- Clases de error custom (ValidationError, UnauthorizedError, ForbiddenError, NotFoundError)
- Validación y sanitización de inputs (Pydantic v2)
- Protección contra XSS en frontend (DOMPurify)
- Helmet headers en backend

**Historias de usuario implementadas:**
- US-068: Manejo de errores estandarizado en backend
- US-074: Validación y sanitización de inputs

**¿De qué depende?**
- Change 01 (FastAPI setup)

**Relación con otras historias de usuario:**
- Aplicable a TODOS los endpoints: los errores tienen formato consistente

---

### Change 05: `feature-base-patterns`

**¿Qué cubre?**
- Estructura base de un módulo backend (model.py, schemas.py, repository.py, service.py, router.py)
- Convenciones de nomenclatura (snake_case backend, camelCase frontend)
- Ejemplo de patrón Repository con queries específicas del dominio
- Ejemplo de patrón Service con validaciones de negocio
- Ejemplo de patrón Router con schemas Pydantic Create/Update/Read

**Historias de usuario implementadas:**
- Ninguna nueva (es de refactoring/convención)

**¿De qué depende?**
- Change 01 (BaseRepository y UoW)
- Change 04 (validaciones y errores)

**Relación con otras historias de usuario:**
- Establece el template para todos los cambios posteriores
- Asegura consistencia arquitectónica

---

## SPRINT 1 — Autenticación, Autorización y Seguridad

### Change 10: `auth-system`

**¿Qué cubre?**
- Endpoint POST /api/v1/auth/register (registro de cliente)
- Endpoint POST /api/v1/auth/login (login con rate limiting 5/15min)
- Endpoint POST /api/v1/auth/refresh (renovación de token con rotación)
- Endpoint POST /api/v1/auth/logout (invalidar refresh token)
- JWT access token (30 min) + refresh token (7 días)
- Tabla RefreshToken con versionamiento
- Hashing con bcrypt (cost >= 10)
- Replay attack detection

**Historias de usuario implementadas:**
- US-001: Registro de cliente
- US-002: Login de usuario
- US-003: Refresh de token
- US-004: Logout
- US-002 & US-073: Rate limiting en login (5 intentos/15 min)

**¿De qué depende?**
- Change 01 (backend core, patrones)
- Change 03 (seed data de usuario admin)
- Change 04 (validación y errores)

**Relación con otras historias de usuario:**
- Punto de entrada para acceso al sistema
- TODAS las operaciones autenticadas dependen de esto
- El interceptor JWT del frontend (Change 02) se configura aquí

---

### Change 11: `rbac-authorization`

**¿Qué cubre?**
- Implementación de require_role como dependencia FastAPI
- Protección de rutas por rol (ADMIN, STOCK, PEDIDOS, CLIENT)
- Endpoint PUT /api/v1/admin/users/{id}/role (asignación de roles)
- Validación de que ADMIN no se quita a sí mismo el único rol ADMIN
- Verificación de roles en JWT (payload contiene roles)
- Guard de autorización en todas las rutas protegidas

**Historias de usuario implementadas:**
- US-005: Gestión de roles (RBAC)
- US-006: Protección de rutas por rol

**¿De qué depende?**
- Change 10 (auth system, tokens con roles en JWT)
- Change 03 (roles seeded en BD)

**Relación con otras historias de usuario:**
- Ejecuta las validaciones de cada capa posterior (productos STOCK/ADMIN, pedidos PEDIDOS/ADMIN, etc.)
- Sin RBAC, no hay seguridad operativa

---

### Change 12: `user-management-and-addresses`

**¿Qué cubre?**
- CRUD completo de usuarios (solo ADMIN)
- Endpoint GET /api/v1/usuarios (listar todos)
- Endpoint GET /api/v1/usuarios/{id} (detalle)
- Endpoint PUT /api/v1/usuarios/{id} (actualizar)
- Endpoint DELETE /api/v1/usuarios/{id} (soft delete)
- CRUD completo de direcciones por usuario
- Endpoint POST /api/v1/direcciones (crear)
- Endpoint GET /api/v1/direcciones (listar del usuario autenticado)
- Endpoint PUT /api/v1/direcciones/{id} (actualizar)
- Endpoint PATCH /api/v1/direcciones/{id}/principal (marcar como predeterminada)
- Endpoint DELETE /api/v1/direcciones/{id}

**Historias de usuario implementadas:**
- US-024: Gestión de direcciones de entrega (múltiples por usuario)
- US-025: Ver/editar propias direcciones (CLIENT only)
- US-026: Cambiar dirección predeterminada
- US-027: Eliminar dirección de entrega

**¿De qué depende?**
- Change 11 (RBAC: solo CLIENT ve sus propias direcciones, solo ADMIN gestiona usuarios)
- Change 01 (modelo de datos de DireccionEntrega)

**Relación con otras historias de usuario:**
- Necesario para crear pedidos (Change 30)
- La dirección se snapshots en cada pedido (inmutable)

---

## SPRINT 2 — Catálogo de Productos

### Change 20: `categories-and-hierarchy`

**¿Qué cubre?**
- CRUD de categorías con jerarquía (FK autoreferencial padre_id)
- CTE recursiva en PostgreSQL para obtener árbol completo
- Endpoint POST /api/v1/categorias (crear)
- Endpoint GET /api/v1/categorias (listar con jerarquía)
- Endpoint GET /api/v1/categorias/{id} (detalle)
- Endpoint PUT /api/v1/categorias/{id} (actualizar)
- Endpoint DELETE /api/v1/categorias/{id} (soft delete con validación)
- Validación: no permite ciclos, no permite asignar categoría como padre de sí misma
- Validación: no permite eliminar categoría con productos activos

**Historias de usuario implementadas:**
- US-007: Crear categoría
- US-008: Actualizar categoría
- US-009: Validación de ciclos en jerarquía
- US-010: No eliminar categoría con productos

**¿De qué depende?**
- Change 11 (RBAC: solo ADMIN/STOCK pueden crear categorías)
- Change 01 (modelo de datos de Categoria)

**Relación con otras historias de usuario:**
- Necesario para Change 21 (productos)
- Afecta la búsqueda y filtrado de productos

---

### Change 21: `ingredients-and-allergens`

**¿Qué cubre?**
- Modelo Ingrediente con campo es_alergeno
- CRUD de ingredientes
- Endpoint POST /api/v1/ingredientes (crear)
- Endpoint GET /api/v1/ingredientes (listar)
- Endpoint GET /api/v1/ingredientes/{id} (detalle)
- Endpoint PUT /api/v1/ingredientes/{id} (actualizar)
- Endpoint DELETE /api/v1/ingredientes/{id} (soft delete)
- Tabla ProductoIngrediente (M2M) con campo es_removible
- Validación de alérgenos en UI (badge informativa)

**Historias de usuario implementadas:**
- US-017: Gestionar ingredientes y alérgenos de productos

**¿De qué depende?**
- Change 11 (RBAC: solo ADMIN/STOCK)
- Change 01 (modelo de datos de Ingrediente)

**Relación con otras historias de usuario:**
- Necesario para personalización de pedidos (US-030)
- Información crítica para clientes con restricciones dietarias

---

### Change 22: `products-catalog`

**¿Qué cubre?**
- CRUD completo de productos
- Modelo Producto con precio (NUMERIC), stock_cantidad (INTEGER), disponible (BOOLEAN)
- Tabla ProductoCategoria (M2M)
- Endpoint GET /api/v1/productos (listar con filtros: categoría, búsqueda, rango precio, disponibilidad, paginación)
- Endpoint GET /api/v1/productos/{id} (detalle con ingredientes, categorías, stock)
- Endpoint POST /api/v1/productos (crear: solo ADMIN/STOCK)
- Endpoint PUT /api/v1/productos/{id} (actualizar: solo ADMIN/STOCK)
- Endpoint PATCH /api/v1/productos/{id}/disponibilidad (cambiar disponible: solo ADMIN/STOCK)
- Endpoint DELETE /api/v1/productos/{id} (soft delete: solo ADMIN)
- Validación: el catálogo público solo muestra productos disponibles (disponible=true, eliminado_en IS NULL)
- Para STOCK/ADMIN: pueden ver productos no disponibles

**Historias de usuario implementadas:**
- US-015: Crear producto (con precio, stock, ingredientes)
- US-018: Listar catálogo público con filtros
- US-020: Ver detalle de producto
- US-021: Gestionar stock (actualizar cantidad)
- US-022: Deshabilitar producto (soft delete)

**¿De qué depende?**
- Change 11 (RBAC: filtrado por rol)
- Change 20 (categorías jerárquicas para filtrado)
- Change 21 (ingredientes)
- Change 01 (modelo de datos de Producto)

**Relación con otras historias de usuario:**
- CRÍTICO: sin catálogo, no hay compra posible
- Afecta a carrito (Change 30) y pedidos (Change 31)

---

## SPRINT 3 — Carrito, Pedidos y Máquina de Estados

### Change 30: `shopping-cart-client`

**¿Qué cubre?**
- Implementación de cartStore (Zustand) en frontend
- Persistencia en localStorage
- Acciones: addItem, removeItem, updateQuantity, clearCart
- Selectores: totalItems, totalPrice, getItem, subtotal
- Personalización de items (exclusión de ingredientes con array de IDs)
- El carrito persiste al cerrar navegador, refresh y logout/login
- Duplicación: agregar producto ya existente incrementa cantidad
- Validación: solo excluir ingredientes que el producto tiene

**Historias de usuario implementadas:**
- US-029: Agregar productos al carrito
- US-030: Personalización (excluir ingredientes)
- US-031: Modificar cantidades en carrito
- US-032: Eliminar producto del carrito
- US-033: Ver resumen del carrito (total, items)
- US-034: Carrito persistente (localStorage)

**¿De qué depende?**
- Change 02 (Zustand setup, localStorage middleware)
- Change 22 (catálogo de productos disponibles)

**Relación con otras historias de usuario:**
- Antecedente directo a Change 31 (crear pedido desde carrito)
- Sin carrito, no hay compra

---

### Change 31: `order-creation-and-management`

**¿Qué cubre?**
- Endpoint POST /api/v1/pedidos (crear pedido desde carrito)
  - Recibe items (producto_id, cantidad, personalizacion[])
  - Recibe direccion_id y forma_pago_codigo
  - Valida stock suficiente (SELECT FOR UPDATE dentro de UoW)
  - Genera snapshots: nombre_snapshot, precio_snapshot, direccion_snapshot
  - Calcula totales: subtotal x item + costo_envío
  - Crea Pedido + DetallePedido + primer HistorialEstadoPedido
  - ATÓMICO: todo o nada (UoW)
- Endpoint GET /api/v1/pedidos (listar)
  - CLIENT: solo propios pedidos
  - PEDIDOS/ADMIN: todos los pedidos
  - Filtros: estado, fecha, paginación
- Endpoint GET /api/v1/pedidos/{id} (detalle completo)
  - Incluye detalles, snapshots, historial de estados, pagos
- Endpoint GET /api/v1/pedidos/{id}/historial (historial append-only)
  - Orden: ORDER BY created_at ASC
  - Cada registro: estado_desde, estado_hacia, timestamp, usuario, observación

**Historias de usuario implementadas:**
- US-035: Crear pedido desde carrito (validación, snapshots, atomicidad)
- US-036: Validar stock suficiente (SELECT FOR UPDATE)
- US-037: Snapshot de precios en pedido (inmutable)
- US-038: Snapshot de dirección en pedido (inmutable)
- US-049: Ver propios pedidos (CLIENT)
- US-050: Ver todos los pedidos (ADMIN/PEDIDOS)
- US-051: Historial de estados (audit trail)

**¿De qué depende?**
- Change 30 (carrito con items)
- Change 12 (direcciones de entrega)
- Change 11 (RBAC: CLIENT vs ADMIN/PEDIDOS)
- Change 01 (Unit of Work, transacciones atómicas)
- Change 03 (seed data de formas de pago y estados de pedido)

**Relación con otras historias de usuario:**
- Punto central del sistema: agrupa todas las operaciones
- Antecedente a Change 32 (máquina de estados)
- Antecedente a Change 40 (pagos)

---

### Change 32: `order-state-machine-and-transitions`

**¿Qué cubre?**
- Máquina de estados de 6 estados: PENDIENTE, CONFIRMADO, EN_PREP, EN_CAMINO, ENTREGADO, CANCELADO
- Validación de transiciones (solo las permitidas según FSM)
- Endpoint PATCH /api/v1/pedidos/{id}/estado (avanzar estado manualmente)
  - CONFIRMADO → EN_PREP (PEDIDOS/ADMIN)
  - EN_PREP → EN_CAMINO (PEDIDOS/ADMIN)
  - EN_CAMINO → ENTREGADO (PEDIDOS/ADMIN)
  - Recibe observación opcional que se guarda en historial
- Endpoint PATCH /api/v1/pedidos/{id}/cancelar (cancelar pedido)
  - Desde PENDIENTE (CLIENT/PEDIDOS/ADMIN)
  - Desde CONFIRMADO (PEDIDOS/ADMIN)
  - Desde EN_PREP (solo ADMIN)
  - Si el pedido fue CONFIRMADO: restaurar stock atómicamente
  - Requiere motivo (observación)
- Validación de estados terminales: ENTREGADO y CANCELADO no permiten transiciones salientes
- Toda transición registrada en HistorialEstadoPedido (append-only, nunca UPDATE/DELETE)

**Historias de usuario implementadas:**
- US-039: Confirmar pedido (PENDIENTE → CONFIRMADO automático por pago)
- US-040: Avanzar a EN_PREP (PEDIDOS/ADMIN manual)
- US-041: Avanzar a EN_CAMINO (PEDIDOS/ADMIN manual)
- US-042: Marcar ENTREGADO (terminal)
- US-043: Cancelar pedido (con restauración de stock)
- US-044: Ver historial de estados (audit trail)

**¿De qué depende?**
- Change 31 (creación de pedidos con estados iniciales)
- Change 01 (Unit of Work: decremento/restauración atómica de stock)
- Change 11 (RBAC: permisos por rol)
- Change 03 (seed de estados de pedido)

**Relación con otras historias de usuario:**
- Implementa la lógica central de negocio del sistema
- Interactúa con Change 40 (pagos: transición automática PENDIENTE → CONFIRMADO)
- Sin esto, no hay flujo operativo de pedidos

---

## SPRINT 4 — Integración MercadoPago y Pagos

### Change 40: `mercadopago-integration`

**¿Qué cubre?**
- Tabla Pago: mp_payment_id, mp_status, external_reference, idempotency_key
- Endpoint POST /api/v1/pagos/crear (crear pago)
  - Recibe pedido_id y token de tarjeta (tokenizado vía SDK MP.js)
  - Genera idempotency_key UUID (único por transacción)
  - Llama SDK MercadoPago Python para crear orden
  - Registra en tabla Pago
  - Devuelve mp_payment_id y status inicial
- Endpoint POST /api/v1/pagos/webhook (IPN de MercadoPago)
  - Recibe notificación de MercadoPago (topic=payment)
  - Valida firma del webhook (autenticidad)
  - Consulta API MP para obtener estado real del pago
  - Actualiza registro en Pago (mp_status)
  - Si status = "approved": ATÓMICO vía UoW
    - Transición pedido PENDIENTE → CONFIRMADO (automática)
    - Decremento de stock
    - Registra en HistorialEstadoPedido
  - Si status = "rejected": pedido permanece PENDIENTE
  - Si status = "pending": pedido permanece PENDIENTE (espera confirmación posterior)
  - Responde HTTP 200 inmediatamente (evita reintentos de MP)
- Endpoint GET /api/v1/pagos/{pedido_id} (ver historial de intentos de pago)
- Frontend: integración con SDK MercadoPago.js para tokenización segura (PCI SAQ-A)
  - CardPayment component de @mercadopago/sdk-react
  - Tokenización en browser, token se envía al backend
  - Nunca pasan datos de tarjeta por Food Store

**Historias de usuario implementadas:**
- US-045: Crear pago con MercadoPago (PCI SAQ-A compliant)
- US-046: Procesar webhook IPN (confirmación automática de pago)
- US-047: Validar estado de pago y transición automática
- US-048: Reintentar pago (múltiples intentos por pedido)

**¿De qué depende?**
- Change 32 (máquina de estados: transición automática PENDIENTE → CONFIRMADO)
- Change 31 (pedidos con forma de pago)
- Change 01 (Unit of Work: atomicidad de transición + decremento stock)
- Change 11 (RBAC: solo CLIENT puede crear pago)

**Relación con otras historias de usuario:**
- CRÍTICO: sin esto, no hay operación de cobro
- Afecta la visualización del estado de pago en pedido (Change 50)
- Integración directa con servicio externo (riesgo de fallos)

---

## SPRINT 5 — Panel de Administración y Métricas

### Change 50: `admin-dashboard-and-metrics`

**¿Qué cubre?**
- Panel de administración (solo ADMIN)
- Dashboard con KPIs:
  - Total de ventas (sum(total) de pedidos CONFIRMADO/EN_CAMINO/ENTREGADO)
  - Cantidad de pedidos por estado (gráfico de barras con recharts)
  - Ingresos del mes (recharts: línea temporal)
  - Productos más vendidos (recharts: top 5)
  - Tasa de conversión (pedidos CONFIRMADO / pedidos totales)
- CRUD de productos desde panel (igual que Change 22, pero en UI)
- CRUD de categorías desde panel (igual que Change 20, pero en UI)
- Gestión de stock (actualizar cantidades, cambiar disponibilidad)
- Gestión de pedidos (ver todos, cambiar estado, ver historial)
- Gestión de usuarios (crear, editar, asignar roles, desactivar)
- Vistas: Cards, Tablas con paginación, Modales de confirmación
- Skeleton loaders mientras se cargan datos (TanStack Query)
- Toasts para confirmaciones y errores

**Historias de usuario implementadas:**
- US-052: Dashboard con métricas visuales
- US-053: CRUD de productos desde panel (UI)
- US-054: CRUD de usuarios y asignación de roles (UI)
- US-055: Gestión de stock visual (UI)
- US-056: Visualización de todos los pedidos con filtros (UI)

**¿De qué depende?**
- Change 22 (endpoints CRUD de productos)
- Change 20 (endpoints CRUD de categorías)
- Change 12 (endpoints CRUD de usuarios)
- Change 31 (endpoints de pedidos)
- Change 11 (RBAC: solo ADMIN accede a este panel)
- Change 02 (TanStack Query, recharts, Zustand)

**Relación con otras historias de usuario:**
- UI sobre APIs ya existentes
- Permite gestión operativa visual vs solo API
- Mejora experiencia de administrador

---

### Change 51: `client-storefront-and-checkout`

**¿Qué cubre?**
- Frontend completo para cliente
- Páginas:
  - Catálogo (GET /api/v1/productos con filtros, paginación, búsqueda debounce)
    - Skeleton loaders mientras carga
    - Filtros: categoría, rango precio, búsqueda por nombre
    - Paginación: página actual, total de items
    - Tarjetas de producto con precio, stock, botón "Agregar al carrito"
  - Detalle de producto (GET /api/v1/productos/{id})
    - Imagen, descripción completa, ingredientes con badges de alérgenos
    - Campo de personalización (checkboxes para excluir ingredientes)
    - Input de cantidad + botón "Agregar al carrito"
  - Carrito (cartStore + drawer/modal)
    - Items del carrito con subtotal por item
    - Botones: +/- cantidad, eliminar item
    - Botón "Checkout"
    - Total, costo envío, subtotal
  - Checkout (creación de pedido)
    - Resumen de items
    - Selector de dirección de entrega (GET /api/v1/direcciones)
    - Selector de forma de pago (formas activas)
    - Botón "Confirmar pedido" → POST /api/v1/pedidos
  - Pago (integración MercadoPago)
    - CardPayment component de SDK MercadoPago
    - Tokenización segura en browser
    - Botón "Pagar" → POST /api/v1/pagos/crear
    - Visualización de status (pendiente, procesando, aprobado, rechazado)
  - Mi cuenta (datos del usuario)
    - Perfil: nombre, email, teléfono
    - Mis direcciones (CRUD)
    - Mis pedidos (listado paginado)
      - Estado actual, total, fecha
      - Botón "Ver detalle"
  - Detalle de pedido (GET /api/v1/pedidos/{id})
    - Resumen: estado, total, fecha
    - Items del pedido (snapshots)
    - Historial de estados (timeline)
    - Status de pago (si existe)
    - Botón "Cancelar pedido" (si está en PENDIENTE o CONFIRMADO)
- Autenticación:
  - Login / Register (forms con validación)
  - Interceptor JWT: adjunta token y refresca automático en 401
  - Protected routes: redirect a login si no autenticado
- Estado:
  - authStore: sesión del usuario
  - cartStore: items del carrito
  - paymentStore: estado del pago
  - TanStack Query: productos, pedidos, direcciones

**Historias de usuario implementadas:**
- US-016: Navegar categorías (con filtros jerárquicos)
- US-018: Listar catálogo (búsqueda, paginación, filtros)
- US-019: Ver detalle de producto (incluyendo alérgenos)
- US-020: Ver precio y disponibilidad
- US-025: Gestionar propias direcciones (CRUD)
- US-026: Cambiar dirección predeterminada
- US-027: Eliminar dirección
- US-029: Agregar al carrito
- US-030: Personalización (excluir ingredientes)
- US-031: Modificar cantidades
- US-032: Eliminar del carrito
- US-033: Ver resumen del carrito
- US-034: Carrito persistente
- US-035: Crear pedido (checkout)
- US-044: Ver historial de estados
- US-047: Ver status de pago
- US-057: Navegar catálogo sin login (público)
- US-058: Ver detalle de producto sin login (público)

**¿De qué depende?**
- Change 22 (endpoints de productos)
- Change 12 (endpoints de direcciones)
- Change 31 (endpoints de pedidos, crear pedido)
- Change 40 (endpoints de pagos)
- Change 30 (cartStore implementado)
- Change 02 (TanStack Query, Zustand, Axios con JWT)
- Change 10 (autenticación JWT)

**Relación con otras historias de usuario:**
- Interfaz de usuario para cliente final
- Integra todas las capas anteriores
- Experiencia de compra completa

---

## Matriz de Dependencias

```
SPRINT 0
├─ Change 00: project-scaffolding
│  └─ Nada
├─ Change 01: backend-core-setup
│  └─ Change 00
├─ Change 02: frontend-core-setup
│  └─ Change 00
├─ Change 03: database-seed-and-validation
│  └─ Change 01
├─ Change 04: error-handling-and-validation
│  └─ Change 01
└─ Change 05: feature-base-patterns
   └─ Change 01 + Change 04

SPRINT 1
├─ Change 10: auth-system
│  └─ Change 01 + Change 03 + Change 04
├─ Change 11: rbac-authorization
│  └─ Change 10 + Change 03
└─ Change 12: user-management-and-addresses
   └─ Change 11 + Change 01

SPRINT 2
├─ Change 20: categories-and-hierarchy
│  └─ Change 11 + Change 01
├─ Change 21: ingredients-and-allergens
│  └─ Change 11 + Change 01
└─ Change 22: products-catalog
   └─ Change 11 + Change 20 + Change 21 + Change 01

SPRINT 3
├─ Change 30: shopping-cart-client
│  └─ Change 02 + Change 22
├─ Change 31: order-creation-and-management
│  └─ Change 30 + Change 12 + Change 11 + Change 01 + Change 03
└─ Change 32: order-state-machine-and-transitions
   └─ Change 31 + Change 01 + Change 11 + Change 03

SPRINT 4
└─ Change 40: mercadopago-integration
   └─ Change 32 + Change 31 + Change 01 + Change 11

SPRINT 5
├─ Change 50: admin-dashboard-and-metrics
│  └─ Change 22 + Change 20 + Change 12 + Change 31 + Change 11 + Change 02
└─ Change 51: client-storefront-and-checkout
   └─ Change 22 + Change 12 + Change 31 + Change 40 + Change 30 + Change 02 + Change 10
```

---

## Reglas de Implementación

### Orden Obligatorio por Sprint

1. **SPRINT 0**: Ejecutar en orden: 00 → 01 → 02 → 03 → 04 → 05
   - Todos los cambios posteriores dependen del scaffolding
   - Sin BD seeded, nada funciona
   - Sin patrones base, hay duplicación

2. **SPRINT 1**: Ejecutar en orden: 10 → 11 → 12
   - Auth debe existir antes de autorización
   - Roles deben existir antes de CRUD de usuarios

3. **SPRINT 2**: Ejecutar en orden: 20 → 21 → 22
   - Categorías antes de productos
   - Ingredientes antes de productos
   - Productos necesita ambos

4. **SPRINT 3**: Ejecutar en orden: 30 → 31 → 32
   - Carrito debe existir antes de pedidos
   - Creación de pedidos antes de máquina de estados
   - Máquina de estados con todas las validaciones

5. **SPRINT 4**: Change 40
   - Depende de máquina de estados (transición automática)
   - Puede ejecutarse en paralelo a Change 50

6. **SPRINT 5**: Changes 50 y 51 pueden ejecutarse en paralelo

### Qué NO hacer

- ❌ No implementar un change si sus dependencias no están archivadas
- ❌ No mezclar cambios de diferentes sprints (a menos que no haya dependencia)
- ❌ No hacer cambios directos al código sin pasar por proposal → design → tasks
- ❌ No commitear antes de archivar el change (specs sincronizadas)

### Cómo trabajar con cada change

Para CADA change:

1. **Proponer**: `/opsx:propose nombre-del-change`
   - Lee automaticamente docs/ y specs previas archivadas
   - Genera proposal.md, design.md, tasks.md

2. **Revisar**: Leer y aprobar los tres artefactos
   - Verificar que design respeta arquitectura en capas
   - Verificar que tasks son atómicas (horas, no días)
   - Verificar que las reglas de negocio están todas incluidas

3. **Aplicar**: `/opsx:apply nombre-del-change`
   - Lee design.md y tasks.md
   - Implementa tarea por tarea
   - Marca cada tarea completada

4. **Archivar**: `/opsx:archive nombre-del-change`
   - Sincroniza specs en openspec/specs/
   - Mueve change al historial
   - Siguientes changes usan estas specs como contexto

---

## Sumario de Historias de Usuario por Change

| Change | Historias |
|--------|-----------|
| 00 | US-000 |
| 01 | US-000a, US-000b, US-000d |
| 02 | US-000c, US-000e |
| 03 | US-000b (continuación) |
| 04 | US-068, US-074 |
| 05 | (patrón) |
| 10 | US-001, US-002, US-003, US-004, US-073 |
| 11 | US-005, US-006 |
| 12 | US-024, US-025, US-026, US-027 |
| 20 | US-007, US-008, US-009, US-010 |
| 21 | US-017 |
| 22 | US-015, US-018, US-020, US-021, US-022 |
| 30 | US-029, US-030, US-031, US-032, US-033, US-034 |
| 31 | US-035, US-036, US-037, US-038, US-049, US-050, US-051 |
| 32 | US-039, US-040, US-041, US-042, US-043, US-044 |
| 40 | US-045, US-046, US-047, US-048 |
| 50 | US-052, US-053, US-054, US-055, US-056 |
| 51 | US-016, US-019, US-057, US-058 + integración de todas las capas |

---

## Notas Finales

- **Total: 16 changes** que cubren las 74 historias de usuario
- **Duración estimada**: 
  - SPRINT 0: ~2-3 días (infraestructura)
  - SPRINT 1: ~2 días (auth + RBAC)
  - SPRINT 2: ~2 días (catálogo)
  - SPRINT 3: ~3 días (carrito + pedidos + FSM)
  - SPRINT 4: ~1-2 días (pagos)
  - SPRINT 5: ~2-3 días (UI completa)
  - **Total: ~12-14 días de desarrollo**
  
- **Testing**: Cada change debe ser verificado con tests
- **Documentación**: Las specs archivadas son la documentación viva del sistema
- **Mantenibilidad**: Cada change es independiente, facilita refactoring futuro

---

**Fin del Mapa de Changes — Food Store v5.0 — OPSX**
