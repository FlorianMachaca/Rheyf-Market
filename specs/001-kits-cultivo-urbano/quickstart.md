# Guía de preparación y verificación técnica

Este documento define el orden para poner en marcha la solución y los resultados verificables. El repositorio aún no tiene aplicaciones ni manifiestos de dependencias, así que no se publican comandos ficticios. Al inicializar frontend/backend, actualizar esta guía con comandos comprobados de instalación, migración, ejecución y pruebas, incluyendo versiones requeridas y variables de entorno sin valores secretos.

## Preparación del entorno (pendiente de tareas de inicialización)

1. Instalar las versiones de Node.js/npm (o gestor acordado) que queden fijadas en ambos proyectos y Microsoft SQL Server compatible con el entorno académico.
2. Crear las aplicaciones `frontend/` (React, Vite, TypeScript) y `backend/` (NestJS, TypeScript), documentando versiones exactas en sus manifiestos y archivos de bloqueo.
3. Crear base de datos local de desarrollo, ejecutar migraciones versionadas y cargar las tres categorías iniciales. La cuenta Administrador inicial se provisiona con procedimiento de puesta en marcha que no exponga credenciales ni habilite registro público con rol elevado.
4. Configurar por entorno la conexión SQL, origen del frontend, secreto/configuración de sesión, política de cookies y zona `America/Lima`. Mantener `.env.example` sin valores de credenciales reales y excluir secretos locales del control de versiones.
5. Registrar comandos reales una vez creados: instalación limpia, desarrollo, migración/seed, compilación, lint/format si se adoptan y comandos separados de pruebas unitarias, integración y frontend.

## Datos mínimos para demostración

- Categorías: Verduras, Frutas pequeñas y Plantas aromáticas.
- Un kit publicado por categoría con especie/variedad, dificultad, espacio, luz, contenido, instrucciones aprobadas, imagen con texto alternativo, precio PEN y stock.
- Un distrito habilitado con tarifa fija y al menos uno deshabilitado.
- Una promoción automática y un cupón válidos para comprobar selección del mayor descuento; un cupón inválido/vencido.
- Usuarios Cliente y Administrador creados por los flujos permitidos. No usar contraseñas reales ni crear admin mediante registro público.
- Stock de prueba pequeño, incluida una unidad para pruebas de concurrencia y un pedido elegible para devolución.

## Secuencia de verificación principal

| Caso | Preparación y acción | Resultado esperado |
|---|---|---|
| Catálogo (FR-001..003) | Abrir tienda, buscar, combinar y quitar filtros; consultar producto publicado y producto no publicado. | Solo publicados son visibles; precio S/, stock disponible, contenido, dificultad, espacio/luz e instrucciones claros; no hay coincidencias con explicación. |
| Registro/roles (FR-004..005, BR-007) | Registrar cliente, iniciar/cerrar sesión; intentar acceso admin desde cliente y consultar recurso ajeno. | Registro crea CLIENTE; sesión se revoca; API bloquea el panel y recursos ajenos aunque se invoque directamente. |
| Perfil y dirección | Crear/editar dirección propia; probar distrito con y sin cobertura y una dirección referenciada por pedido. | Dirección válida queda guardada; no se modifica el snapshot histórico; distrito no habilitado impide checkout. |
| Carrito y promociones (FR-006..009, BR-006/012) | Agregar kits, cambiar cantidad, quitar línea; alterar precio/stock; aplicar promoción y cupón simultáneamente. | Cantidad sin stock se rechaza; interfaz muestra precio actual y cambio; una sola mayor rebaja; envío y total se presentan antes de aceptar. |
| Compra aprobada (FR-010..013, BR-002..006/009) | Confirmar con dos líneas; completar simulación aprobada y recargar pedido. | Reserva expira a +15 min; pago y pedido confirmados una sola vez; stock baja una sola vez; snapshot del precio/dirección/totales y eventos se conserva. |
| Rechazo, pendiente y expiración | Rechazar y reintentar; crear pago pendiente; intentar segundo intento; dejar una reserva vencer; simular resolución tardía. | Rechazo no es venta; reintento comparte pedido/reserva; pendiente bloquea otro intento; vencimiento cancela y libera; aprobación tardía se rechaza. |
| Carrera de última unidad (BR-003/004) | Enviar dos checkouts concurrentes que compiten por la misma última unidad en SQL Server. | Solo un pedido adquiere reserva; disponibilidad no queda negativa; rollback deja pedido, pago e inventario coherentes. Repetir solicitud idempotente no duplica efectos. |
| Seguimiento/cancelación | Probar transiciones válida e inválida, cancelación del cliente pendiente y cancelación admin antes de despacho. | Cliente solo consulta lo propio; transiciones se auditan; se libera reserva o stock no enviado exactamente una vez; no hay cancelación después del despacho. |
| Inventario/admin (US4) | Registrar entrada, ajuste y salida; probar salida excesiva; consultar saldo y ledger. | Solo admin; motivo requerido; no saldo negativo ni reducción debajo de reservas; historial atribuye actor y fecha. |
| Reclamo (FR de Libro; BR-016) | Enviar queja/reclamo sin cuenta y consultar por código; atender respuesta. | Recibo incluye código y fecha límite en días hábiles; consulta no revela contacto; respuesta escrita y eventos quedan conservados dentro del plazo definido. |
| Devolución/reembolso (BR-013/014) | Probar cambio de opinión dentro/fuera de siete días, kit abierto y reclamo por defecto; aprobar, recibir e inspeccionar apto/no apto. | Política comercial se aplica solo a cambio de opinión y no limita defecto; reembolso es simulado; stock se repone solo con inspección apta y una vez. |
| Chatbot (FR chatbot) | Consultar ficha/instrucciones disponibles, instrucción ausente, pedido propio/ajeno y pedir asistencia. | Responde desde fuente aprobada; ante falta de dato lo reconoce; exige sesión para privado, limita a datos propios y crea referencia de atención al derivar. |
| Reportes (BR-015) | Preparar en periodo local pagos aprobados, pendientes/rechazados, cancelación, devolución parcial/total. | Ingresos netos = pagos aprobados menos reembolsos; unidades más vendidas restan devueltas; excluye resultados no aprobados y muestra periodo en hora peruana. |

## Estrategia de pruebas

1. **Unitarias backend**: cálculo monetario/redondeo, elegibilidad de promoción, grafo de estados, permisos, plazos de devolución/reclamo y reglas de stock.
2. **Integración SQL Server**: restricciones/FK, snapshots, ledger, reserva/expiración, transacciones, idempotencia, doble aprobación/reembolso y acceso concurrente a última unidad. La prueba de concurrencia debe ejecutarse en SQL Server; no sustituirla con SQLite.
3. **Contrato REST**: validación de OpenAPI/DTO; códigos de error, roles, propiedad del recurso, mensajes seguros y ausencia de datos bancarios/hash/sesión en respuestas.
4. **Integración frontend/API**: sesión cookie, CSRF en mutaciones, errores de stock/cobertura, carrito anónimo reconciliado al login y renderizado de moneda/fecha para Perú.
5. **UI end-to-end**: escenarios independientes de cada historia de usuario y criterios de aceptación de `spec.md`, en viewport móvil/tablet/escritorio y con teclado/lector cuando corresponda.
6. **Accesibilidad y rendimiento**: controles semánticos, foco, contraste y texto alternativo; carga representativa para verificar p95 menor de 2 s en catálogo/carrito a 100 sesiones simultáneas excluyendo red, según NFR-010.
7. **Regresión de documentos y migraciones**: desplegar una base limpia y una base previa, aplicar migraciones y confirmar seed; comprobar que `data-model.md` y OpenAPI coincidan con el esquema y DTO implementados.

No se ha ejecutado ninguna prueba; esta es una estrategia de aceptación para convertir a tareas. El resultado de cada ejecución deberá registrarse con comandos reales, entorno, fecha y salida, sin afirmar éxito antes de ejecutarlo.
