# Instalación y Configuración

## Archivo de configuración principal

Toda la configuración del sistema se centraliza en:

```
/opt/sipdoc/peke-web/resources/pekepbx.conf
```

## Secciones del fichero de configuración

### [app]

```ini
APP_ENV=production          # production | development
APP_DEBUG=false
APP_KEY=                    # Clave de cifrado Laravel (32 chars)
APP_URL=https://mi-pbx.com
APP_VERSION=v36
```

### [database]

```ini
DB_HOST=localhost
DB_PORT=3306
DB_DATABASE=peke_pbx
DB_USERNAME=peke
DB_PASSWORD=
```

### [redis]

```ini
REDIS_HOST=127.0.0.1
REDIS_PORT=6379
REDIS_DATABASE=1
```

### [system]

```ini
PUBLICIP=          # IP pública del servidor
LOCALIP=           # IP local/privada
DOMAIN=            # Dominio principal
MSTDOMAIN=         # Dominio Microsoft Teams
```

### [asterisk]

```ini
ASTERISK_BIN=/usr/sbin/asterisk
BLF_CONFIG_PATH=   # Ruta al fichero de BLF
CUSTOM_DID_PATH=   # Ruta para DIDs personalizados
DEFAULT_OUTBOUND_PROXY=
FAILOVER_OUTBOUND_PROXY=
```

### [ami]

```ini
AMI_HOST=localhost
AMI_PORT=5100
AMI_USER=
AMI_SECRET=
```

### [web]

```ini
JWT_SECRET=                         # Mínimo 32 caracteres
NODE_HTTP_BRIDGE_HOST=localhost
NODE_HTTP_BRIDGE_PORT=3004
NOTIFY_ON_COMPANY_CHANGE=false
NOTIFY_ON_EXTENSION_CHANGE=false
```

### [security]

```ini
SUPER_LOGIN_PORT=1443    # Puerto exclusivo para super administrador
ENFORCE_2FA=false
```

### [services]

```ini
JITSI_DOMAIN=            # Servidor Jitsi Meet
KAMAILIO_RPC_SERVER=     # API RPC de Kamailio
```

### [mail]

```ini
MAIL_DRIVER=smtp
MAIL_HOST=
MAIL_PORT=587
MAIL_USERNAME=
MAIL_PASSWORD=
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=
```

### [regional]

```ini
TIMEZONE=Europe/Madrid
LANGUAGE=es
```

### [cleanup]

```ini
CDR_RETENTION_MONTHS=12
QUEUE_LOG_RETENTION_DAYS=90
```

### [fax]

```ini
HYLAFAX_HOST=localhost
HYLAFAX_PORT=4559
FAX_AUTH_TOKEN=
IMAP_HOST=
IMAP_USER=
IMAP_PASSWORD=
```

### [siptrunks]

```ini
DEFAULT_TRUNK_PEER=
DEFAULT_TRUNK_USER=
DEFAULT_TRUNK_PASS=
FAILOVER_TRUNK_PEER=
```

### [queue]

```ini
QUEUE_CONNECTION=redis
QUEUE_FALLBACK=database
```

### [panel]

```ini
PANEL_HOST=
PANEL_PORT=
PANEL_LICENSE_KEY=
```

## Gestión de procesos (PM2)

El sistema arranca con PM2. Los dos procesos gestionados son:

### PekeAMI (proceso principal)
```bash
pm2 start /opt/sipdoc/peke-web/node-src/app.js --name PekeAMI
```
- Puerto WebSocket JWT: **5050**
- Puerto WebSocket Cookie: **3002**
- Puerto HTTP Bridge: **3004**
- Memoria máxima: 1 GB

### FastAGI Server
```bash
pm2 start /opt/sipdoc/peke-web/node-src/fastagi/fastagi-server.js --name FastAGI
```
- Puerto: **4573** (localhost)
- Memoria máxima: 1 GB

### Comandos útiles

```bash
pm2 list                    # Ver estado de procesos
pm2 restart PekeAMI         # Reiniciar proceso principal
pm2 logs PekeAMI            # Ver logs en tiempo real
pm2 monit                   # Monitor de recursos
```

## Directorio de instalación

```
/opt/sipdoc/peke-web/
├── node-src/           # Backend Node.js
│   ├── app.js          # Entrada principal
│   ├── fastagi/        # Servidor FastAGI
│   ├── modules/        # Módulos: AMI, WebSocket, HTTP, FAX
│   ├── helpers/        # Configuración y utilidades
│   └── models/         # Modelos Sequelize
├── src/                # Backend Laravel
│   ├── app/
│   │   └── Http/Controllers/Api/   # Controladores REST
│   ├── routes/         # Definición de rutas
│   └── database/       # Migraciones y schema SQL
├── resources/
│   └── pekepbx.conf    # Fichero de configuración central
└── socketPhp/          # Bridge PHP-Socket.io (FAX)
```
