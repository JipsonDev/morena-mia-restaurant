# Tiempos de atención del mesero

Mesas y Express muestran la espera del pedido y el tiempo que lleva lista la ronda pendiente más antigua. Pedido detalla cuándo se envió cada ronda, su espera, desde cuándo está lista y, después de entregarla, la hora de entrega y el tiempo que transcurrió desde que estuvo lista.

## Criterios

- La espera empieza en `pedido_rondas.created_at`, al registrar el envío en el servidor. No representa tiempo desde que el cliente se sienta: ese evento aún no se registra.
- La espera de la mesa usa su ronda sin entregar más antigua, incluyendo cuentas pagadas que aún tienen comida pendiente. Al entregarla, pasa a la siguiente ronda pendiente. Las canceladas quedan excluidas.
- El tiempo de lista usa `listo_at` de la ronda, incluso si otras rondas de la misma cuenta siguen en preparación. Al entregar se detiene en `entregado_at`.
- Cada ronda conserva sus fechas propias; una ronda adicional no hereda la apertura ni la hora de lista de la cuenta. Las cuentas pagadas y completamente entregadas salen del espacio operativo conforme al comportamiento existente.
- Se actualiza un reloj compartido cada 15 segundos en las vistas operativas; no se realizan consultas adicionales por contador. El reloj se pausa al ocultar la aplicación y se recalcula al volver. No reinicia las fechas al recargar.
- Registros antiguos sin fecha, fechas inválidas, futuras o incoherentes se muestran sin tiempo calculado. Nunca se sustituye la hora de lista por la de envío ni por la hora de apertura de la pantalla.
- Las horas se presentan según la zona horaria del dispositivo. Los días anteriores incluyen fecha completa. Los tiempos transcurridos se calculan sobre los instantes originales del servidor y el reloj del dispositivo.

Se reutilizan las columnas existentes en MySQL y las respuestas de pedidos/eventos actuales. No requiere migración, nuevas dependencias ni modificaciones de pedidos reales para verificar el cambio.

## Código

- `src/utils/serviceTiming.mjs`: fechas, cálculos por ronda y resumen de cuenta/mesa.
- `src/services/useServiceClock.js`: actualización y suspensión del reloj compartido.
- `src/components/ServiceTiming.js`: indicadores y detalle de rondas.
- Integración en `WaiterApp.js`, `WaiterFloor.js`, `WaiterTableCard.js` y `WaiterExpress.js`.
- `tests/serviceTiming.test.mjs`: separación de rondas, cuentas pagadas, entregas, datos antiguos y fechas inválidas/futuras.

Los scripts de revisión del navegador están en `output/playwright/service-timing-*.js`, con API y socket interceptados en una sesión aislada; no escriben a la base de datos real.

## Verificación

- `npm test`: 83 pruebas correctas, incluidas 7 pruebas específicas de tiempos.
- Navegador: avance automático del reloj sin consultas, retiro de la ronda entregada del resumen, recarga, duración de entrega detenida, rondas del día anterior, fechas faltantes y tiempos visibles mientras se añade una nueva ronda.
- Mesas, Express y Pedido en 320, 390, 768 y 1440 px; zoom 125 % y 150 % y contenedor 17 px más estrecho. Revisión visual en modo oscuro y claro, sin errores JavaScript en estos flujos.
- Se conserva la exigencia de pago completo antes de entregar Express. No se probaron dispositivos Android/iOS físicos.
