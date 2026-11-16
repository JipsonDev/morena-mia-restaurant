# Correcciones de caja

## Cambios

- El resumen devuelve `totales_turno` y `efectivo_esperado` calculados con todos los pagos y movimientos de la caja abierta. El listado visible conserva su límite, pero ya no sirve para calcular el cierre. Incluye turnos iniciados el día anterior.
- El efectivo esperado suma apertura, pagos activos en efectivo e ingresos en efectivo; resta retiros y gastos en efectivo. Anulaciones y devoluciones ya están excluidas de los pagos activos y no se descuentan nuevamente. Los movimientos por tarjeta o transferencia no cambian el efectivo físico esperado.
- Apertura, reapertura, cobro, movimientos, anulaciones, devoluciones, edición y cierre se coordinan mediante una fila bloqueada de `caja_control` dentro de sus transacciones. Se conserva la regla existente de una caja compartida para el restaurante.
- Cerrar exige `caja_id` y monto contado válido. Una pantalla que conserva un turno anterior no puede cerrar el nuevo. Abrir simultáneamente no crea dos cajas. Cobrar y registrar movimientos se ejecuta antes del cierre o se rechaza después.
- Montos negativos, vacíos, no numéricos, infinitos, fuera del rango y con fracciones de centavo se rechazan. Movimientos y cobros deben tener importe positivo.
- Anular o devolver un pago de una caja cerrada exige reabrirla con el permiso existente. Así no se cambian sus ventas después de guardar un cierre. Se conservan autorizaciones y auditoría.
- Se bloquean pulsaciones simultáneas en los formularios de caja y en ventas directas. El carrito no admite cambios mientras se envía. Esto no sustituye una futura clave de idempotencia para reintentos después de perder la respuesta de red.
- El método Mixto se retira del selector y se rechaza en el servidor: aún no existe un desglose que permita saber qué parte fue efectivo y qué parte fue tarjeta. Los pagos mixtos históricos no se reconstruyen automáticamente.
- Se conserva el total del pedido calculado por el servidor. Las ventas directas ya no añaden en pantalla un IVA extra que el backend no sumaba. El desglose considera los precios de productos con IVA incluido, según la configuración actual, y muestra el servicio guardado en la cuenta. IVA cero permanece en cero. El impuesto declarado se calcula en el backend; no se confía en el enviado por el cliente.
- El cobro devuelve la cuenta guardada y su desglose para imprimirlos. Un fallo de lectura antes de responder revierte la transacción. Los errores posteriores de impresión o actualización de pantalla no deben interpretarse como un pago sin registrar.
- En web se reserva la ventana del comprobante al pulsar cobrar y se espera la carga del logo antes de imprimir. Una impresión manual anterior al pago se identifica como precuenta; el ticket pagado muestra su número de comprobante.
- Lecturas antiguas de caja y pedidos no reemplazan respuestas más recientes. Un error temporal al leer pedidos conserva las cuentas ya cargadas.

## Verificación y puesta en uso

- `pnpm test` en backend: 63 pruebas aprobadas, incluidas 15 nuevas de caja.
- `pnpm test` en frontend: 14 pruebas aprobadas, incluida la comparación del desglose del comprobante con el backend.
- `pnpm run build:web`: exportación de Expo verificada.
- Las pruebas de caja usan persistencia simulada y un bloqueo simulado. No demuestran el funcionamiento de las transacciones, DDL o concurrencia en un servidor MySQL real; eso debe comprobarse en una base de pruebas.
- El arranque prepara el esquema de caja antes de aceptar peticiones, incluida `caja_control`. Actualizar backend y frontend juntos: el cierre ahora requiere `caja_id` y la pantalla usa los nuevos totales del resumen.
- No se modificaron cierres, pagos ni datos de producción. No se desplegó en Render. Tampoco se comprobó salida de papel en una impresora física.
- Los comprobantes generados por esta aplicación no incorporan por estos cambios una integración de facturación electrónica.
