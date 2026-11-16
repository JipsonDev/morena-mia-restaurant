# Rondas de cocina, cobro y cuenta completa

Implementado el 10 de septiembre de 2026.

## Comportamiento

- Cada envío crea una `pedido_rondas`, con productos, notas, estado y tiempos propios. Añadir otra ronda no reinicia las anteriores.
- Cocina prepara y marca listas las rondas. El mesero entrega cada ronda lista por separado. Los cambios repetidos al mismo estado no descuentan inventario otra vez; se rechazan saltos de estado y acciones de otro rol.
- El cobro tiene estado independiente en la API (`estado_cobro`). Una cuenta pagada con comida pendiente continúa visible en cocina y meseros. La mesa se libera cuando no quedan cuentas por pagar ni rondas por entregar.
- El inventario se descuenta una sola vez por ronda, al iniciar preparación o al cobrar, lo que ocurra primero. Preparar después del pago no repite el consumo.
- Mover traslada todos los pedidos operativos de la mesa. Las cuentas abiertas se unen a la cuenta destino conservando productos y rondas; la comida pagada pendiente también se traslada sin sumarse al saldo.
- Caja muestra una cuenta por mesa. Solicitar cobro, cobrar y dividir consolidan todos los pedidos abiertos de esa mesa dentro de la transacción.
- La división igualitaria o por productos se calcula en el servidor, distribuyendo los centavos sobrantes de forma determinista e incluyendo el servicio ya guardado. Por productos, toda unidad debe asignarse exactamente una vez y pertenecer a la cuenta.
- Al añadir productos o unir cuentas se invalidan las divisiones anteriores para evitar importes desactualizados. La división sigue siendo una instrucción de reparto para caja: el registro de cobro continúa siendo por el total, sin abonos independientes por comensal.
- Las comandas automáticas y manuales de cocina identifican la ronda y contienen solo sus productos.

## Datos existentes y arranque

Al iniciar el backend se ejecuta `initializeOrderSchema()` antes de aceptar conexiones. Crea las rondas y añade `detalle_pedidos.ronda_id`; agrupa los productos históricos sin ronda en una ronda por pedido. Las ejecuciones posteriores conservan las asignaciones existentes.

Los pedidos antiguos pagados se migran como entregados porque su estado anterior no permite reconstruir si quedaba comida por entregar. Antes del primer arranque de esta versión, revisar esos pedidos y realizar una copia de la base. Detener las versiones anteriores del backend durante el cambio; no deben seguir creando detalles sin ronda.

Actualizar backend y frontend juntos: cocina y meseros usan `PUT /pedidos/:id/rondas/:rondaId`. La ruta de estado de la cuenta rechaza los cambios de preparación y entrega.

Las tablas nuevas necesitan permisos de CREATE y ALTER en la base configurada. Un fallo de la migración impide iniciar el servidor; no se continúa con un esquema incompleto. La implementación conserva `pedidos.estado = PAGADO` para compatibilidad con caja, y calcula la preparación desde las rondas.

## Verificación y límites

Las pruebas automatizadas cubren estados por ronda, pago antes de preparar, entrega parcial, stock sin doble descuento, traslado de una mesa con varios pedidos y comida pagada, conservación de centavos, asignaciones inválidas, bloqueos de cuentas, migración reiniciable e impresión por ronda. La persistencia de las pruebas se simula: falta ejecutar la migración y probar concurrencia y rollback contra una base MySQL de pruebas.

La exportación web de Expo compila. Sin una impresora física todavía solo puede comprobarse la generación, la cola y el agente en simulación, no la salida de papel.


## Express: pago antes de entregar e historial diario

- Express y Delivery requieren pago completo registrado en caja antes de marcar una ronda como entregada. La regla se valida con la cuenta bloqueada en MySQL; no depende de los botones del navegador. Abonos no habilitan la entrega. Preparar y marcar listo siguen siendo posibles antes de pagar. El servicio de mesas conserva su flujo de entregar y cobrar después.
- El sistema no implementa una excepción de pago contra entrega, ni considera una app de reparto como comprobación automática del pago. Para esos casos hará falta un flujo de liquidación específico.
- Pedidos antiguos entregados sin cobrar siguen activos con su saldo real y un aviso. No se modifican pagos ni entregas históricas.
- Express → Historial por día muestra exclusivamente cuentas pagadas sin rondas pendientes de entrega. Se agrupan por el día de cierre (el posterior entre pago y última entrega), usando la fecha del servidor MySQL. No se borran registros: se consultan en una vista separada, de 50 pedidos por página, con detalle de productos.
- Pedido distingue cocina, pago y entrega; separa total y saldo registrado. En escritorio, productos a la izquierda y resumen/acciones a la derecha; en móvil, apilados. Una cuenta recién completada permanece consultable en el detalle seleccionado, aunque deja de contarse en Activos.
- Verificación: 83 pruebas backend, 27 frontend y exportación web aprobadas. Prueba SQL en copia temporal confirma el filtro de historial y el día de cierre. Revisión visual de Pedido en escritorio y móvil e historial diario, sin cobrar ni entregar pedidos reales.
