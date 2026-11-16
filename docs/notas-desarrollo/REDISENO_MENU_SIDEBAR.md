# Menú y navegación de administración

## Diseño

- Paleta de la referencia: superficies blancas, azul marino y dorado; compatibilidad con modo oscuro.
- Sidebar con grupos, iconos Lucide SVG, ilustraciones vectoriales de cafetería y un degradado suave con `expo-linear-gradient` 56.0.4, elegido por Expo para SDK 56.
- Navegación contraíble con preferencia local, etiquetas también en modo compacto, contador de pedidos abiertos y drawer en móvil.
- Catálogo adaptable al ancho real: tarjetas o lista, filtros por estado y categoría, búsqueda sin tildes, orden por nombre/precio/stock y lotes de 24 resultados.
- Formularios separados en información, existencias/receta y fotografía; errores visibles junto al guardado y confirmación de descarte dentro del sistema.

## Reglas de negocio

- Disponible, agotado y oculto son estados distintos. Publicar no incrementa existencias.
- Un producto nuevo empieza con cero unidades. Las cantidades no modificadas no sobrescriben ventas concurrentes.
- El formulario elige stock propio, vínculo directo 1:1 o receta. El servidor rechaza combinaciones ambiguas.
- Las recetas usan la unidad real del insumo: no se convierte g/kg automáticamente. Se validan cantidades positivas, insumos activos y reservas.
- Estimación de unidades elaborables y costo de ingredientes; los costos incompletos se indican y no se presentan como ganancia.
- Categorías inactivas ocultan sus productos del catálogo operativo y bloquean nuevas reservas. Se conservan pedidos e historial. No se reactiva una categoría con nombre activo duplicado.
- Eliminar productos con historial continúa bloqueado; ocultarlos conserva sus registros.

## Verificación

- Frontend: 33 pruebas; backend: 85 pruebas.
- `verifyAdmin.mjs`: guardado atómico, categorías, fuente de stock única, unidades y rechazo de ventas en categorías inactivas.
- `verifyStockReservations.mjs`: concurrencia por última unidad, recetas compartidas, preparación, cancelación y migración repetible.
- Las pruebas de integración utilizan copias temporales de MySQL y eliminan solo esas copias.
- Exportación web correcta. Revisión visual de sidebar expandido/compacto, catálogo/lista, búsqueda y formularios en escritorio y móvil.

No se añadieron descuentos automáticos ni promociones programadas: el acceso de promoción conserva el modelo existente de producto promocional.
