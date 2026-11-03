# Plan técnico: tienda de kits de cultivo urbano

**Rama de referencia**: `001-kits-cultivo-urbano` (no creada en Git) | **Fecha**: 2026-11-03 | **Especificación**: [spec.md](spec.md)

**Estado**: diseño técnico listo para descomponer en tareas

## Resumen

Construir una tienda web modular para descubrir y comprar kits de cultivo de verduras, frutas pequeñas y plantas aromáticas en espacios reducidos. La solución será una aplicación React/Vite/TypeScript y una API REST NestJS/TypeScript conectadas a Microsoft SQL Server. El backend será un monolito modular con límites de dominio explícitos; SQL Server será la fuente de verdad para precios de pedido, pagos de prueba, reservas y movimientos de inventario. El checkout, la conversión de reserva a venta, los reembolsos simulados y la liberación de stock se ejecutarán como transacciones idempotentes.

## Contexto técnico

| Área | Decisión |
|---|---|
| Lenguaje | TypeScript en frontend y backend; SQL para migraciones y consultas específicas de persistencia. |
| Frontend | React y Vite; aplicación web adaptable en español, organizada por dominios y componentes compartidos. |
| Backend | NestJS; API REST versionada, módulos de dominio, DTOs validados, servicios de aplicación y adaptadores de persistencia. |
| Persistencia | Microsoft SQL Server relacional. Migraciones versionadas como fuente de evolución del esquema. |
| Contrato | OpenAPI 3.1 como contrato REST documentado en `contracts/openapi.yaml`. |
| Identidad | Cuenta con un solo rol de negocio (`CLIENTE` o `ADMINISTRADOR`), sesión opaca revocable en cookie `HttpOnly`, `Secure` en despliegue HTTPS y `SameSite`; hash Argon2id para contraseñas. Visitante no autenticado no es un rol persistido. |
| Pruebas | Estrategia por capas: unitarias, integración con SQL Server, contrato/API, UI de flujos y pruebas de concurrencia del checkout. No se fijan herramientas/comandos hasta que se creen los proyectos y se elijan dependencias. |
| Plataforma | Navegadores actuales de escritorio, tablet y móvil; servidor Node.js compatible con la versión LTS que se fije al inicializar los proyectos. |
| Rendimiento | Objetivo heredado de NFR-010: p95 < 2 s para búsqueda y carrito con 100 sesiones simultáneas, excluyendo latencia de red. |
| Restricciones | Solo pagos y reembolsos simulados; no se solicitan ni almacenan datos bancarios. Moneda PEN, importes con dos decimales, presentación de fecha en zona `America/Lima`. No integrar transportistas ni IA externa. |
| Alcance | Una tienda, inventario lógico único, tres categorías iniciales; cliente y administrador. Sin marketplace ni almacenes múltiples. |

Las dependencias concretas de acceso a datos, componentes visuales, pruebas y almacenamiento de imágenes se seleccionarán durante las tareas de inicialización, documentando versiones y comandos reales. Las imágenes podrán referenciar recursos administrados por el catálogo; la especificación no define un proveedor de almacenamiento.

## Arquitectura y comunicación

El frontend usa rutas y módulos de presentación para catálogo, identidad, carrito/checkout, cuenta/pedidos, atención y administración. La interfaz no decide precios finales ni permisos: presenta datos calculados por la API y envía comandos declarativos. El cliente REST central agrega autenticación, serialización, manejo uniforme de errores y fechas/importes.

La API expone `/api/v1`; controladores NestJS validan DTOs, aplican guards de sesión/rol y delegan en servicios de aplicación. Los servicios coordinan reglas de dominio y repositorios; solo la capa de persistencia conoce SQL Server. No se implementa una capa distribuida ni microservicios. La autorización se comprueba en backend para cada operación y para cada recurso del cliente.

El estado de dominio y las operaciones atómicas residen en SQL Server. La lógica de catálogo de lectura, promociones, carrito y chatbot consulta servicios de dominio; chatbot solo usa contenido publicado y aprobado. El frontend conserva únicamente el carrito anónimo temporal; al iniciar sesión sus líneas se validan y se asocian a la cuenta. Al cerrar checkout, la API recalcula todos los importes, verifica cobertura y stock, crea pedido y reserva por 15 minutos en una transacción corta. Las llamadas de simulación no mantienen una transacción SQL abierta: al obtener aprobación, una segunda transacción convierte la reserva una sola vez y registra pago/eventos/movimientos. El vencimiento se procesa de forma repetible por una operación de mantenimiento invocada por el backend; no requiere servicio externo.

Los cambios de estado usan transiciones explícitas y dejan historial. Las operaciones administrativas críticas registran actor y fecha; no se guarda información sensible innecesaria. Se usa UTC en persistencia y se convierte a `America/Lima` en vistas y cálculos de calendario de Perú.

## Puertas de constitución

| Principio | Resultado | Evidencia de diseño |
|---|---|---|
| Simplicidad y alcance aprobado | Cumple | Monolito modular, una API, una tienda e inventario; sin integraciones no aprobadas. |
| Cultivo accesible y responsable | Cumple | Modelo incluye dificultad, espacio, luz, especie, contenido e instrucciones; contenido ambiental requiere respaldo del producto. |
| Español, Perú y accesibilidad | Cumple | Interfaz en español, importes PEN, zona Lima, responsive y criterios de accesibilidad en UI. |
| Stack y responsabilidades | Cumple | React/Vite, NestJS, SQL Server y separación de capas/módulos. |
| Seguridad e integridad | Cumple | Sesión revocable, Argon2id, guards, validación y transacciones de stock/pedido/pago. |
| Calidad y aceptación demostrable | Cumple | Flujos de verificación incluidos en quickstart; suite detallada se convierte a tareas sin afirmar pruebas ejecutadas. |
| Documentación | Cumple | Plan, modelo, contrato, decisiones y guía de verificación enlazados. |

**Revisión posterior a diseño**: no se identifican excepciones a la constitución ni ampliaciones del alcance. No se crea `tasks.md` en esta etapa.

## Estructura objetivo del repositorio

```text
frontend/
  src/
    app/                 # inicio, rutas, configuración visual y proveedores
    modules/
      catalog/           # búsqueda, categorías y ficha de kit
      identity/          # registro, sesión y perfil
      cart-checkout/     # carrito y resumen/confirmación
      orders/            # historial y seguimiento
      support/           # Libro de Reclamaciones, devoluciones y chatbot
      admin/             # dashboard, catálogo, inventario, pedidos, reportes
    shared/              # controles accesibles, formato PEN/fecha, cliente API
    assets/
  tests/                 # UI, accesibilidad y flujos de cliente/admin
backend/
  src/
    main.ts
    common/              # validación, errores, sesión, permisos, filtros
    modules/
      identity/ customers/ catalog/ categories/ inventory/
      cart/ checkout/ orders/ simulated-payments/ promotions/
      complaints/ returns/ chatbot/ reports/
    persistence/         # SQL Server, migraciones, transacciones
  test/                  # unitarias, integración, API y concurrencia
database/
  migrations/            # esquema, índices, restricciones y datos de categorías iniciales
  reference/             # diagrama y diccionario sincronizados con data-model.md
docs/
  architecture/          # ADR y decisiones persistentes que nazcan en implementación
specs/001-kits-cultivo-urbano/
  spec.md plan.md research.md data-model.md quickstart.md
  contracts/openapi.yaml checklists/
```

### Límites de los módulos backend

| Módulo | Responsabilidad y dependencias |
|---|---|
| `identity` | Registro solo de cliente, inicio/cierre, sesiones y guard de roles; expone identidad autenticada. |
| `customers` | Perfil y direcciones; no autoriza acceso a datos ajenos. |
| `catalog` / `categories` | Catálogo público, categorías iniciales, filtros, detalle e instrucciones aprobadas. |
| `inventory` | Stock disponible, reservas, liberaciones y ledger de movimientos; API administrativa de ajustes. |
| `cart` | Carrito de visitante/cliente, cantidades y vista de precios actualizados. No reserva stock. |
| `checkout` | Cobertura, cotización final, descuento único, creación transaccional de pedido y reserva. |
| `orders` | Consulta propia, transiciones permitidas e historial. |
| `simulated-payments` | Intentos aprobados/rechazados/pendientes y reembolsos sin datos financieros. Idempotencia y enlace con pedido. |
| `promotions` | Vigencia, elegibilidad y selección automática del ahorro mayor entre promoción/cupón. |
| `complaints` | Libro accesible sin cuenta, código, vencimiento hábil, respuesta y seguimiento. |
| `returns` | Evaluación de elegibilidad, recepción/inspección, reembolso simulado y reposición condicionada. |
| `chatbot` | Respuestas desde catálogo/instrucciones aprobadas y creación de solicitudes cuando no resuelva. |
| `reports` | Agregados por intervalo; ventas netas y unidades más vendidas según BR-015. Solo administrador. |

## Orden técnico para posterior descomposición

1. Base de aplicación, configuración segura, migración inicial y contrato de errores.
2. Identidad/sesiones/roles, perfil y direcciones.
3. Categorías, catálogo público, imágenes referenciadas y búsqueda.
4. Inventario y ledger; carrito y promociones.
5. Cotización, checkout, reserva, pago simulado, expiración, cancelación y seguimiento.
6. Panel de catálogo, pedidos, clientes, stock y reportes.
7. Libro de Reclamaciones, devoluciones y reembolsos simulados.
8. Chatbot de contenido acotado, comprobaciones integrales y documentación ejecutable.

La secuencia preserva la entrega incremental y permite validar los invariantes comerciales antes de construir los módulos de atención secundarios.

## Artefactos de diseño

- Decisiones y alternativas: [research.md](research.md).
- Entidades, relaciones, campos, restricciones e índices: [data-model.md](data-model.md).
- Operaciones REST y esquemas: [contracts/openapi.yaml](contracts/openapi.yaml).
- Secuencia de preparación, validación y flujos de demostración: [quickstart.md](quickstart.md).
- Requisitos normativos del producto: [spec.md](spec.md).

## Complejidad añadida

No hay excepciones a la constitución. La separación por módulos refleja dominios requeridos por la especificación; no introduce servicios desplegables por separado. La tabla de eventos y el ledger de inventario se justifican por seguimiento, conciliación y protección contra dobles efectos.
