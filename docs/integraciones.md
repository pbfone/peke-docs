# Integraciones

## Microsoft Teams

PekePBX puede integrarse con Microsoft Teams para enrutar llamadas desde/hacia el entorno Teams de una empresa.

**Configuración:**
```ini
# pekepbx.conf
[system]
MSTDOMAIN=tenant.teams.microsoft.com
```

**Casos de uso:**
- Llamadas entrantes al DID → extensión Teams del usuario
- Click-to-call desde Teams hacia la red PSTN via PekePBX

---

## Jitsi Meet

Integración para videoconferencia corporativa.

**Configuración:**
```ini
[services]
JITSI_DOMAIN=meet.mi-empresa.com
```

Las salas de conferencia de PekePBX pueden lanzar una sesión Jitsi Meet asociada. El enlace se genera automáticamente y se distribuye a los participantes.

---

## HylaFAX

Servidor de FAX open source integrado para envío y recepción de documentos.

**Flujo de recepción:**
```
Llamada FAX entrante → Asterisk → HylaFAX → PDF → Email al destinatario
```

**Flujo de envío:**
```
Panel web → API Laravel → Node.js FAX Service → HylaFAX → Línea PSTN
```

**Configuración:**
```ini
[fax]
HYLAFAX_HOST=localhost
HYLAFAX_PORT=4559
FAX_AUTH_TOKEN=token-seguro

# Mail to Fax (IMAP)
IMAP_HOST=mail.empresa.com
IMAP_USER=fax@empresa.com
IMAP_PASSWORD=contraseña
```

---

## Kamailio

Proxy SIP para enrutamiento de llamadas. PekePBX se comunica con Kamailio a través de su API RPC para actualizaciones de configuración dinámica.

**Configuración:**
```ini
[services]
KAMAILIO_RPC_SERVER=http://localhost:5060/rpc
```

**Operaciones:**
- Recarga de rutas SIP tras cambios en trunks
- Gestión de usuarios registrados

---

## PowerDNS (opcional)

Gestión dinámica de registros DNS. Usado en despliegues donde PekePBX gestiona subdominios de clientes automáticamente.

---

## LDAP / Active Directory

Autenticación de administradores mediante directorio corporativo.

**Configuración en Laravel:**
```env
LDAP_HOST=ldap.empresa.com
LDAP_PORT=389
LDAP_BASE_DN=dc=empresa,dc=com
LDAP_USERNAME=cn=admin,dc=empresa,dc=com
LDAP_PASSWORD=contraseña
```

---

## FFmpeg

Usado internamente para transcodificación de audio:
- Conversión de grabaciones a MP3/OGG para el panel web
- Procesamiento de ficheros de audio para IVR y música en espera

Debe estar instalado en el servidor:
```bash
apt install ffmpeg
```
