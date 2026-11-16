# Inicio de sesión Morena Mia

## Skills de GitHub instaladas y aplicadas

| Uso | Fuente | Instalación local |
| --- | --- | --- |
| Dirección visual | https://github.com/anthropics/skills/tree/main/skills/frontend-design | `C:/Users/Jipson/.codex/skills/github-morenamia-design` |
| Estructura React Native | https://github.com/vercel-labs/agent-skills/tree/main/skills/react-native-skills | `C:/Users/Jipson/.codex/skills/github-morenamia-react-native` |
| Seguridad React y Express | https://github.com/openai/skills/tree/main/skills/.curated/security-best-practices | `C:/Users/Jipson/.codex/skills/github-morenamia-security` |

En la revisión se aplicaron también las skills locales `ui-ux-pro-max-github`, `accessible-components` e `impeccable`, centradas en disposición, accesibilidad y adaptación. Las búsquedas UI/UX devolvieron pautas verificadas sobre autenticación accesible, ausencia de desbordamiento y dimensiones responsivas de React Native. Se consultó Expo SDK 56 según `mi-restaurante/AGENTS.md`.

## Referencias consultadas

Se leyeron los archivos fuente originales de shadcn/ui:

- https://github.com/shadcn-ui/ui/tree/main/apps/v4/registry/new-york-v4/blocks/login-03
- https://github.com/shadcn-ui/ui/tree/main/apps/v4/registry/new-york-v4/blocks/login-04

Se adoptaron como referencia la composición centrada, el ancho limitado y la agrupación de campos. La implementación se conserva en React Native/Expo; no se añadieron controles de acceso social ni acciones que el sistema no soporta.

## Dirección visual vigente

Paleta: fondo azul marino `#0B1220`, superficie `#111B2D`, campos `#17243A`, ámbar `#F6B325`, texto `#F7F4E9`, texto secundario `#A8B3C7`.

Logo real centrado arriba y un único formulario de ancho máximo 448 px incluyendo sus márgenes interiores. Mismo eje central en escritorio, tablet y móvil. Espaciado de 4/8/12/16/20/24/32 px. Al reducir la altura se conserva el flujo desde arriba con desplazamiento vertical; no se fuerza un centrado que oculte campos. Botón principal de 52 px, pestañas y botones auxiliares de al menos 44 px. El ámbar identifica la acción principal y el foco.

Manrope regular y medium se sirven desde `assets/fonts/`, con licencia OFL incluida. Fuente original: https://github.com/googlefonts/manrope. No se requieren llamadas a Google Fonts al abrir el sistema. Se conserva el logo original del restaurante. Se retiró la composición de marca de dos columnas y su ilustración decorativa para priorizar la tarea de acceso.

## Estructura

- `src/screens/LoginScreen.js`: acceso, cambio de método y creación inicial del PIN.
- `src/components/login/LoginBrand.js`: logo centrado.
- `src/components/login/LoginField.js`: etiquetas, foco y errores asociados.
- `src/components/login/loginStyles.js`: colores, tipografía y espaciado compartidos por el acceso.
- `src/utils/loginLogic.mjs`: validación.

La creación del PIN se realiza dentro del formulario. Las contraseñas y PIN no se guardan en almacenamiento ni se añaden a registros; el flujo conserva sus validaciones y bloqueo de doble envío.

## Corrección de caché de desarrollo

El navegador conservaba un service worker de la aplicación que podía devolver una versión antigua del bundle cuando Metro no respondía. La reproducción mostró el formulario antiguo desplazado 331 px respecto al centro tras recargar.

`public/sw.js` ahora excluye bundles de desarrollo del fallback y renueva su versión de caché. `src/services/pwa.js` desactiva el registro del worker propio en sesiones Expo de desarrollo. La caché PWA de producción sigue disponible. Se añadió una prueba que reproduce la interceptación incorrecta antes del arreglo.

## Evidencia de adaptación

Capturas y mediciones en Chromium mediante Playwright, no pruebas en dispositivos físicos:

| Tamaño | Desbordamiento horizontal | Desviación horizontal del formulario | Distribución vertical |
| --- | --- | --- | --- |
| 1440 × 900 | Ninguno | 0 px | Centrada |
| 768 × 1024 | Ninguno | 0 px | Centrada |
| 390 × 844 | Ninguno | 0 px | Centrada |
| 320 × 568 | Ninguno | 0 px | Desplazamiento desde arriba |
| 844 × 390 | Ninguno | 0 px | Desplazamiento desde arriba |

Capturas: `output/playwright/login-centered-*.png`. Verificación: `output/playwright/review-centered-login.js`.

Validación de campos vacíos, selección por teclado, contraseña visible/oculta, PIN y acceso al botón al desplazar en alturas cortas. El historial de seguridad se mantiene en `SEGURIDAD_LOGIN.md`.

El acceso real con la base de datos continúa pendiente: MySQL local aborta al cargar su tabla interna `mysql.plugin`. No se modificaron sus tablas internas. Las rutas de autenticación se verificaron con HTTP real y base de datos sustituida en las pruebas anteriores.

## Resultado de la revisión

55 pruebas de frontend aprobadas, incluida la regresión de caché; exportación web completada y cero errores de consola en la revisión. Se verificaron campos vacíos, PIN incompleto, cambio de método por teclado, mostrar/ocultar contraseña y apertura/cierre de instalación. La configuración inicial de PIN se comprobó en el navegador con respuesta de API simulada, sin crear usuarios reales.

La revisión independiente detectó el salto de línea de la pestaña en 320 px. Tras acortar sus etiquetas visibles a «Contraseña» y «PIN», conservando nombres accesibles completos, se volvió a capturar la misma matriz de tamaños y la revisión dio el hallazgo por resuelto. No se probó en dispositivos físicos ni se resolvió el problema externo de MySQL.
