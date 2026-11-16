# Pedidos del administrador

Nueva pantalla independiente OrdersScreen. Lista compacta en escritorio y tarjetas en móvil, paleta compartida clara/oscura, resumen de operación, búsqueda por pedido/mesa/cliente/producto, filtros de canal y preparación, orden por antigüedad y carga incremental.

El detalle muestra productos, modificadores, notas, rondas, total, abonos y saldo. Preparación y pago se muestran separados: un pedido pagado con entrega pendiente permanece en curso. Los pedidos anteriores aún abiertos no se etiquetan como pedidos de hoy. La vista general incluye los datos del endpoint operativo: cuentas abiertas y pedidos creados hoy, no todo el historial.

Se retiraron los botones antiguos que llamaban a la actualización de la cuenta completa para avanzar preparación (el servidor los rechazaba). Ahora cada acción apunta a la ronda correspondiente, con confirmación y protección contra doble envío. Express impide entregar sin pago completo y Caja sigue siendo el lugar para cobrar. La cancelación envía los IDs de productos revisados para detectar nuevas líneas concurrentes; los abonos impiden ofrecer cancelar directamente.

Los filtros incluyen rondas mixtas: una cuenta con una ronda preparando y otra lista aparece en ambos filtros. Cancelados cambia a vista general si se estaba en curso.

## Corrección de modales de Mesas

MenuModal desmonta las ventanas cerradas y no anima contenido vaciado. Se elimina el parpadeo de Confirmar acción y el espacio vacío central. TablesScreen mantiene un solo modal y conserva el formulario al volver desde descartar/desactivar/eliminar. El estado de edición se compara con la captura al abrir, no con un número sugerido que puede cambiar por eventos externos. Los errores de acciones se muestran dentro de la confirmación y la X se bloquea mientras se guarda.

## Validación

39 pruebas frontend aprobadas y compilación web correcta. Revisión visual en modo claro/oscuro, escritorio/móvil, búsqueda/filtro Delivery y detalle por rondas. Confirmar preparación y volver no altera pedidos; cierre del detalle comprobado con cero diálogos residuales. Las pruebas de navegación no guardaron cambios sobre pedidos reales.
