# Rediseño de caja

La caja usa una pantalla de facturación con paleta azul, gris y blanco, navegación lateral en escritorio y navegación horizontal en móvil. El tema oscuro conserva la misma jerarquía.

- **Facturación:** cliente, documento, detalle de productos, resumen, métodos de pago, efectivo recibido, importe exacto y cambio. Las cuentas confirmadas conservan sus cantidades y totales registrados.
- **Nueva venta:** catálogo con búsqueda y categorías, cantidades editables y observaciones. Confirmar envía el pedido a cocina; después permite cobrar y emitir el comprobante.
- **Cuentas pendientes:** búsqueda y filtros para seleccionar la cuenta que se va a facturar.
- **Comprobantes:** búsqueda de pagos recientes y acceso a los flujos existentes de anulación y devolución.
- **Mi caja:** totales del turno, efectivo esperado, movimientos y acceso a cierres.

La presentación está en `mi-restaurante/src/modules/CashierWorkspace.js`; las operaciones y validaciones siguen en `CashierApp.js`. Se limitó el estilo de altura de la raíz web para que las ventanas modales no agreguen pantallas vacías ni desplacen la cabecera.

Validación: 14 pruebas del frontend aprobadas y exportación web completada. Revisión visual a 390, 910 y 1440 píxeles, incluyendo catálogo, cantidades del borrador, búsqueda, pago exacto e insuficiente y temas claro/oscuro. No se registraron cobros ni movimientos de caja durante la revisión visual.
