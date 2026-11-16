# Pagos, historial y reimpresión

## Uso

- En **Facturación**, selecciona una cuenta. **Cobrar saldo** liquida lo pendiente; **Por partes** permite ingresar un abono o elegir una parte pendiente enviada por el mesero. El resto sigue por cobrar y la mesa no se libera por un abono.
- **Mixto** permite repartir el importe entre varios métodos. El desglose debe sumar exactamente lo que se cobra; el efectivo recibido y el cambio corresponden únicamente a su componente en efectivo.
- **Comprobantes** consulta el historial completo en páginas de 20 resultados. Incluye filtros por cliente/documento/número, fechas, método, estado y cajero. En registros antiguos sin cajero individual se usa el responsable de la caja.
- **Ver** abre los datos del comprobante y **Reimprimir** vuelve a usar esos datos sin crear otro pago. Se conserva número, fecha, cliente, productos, importes y saldo al emitir. Los comprobantes antiguos sin copia se identifican explícitamente como reconstruidos.

## Consistencia

Cada cobro nuevo conserva un identificador único. Una solicitud cuyo resultado no se conoce permanece guardada en este dispositivo: **Reintentar cobro pendiente** recupera el resultado usando el mismo identificador, incluso después de recargar. Cambiar de importe no reutiliza un cobro confirmado.

Los pagos y el desglose se guardan en una transacción. El servidor valida el saldo, las divisiones y los centavos bajo bloqueo. El arqueo incluye solo los componentes en efectivo y excluye pagos anulados/devueltos una sola vez. La asignación de base, impuesto y servicio conserva los centavos al anular y volver a cobrar.

Una anulación reabre el saldo correspondiente. Devolver un abono de una cuenta abierta resta ese abono; devolver un pago de una cuenta ya liquidada conserva la cuenta cerrada, como antes. Anulaciones y devoluciones siguen requiriendo autorización y una caja abierta compatible con el pago.

Las cuentas con abonos no permiten modificar productos, rehacer el reparto ni fusionarse con otra cuenta hasta revertir esos pagos. Una cuenta con comprobantes no puede eliminarse, para conservar el historial.

## Impresión

La caja usa el diálogo de impresión del navegador para tickets de 80 mm. **Enviado a impresión** significa que se abrió ese diálogo: no confirma que salió papel ni distingue si se canceló. Los fallos al abrirlo se registran para reintentar. La salida física queda pendiente de probar cuando se conecte una impresora de tickets.

## Verificación

Pruebas del frontend y backend, exportación web, revisión visual del abono mixto y del historial, y prueba con MySQL en una copia local desechable. Las pruebas no cobraron cuentas de la base original.

Para repetir la prueba de integración desde `restaurante-backend`:

```powershell
node --experimental-test-module-mocks scripts/verifyCashLedger.mjs
```

El script solo acepta MySQL local, copia la base configurada a `codex_cash_test_<timestamp>` y elimina exclusivamente esa copia al terminar.
