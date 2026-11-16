# Revisión limitada de seguridad del acceso

Fecha: 17 de octubre de 2026. Alcance: inicio de sesión por contraseña, PIN, configuración inicial de PIN y registro de usuarios. No equivale a una auditoría de todo el sistema.

## Cambios aplicados

1. **Crítico: registro sin autorización.** `restaurante-backend/routes/auth.js`, ruta `POST /register`, estaba montada públicamente desde `server.js` y ejecutaba la creación sin verificar token ni rol. Un visitante podía solicitar una cuenta administrativa. Ahora exige `verifyToken` y `verifyRole('admin')`; las pruebas HTTP verifican 401 para anónimos, 403 para meseros y éxito para administradores.
2. **Alto: intentos de contraseña y configuración de PIN sin límite.** Ambas rutas comparten ahora un presupuesto de 10 fallos por dirección IP cada 2 minutos, mediante `express-rate-limit`. El acceso con PIN permite 5 fallos. Devuelve 429 y `Retry-After`; las pruebas verifican que el bloqueo evita consultar la base de datos. No se confía en encabezados proxy suministrados por el visitante.
3. **Medio: tipos y PIN ambiguos.** Los campos de acceso se validan como cadenas con tamaños acotados. El PIN debe ser exactamente una cadena de cuatro dígitos; ya no se admite un prefijo válido seguido de más caracteres en estas rutas. Las entradas malformadas devuelven 400.
4. **Medio: diagnóstico interno en respuestas públicas.** Los errores de conexión ya no muestran nombres de variables de base de datos ni rutas `.env`. Los diagnósticos permanecen en el servidor; la respuesta pública es genérica. Las respuestas de autenticación incluyen `Cache-Control: no-store`.
5. **Experiencia ante errores de red.** Las tres solicitudes de autenticación tienen un tiempo límite de 15 segundos. El mensaje de interfaz indica cómo reintentar sin exponer direcciones internas. Se conservan el bloqueo de doble envío y los campos de secreto enmascarados.

## Límites y siguiente trabajo

- El limitador usa memoria de un proceso. Reiniciar el servidor reinicia contadores; varios procesos o servidores requieren un almacén compartido. La limitación por IP puede afectar a equipos que comparten una conexión. Un despliegue detrás de proxy requiere configurar explícitamente su topología de confianza.
- Se conserva la arquitectura actual de sesiones JWT y su persistencia. Para producción se recomienda diseñar una migración web a cookies HttpOnly y protección CSRF, y almacenamiento seguro nativo; requiere revisar también sockets y resto de clientes.
- Los JWT existentes pueden conservar permisos hasta caducar. Comprobar el estado actual del trabajador y revocar sesiones es trabajo adicional de autorización para todo el sistema.
- El PIN de cuatro dígitos es una comodidad para equipos operativos controlados; no debe considerarse autenticación fuerte para administración expuesta a Internet. Conviene separar acceso administrativo y adoptar un segundo factor antes de una exposición pública.
- La autenticación con usuarios reales no se pudo completar porque MySQL local aborta al inicializar `mysql.plugin`. Se verificaron las rutas con HTTP real y persistencia sustituida en pruebas, sin crear usuarios reales.

## Evidencia

`restaurante-backend/tests/authSecurity.test.js`: autorización de registro, validación de tipos/PIN, errores públicos sin diagnóstico, límite compartido, acceso válido, configuración inicial y bloqueo de PIN. Las pruebas nunca se conectan a la base de datos real ni escriben usuarios reales.
