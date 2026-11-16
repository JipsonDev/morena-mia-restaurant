# Reparación de arranque MySQL — 17 de octubre de 2026

Autorizada por el usuario para recuperar el sistema y continuar con el acceso de mesero.

## Respaldo previo

- Copia completa de `C:/xampp/mysql/data` y configuración `C:/xampp/mysql/bin/my.ini` en `D:/MorenaMia-respaldos/mysql-20261017-135026`.
- 1.139 archivos, 420.584.002 bytes. Cada archivo fue comparado contra el original mediante SHA-256.
- Manifiesto: `manifest-sha256.csv` dentro del respaldo.
- Se comprobó que la instancia adicional de LinkLocker usa otro directorio y el puerto 33413; no se detuvo ni modificó.

## Diagnóstico y reparación

El arranque fallaba al buscar `aria_log.00000002`, impedía cargar el motor Aria y finalmente `mysql.plugin`. Se comprobaron las 24 tablas Aria sin iniciar el servidor. `mysql.db` presentó tamaño de archivo inconsistente; las demás pasaron.

- Se reparó únicamente `mysql/db.MAI` con `aria_chk --recover --backup --ignore-control-file` y se comprobó de nuevo correctamente.
- El registro incompleto `aria_log.00000003` se trasladó a `quarantine/` dentro del respaldo, conservando `aria_log_control`.
- MySQL se inició normalmente con su configuración XAMPP en el puerto 3306, sin deshabilitar autenticación ni usar modos de recuperación InnoDB.
- La API se inició con `pnpm start` en el puerto 3000.

## Verificación

- Conexión de la aplicación a MySQL correcta.
- `CHECK TABLE` sobre las 31 tablas de la base configurada: sin errores.
- Lecturas de trabajadores, pedidos, pagos, productos y mesas: correctas.
- `CHECK TABLE mysql.db, mysql.plugin`: OK.
- `GET http://localhost:3000/health`: `ok: true`.

No se cambiaron contraseñas, PIN, roles ni usuarios. El inicio de sesión como mesero requiere que el usuario proporcione o introduzca su credencial existente. El control del navegador integrado está desactivado en los ajustes de Work; no se intentó eludir esa preferencia.

Referencia de recuperación Aria: https://mariadb.com/docs/server/server-usage/storage-engines/aria/aria-faq

El respaldo se conserva completo y no se programó su eliminación. Para restaurar una copia física se debe detener primero la instancia de MySQL correspondiente; no se debe copiar sobre un servidor en ejecución.

## Comprobación posterior de tablas internas

Después del arranque, `CHECK TABLE` indicó que `mysql.plugin` y otras tablas Aria internas necesitaban reparación de metadatos. Se ejecutó `REPAIR TABLE` sobre las afectadas. En la sesión de reparación se usó `aria_sort_buffer_size = 8388608` para reconstruir índices, sin modificar la configuración permanente del servidor.

Las 24 tablas Aria internas terminaron con estado OK. Se efectuó apagado limpio mediante `SHUTDOWN`, arranque normal con la configuración original y comprobación posterior de las 55 tablas: 31 del restaurante y 24 internas, sin errores. La API sigue disponible después del reinicio.

## Corrección final de estadísticas y resultado confirmado

El primer reinicio reveló un índice inválido en `mysql.table_stats`. La reparación SQL y la recuperación offline sobre ese índice fallaron por su definición dañada. Con MySQL detenido, se conservó el índice fallido en `quarantine/table_stats-after-restart.MAI`, se recuperó la definición del índice desde el respaldo SHA-256 verificado y se reconstruyó con `aria_chk --recover --backup --ignore-control-file --sort_buffer_size=8M`. No se sustituyó el archivo de datos `.MAD` del servidor activo por una copia vacía.

Tras el siguiente arranque normal se repitieron las comprobaciones: 55 tablas comprobadas, cero incidencias, dos cuentas activas de mesero encontradas y API con `ok: true`. Este es el resultado final confirmado; sustituye la comprobación intermedia anterior que aún presentaba el fallo de `table_stats`.

El inicio de sesión sigue pendiente de la credencial de mesero del usuario. No se generaron tokens para suplantar cuentas, no se probaron PIN por tanteo y no se cambiaron contraseñas ni PIN.
