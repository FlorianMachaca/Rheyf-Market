<!-- Sync Impact Report
Version change: 1.0.0 → 2.0.0 (redefinición del objetivo comercial del proyecto)
Modified principles: nuevo II. Cultivo accesible y responsable; antiguos II–V renumerados; seguridad e integridad comercial conservadas en V, con alcance adaptado a kits de cultivo
Added/renamed sections: Stack y alcance funcional → Alcance comercial; Calidad y aceptación conservada
Removed sections: ninguna
Follow-up TODOs: confirmar la fecha original de ratificación
-->
# RHEYF MARKET Constitution

## Core Principles

### I. Simplicidad y evolución incremental
Construir una solución sencilla, organizada y escalable. Priorizar las funciones necesarias para una primera versión funcional; evitar complejidad innecesaria y duplicación. Toda funcionalidad debe corresponder a un requisito especificado y aprobado. Proponer las mejoras fuera de alcance antes de implementarlas.

### II. Cultivo accesible y responsable
RHEYF MARKET se dedica a facilitar el cultivo de verduras, frutas pequeñas y plantas aromáticas mediante kits pensados para departamentos, balcones, terrazas y otros espacios reducidos. La oferta y sus instrucciones deben ayudar a personas con distintos niveles de experiencia a empezar y cuidar sus cultivos. Promover el uso responsable de recursos y prácticas sostenibles; toda afirmación ambiental debe ser clara y estar respaldada por información del producto, sin promesas no verificadas.

### III. Experiencia en español y accesible
Toda interfaz, mensaje, botón, formulario y notificación debe estar en español, ser claro y fácil de usar. Mostrar importes en soles peruanos (S/) y fechas y horas con formatos adecuados para Perú. La experiencia debe ser moderna, consistente, intuitiva, adaptable a computadoras, tabletas y celulares, y considerar accesibilidad.

### IV. Arquitectura modular y responsabilidades claras
Usar React, Vite y TypeScript en el frontend; NestJS y TypeScript en el backend; Microsoft SQL Server como base de datos, y una API REST entre frontend y backend. Organizar módulos mantenibles y separar las responsabilidades de presentación, lógica de negocio, API y persistencia.

### V. Seguridad e integridad de las operaciones
Implementar autenticación y autorización por roles. Proteger las contraseñas con algoritmos seguros de hash; validar datos en frontend y backend; y restringir las operaciones administrativas a usuarios autorizados. No guardar contraseñas en texto plano, incluir secretos o credenciales en el código, ni usar o almacenar datos reales de tarjetas bancarias.

Mantener coherentes las reglas de negocio y los datos persistidos. Las compras deben preservar la integridad de pedidos e inventario, conservar el precio histórico y evitar confirmar cantidades superiores al stock. La gestión incluye kits, categorías, precios, inventario, clientes, pedidos, pagos simulados, promociones, reclamos y devoluciones. Los pagos son exclusivamente de prueba, con resultados aprobados, rechazados o pendientes y sin cobros reales. El chatbot atiende consultas sobre productos e instrucciones de cultivo, compras, pedidos y reclamos, y permite registrar solicitudes de atención.

## Alcance comercial

La plataforma de comercio electrónico RHEYF MARKET permite a clientes descubrir y comprar kits para cultivar verduras, frutas pequeñas y plantas aromáticas en espacios reducidos, y a administradores gestionar catálogo, inventario y operaciones de la tienda. La oferta debe identificar el tipo de cultivo, el nivel de dificultad, el contenido del kit y las instrucciones pertinentes. La sostenibilidad y la facilidad de cultivo orientan la selección y presentación de productos, sin ampliar el alcance a productos ajenos a este propósito salvo aprobación expresa.

La versión demostrable debe permitir registro e inicio de sesión, búsqueda y consulta de productos, carrito y compra, simulación de pagos, seguimiento de pedidos, administración de productos, clientes, inventario y pedidos, gestión de promociones, registro y atención de reclamos y devoluciones, y uso del chatbot.

## Calidad y aceptación

Mantener código limpio, modular y mantenible, gestionar errores con mensajes comprensibles y mantener documentación de requisitos, arquitectura, base de datos, instalación, ejecución y pruebas. Verificar las funciones principales con pruebas y validar reglas de negocio y consistencia de datos. Una funcionalidad solo se considera terminada cuando su funcionamiento puede comprobarse mediante la interfaz; la demostración no debe requerir inspeccionar el código fuente.

## Governance

Esta constitución establece los principios obligatorios para especificar, diseñar, implementar y revisar el proyecto. Toda propuesta de cambio debe documentar los principios o secciones afectados, su motivo y las consecuencias para requisitos existentes. Aprobar y registrar las enmiendas en esta constitución antes de aplicarlas como regla del proyecto.

Aplicar versionado semántico: incrementar MAJOR ante la eliminación o redefinición incompatible de principios u objetivo comercial; MINOR al añadir principios o ampliar sustancialmente las obligaciones; PATCH para aclaraciones y correcciones sin cambio sustantivo. En revisiones de especificaciones, planes, tareas y cambios, comprobar su cumplimiento con esta constitución y resolver cualquier conflicto antes de considerar aprobado el trabajo. Mantener la documentación del proyecto alineada con las reglas vigentes.

**Version**: 2.0.0 | **Ratified**: TODO(RATIFICATION_DATE): confirmar la fecha original de adopción | **Last Amended**: 2026-11-03

