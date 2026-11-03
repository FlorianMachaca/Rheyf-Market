# Feature Specification: Tienda de kits de cultivo urbano

**Feature Branch**: `001-kits-cultivo-urbano` (referencia; no se creó una rama Git)

**Created**: 2026-11-03

**Status**: Ready for planning

**Input**: User description: "Desarrollar RHEYF MARKET como plataforma de comercio electrónico especializada en kits para cultivar verduras, frutas pequeñas y plantas aromáticas en espacios reducidos, con tienda para clientes, administración, chatbot, pagos simulados e inventario íntegro. No comenzar a programar."

## Clarifications

### Session 2026-11-03

- Q: ¿Qué roles y permisos tendrá la tienda? → A: Visitante explora y arma un carrito temporal; Cliente compra y consulta solo sus datos; Administrador gestiona la tienda. El registro público crea solo Clientes y un Administrador autorizado asigna o revoca el rol administrativo.
- Q: ¿Qué atributos definen el catálogo de kits? → A: Cada kit incluye categoría, especie/variedad, dificultad (Principiante, Intermedio o Avanzado), espacio y luz recomendados, contenido y cantidades, instrucciones de siembra y cuidado, imágenes, precio, publicación y stock. Categorías iniciales: Verduras, Frutas pequeñas y Plantas aromáticas.
- Q: ¿Cómo se prepara y confirma una compra? → A: Se permite carrito temporal sin cuenta, pero checkout requiere autenticación. Solo se entrega en distritos habilitados y se aplica una tarifa fija por distrito, visible antes de confirmar.
- Q: ¿Cómo se gestionan los pagos de prueba? → A: Cada intento se conserva como Aprobado, Rechazado o Pendiente. No se ingresan datos bancarios. Un rechazo permite reintentar durante la reserva; un intento pendiente bloquea nuevos intentos hasta resolverse o vencer.
- Q: ¿Cuándo se reserva y descuenta el inventario? → A: Al crear el pedido se reservan todas las unidades por 15 minutos en una única operación atómica. La aprobación convierte la reserva en venta; la cancelación o vencimiento la libera. Nunca se puede aprobar después del vencimiento.
- Q: ¿Cuáles son los estados del pedido? → A: Pendiente de pago → Confirmado → En preparación → Despachado → Entregado. Puede cancelarse mientras está pendiente o por un Administrador antes del despacho; toda transición registra fecha, responsable y estado anterior/nuevo. El seguimiento de transporte es manual, sin integración con transportistas.
- Q: ¿Qué reglas tienen reclamos y devoluciones? → A: El Libro de Reclamaciones está accesible sin cuenta y genera código; la tienda emite respuesta escrita dentro de 15 días hábiles, según la información vigente de Indecopi. El cambio de opinión se solicita dentro de 7 días calendario de la entrega para kits completos, sellados y sin uso. Este plazo no limita reclamos legales por defectos o falta de conformidad. Las solicitudes transitan Recibida → En revisión → (Requiere información) → Resuelta → Cerrada. El Administrador resuelve y registra un reembolso simulado; el stock se repone solo tras inspección favorable.
- Q: ¿Qué información puede usar el chatbot? → A: Responde solo con información publicada del catálogo e instrucciones, sin inventar recomendaciones. Consultar pedidos o reclamos exige autenticación y solo muestra datos propios. Si no resuelve una pregunta, crea una solicitud de atención con referencia.
- Q: ¿Cómo se calculan promociones y reportes? → A: Como máximo se aplica un descuento por pedido; cupón y promoción automática no se acumulan. Ventas netas son pagos aprobados menos reembolsos; se excluyen pagos pendientes, rechazados y pedidos cancelados. Los más vendidos se ordenan por unidades pagadas menos devueltas, con fechas en hora de Perú.


## User Scenarios & Testing *(mandatory)*

### User Story 1 - Explorar y elegir kits de cultivo (Priority: P1)

Como visitante, quiero descubrir kits de cultivo adecuados para mi espacio, experiencia e intereses, para elegir un producto con información suficiente antes de comprarlo.

**Why this priority**: Encontrar y entender la oferta es la entrada principal a la tienda y sirve incluso antes de registrarse.

**Independent Test**: Un visitante abre la tienda, recorre categorías, busca y filtra el catálogo, y consulta una ficha con precio, disponibilidad, contenido e instrucciones.

**Acceptance Scenarios**:

1. **Given** que hay productos activos agrupados en categorías, **When** el visitante abre la tienda, **Then** puede ver banners, ofertas vigentes, productos destacados y categorías disponibles.
2. **Given** que hay productos que coinciden con una búsqueda, **When** el visitante busca por nombre o aplica filtros de precio, tipo de cultivo o dificultad, **Then** ve solo resultados coincidentes y puede quitar los filtros.
3. **Given** que el visitante abre un producto disponible, **When** consulta su detalle, **Then** ve imágenes, precio en S/, stock disponible, contenido del kit e instrucciones de cultivo.
4. **Given** que una búsqueda o combinación de filtros no coincide con productos, **When** se muestran los resultados, **Then** la tienda informa que no hay coincidencias y permite cambiar la búsqueda.

---

### User Story 2 - Crear cuenta y preparar una compra (Priority: P1)

Como cliente, quiero registrarme, administrar mis datos y direcciones y añadir kits a un carrito, para preparar una compra con cantidades y costos claros.

**Why this priority**: El cliente necesita guardar su identidad y destino y revisar el pedido antes de confirmar la compra.

**Independent Test**: Una persona crea una cuenta, inicia sesión, registra una dirección, agrega productos al carrito, cambia cantidades y comprueba subtotales, descuentos, envío y total antes de pagar.

**Acceptance Scenarios**:

1. **Given** datos de registro válidos y una cuenta inexistente, **When** la persona envía el registro, **Then** se crea su cuenta y puede iniciar sesión; los errores de campos inválidos o duplicados se explican claramente.
2. **Given** un cliente autenticado, **When** crea, edita o elimina una dirección que no está en uso por un pedido ya realizado, **Then** los cambios quedan reflejados en su libreta de direcciones.
3. **Given** un producto con stock, **When** el cliente lo agrega al carrito o modifica su cantidad, **Then** el carrito muestra las cantidades, subtotales y total calculados y no admite una cantidad superior al stock disponible.
4. **Given** que el precio o stock cambió desde que se agregó un producto, **When** el cliente revisa el carrito o inicia el pago, **Then** ve el valor y disponibilidad vigentes antes de confirmar.
5. **Given** que un visitante arma un carrito y luego inicia sesión, **When** accede a su cuenta, **Then** el carrito se conserva y cada línea vuelve a validarse contra precio y stock vigentes.
6. **Given** un cliente autenticado con dirección en un distrito no atendido, **When** inicia checkout, **Then** recibe aviso de falta de cobertura y no puede confirmar.

---

### User Story 3 - Confirmar compra y consultar el pedido (Priority: P1)

Como cliente, quiero completar una compra mediante un pago simulado y consultar su estado, para conocer el resultado y hacer seguimiento de los kits pedidos.

**Why this priority**: La compra íntegra y su seguimiento constituyen el valor central del comercio electrónico.

**Independent Test**: Un cliente confirma un carrito disponible, elige una dirección y un resultado de pago de prueba, y consulta el pedido y su historial.

**Acceptance Scenarios**:

1. **Given** un carrito con productos disponibles y una dirección válida, **When** el cliente revisa el resumen, **Then** puede verificar productos, cantidades, precios, descuentos aplicados, envío y total antes de confirmar.
2. **Given** el resumen aceptado, **When** el cliente ejecuta la simulación de pago aprobada, **Then** se registra el pago simulado, se confirma el pedido, se conserva el precio unitario de cada línea y se descuenta el stock una sola vez.
3. **Given** un pago simulado rechazado, **When** termina el intento, **Then** el cliente recibe el resultado, el pedido no aparece como pagado y no se descuenta stock como compra confirmada.
4. **Given** un pago simulado pendiente, **When** termina el intento, **Then** el pedido y el pago aparecen como pendientes y el cliente puede consultar el estado actualizado sin repetir la compra por error.
5. **Given** un pedido del cliente, **When** consulta su historial o detalle, **Then** ve fecha, líneas, total, pagos y cambios de estado en orden cronológico.
6. **Given** que otro cliente agotó el último stock durante el proceso de compra, **When** se confirma la compra, **Then** no se crea una venta por encima del stock y el cliente recibe una explicación para ajustar el carrito.
7. **Given** que el checkout reserva las últimas unidades, **When** transcurren 15 minutos sin aprobación, **Then** el pedido se cancela, se libera el stock y un resultado tardío no puede aprobarlo.
8. **Given** un intento de pago Rechazado y reserva vigente, **When** el cliente reintenta, **Then** se registra un nuevo intento en el mismo pedido sin duplicar la reserva.
9. **Given** un pago Pendiente, **When** el cliente intenta un segundo pago, **Then** no se acepta el nuevo intento hasta resolver o vencer el primero.

---

### User Story 4 - Administrar la operación de la tienda (Priority: P2)

Como administrador autorizado, quiero gestionar el catálogo, inventario, clientes, pedidos y pagos de prueba, para mantener la tienda y atender las operaciones diarias.

**Why this priority**: La operación administrativa mantiene disponible y confiable la oferta que usan los clientes.

**Independent Test**: Un administrador autorizado consulta indicadores y puede crear o editar productos, categorías, precios e imágenes, registrar movimientos de stock, cambiar estados permitidos y consultar clientes y pagos simulados; una cuenta de cliente no puede realizar estas acciones.

**Acceptance Scenarios**:

1. **Given** una cuenta con permisos administrativos, **When** abre el panel, **Then** consulta estadísticas de ventas, ingresos y pedidos con un periodo identificable.
2. **Given** un administrador autorizado y datos de producto válidos, **When** crea o edita productos, categorías, precios, imágenes, contenido del kit o instrucciones, **Then** los cambios se reflejan en el catálogo según su estado de publicación.
3. **Given** un movimiento de entrada, ajuste o salida de inventario, **When** se registra con motivo y cantidad, **Then** el stock resultante y el historial de movimientos pueden consultarse.
4. **Given** pedidos y pagos existentes, **When** un administrador autorizado los consulta o actualiza un estado permitido, **Then** se registra quién realizó el cambio, cuándo y cuál fue la transición.
5. **Given** una sesión de cliente sin permiso administrativo, **When** intenta abrir o ejecutar una función de administración, **Then** el acceso es denegado y los datos administrativos no son revelados.
6. **Given** ventas registradas en el periodo elegido, **When** el administrador consulta los reportes, **Then** puede ver ventas e identificar los productos más vendidos con criterios y periodo visibles.
7. **Given** cualquier Cliente o Visitante, **When** intenta asignarse rol Administrador o acceder al panel, **Then** se deniega la acción.
8. **Given** una modificación de inventario, **When** el administrador ingresa cantidad inválida o un ajuste dejaría stock negativo, **Then** se rechaza sin cambiar el stock.
9. **Given** un pedido en Confirmado, **When** el Administrador actualiza el seguimiento, **Then** solo puede avanzar por En preparación, Despachado y Entregado, y cada cambio queda auditado.

---

### User Story 5 - Promocionar productos y resolver solicitudes (Priority: P2)

Como administrador autorizado, quiero administrar promociones, cupones, reclamos y devoluciones, para ofrecer descuentos controlados y dar seguimiento a problemas de clientes.

**Why this priority**: Las promociones y la atención posterior a la compra completan la operación comercial, aunque no impiden demostrar la compra básica.

**Independent Test**: Un administrador crea una promoción con vigencia y condiciones visibles, procesa un reclamo o solicitud de devolución y el cliente puede consultar el resultado o estado.

**Acceptance Scenarios**:

1. **Given** una promoción o cupón vigente y un pedido que satisface sus condiciones, **When** el cliente lo aplica en el carrito, **Then** ve el descuento antes de confirmar y el total se recalcula.
2. **Given** un cupón vencido, inválido o que no cumple condiciones, **When** el cliente intenta aplicarlo, **Then** no se aplica el descuento y se informa el motivo sin perder el carrito.
3. **Given** un pedido del cliente, **When** registra un reclamo o solicitud de devolución con motivo y descripción, **Then** recibe un número de referencia y puede consultar su estado.
4. **Given** una solicitud abierta, **When** un administrador autorizado la revisa, responde y cambia su estado, **Then** queda constancia de la resolución y el cliente puede verla.
5. **Given** un visitante, **When** busca el Libro de Reclamaciones, **Then** encuentra un enlace visible y puede enviar una queja o reclamo sin crear cuenta.
6. **Given** un reclamo registrado, **When** el administrador lo atiende, **Then** registra y emite respuesta escrita dentro de 15 días hábiles.
7. **Given** una solicitud de cambio de opinión posterior a 7 días o un kit usado/abierto, **When** se evalúa, **Then** no califica para la política comercial; un reclamo por defecto o falta de conformidad se evalúa de forma independiente.
8. **Given** una devolución aprobada, **When** se recibe e inspecciona el kit, **Then** el Administrador registra reembolso simulado y solo repone stock si el producto es apto para venta.

---

### User Story 6 - Consultar al asistente de atención (Priority: P3)

Como visitante o cliente, quiero consultar productos, instrucciones de cultivo, compras, pedidos y reclamos, para resolver dudas sin recorrer varias secciones.

**Why this priority**: El asistente mejora el acceso a información y deriva a atención humana los casos no resueltos.

**Independent Test**: Una persona consulta información disponible, recibe una respuesta basada en la tienda y, si el asistente no puede resolverla, registra una solicitud de atención.

**Acceptance Scenarios**:

1. **Given** información disponible sobre un producto o instrucciones, **When** la persona pregunta por ella, **Then** el asistente ofrece una respuesta clara y pertinente.
2. **Given** que el cliente consulta datos personales de un pedido o reclamo, **When** solicita el estado, **Then** el sistema exige autenticación y solo muestra información propia.
3. **Given** una pregunta que el asistente no puede resolver, **When** la persona solicita ayuda, **Then** puede registrar una solicitud de atención y recibe una referencia o confirmación.
4. **Given** una consulta sobre pedidos o reclamos, **When** la envía un visitante o un cliente sobre datos ajenos, **Then** el chatbot no revela información personal.
5. **Given** el chatbot no encuentra instrucciones de cultivo, **When** recibe una pregunta, **Then** comunica que no tiene la respuesta y ofrece crear una solicitud, sin inventar consejos.

### User Story 7 - Aplicar promociones y consultar reportes (Priority: P2)

Como cliente, quiero conocer el descuento y total final antes de pagar; como administrador, quiero consultar cifras calculadas de forma uniforme.

**Why this priority**: La claridad de precios permite decidir la compra y los reportes permiten gestionar la tienda.

**Independent Test**: Una promoción y un cupón elegibles se prueban contra el mismo carrito; el administrador genera un reporte para un periodo con pagos aprobados y reembolsos.

**Acceptance Scenarios**:

1. **Given** un producto en promoción y un cupón válido, **When** se calcula el carrito, **Then** se aplica solo un descuento, se muestra su base y no se suman ambos beneficios.
2. **Given** pedidos aprobados, rechazados, pendientes, cancelados y reembolsados, **When** el administrador genera ventas netas, **Then** solo suma aprobados y resta reembolsos; más vendidos descuenta unidades devueltas.

### Edge Cases

- Producto sin stock, despublicado o con precio actualizado mientras está en el carrito.
- Cantidad solicitada supera el stock al agregar, revisar el carrito o confirmar la compra.
- Dos compras intentan confirmar simultáneamente la última unidad.
- Pago rechazado, pendiente sin resolución, reintento o notificación repetida del mismo resultado.
- Carrito vacío, cupón vencido, cupón no acumulable o promoción que vence durante el checkout.
- Cambio de dirección después de un pedido confirmado; el pedido conserva la dirección usada al comprar.
- Intento de consultar pedido, reclamo o perfil de otro cliente.
- Transición de pedido o solicitud no permitida, y fallo al guardar parte de una operación que modifica pedido, pago e inventario.
- Reclamo o devolución para un pedido que no pertenece al cliente o que ya tiene una solicitud abierta.
- Pregunta sin información suficiente o chatbot indisponible; ofrecer una vía para registrar solicitud.
- Catálogo vacío, filtros sin coincidencias, imagen no disponible o error al cargar resultados.
- Precio, descuento, cantidades y totales deben mantener dos decimales y reglas de redondeo consistentes.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: La tienda MUST presentar productos destacados, banners, ofertas vigentes y categorías de kits para verduras, frutas pequeñas y plantas aromáticas.
- **FR-002**: El visitante MUST poder buscar productos y filtrar por precio, tipo de cultivo y dificultad, y MUST poder quitar filtros aplicados.
- **FR-003**: Cada ficha de producto MUST mostrar imágenes, nombre, descripción, precio en soles (S/), disponibilidad, contenido del kit e instrucciones de cultivo.
- **FR-004**: El sistema MUST permitir que una persona registre una cuenta de cliente e inicie y cierre sesión, con errores de validación comprensibles.
- **FR-005**: El cliente autenticado MUST poder consultar y actualizar su perfil y administrar varias direcciones de entrega.
- **FR-006**: El cliente MUST poder agregar productos al carrito, cambiar cantidades y quitar productos; el sistema MUST mostrar subtotales y total recalculados.
- **FR-007**: Antes de confirmar un pedido, el sistema MUST presentar cantidades, precios vigentes, descuentos, costo de entrega y total, e informar de cambios de precio o disponibilidad.
- **FR-008**: El sistema MUST rechazar cantidades mayores al stock disponible y MUST impedir que compras concurrentes confirmen más unidades que las disponibles.
- **FR-009**: El sistema MUST calcular y conservar en el pedido el precio unitario, cantidad, subtotal, descuento y total correspondientes al momento de la compra.
- **FR-010**: El sistema MUST permitir seleccionar entre resultados de pago simulado aprobado, rechazado y pendiente, sin realizar cobros reales ni solicitar o almacenar datos bancarios reales.
- **FR-011**: El sistema MUST mantener un registro consultable de intentos de pago y sus resultados; un rechazo no puede marcarse como pago aprobado.
- **FR-012**: El sistema MUST descontar inventario una sola vez cuando la compra se confirma mediante pago aprobado y registrar los movimientos relacionados con pedidos y ajustes administrativos.
- **FR-013**: El cliente MUST poder consultar sus pedidos, detalle, pago, estado actual y cronología de cambios, y el sistema MUST conservar la dirección usada en cada pedido.
- **FR-014**: El administrador autorizado MUST poder consultar un dashboard de ventas, ingresos y pedidos, con periodo de consulta identificable.
- **FR-015**: El administrador autorizado MUST poder crear, editar, publicar, despublicar y consultar productos y categorías, incluidos precios, imágenes, contenido del kit e instrucciones.
- **FR-016**: El administrador autorizado MUST poder consultar y registrar entradas, salidas y ajustes de inventario con motivo, fecha, responsable y cantidad.
- **FR-017**: El administrador autorizado MUST poder consultar clientes y usuarios, y gestionar pedidos, estados permitidos y pagos simulados sin exponer contraseñas.
- **FR-018**: El sistema MUST aplicar permisos por rol y denegar toda operación administrativa a cuentas sin autorización.
- **FR-019**: El administrador autorizado MUST poder crear y gestionar promociones y cupones con vigencia y condiciones; el cliente MUST ver cualquier descuento aplicado antes de confirmar.
- **FR-020**: El cliente MUST poder registrar reclamos y solicitudes de devolución vinculados a sus pedidos, consultar su estado y recibir una confirmación de registro.
- **FR-021**: El administrador autorizado MUST poder revisar reclamos y devoluciones, responder, cambiar sus estados permitidos y dejar un historial de atención.
- **FR-022**: El administrador autorizado MUST poder consultar reportes de ventas y productos más vendidos para un periodo seleccionado.
- **FR-023**: El chatbot MUST responder consultas sobre productos, instrucciones de cultivo, compras, pedidos y reclamos usando la información disponible para ese usuario.
- **FR-024**: El chatbot MUST proteger la información de pedidos y reclamos mediante autenticación y autorización, y MUST permitir registrar una solicitud de atención cuando no resuelva la consulta.
- **FR-025**: El sistema MUST registrar fechas y responsables de cambios de estado de pedidos, pagos y solicitudes de atención.
- **FR-026**: El sistema MUST presentar textos, validaciones, notificaciones y estados de interacción en español y mostrar precios en S/.

- **FR-027**: El registro público MUST asignar el rol Cliente; una cuenta no puede elegirse ni elevarse a Administrador por sí misma. Un Administrador autenticado es el único rol que puede asignar o revocar el rol administrativo, sin poder dejar la tienda sin un Administrador activo.
- **FR-028**: El carrito de visitante MUST ser temporal y conservarse al registrarse o iniciar sesión; antes de comprar, el sistema MUST revalidar precio, stock y cantidades enteras positivas.
- **FR-029**: La entrega MUST ofrecerse solo en distritos habilitados; el Administrador MUST mantener una tarifa fija por distrito y el checkout MUST mostrarla antes de confirmar.
- **FR-030**: El sistema MUST reservar las unidades de un pedido Pendiente de pago durante 15 minutos y liberar la reserva al cancelar o vencer; toda reserva y liberación se aplica una sola vez.
- **FR-031**: El sistema MUST permitir reintentar un pago Rechazado mientras la reserva esté vigente; un intento Pendiente bloquea otro intento y solo un Administrador puede resolverlo como Aprobado o Rechazado antes del vencimiento.
- **FR-032**: El pedido MUST seguir las transiciones Pendiente de pago → Confirmado → En preparación → Despachado → Entregado; puede pasar a Cancelado antes de confirmarse o por un Administrador antes del despacho. Cada transición queda auditada.
- **FR-033**: Una cancelación de pedido ya aprobado MUST registrar reembolso simulado; antes del despacho, el stock no enviado vuelve a estar disponible. Después del despacho, la devolución se gestiona mediante solicitud y recepción, no por cancelación directa.
- **FR-034**: El kit MUST incluir categoría, especie o variedad, nivel de dificultad, espacio/luz recomendados, contenido y cantidades, instrucciones de siembra/cuidado e imágenes. Solo se publica si cuenta con información, categoría y precio obligatorios.
- **FR-035**: El Libro de Reclamaciones MUST permanecer visible y disponible para visitantes sin cuenta, generar un código correlativo y permitir al consumidor obtener copia de la solicitud. El formulario MUST guardar nombre, documento de identidad y contacto del consumidor, producto/servicio, importe, detalle, pedido concreto y solución que solicita.
- **FR-036**: El Administrador MUST responder por escrito a cada queja o reclamo en un máximo de 15 días hábiles, conservar fecha/canal/respuesta y dejarla accesible al consumidor.
- **FR-037**: Una devolución comercial por cambio de opinión MUST solicitarse dentro de 7 días calendario de la entrega y el kit debe estar completo, sellado y sin uso; la evaluación no limitará derechos legales por defectos o falta de conformidad.
- **FR-038**: El Administrador MUST resolver la devolución y registrar la recepción e inspección; el reembolso es simulado y el stock solo se repone si el kit está apto para venta.
- **FR-039**: El chatbot MUST usar información publicada de productos e instrucciones, indicar cuando no conoce la respuesta y crear una solicitud de atención con referencia si el usuario lo pide.
- **FR-040**: El chatbot MUST requerir autenticación para consultas de pedidos/reclamos y limitar la respuesta a registros del usuario autenticado.
- **FR-041**: Las promociones automáticas y cupones MUST ser excluyentes por pedido; el sistema MUST mostrar base del descuento, importe y total antes de compra.
- **FR-042**: El reporte de ventas MUST sumar pagos aprobados y restar reembolsos simulados; MUST excluir pagos pendientes, rechazados y pedidos cancelados.
- **FR-043**: El reporte de productos más vendidos MUST ordenar por unidades pagadas menos unidades devueltas para el periodo visible seleccionado.
- **FR-044**: Las fechas de los reportes y eventos comerciales MUST expresarse con zona horaria de Perú; el plazo de reclamos se calcula en días hábiles.
- **FR-045**: Una cuenta Cliente MUST poder consultar sus solicitudes de atención y sus respuestas; el Visitante que deje una solicitud recibe referencia y medio de seguimiento sin acceder a pedidos.
- **FR-046**: Las quejas y solicitudes de atención MUST transitar Recibida, En revisión, Requiere información, Resuelta y Cerrada, con historial de estado; un reclamo debe ser respondido dentro de su plazo aunque espere información adicional.
- **FR-047**: Las devoluciones MUST transitar Recibida, En evaluación, Aprobada o Rechazada, Producto recibido, Reembolsada y Cerrada; las transiciones registran quién actuó, cuándo y la decisión.
- **FR-048**: El primer Administrador MUST habilitarse fuera del registro público mediante un procedimiento seguro; la administración MUST impedir que se desactive o revoque el último Administrador activo.
### Non-Functional Requirements

- **NFR-001**: La experiencia MUST adaptarse a computadoras, tabletas y celulares y permitir completar las tareas principales con navegación comprensible y controles accesibles.
- **NFR-002**: Todas las operaciones que cambien conjuntamente pedido, pago e inventario MUST conservar la integridad ante éxito, error o solicitudes repetidas; no debe quedar una compra parcialmente confirmada.
- **NFR-003**: Las contraseñas MUST almacenarse únicamente mediante un mecanismo seguro de hash; la información sensible y las credenciales MUST mantenerse fuera de las respuestas visibles a usuarios no autorizados.
- **NFR-004**: El sistema MUST validar los datos ingresados por clientes y administradores y presentar errores accionables sin revelar información sensible.
- **NFR-005**: Las funciones y datos administrativos MUST permanecer inaccesibles a clientes no autorizados, incluso si intentan acceder directamente a una dirección protegida.
- **NFR-006**: La interfaz MUST usar español, soles peruanos y formatos de fecha y hora adecuados para Perú de manera consistente.
- **NFR-007**: La información persistida MUST conservar relaciones válidas entre clientes, direcciones, pedidos, líneas, pagos, productos, movimientos de inventario y solicitudes.
- **NFR-008**: La tienda MUST comunicar estados de carga, resultados vacíos, errores y confirmaciones con mensajes claros, sin perder los datos ingresados cuando sea posible recuperarse del error.
- **NFR-009**: Las tareas principales de registro, búsqueda, compra, seguimiento y administración MUST poder demostrarse desde la interfaz, sin inspeccionar el código fuente.
- **NFR-010**: Para hasta 100 sesiones simultáneas, el 95% de las búsquedas de catálogo y consultas del carrito MUST responder en menos de 2 segundos, excluyendo la latencia de red del usuario.
- **NFR-011**: El sistema MUST recopilar y mostrar solo datos personales necesarios para cuenta, entrega, compra y atención, limitar el acceso por rol y retenerlos según las obligaciones legales aplicables.
- **NFR-012**: Las sesiones administrativas MUST expirar tras 15 minutos sin actividad y las sesiones Cliente tras 30 minutos.
- **NFR-013**: Las fechas y horas de la tienda MUST presentarse en hora de Perú; los reclamos MUST incluir una fecha límite calculada en días hábiles.
- **NFR-014**: Las acciones de gestión y estados MUST conservar actor, fecha, entidad afectada y cambio para revisión administrativa.
- **NFR-015**: Las instrucciones y afirmaciones de cultivo MUST derivarse del contenido aprobado del kit; si falta información, el sistema no presenta recomendaciones inventadas.

### Business Rules

- **BR-001**: Un cliente puede realizar varios pedidos y cada pedido puede incluir varios productos.
- **BR-002**: El precio de cada línea se conserva como precio histórico del pedido, aunque el precio actual del producto cambie.
- **BR-003**: Una compra no puede confirmarse por una cantidad superior al stock disponible, incluso ante solicitudes simultáneas.
- **BR-004**: Un pedido pendiente reserva unidades durante 15 minutos; el pago aprobado convierte la reserva en venta una sola vez y el rechazo no confirma la compra.
- **BR-005**: Un intento pendiente no se presenta como aprobado ni permite otro intento; sin aprobación en 15 minutos, el pedido se cancela y la reserva se libera.
- **BR-006**: Los totales corresponden a la suma de subtotales menos descuentos más cargos de entrega informados antes de confirmar; las reglas de redondeo son uniformes.
- **BR-007**: Un cliente solo puede consultar sus propios perfiles, direcciones, pedidos, pagos y solicitudes; el personal solo puede ejecutar funciones otorgadas por su rol.
- **BR-008**: Los cambios de estado deben ser válidos, conservar su historial y tener fecha y responsable.
- **BR-009**: Una compra, pago e impacto en el inventario deben confirmarse de forma coherente; un fallo no puede dejar datos contradictorios.
- **BR-010**: Los estados de pedido siguen únicamente las transiciones definidas en Clarifications; el cliente no puede cambiar estados de preparación o despacho.
- **BR-011**: Los distritos sin cobertura no pueden generar pedidos y toda tarifa de entrega se muestra antes de confirmar.
- **BR-012**: Solo un descuento es aplicable por pedido; precio, descuento y cargos visibles antes de confirmar forman parte del historial inmutable.
- **BR-013**: El plazo comercial de devolución por cambio de opinión es 7 días calendario desde la entrega, con kit completo, sellado y sin uso; no reduce derechos legales por defecto o falta de conformidad.
- **BR-014**: Un reembolso simulado no repone stock hasta inspección favorable del producto devuelto.
- **BR-015**: Ventas netas excluyen pedidos no aprobados y restan reembolsos; las unidades más vendidas se netean contra devoluciones.
- **BR-016**: Cada queja o reclamo debe recibir una respuesta escrita dentro de 15 días hábiles y conservar código y evidencia de respuesta.

### Key Entities *(include if feature involves data)*

- **Cliente**: Cuenta, datos de perfil, credenciales protegidas y rol; puede tener direcciones y pedidos.
- **Dirección**: Datos de entrega asociados a un cliente; el pedido conserva una copia de los datos utilizados al confirmar.
- **Producto**: Kit con categoría, especie/variedad, dificultad, espacio y luz recomendados, precio, imágenes, contenido, instrucciones y estado de publicación. El inventario registra stock físico, reservado y disponible.
- **Categoría**: Agrupación de productos del catálogo.
- **Carrito**: Selección temporal de productos y cantidades de un cliente.
- **Pedido**: Compra de un cliente con líneas, dirección histórica, importes, estado, fecha y eventos.
- **Línea de pedido**: Producto, cantidad, precio histórico y subtotal al confirmar.
- **Pago simulado**: Intento asociado a un pedido, resultado, importe y fechas de cambio.
- **Movimiento de inventario**: Entrada, ajuste, reserva, venta, liberación o devolución con cantidad, motivo, fecha y responsable.
- **Tarifa de entrega**: Distrito habilitado y tarifa fija vigente mostrada durante checkout.
- **Reserva de inventario**: Unidades retenidas por un pedido Pendiente de pago y hora de expiración.
- **Reembolso simulado**: Registro de devolución del importe simulado, motivo, pedido y decisión administrativa.
- **Libro de Reclamaciones**: Queja o reclamo con datos del consumidor, importe, producto/servicio, detalle, pedido, código y respuesta escrita.
- **Promoción o cupón**: Descuento, periodo de vigencia, condiciones y estado.
- **Reclamo o devolución**: Solicitud de un cliente vinculada a un pedido, motivo, descripción, estado, respuesta e historial.
- **Solicitud de atención**: Consulta derivada al equipo de atención, referencia, cliente si está autenticado, descripción y estado.
- **Evento de estado**: Cambio de estado de pedido, pago o solicitud con fecha, responsable y estado anterior y nuevo.
- **Tarifa de entrega**: Distrito habilitado y tarifa fija visible en checkout.
- **Reserva de stock**: Cantidad del producto apartada para un pedido, con vencimiento a los 15 minutos y estado de conversión o liberación.
- **Reembolso simulado**: Devolución ficticia total o parcial del importe con causa, importe, pedido y fecha.
- **Hoja del Libro de Reclamaciones**: Código, fecha, consumidor, producto/servicio, importe, detalle, pedido solicitado, respuesta y fecha límite.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Al menos 9 de cada 10 participantes de una prueba guiada encuentran un kit y explican precio, contenido, dificultad, espacio y luz recomendados sin ayuda.
- **SC-002**: Al menos 9 de cada 10 clientes de prueba pueden completar registro, agregar productos, revisar el total y llegar al resultado de pago simulado en menos de 5 minutos.
- **SC-003**: En todas las pruebas de compra, los subtotales, descuentos, entrega y total visibles coinciden con el pedido registrado hasta el céntimo.
- **SC-004**: En todas las pruebas con stock limitado, compras simultáneas y vencimiento de reservas, las unidades aprobadas no superan el stock y cada reserva/venta/liberación se aplica exactamente una vez.
- **SC-005**: El 100% de los pagos simulados de prueba conserva su resultado aprobado, rechazado o pendiente y aparece correctamente en el pedido relacionado.
- **SC-006**: El 100% de los intentos de clientes sin rol autorizado de acceder a datos o acciones administrativas son rechazados.
- **SC-007**: Al menos 9 de cada 10 participantes pueden consultar el estado de un pedido propio, registrar una solicitud de atención y encontrar su confirmación sin asistencia.
- **SC-008**: Un administrador de prueba puede completar desde la interfaz la actualización de un producto, un ajuste de stock, la atención de una solicitud y la consulta de ventas y productos más vendidos.

## Assumptions

- El lanzamiento inicial atiende a clientes y administradores en español y muestra importes en soles peruanos.
- Los pagos son simulaciones de demostración; no existe una pasarela real ni se capturan datos de tarjetas.
- Se asume inicialmente una única tienda y un único inventario lógico; no se especifican sucursales ni vendedores externos.
- El visitante puede explorar catálogo y fichas sin cuenta; se requiere una cuenta para comprar y consultar información personal.
- Los resultados de pago son seleccionables en el entorno de demostración y se distinguen claramente de pagos reales.
- Un pedido pendiente de pago retiene stock durante 15 minutos; vencido ese plazo se cancela y libera la reserva.
- Los cupones vencidos o no elegibles se rechazan; una promoción automática y un cupón nunca se acumulan en un pedido.
- El periodo del dashboard y los reportes puede seleccionarse; las estadísticas se calculan con pedidos registrados y sus estados de pago.
- El chatbot solo utiliza contenido aprobado por el administrador y deriva preguntas no resueltas a una solicitud de atención.
- La tienda configura distritos atendidos y una tarifa fija por distrito; el checkout muestra la tarifa antes de confirmar.
- La primera cuenta Administrador se establece mediante un proceso seguro de puesta en marcha, nunca por registro público.
- Los estados de reclamos y devoluciones usan los flujos definidos en los requisitos funcionales; se conserva su historial.
- La tienda no incorpora funciones de marketplace, recomendaciones automáticas, dispositivos conectados ni seguimiento de transportistas.








- No se integran pasarelas de pago reales, transportistas, marketplace ni dispositivos conectados; los pagos y reembolsos son simulados.
- El Libro de Reclamaciones debe contar con enlace visible y accesible y respuesta escrita en un máximo de 15 días hábiles, según la información de [Indecopi](https://consumidor.gob.pe/libro-de-reclamaciones/) y la [Ley publicada en El Peruano](https://busquedas.elperuano.pe/dispositivo/NL/2457203-1).




- **SC-009**: El 100% de los reclamos de prueba recibe código, fecha límite y respuesta escrita dentro de 15 días hábiles.
- **SC-010**: Con 100 sesiones simultáneas, el 95% de búsquedas y consultas de carrito responde en menos de 2 segundos, excluyendo latencia de red.
- **SC-011**: Los reportes de prueba coinciden con pagos aprobados menos reembolsos y unidades pagadas menos devoluciones, según el periodo seleccionado.


