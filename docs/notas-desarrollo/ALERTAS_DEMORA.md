# Alertas de atención

Los límites se configuran en **Administración → Ajustes → Atención** y se guardan en MySQL. Valores iniciales: 20 minutos desde el envío a cocina y 5 minutos desde que la ronda está lista. Se admiten minutos enteros entre 1 y 240 y se pueden desactivar los avisos.

## Comportamiento

- El servidor usa su reloj para determinar las demoras. Solo consulta rondas activas del mesero autenticado; incluye pedidos pagados que todavía requieren entrega.
- La fase de cocina termina al marcar la ronda como lista. La fase lista utiliza `listo_at`. Las fechas faltantes, futuras o inconsistentes no generan tiempos inventados.
- Cada combinación mesero/ronda/fase tiene un registro único en `ronda_alertas`. Una transacción bloquea al mesero al reclamar avisos: dos dispositivos no pueden notificar la misma fase. Los avisos nuevos se agrupan con un mínimo de 120 segundos entre grupos y sin sonido.
- La aplicación consulta cada 30 segundos mientras está en primer plano y al cambiar las fases de los pedidos. Se detiene en segundo plano. Estos avisos son internos; no son notificaciones push con la aplicación cerrada.
- La bandeja conserva las demoras activas aunque ya se hayan avisado. Desaparecen al resolverlas o dejar de superar el límite. Cambiar los límites o desactivar/reactivar no reinicia los avisos anteriores.
- Si falla la conexión se conserva la última consulta con una advertencia. Si falla la confirmación del aviso, no se inventa una notificación confirmada; la ronda sigue visible en la bandeja.
- Express conserva el requisito de cobro completo antes de entregar.

## Interfaz

Pedido agrupa Tiempos y Seguimiento en pestañas accesibles. El estado local/enviado/pendiente está integrado en el encabezado de la cuenta. Las rondas entregadas se consultan bajo demanda. Mesas y Express muestran cuántas rondas tienen demora; la campana abre la bandeja con enlaces a cada pedido.

## Verificación

Pruebas de límites, cambios de fase, configuración, concurrencia, deduplicación, pausa entre grupos y respuestas fallidas. Pruebas de navegador con todas las API y sockets interceptados: sin pedidos, cobros ni entregas reales. Se verificaron 320/390/768/1440 px, zoom 125/150 %, pestañas con teclado, foco, Escape, estados sin conexión y guardado simulado de ajustes. Las capturas y scripts están en `output/playwright/service-alerts-*` y `service-panel-*`.
