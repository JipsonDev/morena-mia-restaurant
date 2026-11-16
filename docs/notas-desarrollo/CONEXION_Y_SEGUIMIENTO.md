# Conexión y seguimiento del pedido

## Estado de conexión y envío

El mesero ve la última sincronización correcta de mesas y pedidos, con hora y segundos. La hora cambia cuando una lectura válida actualiza esos datos; conectar el socket, cambiar de pantalla o fallar una consulta no la adelanta. Se distinguen sincronización en curso, conexión en vivo, reconexión y errores al actualizar.

El detalle del pedido, el acceso fijo al pedido en Menú y los borradores Express diferencian:

- **Guardando en este equipo:** la escritura local aún no ha terminado.
- **Guardado en este equipo · sin enviar:** persistencia local confirmada para ese usuario y dispositivo.
- **No guardado en este equipo:** falló el almacenamiento; conservar la página abierta.
- **Pendiente de confirmar:** se conserva la solicitud original para reintentar con el mismo identificador, incluso tras recarga. Recuperar conexión no la convierte en confirmada.
- **Enviado · confirmado por el servidor:** existe una respuesta válida para la cuenta y destino originales. Una respuesta incompleta o de otra cuenta no limpia el borrador.

Las escrituras locales en cola solamente actualizan el estado de guardado de su revisión más reciente, evitando que una escritura antigua marque como guardados cambios posteriores que fallaron.

## Seguimiento desde el detalle

«Seguimiento del pedido» se consulta al desplegarlo en Pedido o en el detalle del historial Express. Incluye quién tomó el pedido, qué productos se enviaron por ronda, quién la añadió, quién la entregó y quién registró cada cobro. También conserva cobros anulados/devueltos y sus reversos cuando están registrados.

El mesero asignado se identifica aparte: no se presenta automáticamente como autor de acciones realizadas por otras personas. Los responsables nuevos de creación y entrega se toman de la sesión autenticada y se guardan dentro de la misma transacción. Un reintento de entrega no reemplaza al responsable original. Las rondas conservan su cuenta de origen al unir cuentas.

Los registros antiguos aprovechan la auditoría cuando existe una asociación inequívoca. Si falta un responsable se muestra **Usuario no registrado**; nunca se atribuye un cobro al dueño de la caja por deducción. Las entradas antiguas sin número de ronda se indican expresamente.

## Persistencia y acceso

- Migración aditiva, repetible al iniciar el servidor: cinco columnas opcionales en `pedido_rondas` para origen y responsables de creación/entrega con nombre histórico. No rellena identidades antiguas ni cambia estados o importes.
- Endpoint autenticado `GET /api/pedidos/:id/seguimiento`, permitido a administración, mesero y caja. Consulta acotada a la cuenta y sus rondas; no entrega documentos fiscales, referencias bancarias ni el JSON completo de auditoría.
- Carga bajo demanda, cancelación al cambiar de cuenta/cerrar el detalle, estados de error y reintento. Los contadores de tiempo no repiten las consultas de seguimiento.

Implementación: `waiterSync.mjs`, `useWaiterDrafts.js`, `OrderTracking.js`, `WaiterNavigation.js`, `WaiterApp.js`, `ExpressDraftList.js`, `ExpressHistory.js`, `services/orderTracking.js` y `routes/pedidos.js`.

Las pruebas de navegador usan una sesión aislada con toda la API y el socket interceptados. La lectura de compatibilidad con MySQL consulta registros existentes sin crear pedidos, pagos ni entregas de prueba.

## Verificación

- Frontend: `npm test`, 88 pruebas correctas.
- Backend: `node --experimental-test-module-mocks --test`, 103 pruebas correctas.
- Navegador: hora de sincronización conservada tras HTTP 503, actualización correcta posterior, fallo de almacenamiento visible y recuperación, envío sin respuesta, recuperación al recargar, respuesta malformada sin pérdida del pendiente y reintento con idéntica solicitud.
- Seguimiento: cuatro responsables distintos, error y respuesta inválida con reintento, teclado/estado expandido, consulta desde historial cerrado y ausencia de consultas nuevas por los contadores de tiempo.
- Revisión en 320, 390, 768 y 1440 px, zoom 125 % y 150 %, contenedor 17 px más estrecho y temas claro/oscuro. Sin errores JavaScript en los flujos comprobados.
- Migración aplicada: cinco columnas verificadas en MySQL; consulta de seguimiento sobre un pedido existente correcta, sin modificar sus datos.
