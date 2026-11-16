# Menú y Express del mesero

## Diseño y estructura

Se aplicaron las skills UI/UX Pro Max, frontend design, Vercel React Native, componentes accesibles, depuración sistemática, networking de Expo y verificación. Se mantuvieron React Native, FlatList, Lucide, la tipografía Manrope y la paleta azul marino/ámbar existentes. No se añadieron dependencias.

- `WaiterCatalog.js`: vista de lista por defecto en móvil y tarjetas en pantallas amplias; selector manual de ambas vistas, buscador, categorías, disponibilidad, orden y destino. El acceso al pedido queda fijo debajo del catálogo con cantidad e importe.
- `WaiterProductCard.js`: precio, existencias y acciones consistentes en lista/tarjeta. Las imágenes y sus sustitutos ocupan el mismo alto en la cuadrícula. Con poco ancho, precio y cantidades pasan debajo del nombre. Los controles conservan al menos 44 px CSS.
- `WaiterExpress.js`: creación compacta de pedidos para llevar/delivery; filtros de cobro, preparación y entrega; búsqueda por cliente, número, teléfono y dirección. Filas con columnas en escritorio y estados debajo del cliente en móvil.
- `WaiterUI.js`: botones, buscador con foco visible y limpieza, filtros horizontales sin crecimiento vertical y texto compartidos. Los botones de filtro exponen `aria-pressed` en web.
- `ExpressHistory.js`: consulta de fecha explícita, navegación por día, detalle desplegable y estados de carga, error, respuesta inválida, vacío y contenido.
- El modo claro tiene colores de éxito y advertencia más oscuros para mejorar la lectura sobre blanco. Cambios limitados al módulo de mesero.

## Correcciones de comportamiento

1. El buscador del catálogo incluye descripción y categoría, normalizando acentos.
2. Una mención a promociones en la descripción ya no oculta un producto ordinario del Menú. La clasificación sigue el nombre o la categoría porque el modelo actual no tiene un indicador explícito de promoción.
3. Se mantienen vista, disponibilidad, orden y filtros operativos al cambiar de sección durante la sesión. Si una categoría desaparece, se vuelve a Todo.
4. Los productos agotados y el límite de stock del borrador deshabilitan añadir; disminuir sigue disponible para liberar cantidad. Un envío pendiente bloquea ambas acciones y el cambio de destino, con un mensaje específico en lugar de ofrecer crear otro destino.
5. Express distingue total y saldo restante después de un abono. Una entrega antigua sin cobro se señala para revisión en caja.
6. Las rondas listas de un pedido mixto aparecen en Por entregar si está pagado, aunque otras sigan preparándose. En cocina incluye preparación pagada y no pagada. Los filtros de trabajo pueden solaparse: un pedido puede tener cobro y preparación pendientes.
7. Escribir una fecha no dispara consultas por cada carácter. Se rechazan fechas incompletas y días inexistentes antes de consultar. Las solicitudes anteriores se cancelan y sus respuestas no se aplican después de abandonar la vista.
8. Las confirmaciones transitorias de añadido no se superponen a Express ni al acceso fijo al pedido del catálogo.

## Verificación

- `npm test`: 69 pruebas correctas, incluidas 8 regresiones nuevas del catálogo, fechas y estados Express. Los defectos de búsqueda, clasificación, fechas y rondas mixtas se reprodujeron antes de aplicar las correcciones.
- `npm run build:web`: exportación correcta.
- `git diff --check`: sin errores en los archivos modificados.
- Navegador: Menú y Express en 320, 390, 446, 768, 1024 y 1440 px, escalas CSS de 100 %, 125 % y 150 %, y contenedor 17 px más estrecho. Se verificaron además lista/tarjetas con cantidades activas y acceso fijo al pedido.
- Flujos: cambio de destino, navegación con filtros conservados, búsqueda por descripción, agotados, orden de precios, vacíos y limpieza, límite de stock y decremento, fecha inválida sin solicitud, error y respuesta malformada del historial con reintento, rondas mixtas y abonos.
- Envío no confirmado simulado con HTTP 503: el borrador se conservó y Menú bloqueó edición/destino. La solicitud se interceptó íntegramente en el navegador; no llegó al backend.
- Se usó una sesión aislada con API/socket simulados, sin crear pedidos o ventas en MySQL. No se validó impresión física ni ejecución en Android/iOS físicos.

Scripts y capturas están en `output/playwright/`: `menu-express-fixture.js`, `check-menu-express.js`, `menu-express-responsive.js`, `menu-pending-test.js`, `menu-express-states.js`, `menu-express-final.js`, `menu-complete-*`, `express-complete-*`.
