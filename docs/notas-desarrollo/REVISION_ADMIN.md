# Correcciones del administrador — 13 de septiembre de 2026

## Formularios y catálogo

- React Native Web no implementa `Alert.alert`: los errores de los formularios se perdían. Administración y menú ahora muestran avisos web y conservan los diálogos nativos en Android/iOS.
- Si falla la lectura de una receta, no se abre el producto con una receta vacía. Una respuesta tardía tampoco reemplaza otro producto seleccionado.
- Producto y receta se guardan en una sola transacción. Los clientes anteriores pueden omitir `receta` para conservarla; una lista vacía explícita la elimina.
- Una receta inválida, repetida o con insumos inexistentes revierte el guardado completo. Las cantidades deben ser positivas, con hasta tres decimales.
- Crear un producto respeta su disponibilidad inicial. Ahora se pueden vaciar descripción e imagen de forma explícita.
- Editar datos generales sin cambiar la cantidad no vuelve a enviar el stock leído al abrir el formulario; evita sobrescribir consumos ocurridos entretanto.
- Se bloquean envíos simultáneos desde Guardar en productos, mesas, ubicaciones, equipo, inventario, movimientos y extras.

## Inventario y personal

- Precios, costos y existencias inválidos o negativos se rechazan. El formulario acepta coma decimal; las unidades de producto deben ser enteras. Las entradas/salidas de inventario deben ser mayores que cero; un ajuste a cero es válido si las reservas lo permiten.
- Vaciar proveedor o categoría de inventario ya borra el valor anterior.
- La edición de usuario valida correo y PIN antes de modificar credenciales y guarda los campos juntos. Un correo duplicado no deja una contraseña cambiada a medias.
- Crear usuarios respeta el estado inactivo. Editar un usuario inexistente devuelve 404. El administrador no puede desactivarse o quitarse su propio rol por accidente.
- Los fallos de lectura conservan las listas anteriores y muestran un aviso con Reintentar. La reconexión actualiza los datos. Las operaciones de pedidos actualizan productos e inventario.
- El menú y las alertas de existencias usan stock disponible tras reservas. El indicador de mesas ocupadas cuenta las mesas actualmente ocupadas, en vez de mesas visitadas durante el día.

## Verificación

- Backend: `pnpm test`, 85 pruebas aprobadas.
- Frontend: `pnpm test`, 28 pruebas aprobadas.
- `pnpm build:web`: compilación correcta.
- Backend: `node --experimental-test-module-mocks scripts/verifyAdmin.mjs`, pruebas de rollback de producto/receta, validaciones, campos vacíos, usuario inactivo y edición de credenciales sin cambios parciales.
- Backend: `node --experimental-test-module-mocks scripts/verifyStockReservations.mjs`, regresión de concurrencia, reservas, cancelaciones y recetas.
- Ambos scripts escriben exclusivamente en copias temporales de la base local, que eliminan al terminar.
- Navegador: panel autenticado y catálogo cargados; el formulario vacío de producto muestra el aviso de nombre obligatorio. No se enviaron productos, usuarios ni movimientos de prueba a la base real.

Esta revisión cubre los errores descritos; no certifica todos los flujos del administrador ni modifica las reglas de cálculo de reportes financieros o revocación de sesiones.
