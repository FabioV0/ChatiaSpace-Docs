"# ¿Existe un dashboard web para administración?

## Respuesta breve
**No, en la versión v1.0 no hay un dashboard web obligatorio.** La administración se realiza mediante:

1. **Endpoints de API REST** (requieren `X-API-Key`)
2. **Comandos de WhatsApp** (desde el número asociado al tenant)
3. **Acceso directo a la base de datos** (PostgreSQL)
4. **Variables de entorno y archivos de configuración**

## Detalles de administración disponible

### API de administración (endpoints protegidos)
- `GET /admin/tenants` - Lista todos los tenants
- `POST /admin/tenants` - Crea un nuevo tenant
- `POST /admin/admin-users` - Crea admin de tenant o global
- `POST /admin/secrets` - Actualiza credenciales cifradas de un tenant
- `POST /admin/tenant-config` - Actualiza configuración flexible (JSONB)
- `GET /health` - Verificación de salud del sistema

### Comandos de WhatsApp (para admins autenticados)
- `/admin_login` - Inicia sesión de admin
- `/admin_estado` - Verifica estado de sesión admin
- `/admin_logout` - Cierra sesión admin
- `/tomar` / `/soltar` - Control de handoff humano
- `/sync_sheets`, `/sync_drive`, `/sync_all` - Sincronización de Google
- `/aprobar_pago <id>` / `/rechazar_pago <id> [motivo]` - Gestión de pagos
- `/estado` - Estado actual de la conversación

## Próximos pasos según el blueprint
El **dashboard web es opcional y se планирует para fases posteriores** (ver sección 5 del blueprint: "Dashboard web opcional en fases posteriores, no obligatorio en v1").

Si necesitas una interfaz web para administración, tendrás que:
1. Construirla tú mismo consumiendo los endpoints de API ya existentes
2. Esperar a versiones futuras donde se incluya como feature opcional
3. Integrar con herramientas externas como Metabase, Superset, etc., conectándolas directamente a la base de datos PostgreSQL

## Cómo empezar a administrar sin dashboard
1. Levanta el sistema con `docker-compose up --build -d`
2. Ejecuta el bootstrap: `docker-compose exec app python scripts/bootstrap.py`
3. Usa los endpoints de API con tu `ADMIN_API_KEY` (ver tutorial.md sección 5)
4. O bien, usa WhatsApp enviando comandos al número configurado del tenant

¡La API REST está completamente funcional y documentada en el código!
"