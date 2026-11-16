# Mesas del administrador

- Vista independiente `TablesScreen`, con tarjetas SVG, número, sillas configuradas, ubicación y pedidos activos.
- Paleta compartida con el administrador: disponible verde, ocupada ámbar, cuenta solicitada rosa, reservada azul e inactiva gris. Adaptable a claro/oscuro y escritorio/móvil.
- Resumen de mesas activas, disponibles, en servicio y capacidad total. Búsqueda, filtros de estado/ubicación y paginación incremental.
- Formulario con vista previa, validación de enteros y ubicaciones activas, aviso de cambios sin guardar y bloqueo de doble envío.
- Ediciones envían únicamente metadatos modificados: no reenvían un estado antiguo ni sobrescriben capacidad o ubicación sin cambios.
- Reservar/liberar, solicitar cuenta/continuar atención y reactivar usan confirmaciones dentro de la aplicación y muestran errores del servidor.
- Desactivar conserva historial. Eliminar solo permite mesas sin pedidos históricos. Con pedidos activos se impide desactivar, liberar, reservar, renumerar o eliminar.
- Servidor: bloqueo por catálogo de mesas dentro de la base, transacciones y bloqueo de fila compartido con creación de pedidos. Evita números activos duplicados entre solicitudes simultáneas y carreras durante eliminación. No cambia el esquema ni los datos existentes.
- Se retiró la vista antigua de Mesas de `AdminSectionScreen` para evitar mantener dos implementaciones.

## Verificación

- Frontend: 35 pruebas aprobadas.
- Backend: 86 pruebas aprobadas.
- Integración en una copia temporal local de MySQL: creación simultánea del mismo número (una creación y un conflicto), ubicación inválida, edición sin alterar reserva, mesa inactiva, pedidos activos, historial y eliminación. La copia se elimina al finalizar.
- Compilación web de producción aprobada.
- Revisión visual en navegador: modo claro/oscuro, escritorio y móvil, formularios de creación/edición, validación de sillas y descarte de cambios. No se guardaron modificaciones de prueba sobre mesas reales.
