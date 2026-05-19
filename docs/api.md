# API REST

La API REST está construida con Laravel 12 y utiliza autenticación JWT.

## Autenticación

### Login
```http
POST /api/v2/login
Content-Type: application/json

{
  "username": "user@empresa.com",
  "password": "contraseña"
}
```

**Respuesta:**
```json
{
  "token": "eyJ...",
  "expires_in": 3600
}
```

### Refresh de token
```http
GET /api/v2/refresh
Authorization: Bearer {token}
```

### Logout
```http
GET /api/v2/logout
Authorization: Bearer {token}
```

### Magic Login (enlace de acceso automático)
```http
GET /api/v2/magicLogin/{userId}
Authorization: Bearer {token}
```

---

## Empresas

```http
GET    /api/v2/company              # Listar empresas
GET    /api/v2/company/{id}         # Obtener empresa
POST   /api/v2/company              # Crear empresa
PUT    /api/v2/company/{id}         # Actualizar empresa
DELETE /api/v2/company/{id}         # Eliminar empresa
```

---

## Extensiones

```http
GET    /api/v2/extension              # Listar extensiones
GET    /api/v2/extension/{id}         # Obtener extensión
POST   /api/v2/extension              # Crear extensión
PUT    /api/v2/extension/{id}         # Actualizar extensión
DELETE /api/v2/extension/{id}         # Eliminar extensión
```

---

## Colas (Queues)

```http
GET    /api/v2/queue                  # Listar colas
GET    /api/v2/queue/{id}             # Obtener cola
POST   /api/v2/queue                  # Crear cola
PUT    /api/v2/queue/{id}             # Actualizar cola
DELETE /api/v2/queue/{id}             # Eliminar cola
```

---

## DIDs (Números directos)

```http
GET    /api/v2/did                    # Listar DIDs
GET    /api/v2/did/{id}               # Obtener DID
POST   /api/v2/did                    # Crear DID
PUT    /api/v2/did/{id}               # Actualizar DID
DELETE /api/v2/did/{id}               # Eliminar DID
```

---

## CDR (Registros de llamadas)

```http
GET    /api/v2/cdr                    # Listar CDR con filtros
GET    /api/v2/cdr/{id}               # Obtener registro
```

Parámetros de filtro disponibles:
- `dateFrom` / `dateTo`
- `extension`
- `direction` (inbound | outbound)
- `companyId`

---

## Métricas y Dashboard

```http
GET    /api/v2/metrics                # Métricas generales del sistema
```

---

## Listas negras

```http
GET    /api/v2/blacklist              # Listar números bloqueados
POST   /api/v2/blacklist              # Añadir número
DELETE /api/v2/blacklist/{id}         # Eliminar bloqueo
```

---

## Salas de conferencia (Video)

```http
GET    /api/v2/videoRoom              # Listar salas
POST   /api/v2/videoRoom              # Crear sala
DELETE /api/v2/videoRoom/{id}         # Eliminar sala
```

---

## Otros endpoints

```http
GET/POST /api/v2/alias                # Gestión de alias de teléfono
GET/POST /api/v2/dialpin              # Prefijos y PINs de marcación
GET/POST /api/v2/event                # Registro de eventos
GET/POST /api/v2/banned               # Bloqueo de IPs
```

---

## HTTP Bridge API (Node.js — Puerto 3004)

Endpoints para control en tiempo real de llamadas. Uso interno entre Laravel y Node.js.

### Estado de dispositivos (BLF)
```http
GET    /api/device/state              # Obtener estado de todos los dispositivos
PUT    /api/device/state              # Actualizar estado de dispositivo
DELETE /api/device/state              # Eliminar estado de dispositivo
```

### Control de llamadas
```http
GET    /api/call/transferable         # Verificar si una llamada es transferible
GET    /api/call/transfer             # Transferir llamada
POST   /api/manager/call              # Originar llamada via AMI
```

### Gestión de Asterisk
```http
GET    /api/manager/reloadDialplan    # Recargar el dialplan
GET    /api/manager/reloadQueues      # Recargar configuración de colas
```

### Agentes de cola
```http
GET    /api/queueMember/changeStatus  # Cambiar estado de agente (disponible/pausado)
GET    /api/queue/memberStatus        # Obtener estado de agentes
```

---

## Códigos de respuesta

| Código | Significado |
|--------|-------------|
| 200 | OK |
| 201 | Creado correctamente |
| 400 | Petición incorrecta |
| 401 | No autenticado |
| 403 | Sin permisos |
| 404 | No encontrado |
| 422 | Error de validación |
| 500 | Error del servidor |
