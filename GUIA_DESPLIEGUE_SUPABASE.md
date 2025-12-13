# Guía de Despliegue de Supabase en Easypanel

Esta guía detalla las opciones para desplegar Supabase en tu VPS gestionado con Easypanel. Hemos analizado el repositorio oficial de Supabase para proporcionarte la configuración exacta.

## Resumen de la Arquitectura
Supabase no es una sola imagen de Docker; es un conjunto de servicios orquestados que funcionan juntos. Al desplegar "Supabase", en realidad estás desplegando:

*   **Studio**: El panel de control (Dashboard) web.
*   **Kong**: API Gateway que gestiona las peticiones y rutas.
*   **Auth (GoTrue)**: Servicio de autenticación y gestión de usuarios.
*   **Rest (PostgREST)**: Convierte tu base de datos en una API RESTful automáticamente.
*   **Realtime**: Escucha cambios en la base de datos y los transmite vía Websockets.
*   **Storage**: API compatible con S3 para guardar archivos.
*   **ImgProxy**: Redimensionamiento y optimización de imágenes.
*   **Meta**: API para gestionar la estructura de Postgres.
*   **Functions**: Runtime para Edge Functions (Deno).
*   **Analytics (Logflare)**: Sistema de logs y analíticas.
*   **Database (Postgres)**: El corazón del sistema, con extensiones preinstaladas (pgvector, postgis, etc.).
*   **Vector**: Para búsquedas vectoriales (IA).

---

## Opción 1: Plantilla de Easypanel (RECOMENDADA)
Easypanel cuenta con una **plantilla oficial de "1-Click"** para Supabase. Esta es la forma más segura y rápida, ya que Easypanel gestionará los volúmenes, redes y reinicios por ti.

1.  Entra a tu panel de Easypanel.
2.  Ve a **Projects** > **Create Service** (o "Templates").
3.  Busca **"Supabase"**.
4.  Selecciona la plantilla.
5.  Easypanel te pedirá configurar las variables clave (Passwords, Secrets).
    *   Genera contraseñas fuertes para `POSTGRES_PASSWORD`, `JWT_SECRET`, etc.
6.  Haz clic en **Create**.

**Ventajas:**
*   Integración nativa con la UI de Easypanel.
*   Gestión automática de SSL/HTTPS (Traefik) y Dominios.
*   Logs visibles en el panel.

---

## Opción 2: Docker Compose Manual (Tu Petición)
Si prefieres tener el control total o hacerlo manualmente como indicaste ("descargar lo del repositorio"), utilizaremos **Docker Compose**.

He descargado y preparado los archivos necesarios en tu carpeta `docker`:
1.  `docker-compose.yml`: El archivo de orquestación oficial.
2.  `.env`: Archivo de variables de entorno que NECESITAS configurar.

### Pasos para despliegue manual en VPS:

1.  **Subir archivos**: Sube la carpeta `docker` que he creado a tu VPS (por ejemplo, a `/home/usuario/supabase`).
2.  **Configurar Secretos**:
    *   Edita el archivo `.env`.
    *   **CRÍTICO**: Genera claves reales para `JWT_SECRET`, `ANON_KEY` y `SERVICE_ROLE_KEY`. No uses los valores por defecto en producción.
    *   Puedes usar scripts online o la CLI de Supabase para generar estos JWTs.
3.  **Ejecutar**:
    Conéctate por SSH a tu VPS, ve a la carpeta y ejecuta:
    ```bash
    docker compose up -d
    ```
4.  **Acceso**:
    *   La API (Kong) estará en el puerto `8000`.
    *   El Studio (Dashboard) estará en el puerto `3000`.
    *   Necesitarás configurar un Proxy Inverso (como el Traefik de Easypanel o Nginx) para mapear tus dominios a estos puertos.

### Notas Importantes sobre Volúmenes
El `docker-compose.yml` está configurado para usar carpetas locales (`./volumes/...`) para persistir los datos (base de datos, storage).
*   **Asegúrate de hacer backups** de la carpeta `./volumes` regularmente.
*   Si borras la carpeta, pierdes la base de datos.
