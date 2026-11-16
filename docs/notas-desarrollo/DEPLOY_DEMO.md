# Demo deploy: Vercel + Render + Aiven MySQL

Esta guia deja la demo separada asi:

- Frontend Expo Web: Vercel
- Backend Node/Express + Socket.IO: Render Web Service
- Base de datos: Aiven Free MySQL

## 1. Crear MySQL gratis en Aiven

1. Crea un servicio Aiven for MySQL Free.
2. Copia estos datos del panel de Aiven:
   - Host
   - Port
   - User
   - Password
   - Database
3. Importa tu base local a Aiven. Ejemplo:

```bash
mysqldump -u root morena_mia > morena_mia_demo.sql
mysql -h AIVEN_HOST -P AIVEN_PORT -u avnadmin -p --ssl-mode=REQUIRED defaultdb < morena_mia_demo.sql
```

## 2. Backend en Render

Opcion recomendada: sube este repo a GitHub y usa el `render.yaml` de la raiz.

Variables necesarias en Render:

```env
NODE_ENV=production
JWT_SECRET=pon-un-secreto-largo
CORS_ORIGIN=https://TU-FRONTEND.vercel.app
DB_HOST=tu-host-aiven.aivencloud.com
DB_PORT=12345
DB_USER=avnadmin
DB_PASSWORD=tu-password
DB_NAME=defaultdb
DB_SSL=true
DB_SSL_REJECT_UNAUTHORIZED=false
```

Health check:

```text
/health
```

Cuando Render termine, guarda la URL publica, por ejemplo:

```text
https://your-api.onrender.com
```

## 3. Frontend en Vercel

En Vercel, configura el proyecto apuntando a la carpeta `mi-restaurante`.

Build command:

```bash
npx expo export --platform web
```

Output directory:

```text
dist
```

Variable de entorno:

```env
EXPO_PUBLIC_API_URL=https://your-api.onrender.com
```

Despues del deploy, vuelve a Render y actualiza `CORS_ORIGIN` con la URL final de Vercel.

## 4. Notas para demo gratis

- Render Free puede dormirse si no recibe trafico; el primer acceso puede tardar.
- Aiven Free MySQL es suficiente para demo pequena, pero limita recursos.
- Las imagenes subidas al backend quedan en `uploads`; en Render Free no conviene depender de archivos persistentes para produccion.
