# Borradores Express por cliente

Cada acción «Para llevar» o «Delivery» crea un borrador independiente. En Express → Borradores se puede buscar, continuar o eliminar cada cliente; también se recuperan desde el acceso a borradores del encabezado. Los nombres repetidos conservan identificadores distintos.

Cada borrador mantiene productos, cantidades, extras, notas, tipo de pedido, nombre, teléfono, dirección y referencia. Se guarda localmente por usuario y equipo; no es una reserva de existencias ni se envía a cocina hasta confirmar el pedido. Al enviar se revisan precios y disponibilidad.

El formato anterior se migra a versión 2 sin cambiar la clave de almacenamiento. Se conservan los borradores de mesas, las rondas de cuentas existentes y el contenido exacto de solicitudes pendientes. Un envío sin confirmar bloquea edición y eliminación de ese cliente; su reintento conserva el mismo `request_id`. Los demás clientes pueden seguir trabajando. Completar uno elimina solamente su borrador.

## Verificación

- `npm test`: 76 pruebas correctas, incluidas 7 pruebas específicas de separación, migración, eliminación y protección de pendientes.
- `npm run build:web`: exportación web correcta.
- Navegador aislado con API y socket simulados: tres clientes simultáneos, recuperación al recargar, productos/notas/direcciones independientes, envío individual, respuesta HTTP 503, reintento idéntico tras recarga, eliminación con confirmación y ronda adicional sin heredar el cliente de otro borrador.
- Vista y recuperación desde el modal verificadas en 320, 390, 768 y 1440 px; ampliación de 125 % y 150 %, contenedor 17 px más estrecho, controles de al menos 44 px CSS y búsqueda sin acentos con clientes de igual nombre. Sin errores JavaScript en estos flujos.
- No se crearon pedidos ni ventas en la base de datos real. La sincronización de borradores entre equipos queda fuera de este cambio.

Código principal: `orderDrafts.mjs`, `useWaiterDrafts.js`, `ExpressDraftList.js`, `WaiterExpress.js` y `WaiterApp.js`. Scripts de revisión en `output/playwright/express-drafts-*.js` y `send-express-drafts.js`.
