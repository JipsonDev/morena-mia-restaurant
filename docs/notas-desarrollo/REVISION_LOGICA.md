# Revisión del flujo de mesero, pedidos y caja

Fecha: 10 de septiembre de 2026.

## Correcciones aplicadas

- **Cuentas después de medianoche:** los pedidos abiertos siguen visibles y mantienen ocupada su mesa aunque sean del día anterior. El listado operativo tampoco pierde cuentas abiertas al superar 100 pedidos. El historial mantiene su límite independiente.
- **Carga de datos del mesero:** se eliminaron las mesas y productos ficticios que aparecían ante errores de conexión o catálogos vacíos. Un error conserva los últimos datos cargados y permite reintentar desde el menú.
- **Doble pulsación al enviar:** una exclusión inmediata impide dos envíos simultáneos. Mientras se espera la respuesta se bloquea la edición del pedido. Un fallo permite reintentar; el carrito solo se limpia tras una respuesta exitosa.
- **Pedidos express:** los pedidos listos para llevar o delivery permiten marcar su entrega. Abrir un pedido express existente ya no muestra el borrador compartido de un pedido nuevo; la división se reinicia al cambiar de pedido.
- **Solicitud de cuenta:** solicitar cobro o imprimir precuenta conserva el estado de cocina. Una cuenta cerrada rechaza estas operaciones.
- **Pago real:** cambiar el estado general a PAGADO requiere usar el flujo de caja. Caja rechaza montos inválidos o distintos del total pendiente; el flujo actual registra un pago completo.
- **Precios y cantidades:** el servidor usa el precio del catálogo, calcula el servicio y el total en centavos y rechaza cantidades fraccionarias/no válidas y extras negativos/no finitos. Los extras personalizados siguen permitidos.
- **Segundas rondas:** cada envío conserva su propio estado y descuenta inventario al preparar o cobrar, una sola vez. La comanda automática incluye únicamente las líneas nuevas; la cuenta conserva todas las líneas.
- **Stock directo:** se consulta y bloquea el stock actual por producto, evitando reutilizar un saldo obsoleto entre líneas. La disponibilidad se evalúa correctamente antes de actualizar el stock, evitando declarar agotado un producto que aún tiene unidades.
- **Creación en una mesa:** se valida que exista y esté activa y se bloquea su fila antes de buscar su cuenta abierta, serializando las creaciones simultáneas de ese flujo.
- **Servicio visible:** al consultar una cuenta guardada se muestra el servicio incluido en su importe, en lugar de mostrar cero.

## Mejoras de lógica pendientes, en orden de prioridad

### Ya implementado: rondas, cobro independiente y cuenta completa

La cocina y el mesero operan por ronda; pagar no retira comida pendiente. Mover, dividir, solicitar cobro y cobrar abarcan toda la cuenta abierta de la mesa. El servidor valida las asignaciones y distribuye los centavos. Detalles, migración y límites en [RONDAS_Y_CUENTAS.md](RONDAS_Y_CUENTAS.md).

### Pendiente: abonos por comensal

La división es una instrucción para caja. El cobro todavía liquida la cuenta completa; no registra saldos y abonos independientes por parte.

### 3. Evitar duplicados por reintentos de red

El bloqueo del botón resuelve pulsaciones simultáneas en esa pantalla, pero una pérdida de conexión después de guardar puede provocar un reintento ambiguo. Enviar un identificador único por intento lógico y guardarlo con una restricción única; un reintento debe devolver el mismo pedido.

Registrar las tareas de impresión y notificación junto con el pedido, y procesarlas con reintentos. Así, un error posterior al commit no se interpreta como si el pedido no se hubiera guardado.

### 4. Registrar el consumo de inventario por detalle

La devolución actual recalcula cantidades con la receta vigente. Si una receta cambia entre preparación y cancelación, puede devolver cantidades diferentes de las consumidas. Guardar los movimientos originales por detalle y revertir esos movimientos.

Definir también la regla para cancelar comida ya preparada: puede corresponder a una merma, sin devolver ingredientes al stock. La unión ahora conserva el indicador de consumo de cada ronda; aún falta un registro inmutable de los ingredientes realmente consumidos.

### 5. Conservar borradores y hacer fiable la sincronización

Guardar carrito, notas y datos express por usuario y mesa/pedido en almacenamiento local. Recuperarlos al reiniciar la app. Distinguir claramente un borrador de una orden confirmada.

La sincronización actual puede descartar una petición de actualización si ya hay otra en curso; una respuesta vieja también puede reemplazar datos recientes. Usar versiones o secuencias de actualización y programar una nueva lectura cuando lleguen eventos durante una sincronización.

### 6. Centralizar reglas y preparar migraciones

Definir una matriz de transiciones por rol: cocina prepara, mesero entrega y caja registra pagos. Varias rutas hoy permiten cambios de estado a todos esos roles.

Mover CREATE/ALTER TABLE fuera de las peticiones y ejecutar migraciones antes de iniciar el servidor. Algunos flujos todavía pueden ejecutar DDL durante una transacción; MySQL puede confirmar implícitamente operaciones anteriores. También conviene que las lecturas de mesas no cancelen pedidos ni reparen estados mediante escrituras automáticas.

## Validación

- `pnpm test` en `restaurante-backend`: 48 pruebas de cuentas, rondas, importes, flujo e impresión.
- `pnpm test` en `mi-restaurante`: 13 pruebas de cuentas, cocina, selección de entregas y exclusión de envíos.
- `pnpm run build:web` en `mi-restaurante`: exportación web de Expo.
- Las pruebas de rutas sustituyen base de datos y notificaciones. Comprueban lógica, consultas y llamadas de transacción; no sustituyen una prueba de concurrencia y rollback contra MySQL real.
- Los tests del backend usan el soporte experimental de sustitución de módulos de Node; ejecutados con Node 26.3.0. No se agregaron dependencias.
- No se modificaron datos de la base de datos ni se desplegó la aplicación. Queda por validar el recorrido completo en un dispositivo, con la base de pruebas y las impresoras.
