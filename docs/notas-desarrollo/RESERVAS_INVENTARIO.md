# Reservas al confirmar comandas

Implementado el 13 de septiembre de 2026.

- Confirmar una ronda reserva existencias en la misma transacción que guarda sus detalles, la clave de reintento y la comanda de impresión. Si falta un producto o insumo, se revierte toda la operación.
- La disponibilidad para vender es el stock físico menos las reservas. Los borradores no reservan existencias.
- Las recetas descuentan la disponibilidad del insumo más limitado. Se suman los ingredientes compartidos por productos y variantes, con precisión de tres decimales.
- La reserva conserva los ingredientes y cantidades confirmados. Cambiar una receta después no modifica comandas existentes.
- Se convierte la reserva en consumo al iniciar preparación o completar el cobro, según cuál ocurra primero, conservando el comportamiento previo del cobro. La segunda operación no vuelve a descontar.
- Cancelar antes del consumo libera la reserva. Se conserva la regla anterior de restaurar existencias al cancelar un pedido consumido; distinguir devoluciones y mermas de alimentos preparados requiere un flujo adicional.
- No se permite borrar pedidos con historial de reservas; deben cancelarse. Tampoco se permite reducir stock por debajo de lo comprometido. Si ya había faltantes, se permite reponer parcialmente.
- Administración muestra cantidades físicas, reservadas y disponibles de los insumos. El menú usa disponibilidad vendible y se actualiza tras eventos de pedidos.

## Activación local

El arranque agrega `stock_reservado` a productos e insumos y crea `stock_reservas`. Después reserva rondas anteriores no canceladas cuyo inventario aún no se descontó. La migración es repetible. No cambia el stock físico ni cancela pedidos antiguos. Si los compromisos antiguos superan las existencias, los informa y bloquea nuevas ventas de ese recurso.

Se creó una copia de seguridad local antes de activar el cambio: `restaurante-backend/.tmp-runtime/antes-del-arranque-1789320770350.sql`. Se migró una ronda anterior sin detectar faltantes.

## Verificación

- Backend: `pnpm test`, 85 pruebas aprobadas.
- Frontend: `pnpm test`, 27 pruebas aprobadas.
- Compilación: `pnpm build:web`, correcta.
- Integración: desde backend, `node --experimental-test-module-mocks scripts/verifyStockReservations.mjs`. Crea una copia temporal de la base MySQL local y la elimina al terminar. Comprueba ventas simultáneas de la última unidad, recetas compartidas, reintentos, cancelaciones, cobro previo a preparación, cambios de receta, bloqueo de borrado y ajustes incompatibles, rollback de comandas y migración con faltantes.
- La aplicación sirve en localhost:8081. La revisión visual del inventario quedó limitada por la sesión cerrada del navegador; no se ingresaron credenciales ni se crearon pedidos de prueba en la base real.
