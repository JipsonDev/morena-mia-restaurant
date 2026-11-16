# Módulo de mesero — revisión de diseño y lógica

## Diseño y estructura

- Skills aplicadas: UI/UX Pro Max (GitHub), frontend design, Vercel React Native, componentes accesibles, depuración sistemática y verificación.
- Referencias consultadas: https://github.com/Adminiumjs/point-of-sale (zonas, estados y continuidad del ticket) y https://github.com/KirkGamo/restaurant_pos (plano de mesas y estados operativos). Implementación propia con los componentes del proyecto.
- `WaiterFloor.js` separa la pantalla del salón del módulo principal: búsqueda, estados, zonas, orden numérico y vista compacta.
- `WaiterTableCard.js` sustituye el modelo WebGL por un plano SVG sin perspectiva ni animación. Color y texto comunican el estado; número, plazas y saldo se muestran por separado. El esquema limita a 16 las sillas dibujadas; la etiqueta mantiene la capacidad configurada completa.
- Cabecera y menú lateral más compactos, logo original, contador del borrador en Pedido y controles de al menos 44 px en los flujos revisados.
- Paleta clara de blanco/gris, azul marino y ámbar aplicada únicamente al mesero; modo oscuro azul marino conservado.
- Menú con tarjetas más pequeñas, sin círculos decorativos, categorías y buscador compactos. La selección de destino ya no ocupa dos líneas explicativas si hay una mesa elegida.
- Pedido con resumen más corto, etapas de cocina/pago/entrega legibles y cantidades separadas del nombre en móvil. Eliminar tiene botón visible además del gesto.

## Correcciones

1. Añadir el mismo producto no incrementa simultáneamente todas las variantes con notas y extras. Solo se incrementa la línea original sin personalizar; si no existe, se crea una nueva.
2. Aumentar cantidades comprueba el stock actual sumando todas las variantes del producto, incluidas las promociones. Un producto retirado del catálogo no puede aumentar; sí puede reducirse o eliminarse.
3. Los avisos de stock y confirmaciones de añadido salen del actualizador de estado. Un rechazo sin cambio de carrito muestra el aviso inmediatamente.
4. Se reutiliza la cola de actualización probada en cocina: un evento durante una consulta invalida la respuesta anterior y provoca otra lectura. También se invalidan lecturas al recibir una mesa completa.
5. Actualizar datos ya cargados no sustituye toda la pantalla por el cargador inicial. Las consultas tienen un límite de 15 segundos.
6. Cuenta solicitada se reconoce también desde la cuenta local mientras se actualiza la mesa. Capacidad se expresa como plazas y no inventa comensales presentes.
7. Catálogo de una columna en pantallas pequeñas no utiliza `columnWrapperStyle`, incompatible con esa configuración de FlatList.

## Verificación

- `npm test`: 61 pruebas correctas, incluidas 6 regresiones de carrito y las pruebas de concurrencia de la cola.
- `npm run build:web`: exportación web correcta.
- Playwright: escritorio 1440 px, tablet 768 px y móviles 390/320 px; filtros, búsqueda, limpieza, plano/vista compacta, menú, variantes, extras, cantidades, stock, borradores, teclado y cierre del menú lateral.
- Caso reproducido en UI: café de $3.50 con leche de $1.00 y nota, más cuatro cafés normales; total $18.50 y rechazo del sexto café con stock de cinco.
- Pruebas de interfaz con respuestas API y socket simuladas en una sesión de navegador aislada. No constituyen una validación de ventas reales ni de dispositivos Android/iOS físicos. Las escrituras de negocio se bloquearon en esa sesión; no se crearon pedidos ni ventas de prueba en MySQL.
- Capturas y scripts: `output/playwright/waiter-*`, `review-waiter.js`, `waiter-variants.js`, `finish-waiter.js`, `capture-waiter-final.js`.

## Corrección posterior: columnas que se apilaban

La captura del usuario permitió detectar un caso que las pruebas iniciales no cubrieron: la ventana podía ser más ancha que el contenedor del módulo. Se reprodujo con una ventana de 446 px y un contenedor de 426 px: las dos tarjetas de 202 px se apilaban dejando media fila vacía. Además, el redondeo de `onLayout` con zoom al 125 % podía exceder el espacio disponible por décimas de píxel.

- El módulo y el salón miden ahora su propio contenedor mediante `onLayout`.
- Las tarjetas usan medidas redondeadas hacia abajo y un margen de un píxel para absorber el redondeo de la plataforma. El catálogo utiliza el mismo margen.
- Por debajo de 300 px efectivos, borradores sigue disponible en el menú lateral y no ocupa espacio en la cabecera. Caso reproducido a 320 px y zoom al 150 %: el logo pasó de 33.5 px a 134 px visibles.
- Regresión de navegador: `reproduce-responsive.js` falla con el diseño anterior; `verify-responsive-fix.js` comprueba las dos tarjetas en la misma fila con el contenedor reducido.
- Matriz de 30 casos: anchos de 320, 360, 390, 412, 446, 600, 768, 1024, 1280 y 1440 px, combinados con escalas CSS de 100 %, 125 % y 150 %, y 17 px menos de contenedor. Se comprobaron también barra de desplazamiento, borrador activo, botones del catálogo y acceso a borradores desde el menú lateral. Esto prueba escala de contenido en navegador, no una medición en dispositivos físicos.
- `npm test`: 61 correctas; `npm run build:web`: exportación correcta; `git diff --check`: sin errores en los archivos modificados.
