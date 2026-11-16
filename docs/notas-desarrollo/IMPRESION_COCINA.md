# Cocina e impresión automática de comandas

## Qué quedó corregido

- Cocina mantiene pendientes, preparaciones y pedidos listos del día anterior en su columna correspondiente. Su consulta incluye todas las cuentas abiertas y un historial reciente, sin recortar los activos a 160 pedidos.
- Las lecturas se coordinan para no aplicar una respuesta vieja cuando llega un evento nuevo. Si ocurre un evento durante una lectura, se realiza otra lectura.
- Los tiempos y alertas de retraso avanzan incluso sin cambios en los pedidos. Las adiciones a un pedido pendiente generan un nuevo aviso por sus productos.
- Se bloquean pulsaciones simultáneas en acciones de cocina. El servidor compara los productos vistos con los actuales antes de avanzar el estado; si se añadió una ronda, pide revisarla primero.
- Avisar al mesero deja de ocultar errores de red. Los botones de impresión individual y por lotes ahora crean trabajos reales.
- Se conservan modificadores largos al consultar pedidos y se continúa el texto de las observaciones en la comanda; antes se truncaba en SQL y en el ticket.

## Flujo automático

1. El mesero envía el pedido.
2. El backend guarda el pedido y una copia del texto de su comanda en `print_jobs`, dentro de la misma transacción.
3. El agente del equipo conectado a la impresora consulta la cola cada dos segundos. Socket.IO solo lo despierta antes; perder el evento no pierde la comanda.
4. Reclama un trabajo pendiente y guarda su ticket localmente. Lo envía al controlador de impresión o lo simula, según el modo configurado.
5. Guarda el resultado local antes de confirmarlo al servidor. Si falla la confirmación, la reintenta sin volver a imprimir. Las comandas se reclaman en orden de creación.

El navegador de cocina no tiene que abrir un diálogo de impresión y no necesita estar abierto para que el agente imprima.

Estados visibles:

| Estado | Significado |
| --- | --- |
| PENDIENTE | Guardado en la cola, esperando al agente. |
| PROCESANDO | Reclamado por un agente. |
| ENVIADO | El comando/controlador aceptó el trabajo. No demuestra salida física del papel. |
| SIMULADO | Se generó el archivo, sin imprimir papel. |
| ERROR | Falló el procesamiento. Revisar equipo/controlador antes de reimprimir. |
| REVISAR | El agente se interrumpió o no confirmó a tiempo; el resultado es incierto. No se reimprime a ciegas. |

Los botones manuales de cocina generan una **copia completa del pedido actual**. La impresión automática de una ronda contiene solo sus productos nuevos. Si hay un error en una ronda, revisar el archivo original en `print-jobs` antes de imprimir una cuenta completa, para evitar repetir productos anteriores.

## Probar ahora sin impresora

Desde `restaurante-backend`:

```powershell
pnpm print:demo
```

Esta prueba crea una cola HTTP aislada, inicia el agente real, genera una comanda y espera la confirmación `SIMULADO`. No conecta con la base del restaurante ni consume pedidos reales. Guarda el ejemplo en `print-jobs/demo/<ejecucion>/` y termina ambos procesos.

Para consultar configuración e impresoras instaladas, sin imprimir:

```powershell
pnpm print:check
```

## Conectar la futura impresora de tickets

El camino incluido usa un controlador de Windows o CUPS. En Windows se envía texto a una impresora por su nombre mediante [Out-Printer de Microsoft](https://learn.microsoft.com/nb-no/powershell/module/microsoft.powershell.utility/out-printer?view=powershell-5.1). USB, Bluetooth o red deben aparecer como una impresora instalada en ese equipo. No se implementó conexión Bluetooth directa desde el teléfono ni un transporte ESC/POS sin controlador.

1. Instalar el controlador del fabricante y configurar el papel del ticket. La plantilla actual tiene aproximadamente 42 columnas; comprobar tamaño de letra, márgenes y ajuste al ancho real de 58 u 80 mm. Corte y apertura de cajón dependen del controlador; no se añadieron comandos ESC/POS.
2. Configurar el mismo secreto nuevo `PRINT_AGENT_TOKEN` en el backend y en el agente. Es independiente del JWT de usuarios; no usar el valor de ejemplo.
3. En el equipo de la impresora, completar `.env`:

```dotenv
PRINT_SERVER_URL=http://localhost:3000
PRINT_AGENT_TOKEN=el-mismo-secreto-del-backend
PRINT_MODULE=cocina
PRINT_MODE=system
PRINT_PRINTER=Nombre exacto de la impresora de cocina
```

Si el backend está alojado en otro equipo o en Render, usar su URL, preferentemente HTTPS. El agente se ejecuta donde está instalada la impresora, no en Render. `Configuración > impresora_cocina` tiene prioridad sobre `PRINT_PRINTER`; debe coincidir con el nombre en el equipo del agente.

4. Reiniciar el backend para preparar `print_jobs`. La cuenta de MySQL necesita permiso para crear esa tabla.
5. Ejecutar `pnpm print-agent` en el equipo de la impresora y mantenerlo abierto. Para operación diaria, configurar ese comando en un servicio o tarea de inicio de Windows; no se creó una tarea de inicio automáticamente.
6. Enviar una comanda de prueba y verificar papel, acentos, observaciones, ancho y corte. Después revisar el estado en cocina.

Ejecutar un solo agente por módulo y directorio de salida. Para otra estación, usar otro `PRINT_OUTPUT_DIR`. Los archivos locales contienen los textos de los tickets y sus confirmaciones: conservarlos para investigar fallos, no borrar una confirmación pendiente para forzar reimpresión.

Las precuentas de caja mantienen su transporte anterior por Socket.IO cuando se ejecuta el agente con `PRINT_MODULE=caja`; esta cola durable nueva cubre las comandas de cocina. No se ha afirmado recuperación offline para los eventos antiguos de caja.

## Validación realizada y límites

- 31 pruebas del backend y 8 del frontend: importes, cocina, transiciones, cola, autorización del agente y recuperación de confirmaciones.
- Prueba con un ejecutable de impresora ficticia: recibió exactamente el archivo, con nombres y rutas que contienen espacios.
- `pnpm print:demo`: agente real por HTTP, recogida automática y confirmación de una comanda simulada.
- Exportación web de Expo completada.
- No se usó una impresora física ni se ejecutaron estas pruebas contra la base MySQL real. El flujo SQL se verificó con sustituciones de base de datos; falta la prueba de integración con MySQL y el controlador del equipo definitivo.
- En este PC solo se detectaron impresoras virtuales, y aún no están configurados `PRINT_AGENT_TOKEN` ni la impresora física. Por tanto, la impresión real queda preparada en código, pero no activada en el restaurante.
- Sigue pendiente separar las rondas de cocina de la cuenta completa: la pantalla aún agrupa los productos por pedido y sus tiempos son del pedido. El nuevo control evita avanzar una lista obsoleta, pero no introduce estados independientes por ronda.

## Rondas y cuenta completa (implementado)

Cada envío crea una ronda identificada en cocina y en la comanda. Reimprimir desde cocina incluye solo esa ronda. La impresión automática sigue guardándose en la misma transacción que el pedido; pagar, entregar o mover una mesa no imprime nuevamente sus productos.

La comanda guardada conserva la mesa y el texto del momento del envío. Al mover una mesa, cocina actualiza el destino en pantalla; una comanda que ya está en papel no cambia. Una reimpresión manual usa la mesa actual.
