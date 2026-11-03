# AGENTS.md — Rheyf-Market

## 1. Información del proyecto

Nombre: Rheyf-Market.

Descripción: Plataforma de comercio electrónico especializada en kits para cultivar verduras, frutas pequeñas y plantas aromáticas en espacios reducidos, como departamentos, balcones y terrazas.

## 2. Tecnologías

- Frontend: React, Vite y TypeScript.
- Backend: NestJS y TypeScript.
- Base de datos: Microsoft SQL Server.
- Comunicación entre frontend y backend mediante API REST.
- Control de versiones con Git y GitHub.

## 3. Organización

- `frontend/`: tienda para clientes y panel administrativo.
- `backend/`: API, servicios y lógica de negocio.
- `database/`: scripts SQL, tablas y procedimientos para el catálogo, inventario, pedidos y operaciones relacionadas.
- `docs/`: documentación técnica y funcional del comercio de kits de cultivo.
- `.specify/`: configuración, memoria y constitución de Spec Kit.
- `specs/`: especificaciones aprobadas y estado del producto.

## 4. Convenciones

- Mantener el código modular y organizado; utilizar nombres descriptivos.
- Respetar la arquitectura y las especificaciones aprobadas; no ampliar el alcance sin aprobación.
- Mantener toda la interfaz, mensajes y contenido de atención en español.
- Mostrar precios y totales en soles peruanos (S/) y utilizar formatos adecuados para Perú.
- Describir con claridad el tipo de cultivo, dificultad, contenido e instrucciones de cada kit; no publicar afirmaciones ambientales que no estén respaldadas.
- Validar los datos en frontend y backend y mantener la coherencia entre pedidos, pagos simulados e inventario.
- Proteger las rutas y operaciones administrativas según los roles autorizados.
- Usar únicamente pagos simulados; no guardar información bancaria real.
- Proteger las contraseñas con hash seguro y no incluir credenciales ni secretos en el código.

## 5. Ejecución y pruebas

Documentar los comandos reales de instalación, ejecución y pruebas cuando se configure cada aplicación. No inventar comandos ni afirmar que las pruebas pasaron sin ejecutarlas.

## 6. Control de versiones

Utilizar Git con ramas para las funcionalidades. Realizar commits descriptivos y verificar las pruebas antes de integrar cambios.

Las reglas del producto viven en `.specify/memory/constitution.md` y las especificaciones y el estado del producto en `specs/`.
