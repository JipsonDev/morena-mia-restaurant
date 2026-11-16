# Diseño y navegación de servicio — 12 septiembre 2026

## Cambios

- Paleta compartida índigo para acciones; verde disponible, ámbar ocupada, rosa por cobrar y violeta reservada. Mesero y cocina utilizan los tokens de administración; caja deriva su paleta de los mismos valores. Acceso adopta la misma identidad.
- Mesas con modelos geométricos Three.js renderizados mediante un único contexto WebGL2. Se dibujan una vez por capacidad/estado/tema y se reutilizan como imágenes; no hay bucle de animación ni contexto por tarjeta. Caché limitada a 64 modelos. SVG de respaldo para dispositivos sin WebGL2. Se muestran hasta 16 sillas gráficas y el excedente se indica junto a la capacidad total. No es un plano físico editable ni un visor con rotación.
- Barra adaptable con las cinco secciones visibles, botones identificados para lectores de pantalla, menú lateral compacto, usuario, estado de conexión y destino actual. El mismo controlador atiende ambas navegaciones.
- Catálogo con esquinas de 6 px, categorías compactas, botones de 40 px, imágenes con respaldo si fallan y colores de foco para teclado.

## Correcciones funcionales

- Menú y Promos se pueden consultar sin elegir mesa y después de abrir un Express enviado. El catálogo ya no muestra el aviso que impedía entrar desde la barra superior.
- Antes de añadir se selecciona explícitamente mesa, pedido para llevar o delivery. Un Express abierto permite añadir una nueva ronda a la misma cuenta. Las cuentas cerradas o con abonos quedan protegidas.
- No se elige automáticamente la primera mesa al cargar la aplicación. Volver a preparar un Express conserva el borrador y los datos del cliente en el almacenamiento local del usuario y equipo.
- Las cuentas con abonos bloquean la preparación de adiciones incompatibles con el servidor. El control de inventario suma las líneas del mismo producto, incluidas las variantes de promoción.
- Una carga inicial fallida no se presenta silenciosamente como catálogo vacío: hay reintento explícito y recuperación al reconectar.
- Los 401 de la API cierran el estado React de la sesión. Respuestas antiguas no borran una sesión posterior; las credenciales se adjuntan únicamente a la API del restaurante.
- Validación de números/capacidades enteros positivos. Editar una mesa bloquea su fila antes de comprobar pedidos activos. No se permite ocultarla, liberarla o reservarla con pedidos activos. Reactivar verifica números duplicados; editar una mesa inexistente devuelve 404.

## Validación y alcance

- 27 pruebas de frontend y 80 de backend aprobadas; exportación web aprobada.
- Revisión real del módulo de mesero, WebGL2 (12 modelos), tema claro/oscuro, ancho móvil 390 px, selector de destino, menú lateral y recorrido Delivery existente → Menú. Sin cobros ni nuevas comandas durante la revisión.
- Caja, cocina y administración compilan con la paleta compartida; no se hizo una revisión visual completa de todos sus diálogos con usuarios de cada rol.
- La impresión física sigue pendiente de disponer de impresora. Los borradores se recuperan tras recargar en el mismo navegador/equipo; no se sincronizan entre dispositivos.

## Siguientes mejoras propuestas

1. Tiempos de atención y alertas configurables por mesa, ronda y etapa de cocina.
2. Fotografías y descripciones homogéneas del catálogo, disponibilidad por turno y productos favoritos.
3. Plano del salón configurable desde administración: posición, forma, capacidad y zonas. Conservar una vista compacta para operar en teléfonos.


## Borradores, reintentos y rondas Express

- Borradores separados por usuario y mesa/Express. Incluyen productos, modificadores, notas y datos del cliente para un nuevo Express. El botón Borradores permite recuperar el destino después de recargar.
- Antes de enviar se consultan productos, disponibilidad, existencia de la mesa, estado de la cuenta Express y configuración de servicio. Los cambios de precio o servicio requieren revisar el total y volver a pulsar enviar. El servidor también rechaza cambios de precio/servicio ocurridos después de esa consulta.
- Cada envío guarda primero su identificador y contenido localmente. Si no se confirma la respuesta, se conserva el envío original y se bloquea la edición de esas líneas. Reintentar utiliza el mismo contenido e identificador. No se descarta automáticamente un envío de resultado desconocido.
- MySQL guarda el identificador por usuario en pedido_solicitudes dentro de la misma transacción que pedido, ronda y trabajo de impresión. Una petición repetida devuelve el resultado original; reutilizar la clave para otro contenido devuelve conflicto. Los clientes antiguos sin clave siguen siendo compatibles, pero no reciben esta protección.
- Añadir nueva ronda en un Express abierto utiliza pedido_id. Conserva cuenta, cliente, entrega, historial y preparación anterior; imprime únicamente los nuevos productos. Bloquea cuentas pagadas, canceladas, con abonos o que cambiaron de destino. Al añadir se invalida una división anterior y se quita la solicitud de cobro pendiente para revisar el nuevo total.
- El inventario se comprueba antes del envío, pero no se reserva por mantener un borrador; se sigue descontando al iniciar preparación con los controles transaccionales existentes.
- Prueba SQL reproducible: desde restaurante-backend ejecutar `node --experimental-test-module-mocks scripts/verifyOrderRequests.mjs`. Clona la base local en una base temporal, verifica peticiones simultáneas, rollback, reintento después del cierre, nueva ronda y una impresión por ronda; elimina únicamente su propia copia al terminar.
- Revisión de navegador: crear un borrador local, recargar, recuperarlo en Mesa 5 y retirar el artículo de prueba; comprobar Delivery existente → Añadir nueva ronda → catálogo con el mismo número de cuenta. Sin enviar comandas ni realizar cobros reales.


## Ajuste de cabecera y modo oscuro

- Fondos grafito, superficies neutras escalonadas y un único acento azul en modo oscuro. Estados de éxito, aviso y error conservan colores semánticos con fondos apagados.
- Cabecera del mesero: logo centrado con ancho limitado en teléfonos; usuario y controles adicionales desde 600 px; navegación en una fila a partir de 1280 px y fila independiente en tablets para evitar compresión.
- Botones de cabecera con área táctil de 44 px. En móvil, el selector de tema está en el menú lateral. Los cinco destinos permanecen visibles.
- Menú lateral con logo, identificación del mesero, estado de conexión, destino y navegación; estados de pulsación y selección coherentes.
- Logo compartido admite ancho explícito, mantiene proporción y no supera el contenedor.
- Exportación web verificada. Revisión visual de la cabecera y menú en móvil y tablet, incluyendo 320, 390 y 768 px, y navegación de escritorio a 1366 px.


## Paleta cálida de cafetería

La revisión posterior sustituye el grafito y azul del modo oscuro por espresso, crema y caramelo, incluyendo logo, cabecera, tarjetas y acciones. Las mesas usan salvia para disponibles, caramelo para ocupadas, terracota para cobro y malva para reservas; los modelos 3D siguen estos tonos apagados. Se conserva la distribución adaptable de la cabecera. Compilación web y revisión visual del salón verificadas.


## Paleta final solicitada

Se recuperan los fondos azul marino oscuro, combinados con texto crema y acentos dorados suaves. Los estados usan verde salvia, champán, rosa apagado y lavanda. Esta revisión sustituye los fondos espresso de la propuesta anterior. Cabecera adaptable conservada; revisión visual y exportación web aprobadas.


## Guía de marca aportada por el usuario

La referencia «ChatGPT Image 12 sept 2026, 23_35_15.png» sustituye las propuestas anteriores. Modo oscuro: fondo #0B1220, barras #111B2D, tarjetas #1A2740, superficie elevada #243754, primario #F6B325, hover primario #D99A1F, texto #F7F4E9 y secundario #A8B3C7. Disponible #2ECC71, ocupada #F59E0B, por cobrar #F472B6, error #EF4444 y promoción #8B5CF6. Se aplican a los tokens compartidos, cabecera, logo, estados de mesas y tarjetas de productos. Compilación web y revisión visual verificadas.


## Acceso compacto a borradores

Se elimina la franja persistente de guardado correcto. Los borradores se abren desde el icono con contador de la cabecera o desde el menú lateral. Solo errores de almacenamiento o envíos sin confirmar conservan un aviso visible. El aviso de recuperación se muestra dentro del diálogo. La exportación web pasó; la revisión visual de esta modificación quedó limitada por sesión vencida en el navegador.
