# Sinaptyca Outreach Studio

Panel interno de Sinaptyca para editar borradores y enviar **correos autorizados** por medio de la API de Resend. Es independiente de OnlySend, que puede seguir funcionando normalmente.

## Estructura

```text
sinaptyca-outreach-studio/
├── public/
│   ├── _worker.js      # Cloudflare Pages Function en modo avanzado
│   └── index.html      # Archivo de entrada para el despliegue de Pages
├── .gitignore
└── README.md
```

No requiere Node, `npm install`, compilación ni Wrangler. El archivo `_worker.js` **debe permanecer dentro de `public/`** porque esa será la carpeta de salida de Cloudflare Pages.

## Crear repositorio en GitHub

1. En GitHub crea un repositorio **privado** llamado `sinaptyca-outreach-studio` sin inicializar archivos opcionales.
2. Sube el contenido de esta carpeta al repositorio. Mantén la estructura `public/_worker.js` y `public/index.html`.
3. La rama de producción debe llamarse `main`.
4. No subas ningún token de administrador ni clave de Resend. Configúralos exclusivamente como secretos en Cloudflare.

## Conectar a Cloudflare Pages (GitHub)

1. En Cloudflare entra en **Workers & Pages → Create application → Pages → Connect to Git**.
2. Conecta GitHub, autoriza solo el repositorio necesario, y selecciona `sinaptyca-outreach-studio`.
3. Parámetros de despliegue:
   - Framework preset: **None**
   - Production branch: `main`
   - Build command: `exit 0` (alternativa: dejarlo vacío, si la interfaz lo permite)
   - Build output directory: `public`
   - Root directory: raíz del repositorio (predeterminada)
4. Crea el proyecto. El Worker se publicará mediante `_worker.js` y responderá a la raíz `/` y a las rutas `/api/*`.
5. No reutilices un proyecto Pages creado con *Direct Upload*: Cloudflare no permite convertirlo después a Git integration. Crea un proyecto nuevo con *Connect to Git*.

## Variables de entorno en Cloudflare

En el proyecto de Pages: **Settings → Variables and Secrets** (los nombres pueden variar según la interfaz).

| Nombre | Valor inicial | Tipo |
| --- | --- | --- |
| `RESEND_API_KEY` | Crea otra API key para este proyecto en Resend; permisos mínimos de envío | **Secret** |
| `ADMIN_TOKEN` | Token aleatorio de 32 bytes; por ejemplo el resultado de `openssl rand -hex 32` | **Secret** |
| `SEND_MODE` | `test` | Plain text |
| `TEST_TO` | Dirección TUYA que puede recibir correos | Plain text |

Asegúrate de que las variables estén presentes en **Production** y de desplegar nuevamente si Cloudflare pide aplicar cambios. Verifica las opciones disponibles de *Preview* para no exponer el panel en entornos de prueba.

## Prueba segura

1. Mantén `SEND_MODE=test`.
2. Abre el dominio `*.pages.dev` asignado por Cloudflare.
3. Ingresa tu `ADMIN_TOKEN`. Haz clic en **Abrir borradores**.
4. Selecciona un borrador y envía una prueba. El campo `Para` puede mostrar la dirección del borrador, pero el servidor **redirige todas las pruebas exclusivamente a `TEST_TO`**.
5. Confirma en Resend que se aceptó el mensaje y en tu buzón que lo recibiste.
6. Comprueba el asunto con el prefijo `[PRUEBA para ...]`.

## Seguridad, cumplimiento y límites

- **Resend prohíbe el cold outreach**, incluso con mensajes personalizados. No uses este sistema para enviar prospectos no solicitantes. Referencia: https://resend.com/legal/acceptable-use.
- `SEND_MODE` tiene valor efectivo `test` a menos que configures literalmente `live`; activar `live` requiere además la confirmación de consentimiento en cada envío.
- Un clic de envío más una confirmación manual son necesarios para cada correo. No hay envíos automáticos por lotes.
- El `ADMIN_TOKEN` protege las APIs, pero este MVP no tiene usuarios, registro de auditoría, rate limiting ni una base de datos de consentimientos. Para uso de producción añade Cloudflare Access y controles de permisos.
- No almacena conversaciones ni consulta correos recibidos. Las respuestas no se pueden leer directamente desde este panel.
- La clave de Resend debe permanecer en Cloudflare, nunca en GitHub ni en mensajes de chat.
- Los correos enviados aquí aparecerán en Resend, pero podrían no figurar en el historial local de OnlySend.
- El borrador de Rápido Print tiene el destinatario vacío, según la última edición realizada; confirma el correo antes de cualquier envío autorizado.
