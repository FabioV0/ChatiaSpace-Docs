# ChatIA Space - Tutorial de Uso

## 1. Arranque rápido

```bash
# Clonar el repositorio (si aplica)
git clone <repo-url>
cd chatia-space

# Copiar variables de entorno
cp env.example .env
# Editar .env con tus valores (ver sección 2)

# Construir y levantar
docker-compose up --build -d
```

El sistema quedará disponible en:
- API: http://localhost:8000
- Health check: http://localhost:8000/health
- Webhook de Meta: http://localhost:8000/webhook (para configurar en WhatsApp Business)

## 2. Variables de entorno clave (.env)

| Variable | Descripción | Ejemplo |
|----------|-------------|---------|
| `POSTGRES_USER` | Usuario de PostgreSQL | `postgres` |
| `POSTGRES_PASSWORD` | Contraseña de PostgreSQL | `postgres` |
| `POSTGRES_DB` | Nombre de la base | `chatia` |
| `DATABASE_URL` | URL de conexión async | `postgresql+asyncpg://postgres:postgres@postgres:5432/chatia` |
| `REDIS_URL` | URL de Redis | `redis://redis:6379/0` |
| `SENTRY_DSN` | DSN de Sentry (opcional en dev) | `https://...@sentry.io/...` |
| `FERNET_KEY` | Clave para cifrado de secretos (base64, 32 bytes) | generar con `python -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"` |
| `ADMIN_API_KEY` | Key para endpoints protegidos de admin | `cambiar-por-una-segura` |
| `META_VERIFY_TOKEN` | Token de verificación de webhook de Meta | `mi-token-verificacion` |
| `META_APP_SECRET` | Secret de la app de Meta (para validar webhooks) | `secret-de-la-app` |

Variables opcionales para integraciones (rellenar según plan):
- `OPENAI_API_KEY`
- `LLAMA_API_BASE_URL` (ej: `https://api.together.xyz/v1`)
- `GOOGLE_SERVICE_ACCOUNT_JSON` (ruta o contenido JSON)
- `GOOGLE_VISION_CREDENTIALS_JSON`

## 3. Bootstrap inicial (crear admin global y primer tenant)

Ejecutar el script de bootstrap:

```bash
docker-compose exec app python scripts/bootstrap.py
```

Esto crea:
- Admin global: usuario `admin@example.com`, password temporal mostrado en salida
- Primer tenant: con plan Básico
- Admin del tenant: usuario `tenant-admin@example.com`, password temporal mostrado

Guarda esas credenciales; deberás cambiarlas en primer uso.

## 4. Acceso a la base de datos

### 4.1. Psql dentro del contenedor
```bash
docker-compose exec postgres psql -U postgres -d chatia
```

### 4.2. Desde tu máquina (si expusiste el puerto)
```bash
psql postgresql://postgres:postgres@localhost:5432/chatia
```

Tablas principales:
- `tenants` → información de clientes
- `tenant_config` → configuración flexible (JSONB)
- `tenant_secrets` → credenciales cifradas
- `leads`, `appointments`, `payment_receipts`, `admin_users`, `audit_log`

### 4.3. Modificar información
Ejemplo: cambiar plan de un tenant
```sql
UPDATE tenants SET plan_id = 'pro' WHERE id = '<tenant-uuid>';
```
Ejemplo: actualizar configuración (JSONB)
```sql
UPDATE tenant_config SET config = config || '{"google_sheet_id": "1abc..."}'::jsonb WHERE tenant_id = '<tenant-uuid>';
```

## 5. Endpoints de administración (requieren X-API-Key)

Obtener tu `ADMIN_API_KEY` del `.env`.

Listar tenants:
```bash
curl -H "X-API-Key: $(grep ADMIN_API_KEY .env | cut -d'=' -f2)" \
     http://localhost:8000/admin/tenants
```

Crear nuevo tenant:
```bash
curl -X POST -H "X-API-Key: <tu-key>" \
     -H "Content-Type: application/json" \
     -d '{
           "business_name": "Mi Negocio",
           "contact_phone": "+5215512345678",
           "plan_id": "basic",
           "status": "active"
         }' \
     http://localhost:8000/admin/tenants
```

Crear admin de tenant:
```bash
curl -X POST -H "X-API-Key: <tu-key>" \
     -H "Content-Type: application/json" \
     -d '{
           "email": "admin@negocio.com",
           "password": "Cambiar123!",
           "tenant_id": "<tenant-uuid>",
           "role": "tenant_admin"
         }' \
     http://localhost:8000/admin/admin-users
```

Actualizar secretos de un tenant (ej: credenciales de Meta):
```bash
curl -X POST -H "X-API-Key: <tu-key>" \
     -H "Content-Type: application/json" \
     -d '{
           "tenant_id": "<tenant-uuid>",
           "meta_access_token": "EAA...",
           "meta_phone_number_id": "123456789",
           "meta_verify_token": "mi-token",
           "meta_app_secret": "secreto-app"
         }' \
     http://localhost:8000/admin/secrets
```

## 6. Comandos de admin vía WhatsApp

Desde el número de WhatsApp asociado al tenant, enviar:

- `/admin_login` → inicia flujo; responder con `usuario:contraseña`
- `/admin_estado` → verifica si tienes sesión admin activa
- `/admin_logout` → cierra sesión admin
- `/tomar` → toma el handoff (pausa IA)
- `/soltar` → suelta el handoff (restaura IA)
- `/estado` → muestra estado actual de la conversación
- `/aprobar_pago <receipt_id>` → aprueba comprobante (solo admin)
- `/rechazar_pago <receipt_id> [motivo]` → rechaza comprobante
- `/sync_sheets` → sincroniza Google Sheets → JSON local
- `/sync_drive` → sincroniza Google Drive → índice local
- `/sync_all` → hace ambas sincronizaciones

## 7. Scheduler y jobs en background

Los jobs corren automáticamente al iniciar el contenedor `app`:

- **Sheets sync** → cada 5 minutos
- **Drive sync** → cada 10 minutos
- **Recordatorios de cita** → cada 1 minuto (24h y 1h antes)
- **Decay de leads** → cada 24 horas
- **Follow‑up post‑cita** → cada 1 minuto

Para ver logs:
```bash
docker-compose logs -f app
```

## 8. Pruebas rápidas

Ejecutar unit tests (no requieren DB externa):
```bash
docker-compose exec app pytest tests/ -v
```

## 9. Detener y limpiar

```bash
docker-compose down        # detiene contenedores
docker-compose down -v     # además elimina volúmenes (datos de PG y Redis)
```

## 10. Próximos pasos

1. Configura tu número de WhatsApp Business en Meta y apunta el webhook a `http://<tu-dominio>:8000/webhook`.
2. Completa las credenciales de Google (Sheets, Drive, Calendar) en `tenant_secrets` según el plan del tenant.
3. Ajusta límites y configuraciones por tenant vía `tenant_config` (JSONB) o mediante el endpoint `/admin/tenant-config`.
4. Monitorea errores en Sentry (si configuraste `SENTRY_DSN`).
5. Usa los endpoints de admin para crear más tenants y admins según necesites.

¡Listo! Tu instancia de ChatIA Space está operativa.
```