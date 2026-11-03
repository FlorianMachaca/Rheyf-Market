# Decisiones técnicas y supuestos

**Fecha**: 2026-11-03 | **Alcance**: decisiones para planificar la especificación `001-kits-cultivo-urbano`; no constituyen implementación.

## Resultado de revisión

La especificación figura como `Ready for planning`; las decisiones de negocio pendientes se resolvieron en su sesión de aclaración. No quedan marcadores `NEEDS CLARIFICATION` que bloqueen el plan. Las siguientes decisiones concretan cómo implementar esas reglas sin extender el producto.

## Decisiones

| Tema | Decisión | Alternativa considerada / motivo |
|---|---|---|
| Topología | Monolito modular NestJS, una aplicación React y una instancia lógica SQL Server. | Microservicios y colas agregan operación y consistencia distribuida innecesarias para una tienda académica de alcance acotado. |
| Límites | Módulos según dominios cliente, catálogo, inventario, compra, atención y administración; llamadas internas por servicios de aplicación. | Un backend organizado por tipo técnico hace difícil localizar las reglas de compra e inventario. |
| API | REST bajo `/api/v1`, contract-first con OpenAPI; JSON UTF-8 y errores con código estable, mensaje en español y detalles de validación. | GraphQL/WebSockets no son requeridos; seguimiento se consulta mediante API REST. |
| Identidad | Cookie de sesión opaca y revocable; el servidor conserva solo digest del identificador de sesión, vencimiento y usuario. Duración por inactividad: Cliente 30 minutos, Administrador 15 minutos conforme NFR-012. Regenerar el identificador al autenticar/cambiar privilegios y revocar al cerrar sesión. Cookies `HttpOnly`, `Secure` en HTTPS, `SameSite`; protección CSRF para mutaciones cuando el despliegue lo requiera. | No persistir JWT de larga duración en almacenamiento web: la sesión revocable facilita aplicar el tiempo de inactividad y retirar acceso administrativo sin una infraestructura de refresh-token. |
| Contraseña | Hash Argon2id con parámetros configurables y versionados; nunca reversible ni escrito en logs. | OWASP recomienda hashes lentos/adaptativos para credenciales; Argon2id es elección actual indicada por la guía. Si el entorno académico no lo soporta, revisar la alternativa antes de cambiar algoritmo. |
| Autorización | Registro crea `CLIENTE`; asignación `ADMINISTRADOR` solo por operador autorizado fuera del registro público. Guard de roles y verificación de propiedad por cada consulta/mutación. | Ocultar rutas en frontend no constituye autorización. |
| SQL Server | Modelo relacional normalizado; `DECIMAL(12,2)` para PEN, UTC en `datetime2`, restricciones FK/UNIQUE/CHECK e índices sobre rutas de consulta. Migraciones versionadas. | No guardar totales calculados solo en UI ni mantener stock en un archivo independiente. |
| Checkout y concurrencia | Transacción corta que verifica y reserva cantidades mediante actualización condicional/bloqueo de filas, crea pedido/líneas y movimientos de reserva, y confirma todo o nada. Índice y orden de adquisición por ProductId reducen contención. Reintentos por deadlock solo ante errores transitorios, con límite. | No se mantiene transacción abierta durante interacción de pago; las reservas se representan como datos con vencimiento de 15 minutos. SQL Server ofrece aislamiento y bloqueos para coordinar escritores concurrentes. |
| Efectos únicos | IDs de idempotencia para confirmar pedido/resolver intento/reembolso y restricciones únicas en relación pedido-pago aprobado y movimientos de conversión/liberación. | Un doble clic, reintento de red o resolución repetida no debe duplicar venta, pago ni reposición. |
| Pago de prueba | El usuario selecciona resultado permitido del simulador; se persisten monto, estado, fecha e intento previo. No se crean campos de tarjeta. Pendiente impide reintento hasta resolución o expiración. | No integrar una pasarela ni simular el almacenamiento de credenciales bancarias. |
| Inventario | `Products.StockOnHand` representa unidades físicas disponibles/no comprometidas; `InventoryReservations` registra unidades apartadas. La disponibilidad para nueva compra es `StockOnHand - reservas activas`. Al pago aprobado, decrementa stock físico y convierte reserva en venta en una única transacción. Cancelación/expiración libera reserva. Los movimientos son ledger append-only. | Evita que stock parezca vendido antes de aprobarse el pago y evita sobreventa durante la reserva. El ledger permite auditoría. |
| Precio histórico | La línea de pedido guarda nombre/SKU, cantidad, precio, descuento asignado, subtotal y total de línea; pedido conserva resumen, tarifa y dirección. Catálogo puede cambiar sin alterar órdenes existentes. | No reconstruir recibos consultando el precio actual del producto. |
| Promociones | Cálculo server-side antes de checkout; gana el mayor ahorro elegible entre cupón y promoción automática, uno como máximo. La cotización persiste en el pedido al confirmar. | Sin acumulación de descuentos como especifica BR-012. |
| Reclamos | Código público aleatorio/correlativo no secuencialmente enumerable; acceso de consulta por código y factor de contacto definido en el flujo; fecha límite calculada con calendario laboral configurado/documentado. Sin contenido del reclamo expuesto a terceros. | Mantiene el acceso sin cuenta indicado por la especificación y reduce exposición de datos personales. La regla legal vigente se debe validar antes del lanzamiento según la nota en spec. |
| Devoluciones | Solicitud por pedido/línea, decisión administrativa y recepción/inspección diferenciadas. Reembolso es registro simulado; inventario vuelve a disponible solo tras inspección favorable. Reposición registra movimiento distinto del reembolso. | Separar reembolso financiero simulado del estado físico de un kit. |
| Chatbot | Buscador/respuestas deterministas sobre catálogo e instrucciones aprobados; no se promete generación externa ni aprendizaje. Guardar pregunta, referencia, estado y respuesta/derivación mínima. Para consultas privadas exigir sesión y limitar a recursos propios. | Una respuesta generativa sin fuente podría inventar consejos de cultivo; no es necesaria para los criterios. |
| Reportes | Agregaciones sobre pagos aprobados menos reembolsos; vendidos netos descuentan unidades devueltas. Intervalos interpretados en `America/Lima`, convertidos a UTC para consultas. | Mantiene una definición consistente con BR-015 y cortes de periodo locales. |
| Imágenes | En catálogo se persiste URL/ruta de imagen, texto alternativo y orden; proveedor/hosting queda pendiente de la tarea de infraestructura y no se impone un servicio externo. | No existe requisito ni autorización para integrar un servicio concreto. |
| Pruebas | Los tests de integración requieren SQL Server real o contenedor compatible confirmado al seleccionar tooling; checkout concurrente debe correr contra SQL Server, pues simuladores SQLite no prueban su aislamiento. Contratos validan DTO y respuestas; flujos UI cubren criterios de aceptación. | No fijar comandos ni dependencias antes de la inicialización. |

## Referencias primarias consultadas

- OWASP, [Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html): usar hashes adaptativos para contraseñas y orientación para Argon2id.
- OWASP, [Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html): mantener significado de sesión del lado servidor y proteger identificadores.
- Microsoft Learn, [Transaction Locking and Row Versioning Guide](https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide?view=sql-server-ver17): aislamiento y coordinación de escrituras en SQL Server.
- OpenAPI Initiative, [OpenAPI Specification](https://spec.openapis.org/oas/): contrato agnóstico para describir API HTTP.

## Puntos que se dejan para tareas de implementación

1. Fijar versiones soportadas de Node.js, React, Vite, NestJS, SQL Server y dependencias luego de verificar compatibilidad del entorno académico.
2. Elegir ORM/driver de SQL Server y definir migraciones; preservar transacciones, restricciones y operaciones condicionales descritas en `data-model.md`.
3. Elegir biblioteca de componentes y estrategia concreta de imágenes; respetar accesibilidad y evitar comprometer un proveedor no solicitado.
4. Materializar un calendario peruano de días hábiles y mantenerlo de acuerdo con la regla comercial/legal aplicable; la especificación exige 15 días hábiles y respuesta documentada.
5. Determinar el mecanismo programado de expiración de reservas dentro del backend (solicitud oportunista más tarea periódica local); debe ser idempotente y no requerir servicio externo.

Estos puntos no bloquean generar tareas: deben resolverse en las tareas de inicialización y no alteran el alcance funcional.
