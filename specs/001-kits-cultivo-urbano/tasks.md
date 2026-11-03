---
description: "Lista de tareas para implementar la tienda de kits de cultivo urbano"
---

# Tareas: RHEYF MARKET — kits de cultivo urbano

**Documentos base**: [spec.md](spec.md), [plan.md](plan.md), [research.md](research.md), [data-model.md](data-model.md), [contrato REST](contracts/openapi.yaml), [guía de verificación](quickstart.md).

**Pruebas**: La constitución exige probar las funciones principales; por ello cada historia incluye tareas de pruebas dirigidas a sus criterios de aceptación. Esta lista planifica esas pruebas; no las ejecuta.

**Stack**: React/Vite/TypeScript, NestJS/TypeScript y Microsoft SQL Server. Los comandos solo se documentan después de fijarlos y verificarlos al crear los proyectos.

## Fase 1: Preparación del proyecto

**Propósito**: crear los proyectos, sus configuraciones y la base para incorporar cambios versionados.

- [ ] T001 [P] Inicializar la aplicación React/Vite/TypeScript y rutas de módulos en `frontend/package.json`, `frontend/vite.config.ts`, `frontend/tsconfig.json` y `frontend/src/app/` según `plan.md`.
- [ ] T002 [P] Inicializar la API NestJS/TypeScript con compilación y estructura modular en `backend/package.json`, `backend/nest-cli.json`, `backend/tsconfig.json` y `backend/src/main.ts`.
- [ ] T003 [P] Preparar migraciones SQL Server versionadas y datos semilla reproducibles en `database/migrations/`.
- [ ] T004 Configurar variables de entorno de ejemplo sin secretos y exclusiones locales en `.env.example`, `frontend/.env.example`, `backend/.env.example` y `.gitignore`.
- [ ] T005 Preparar configuración para pruebas unitarias, integración/API y UI, sin agregar casos de prueba todavía, en `backend/test/`, `frontend/tests/` y la configuración de ambos proyectos.

## Fase 2: Fundamentos compartidos

**Propósito**: habilitar persistencia, API, seguridad y presentación comunes antes de implementar historias.

- [ ] T006 Crear conexión SQL Server, gestión de migraciones y límite transaccional para repositorios en `backend/src/persistence/` y `backend/src/config/database.config.ts`; documentar el driver y las versiones elegidas en `docs/architecture/decisions.md`.
- [ ] T007 Crear tablas iniciales de identidad y sesiones en `database/migrations/001_identity.sql`: `users.email NVARCHAR(254) NOT NULL UNIQUE` normalizado a minúsculas; `password_hash NVARCHAR(255) NOT NULL`; `role VARCHAR(20) NOT NULL CHECK IN ('CLIENTE','ADMINISTRADOR')`; `is_active BIT NOT NULL DEFAULT 1`; `auth_sessions.token_hash BINARY(32) NOT NULL UNIQUE`; `last_seen_at_utc DATETIME2(0) NOT NULL`; `expires_at_utc DATETIME2(0) NOT NULL`; `revoked_at_utc DATETIME2(0) NULL`; claves foráneas a `users`.
- [ ] T008 Implementar validación global de DTO, errores de API estables en español y filtro de excepciones sin filtrar secretos en `backend/src/common/validation/`, `backend/src/common/filters/api-exception.filter.ts` y `backend/src/common/errors/`.
- [ ] T009 Implementar sesiones opacas revocables, cookies `HttpOnly`/`SameSite`/`Secure` sobre HTTPS, expiración por inactividad (15 minutos Administrador; 30 minutos Cliente), protección CSRF, guardas de rol y verificación de propiedad en `backend/src/modules/identity/`, `backend/src/common/guards/` y `backend/src/common/decorators/`.
- [ ] T010 Implementar prefijo REST `/api/v1` y registrar controladores conforme al contrato OpenAPI existente en `specs/001-kits-cultivo-urbano/contracts/openapi.yaml` dentro de `backend/src/main.ts`, `backend/src/app.module.ts` y `backend/src/common/`.
- [ ] T011 Crear cliente REST compartido con credenciales de sesión, CSRF para mutaciones, decodificación uniforme de errores y manejo de expiración en `frontend/src/shared/api/`.
- [ ] T012 Crear formato compartido PEN (`S/`, dos decimales), fechas `America/Lima`, mensajes y controles accesibles/responsive en `frontend/src/shared/format/`, `frontend/src/shared/ui/` y `frontend/src/app/`.
- [ ] T013 Implementar aprovisionamiento inicial seguro de Administrador fuera del registro público y documentarlo sin credenciales en `backend/src/modules/identity/`, `database/migrations/` y `docs/installation.md`; impedir que el alta pública reciba o asigne el rol administrativo.

**Punto de control**: conexión, validación, errores y controles de sesión/rol están disponibles para las historias.

## Fase 3: Historia US1 — Explorar y elegir kits de cultivo (Prioridad: P1, MVP)

**Meta**: permitir explorar portada, ofertas vigentes, categorías, búsqueda/filtros y ficha de kits publicados.

**Prueba independiente**: abrir la tienda como visitante, explorar categorías, filtrar/buscar y leer precio, stock, contenido e instrucciones; comprobar banner, ofertas, estados vacíos y adaptación móvil.

### Pruebas de US1

- [ ] T014 [P] [US1] Añadir pruebas unitarias de búsqueda, filtros, publicación y stock visible en `backend/test/modules/catalog/catalog.service.spec.ts`.
- [ ] T015 [P] [US1] Añadir pruebas de API para categorías, ofertas vigentes, búsqueda y detalle en `backend/test/modules/catalog/catalog.e2e-spec.ts`.
- [ ] T016 [P] [US1] Añadir recorrido de portada, filtros, ficha, resultados vacíos y viewport móvil en `frontend/tests/catalog/catalog.spec.ts`.

### Implementación de US1

- [ ] T017 [US1] Crear migración de catálogo en `database/migrations/002_catalog.sql` respetando `categories.name NVARCHAR(100) NOT NULL UNIQUE`, `slug VARCHAR(120) NOT NULL UNIQUE`, `description NVARCHAR(500) NULL`, `is_active BIT NOT NULL DEFAULT 1`, `sort_order SMALLINT NOT NULL DEFAULT 0`; `products.category_id INT NOT NULL FK categories`, `sku NVARCHAR(60) NOT NULL UNIQUE`, `name NVARCHAR(180) NOT NULL`, `slug NVARCHAR(220) NOT NULL UNIQUE`, `description NVARCHAR(2000) NOT NULL`, `species NVARCHAR(120) NOT NULL`, `variety NVARCHAR(120) NULL`, `difficulty VARCHAR(15) NOT NULL CHECK IN ('PRINCIPIANTE','INTERMEDIO','AVANZADO')`, `recommended_space NVARCHAR(300) NOT NULL`, `light_requirements NVARCHAR(500) NOT NULL`, `cultivation_instructions NVARCHAR(MAX) NOT NULL`, `price DECIMAL(12,2) NOT NULL CHECK (price >= 0)`, `is_published BIT NOT NULL DEFAULT 0`, `is_featured BIT NOT NULL DEFAULT 0`, fechas UTC y `row_version ROWVERSION`; componentes con `name NVARCHAR(150) NOT NULL`, `quantity DECIMAL(10,3) NOT NULL CHECK (quantity > 0)`, `unit NVARCHAR(30) NOT NULL`; imágenes con `image_url NVARCHAR(1000) NOT NULL`, `alt_text NVARCHAR(250) NOT NULL`, `sort_order SMALLINT NOT NULL DEFAULT 0` y como máximo una imagen primaria por producto. Crear también promoción con `kind VARCHAR(12) NOT NULL CHECK IN ('PORCENTAJE','MONTO_FIJO')`, `value DECIMAL(12,2) NOT NULL CHECK (value > 0)`, `starts_at_utc/ends_at_utc DATETIME2(0) NOT NULL CHECK (ends_at_utc > starts_at_utc)`, `minimum_subtotal DECIMAL(12,2) NOT NULL DEFAULT 0`, `is_active BIT NOT NULL`, `created_by BIGINT NOT NULL FK users`, `created_at_utc DATETIME2(0) NOT NULL`; cupón con `code NVARCHAR(50) NOT NULL UNIQUE`, `max_redemptions INT NULL CHECK (max_redemptions > 0)`, `redemption_count INT NOT NULL DEFAULT 0 CHECK (redemption_count >= 0)`, `is_active BIT NOT NULL DEFAULT 1` y tablas de elegibilidad con PK compuestas.
- [ ] T018 [US1] Completar en `database/migrations/002_catalog.sql` el inventario inicial `inventory_balances.on_hand INT NOT NULL CHECK (on_hand >= 0)` con PK/FK por `product_id`, fecha UTC y `row_version`; en componentes usar `product_id BIGINT NOT NULL FK products` y `sort_order SMALLINT NOT NULL DEFAULT 0`; en imágenes usar `product_id BIGINT NOT NULL FK products`, `is_primary BIT NOT NULL DEFAULT 0` y el índice único filtrado `product_id WHERE is_primary=1`; incluir `promotions.name NVARCHAR(150) NOT NULL` y la regla `CHECK (kind <> 'PORCENTAJE' OR value <= 100)`; sembrar Verduras, Frutas pequeñas y Plantas aromáticas.
- [ ] T019 [US1] Implementar repositorios, servicio de catálogo y endpoints de categorías/productos con búsqueda por nombre, precio, cultivo y dificultad en `backend/src/modules/catalog/` y `backend/src/modules/categories/`, conforme a `GET /categories`, `GET /products` y `GET /products/{productId}` en `contracts/openapi.yaml`.
- [ ] T020 [US1] Implementar consulta pública `GET /promotions/active` y DTO de ofertas vigentes sin calcular descuentos autoritativos en el cliente en `backend/src/modules/promotions/` conforme a `contracts/openapi.yaml`.
- [ ] T021 [P] [US1] Construir portada con banners editoriales accesibles, categorías y ofertas provenientes de API en `frontend/src/modules/catalog/pages/HomePage.tsx`, `frontend/src/modules/catalog/components/HomePromotions.tsx` y `frontend/src/modules/catalog/components/HomeCategoryTiles.tsx`.
- [ ] T022 [P] [US1] Construir listado con búsqueda, filtros removibles, paginación y resultados vacíos en `frontend/src/modules/catalog/pages/CatalogPage.tsx` y `frontend/src/modules/catalog/components/ProductFilters.tsx`.
- [ ] T023 [US1] Construir ficha adaptable con imágenes y texto alternativo, precio S/, disponibilidad, componentes/cantidades, dificultad, espacio, luz e instrucciones en `frontend/src/modules/catalog/pages/ProductDetailPage.tsx`.

**Punto de control**: un visitante puede evaluar un kit sin iniciar sesión y la portada no presenta productos despublicados.

## Fase 4: Historia US2 — Crear cuenta y preparar una compra (Prioridad: P1)

**Meta**: registrar e iniciar sesión como Cliente, gestionar perfil/direcciones y conservar un carrito temporal con importes vigentes.

**Prueba independiente**: registrar una cuenta, gestionar una dirección, añadir/cambiar/quitar kits y comprobar cantidades, subtotales, precios vigentes y fusión del carrito al iniciar sesión.

### Pruebas de US2

- [ ] T024 [P] [US2] Añadir pruebas unitarias para registro Cliente, perfil, propiedad de direcciones, reconciliación de carrito y límites de stock en `backend/test/modules/customers/customers.service.spec.ts` y `backend/test/modules/cart/cart.service.spec.ts`.
- [ ] T025 [P] [US2] Añadir pruebas API de duplicidad de correo, acceso a dirección ajena, y carrito anónimo/autenticado en `backend/test/modules/identity/identity.e2e-spec.ts` y `backend/test/modules/cart/cart.e2e-spec.ts`.
- [ ] T026 [P] [US2] Añadir recorrido UI de registro, sesión, direcciones, carrito y recuperación de errores de validación en `frontend/tests/account/cart.spec.ts`.

### Implementación de US2

- [ ] T027 [US2] Crear migración de perfil/dirección en `database/migrations/003_customers.sql`: `customer_profiles.user_id BIGINT PK/FK users`, `first_name NVARCHAR(100) NOT NULL`, `last_name NVARCHAR(100) NOT NULL`, `phone NVARCHAR(30) NULL`, `document_type VARCHAR(12) NULL`, `document_number NVARCHAR(20) NULL`; direcciones con `recipient_name NVARCHAR(150) NOT NULL`, `phone NVARCHAR(30) NOT NULL`, `department/province/district NVARCHAR(100) NOT NULL`, `address_line NVARCHAR(300) NOT NULL`, `reference NVARCHAR(250) NULL`, `is_default BIT NOT NULL DEFAULT 0`, fechas UTC y FK a usuario.
- [ ] T028 [US2] Implementar registro, inicio/cierre de sesión, hash Argon2id, validación de contraseña de al menos 10 caracteres según el contrato y asignación invariable de rol `CLIENTE` en `backend/src/modules/identity/`; exponer `POST /auth/register`, `/auth/login`, `/auth/logout` y `GET /auth/me`.
- [ ] T029 [US2] Implementar perfil propio y CRUD de direcciones con validación de pertenencia y bloqueo de cambios que alteren snapshots de pedidos en `backend/src/modules/customers/`; implementar endpoints `/customers/me` y `/customers/me/addresses`.
- [ ] T030 [US2] Crear persistencia de carrito y líneas en `database/migrations/004_cart.sql`: `user_id BIGINT NULL FK users`, `anonymous_key_hash BINARY(32) NULL UNIQUE`, `expires_at_utc DATETIME2(0) NULL`, restricción para que exista exactamente un propietario autenticado o una clave anónima; `cart_items.quantity INT NOT NULL CHECK (quantity > 0)`, `UNIQUE(cart_id, product_id)` y FKs según `data-model.md`.
- [ ] T031 [US2] Implementar lectura/escritura de carrito visitante y Cliente, reconciliación tras autenticación y recálculo de precio/disponibilidad en `backend/src/modules/cart/`, con `GET/PUT/DELETE /cart` y `POST /cart/quote` del contrato.
- [ ] T032 [P] [US2] Implementar formularios de registro, inicio de sesión, perfil y libreta de direcciones con mensajes en español en `frontend/src/modules/identity/` y `frontend/src/modules/customers/`.
- [ ] T033 [US2] Implementar carrito adaptable, cambio de cantidad, eliminación de línea, visualización de cambios de precio/stock y conservación de carrito anónimo al acceder en `frontend/src/modules/cart-checkout/`.

**Punto de control**: el cliente prepara su carrito y direcciones; el servidor vuelve a validar precio, stock y propiedad.

## Fase 5: Historia US3 — Confirmar compra y consultar el pedido (Prioridad: P1)

**Meta**: cotizar y reservar stock, registrar resultado simulado, confirmar pedidos y permitir seguimiento con historial íntegro.

**Prueba independiente**: comprar con pago aprobado, rechazado y pendiente; reintentar rechazo, comprobar bloqueo de pendiente, vencimiento a 15 minutos, concurrencia de última unidad e historial propio.

### Pruebas de US3

- [ ] T034 [P] [US3] Añadir pruebas unitarias de totales PEN, redondeo, promoción/cupón excluyentes, transiciones e idempotencia en `backend/test/modules/checkout/checkout.service.spec.ts` y `backend/test/modules/simulated-payments/payments.service.spec.ts`.
- [ ] T035 [P] [US3] Añadir pruebas de integración SQL Server para rollback, dos checkouts concurrentes sobre la última unidad, expiración y aprobación repetida en `backend/test/integration/checkout-concurrency.e2e-spec.ts`.
- [ ] T036 [P] [US3] Añadir recorrido UI de checkout, tres resultados de pago, reintento y seguimiento en `frontend/tests/orders/checkout.spec.ts`.

### Implementación de US3

- [ ] T037 [US3] Crear migración de pedidos, líneas, pagos y eventos en `database/migrations/005_orders_inventory_payments.sql`: pedido con `public_code UNIQUEIDENTIFIER NOT NULL UNIQUE`, `user_id BIGINT NOT NULL FK users`, `status VARCHAR(24) NOT NULL CHECK IN ('PENDIENTE_PAGO','CONFIRMADO','EN_PREPARACION','DESPACHADO','ENTREGADO','CANCELADO')`, `currency CHAR(3) NOT NULL DEFAULT 'PEN' CHECK(currency='PEN')`, cada importe `DECIMAL(12,2) NOT NULL CHECK >=0`, snapshots de destinatario/dirección, `shipment_reference NVARCHAR(120) NULL`, fechas UTC y `row_version ROWVERSION`; línea con `product_id BIGINT NOT NULL FK products`, SKU/nombre snapshot, `quantity INT NOT NULL CHECK (quantity > 0)`, precio/descuento/subtotal/total históricos `DECIMAL(12,2) CHECK >=0`, `UNIQUE(order_id, product_id)`; pago con `attempt_number SMALLINT NOT NULL CHECK >0`, `amount DECIMAL(12,2) NOT NULL CHECK >=0`, `status VARCHAR(12) NOT NULL CHECK IN ('PENDIENTE','APROBADO','RECHAZADO','VENCIDO')`, `idempotency_key UNIQUEIDENTIFIER NOT NULL UNIQUE`, `UNIQUE(order_id, attempt_number)`, un solo pago aprobado por pedido y sin datos bancarios; crear `order_status_events` y `payment_events` append-only con actor y fecha.
- [ ] T038 [US3] Crear en `database/migrations/005_orders_inventory_payments.sql` las tablas de inventario/pagos complementarias: reserva `quantity INT NOT NULL CHECK (quantity > 0)`, `status VARCHAR(12) NOT NULL CHECK IN ('ACTIVA','CONVERTIDA','LIBERADA')`, `expires_at_utc DATETIME2(0) NOT NULL`, `UNIQUE(order_id, product_id)`; ledger `kind CHECK IN ('ENTRADA','AJUSTE','SALIDA','RESERVA','VENTA','LIBERACION','CANCELACION','DEVOLUCION')`, `reason NVARCHAR(500) NOT NULL`, `idempotency_key UNIQUEIDENTIFIER NOT NULL UNIQUE`; pagos únicos por pedido/intento; crear `simulated_refunds.amount DECIMAL(12,2) NOT NULL CHECK (amount > 0)`, `status CHECK IN ('PENDIENTE','APROBADO','RECHAZADO')`, `idempotency_key UNIQUEIDENTIFIER NOT NULL UNIQUE`, actor y fecha, dejando la FK opcional a devoluciones para la migración T064.
- [ ] T039 [US3] Implementar cotización server-side con suma de líneas, descuento máximo elegible, tarifa fija habilitada y total `subtotal - descuento + envío` en `backend/src/modules/checkout/` y `backend/src/modules/promotions/promotion-calculator.service.ts`.
- [ ] T040 [US3] Implementar checkout transaccional con bloqueo ordenado por producto, comprobación de disponibilidad, snapshots de precio/dirección, reserva de 15 minutos y consumo de carrito en `backend/src/modules/checkout/checkout.service.ts` y repositorios de `backend/src/modules/inventory/`.
- [ ] T041 [US3] Implementar pago de prueba aprobado/rechazado/pendiente, reintento solo tras rechazo, bloqueo por pendiente, resolución administrativa antes del vencimiento, idempotencia e historial en `backend/src/modules/simulated-payments/`.
- [ ] T042 [US3] Implementar conversión de reserva a venta, vencimiento/cancelación idempotente, liberación, cancelación pagada antes del despacho con reembolso simulado y rechazo de aprobaciones tardías en `backend/src/modules/inventory/`, `backend/src/modules/orders/` y `backend/src/modules/simulated-payments/`.
- [ ] T043 [US3] Implementar creación/cancelación de pedido, lista y detalle propios, pagos y transiciones permitidas con eventos ordenados en `backend/src/modules/orders/`, conforme a `/checkout/orders`, `/orders` y `/orders/{orderId}`.
- [ ] T044 [US3] Implementar expiración repetible de reservas en el backend sin servicio externo y con actor de sistema en `backend/src/modules/checkout/reservation-expiration.processor.ts`.
- [ ] T045 [P] [US3] Implementar resumen de compra con precio histórico, descuentos, tarifa y total previo a aceptar en `frontend/src/modules/cart-checkout/pages/CheckoutPage.tsx`.
- [ ] T046 [US3] Implementar selección de resultado simulado sin solicitar datos bancarios, confirmación clara y vista de pedidos/eventos propios en `frontend/src/modules/orders/`.

**Punto de control**: el pago aprobado descuenta stock una sola vez; rechazo, vencimiento o error no dejan pedido/inventario parcialmente actualizado.

## Fase 6: Historia US4 — Administrar la operación de la tienda (Prioridad: P2)

**Meta**: proporcionar dashboard y gestión autorizada de productos, categorías, clientes, stock, pedidos, pagos y tarifas de entrega.

**Prueba independiente**: administrador crea/edita/publica un kit, ajusta stock, consulta cliente/pedido/pago y avanza un estado; Cliente y Visitante reciben denegación aun llamando la API directamente.

### Pruebas de US4

- [ ] T047 [P] [US4] Añadir pruebas unitarias para permisos, último administrador, transiciones de pedido y ajustes que respetan reservas en `backend/test/modules/admin/admin.service.spec.ts` y `backend/test/modules/inventory/inventory.service.spec.ts`.
- [ ] T048 [P] [US4] Añadir pruebas API para operaciones administrativas/denegación de Cliente y filtros de pedidos/clientes en `backend/test/modules/admin/admin.e2e-spec.ts`.
- [ ] T049 [P] [US4] Añadir recorrido UI del panel, mantenimiento de producto, inventario y pedidos en `frontend/tests/admin/admin-operations.spec.ts`.

### Implementación de US4

- [ ] T050 [US4] Crear `delivery_districts` en `database/migrations/006_delivery_districts.sql` con `department/province/district NVARCHAR(100) NOT NULL`, `delivery_fee DECIMAL(12,2) NOT NULL CHECK (delivery_fee >= 0)`, `is_enabled BIT NOT NULL DEFAULT 1` y `UNIQUE(department, province, district)`.
- [ ] T051 [US4] Implementar altas/ediciones/despublicación de kits y categorías; validar publicación solo con campos obligatorios, precio y categoría; implementar `POST/PATCH /admin/products` y `/admin/categories` en `backend/src/modules/catalog/` y `backend/src/modules/categories/`.
- [ ] T052 [US4] Implementar consulta de perfiles mínimos y asignación/revocación de rol con transacción que impida retirar el último Administrador activo en `backend/src/modules/customers/` y `backend/src/modules/identity/`; nunca devolver `password_hash`.
- [ ] T053 [US4] Implementar entrada, ajuste y salida de stock con motivo requerido, actor, fecha, idempotencia y prohibición de stock negativo o debajo de reservas en `backend/src/modules/inventory/`.
- [ ] T054 [US4] Implementar gestión de cobertura/tarifa de distritos, consulta administrativa de pedidos/pagos y resolución de pago pendiente vigente en `backend/src/modules/checkout/`, `backend/src/modules/orders/` y `backend/src/modules/simulated-payments/` conforme al contrato.
- [ ] T055 [US4] Implementar avance manual y auditado de pedidos, referencia de transporte sin integración externa y cancelación administrativa predespacho en `backend/src/modules/orders/`.
- [ ] T056 [US4] Construir dashboard administrativo con periodo visible, ventas/ingresos/pedidos por estado en `backend/src/modules/reports/` y `frontend/src/modules/admin/pages/AdminDashboardPage.tsx`.
- [ ] T057 [P] [US4] Construir vistas protegidas para catálogo/categorías, clientes, pedidos y pagos en `frontend/src/modules/admin/catalog/ProductsAdminPage.tsx`, `frontend/src/modules/admin/customers/CustomersPage.tsx`, `frontend/src/modules/admin/orders/OrdersPage.tsx` y `frontend/src/modules/admin/payments/PaymentsPage.tsx`.
- [ ] T058 [US4] Construir formulario de movimientos, saldo disponible/reservado y ledger en `frontend/src/modules/admin/inventory/InventoryPage.tsx`.

**Punto de control**: funciones administrativas verifican rol en backend y cada cambio conserva actor/fecha.

## Fase 7: Historia US5 — Promocionar productos y resolver solicitudes (Prioridad: P2)

**Meta**: gestionar promociones/cupones, Libro de Reclamaciones y solicitudes de devolución hasta su resolución.

**Prueba independiente**: crear beneficio vigente, tramitar reclamo sin cuenta y devolución asociada a pedido; consultar referencias/estados como cliente y comprobar plazo, respuesta, reembolso simulado e inspección.

### Pruebas de US5

- [ ] T059 [P] [US5] Añadir pruebas unitarias de elegibilidad promocional, calendario hábil, privacidad de reclamo, elegibilidad de devolución y transiciones en `backend/test/modules/promotions/promotions.service.spec.ts`, `backend/test/modules/complaints/complaints.service.spec.ts` y `backend/test/modules/returns/returns.service.spec.ts`.
- [ ] T060 [P] [US5] Añadir integración/API para correlativo/copia privada del reclamo, estados y devolución sin acceso a pedidos ajenos en `backend/test/modules/support/complaints-returns.e2e-spec.ts`.
- [ ] T061 [P] [US5] Añadir recorridos UI de promoción/cupón, Libro público y seguimiento/resolución de reclamos y devoluciones en `frontend/tests/support/promotions-returns.spec.ts`.

### Implementación de US5

- [ ] T062 [US5] Crear calendario hábil peruano en `database/migrations/007_business_calendar.sql` con `peru_business_calendar.calendar_date DATE PK` y `is_business_day BIT NOT NULL`; cargar fines de semana y días no laborables configurados para calcular los 15 días hábiles, sin cambiar la regla de negocio.
- [ ] T063 [US5] Crear tablas de quejas en `database/migrations/008_complaints.sql` con `complaint_number BIGINT NOT NULL UNIQUE`, token de acceso persistido solo como `BINARY(32) NOT NULL UNIQUE`, `consumer_name NVARCHAR(180) NOT NULL`, `document_type VARCHAR(12) NOT NULL`, `document_number NVARCHAR(20) NOT NULL`, `consumer_email NVARCHAR(254) NOT NULL`, `consumer_phone NVARCHAR(30) NULL`, `copy_channel CHECK IN ('CORREO','PRESENCIAL')`, `kind CHECK IN ('QUEJA','RECLAMO')`, `product_or_service NVARCHAR(250) NOT NULL`, `amount_claimed DECIMAL(12,2) NULL CHECK >=0`, `description NVARCHAR(4000) NOT NULL`, `requested_solution NVARCHAR(2000) NOT NULL`, `status CHECK IN ('RECIBIDA','EN_REVISION','REQUIERE_INFORMACION','RESUELTA','CERRADA')` y `response_due_at_utc DATETIME2(0) NOT NULL`; registrar pedido opcional, eventos, respuesta `NVARCHAR(4000) NULL`, canal y fechas.
- [ ] T064 [US5] Crear `return_requests`, `return_items`, `return_events` y la FK opcional de reembolsos en `database/migrations/009_returns_requests.sql`: `public_code UNIQUEIDENTIFIER NOT NULL UNIQUE`, `order_id BIGINT NOT NULL FK orders`, `user_id BIGINT NOT NULL FK users`, motivo `CHECK IN ('CAMBIO_OPINION','DEFECTO','FALTA_CONFORMIDAD','OTRO')`; estado `CHECK IN ('RECIBIDA','EN_EVALUACION','APROBADA','PRODUCTO_RECIBIDO','REEMBOLSADA','RECHAZADA','CERRADA')`; inspección `CHECK IN ('APTA','NO_APTA')`; `quantity INT NOT NULL CHECK (quantity > 0)`, `unit_refund_amount DECIMAL(12,2) NOT NULL CHECK >=0`, `restock_quantity INT NOT NULL DEFAULT 0 CHECK (restock_quantity >= 0)`, `UNIQUE(return_request_id, order_item_id)` y suma devuelta no mayor a unidades de pedido. Crear `support_requests` con `public_code UNIQUEIDENTIFIER NOT NULL UNIQUE`, usuario y correo opcionales con al menos uno presente, `subject NVARCHAR(180) NOT NULL`, `status CHECK IN ('RECIBIDA','EN_REVISION','REQUIERE_INFORMACION','RESUELTA','CERRADA')`; mensajes `body NVARCHAR(4000) NOT NULL` y eventos con actor/fecha. Agregar FK de `simulated_refunds.return_request_id` una vez creada esta tabla.
- [ ] T065 [US5] Implementar gestión de promoción/cupón, vigencia, elegibilidad, importe mínimo y condiciones; publicar/desactivar sin cambiar snapshots de pedidos en `backend/src/modules/promotions/` y conforme a `/admin/promotions`.
- [ ] T066 [US5] Implementar Libro de Reclamaciones público sin inicio de sesión, datos requeridos, número correlativo, comprobante/copia y consulta protegida por token en `backend/src/modules/complaints/`; calcular máximo 15 días hábiles con `peru_business_calendar`.
- [ ] T067 [US5] Implementar estados auditados de reclamos/solicitudes y registrar respuesta escrita, canal y fecha en `backend/src/modules/complaints/` y `backend/src/modules/support/`; mantener el plazo aunque el consumidor no responda.
- [ ] T068 [US5] Implementar solicitudes propias de devolución, validación de pedido/líneas y regla comercial de 7 días para cambio de opinión sin limitar defecto/falta de conformidad en `backend/src/modules/returns/`.
- [ ] T069 [US5] Implementar ciclo Recibida → En evaluación → Aprobada → Producto recibido → Reembolsada → Cerrada, o Rechazada → Cerrada; reembolso simulado, inspección y reposición idempotente solo si es apto en `backend/src/modules/returns/` y `backend/src/modules/simulated-payments/`.
- [ ] T070 [P] [US5] Implementar formularios y seguimiento público del Libro con visualización de copia/respuesta sin exponer datos de contacto en `frontend/src/modules/support/complaints/`.
- [ ] T071 [US5] Implementar vistas administrativas de cola, plazos, reclamos, devoluciones, recepción e inspección en `frontend/src/modules/admin/support/`.
- [ ] T072 [US5] Implementar vistas cliente para registrar y seguir devolución de pedido propio en `frontend/src/modules/support/returns/`.
- [ ] T073 [US5] Implementar gestión administrativa de promociones/cupones con fechas, condiciones y elegibilidad en `frontend/src/modules/admin/promotions/`.

**Punto de control**: el Libro permite registrar y obtener copia sin cuenta; las devoluciones mantienen las reglas comerciales y su separación de derechos por defecto/falta de conformidad.

## Fase 8: Historia US7 — Aplicar promociones y consultar reportes (Prioridad: P2)

**Meta**: comprobar descuento único de mayor ahorro y reportes netos por intervalo en hora peruana.

**Prueba independiente**: preparar aprobados, pendientes, rechazados, cancelados y reembolsos; comparar promociones y cupón elegibles y cotejar unidades netas con devoluciones.

### Pruebas de US7

- [ ] T074 [P] [US7] Añadir pruebas unitarias para elección del ahorro mayor, exclusión de estados y fechas `America/Lima` en `backend/test/modules/promotions/promotion-calculator.service.spec.ts` y `backend/test/modules/reports/reports.service.spec.ts`.
- [ ] T075 [P] [US7] Añadir integración/API para ingresos aprobados menos reembolsos y más vendidos netos de devoluciones en `backend/test/modules/reports/reports.e2e-spec.ts`.
- [ ] T076 [P] [US7] Añadir recorrido de cliente y administrador que compara beneficios y periodos de reporte en `frontend/tests/admin/reports-promotions.spec.ts`.

### Implementación de US7

- [ ] T077 [US7] Completar la regla de selección automática de un solo descuento y explicar base, ahorro y total en carrito/checkout en `backend/src/modules/promotions/promotion-calculator.service.ts` y `frontend/src/modules/cart-checkout/`.
- [ ] T078 [US7] Implementar reporte de ventas por periodo `America/Lima`: pagos aprobados menos reembolsos, excluir pendientes/rechazados/cancelados, mostrar pedidos/ingresos por estado en `backend/src/modules/reports/` y `GET /admin/reports/sales`.
- [ ] T079 [US7] Implementar reporte de productos más vendidos con unidades pagadas menos devueltas, fechas locales visibles y orden descendente en `backend/src/modules/reports/` y `GET /admin/reports/best-sellers`.
- [ ] T080 [US7] Implementar filtros de periodo, resumen, criterios de cálculo y presentación PEN en `frontend/src/modules/admin/pages/SalesReportPage.tsx` y `frontend/src/modules/admin/pages/BestSellersPage.tsx`.

**Punto de control**: los resultados de reporte se reconcilian con pagos/reembolsos y unidades efectivamente devueltas.

## Fase 9: Historia US6 — Consultar al asistente de atención (Prioridad: P3)

**Meta**: responder consultas sobre kits/instrucciones y datos propios autorizados; derivar dudas no resueltas a soporte.

**Prueba independiente**: consultar contenido publicado, una instrucción ausente, pedido/reclamo propios y ajenos, y solicitar atención; verificar que el bot no inventa información ni revela datos privados.

### Pruebas de US6

- [ ] T081 [P] [US6] Añadir pruebas unitarias de respuesta basada solo en catálogo/instrucciones y rechazo de contenido sin fuente en `backend/test/modules/chatbot/chatbot.service.spec.ts`.
- [ ] T082 [P] [US6] Añadir pruebas API de sesión, propiedad de pedido/reclamo y creación/seguimiento de solicitudes en `backend/test/modules/chatbot/chatbot.e2e-spec.ts`.
- [ ] T083 [P] [US6] Añadir recorrido UI de consultas de catálogo, pedido propio/ajeno y derivación a soporte en `frontend/tests/support/chatbot.spec.ts`.

### Implementación de US6

- [ ] T084 [US6] Implementar servicio de respuesta acotada a producto e instrucciones publicados, referencia de fuente y respuesta explícita si no hay información en `backend/src/modules/chatbot/chatbot.service.ts`.
- [ ] T085 [US6] Implementar consultas autenticadas de pedidos/reclamos propios y registro de solicitud de atención con estado/eventos y referencia en `backend/src/modules/chatbot/chatbot.controller.ts` y `backend/src/modules/support/support.service.ts`.
- [ ] T086 [US6] Implementar interfaz de chatbot, estados accesibles y opciones para crear/consultar solicitud sin pedir datos de pedidos al visitante en `frontend/src/modules/support/chatbot/`.

**Punto de control**: información privada exige sesión y verificación de pertenencia; la respuesta de cultivo siempre señala contenido aprobado.

## Fase 10: Cierre y preocupaciones transversales

**Propósito**: demostrar criterios de calidad y mantener documentación ejecutable.

- [ ] T087 [P] Completar pruebas de accesibilidad por teclado, etiquetas, contraste y texto alternativo en `frontend/tests/accessibility/`.
- [ ] T088 [P] Medir NFR-010 (100 sesiones simultáneas; p95 < 2 s en búsquedas y carrito, excluida latencia de red) y documentar resultados reproducibles en `backend/test/performance/` y `docs/quality/performance.md`.
- [ ] T089 [P] Revisar amenazas de sesión, CSRF, autorización por objeto y exclusión de secretos/datos bancarios; documentar hallazgos/correcciones en `docs/architecture/security-review.md`.
- [ ] T090 Alinear migraciones SQL, DTOs/OpenAPI y diccionario final con `specs/001-kits-cultivo-urbano/data-model.md`, `specs/001-kits-cultivo-urbano/contracts/openapi.yaml` y `database/reference/`.
- [ ] T091 Registrar comandos reales de instalación, migración, ejecución y pruebas, versiones y pasos de demostración en `docs/installation.md` y actualizar `specs/001-kits-cultivo-urbano/quickstart.md` sin comandos no ejecutados.

## Dependencias y orden de ejecución

### Dependencias entre fases

- **Preparación**: tareas T001–T005 independientes por aplicación; completar antes de los fundamentos.
- **Fundamentos**: T006–T013; bloquean las historias por conexión, validación, sesión, seguridad y componentes compartidos.
- **US1 y US2**: ambas comienzan después de Fundamentos y pueden avanzar en paralelo. US3 requiere ambas para catálogo, identidad, dirección y carrito.
- **US3**: precede las operaciones que dependen de pedidos, pagos e inventario vendido.
- **US4**: requiere productos/categorías de US1 y pedidos/pagos/inventario de US3 para completar su panel operacional.
- **US5**: requiere pedidos de US3 y permisos/admin de US4; crea beneficios promocionales y atención posventa.
- **US7**: requiere pagos/reembolsos de US3 y promociones/devoluciones de US5.
- **US6**: requiere catálogo de US1, pedidos de US3 y soporte/reclamos de US5.
- **Cierre**: tras completar las historias que forman parte de la demostración.

### Grafo de dependencias

```text
Preparación → Fundamentos
                 ├──→ US1 ──┐
                 └──→ US2 ──┴──→ US3 ──→ US4 ──→ US5 ──┬──→ US7
                                                       └──→ US6
```

US6 depende de US1, US3 y US5; puede desarrollarse en paralelo con US7 una vez terminada US5. La secuencia no obliga a paralelizar trabajo con recursos limitados.

### Oportunidades de paralelismo

- En preparación: T001, T002 y T003 pueden ejecutarse en paralelo; T004 requiere que existan los proyectos.
- En cada historia, las tareas de prueba marcadas `[P]` pueden escribirse en archivos distintos antes de la implementación.
- US1 y US2 se pueden asignar a personas diferentes tras cerrar Fundamentos.
- En US1, T021 y T022 son independientes; la ficha T023 consume el servicio de catálogo.
- En US4, vistas separadas de catálogo/cliente/pedido y la vista de inventario pueden repartirse tras disponer de los endpoints correspondientes.
- US6 y US7 se pueden desarrollar en paralelo cuando US5 y sus datos estén integrados.

## Ejemplo de ejecución paralela: US1

```text
T014 backend/test/modules/catalog/catalog.service.spec.ts
T015 backend/test/modules/catalog/catalog.e2e-spec.ts
T016 frontend/tests/catalog/catalog.spec.ts

Tras preparar catálogo y contrato:
T021 frontend/src/modules/catalog/pages/HomePage.tsx
T022 frontend/src/modules/catalog/pages/CatalogPage.tsx
```

## Otras oportunidades de ejecución paralela

```text
US2: T024 backend/test/modules/customers/customers.service.spec.ts
     T025 backend/test/modules/identity/identity.e2e-spec.ts
     T026 frontend/tests/account/cart.spec.ts

US3: T034 backend/test/modules/checkout/checkout.service.spec.ts
     T035 backend/test/integration/checkout-concurrency.e2e-spec.ts
     T036 frontend/tests/orders/checkout.spec.ts

US4: T047 backend/test/modules/admin/admin.service.spec.ts
     T048 backend/test/modules/admin/admin.e2e-spec.ts
     T049 frontend/tests/admin/admin-operations.spec.ts

US5: T059 backend/test/modules/promotions/promotions.service.spec.ts
     T060 backend/test/modules/support/complaints-returns.e2e-spec.ts
     T061 frontend/tests/support/promotions-returns.spec.ts

US7: T074 backend/test/modules/promotions/promotion-calculator.service.spec.ts
     T075 backend/test/modules/reports/reports.e2e-spec.ts
     T076 frontend/tests/admin/reports-promotions.spec.ts

US6: T081 backend/test/modules/chatbot/chatbot.service.spec.ts
     T082 backend/test/modules/chatbot/chatbot.e2e-spec.ts
     T083 frontend/tests/support/chatbot.spec.ts
```

Cada grupo lista archivos separados y puede prepararse en paralelo antes de implementar la historia; la ejecución de pruebas requerirá su servicio y entorno.

## Estrategia de entrega

### MVP incremental

1. Completar Preparación y Fundamentos.
2. Entregar US1 (catálogo) como primer MVP demostrable, con su prueba independiente.
3. Añadir US2 (cuenta/carrito) y después US3 para completar la compra simulada, la reserva de inventario y el seguimiento íntegros.
4. Detenerse en el punto de control y verificar los flujos P1 de `quickstart.md` antes de ampliar a administración y atención.

### Incrementos posteriores

1. US4 habilita operación administrativa y configuración de cobertura.
2. US5 añade promociones administrables y atención de reclamos/devoluciones.
3. US7 completa reportes y comprobación de cálculo de descuentos.
4. US6 incorpora asistencia con contenido acotado.
5. Cierre verifica rendimiento, accesibilidad, seguridad y documentación de comandos comprobados.

## Notas

- `[P]` indica archivos distintos y ausencia de dependencia pendiente; `[USn]` vincula cada tarea a su historia.
- La estrategia detalla pruebas porque la constitución exige verificar las funcionalidades principales; ninguna prueba se ha ejecutado al generar esta lista.
- No desarrollar pagos reales, marketplace, transportistas, banner CMS, integración externa de chatbot ni almacenes múltiples.
- Resolver decisiones de dependencias/versiones dentro de T001–T006 y registrarlas; no inventar comandos en documentación.
