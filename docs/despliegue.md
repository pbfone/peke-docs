# Despliegue

## Requisitos del servidor

| Componente | Versión mínima |
|------------|---------------|
| OS | Ubuntu 22.04 LTS |
| PHP | 8.2 |
| Node.js | 18 LTS |
| MySQL | 8.0 |
| Redis | 7.x |
| Asterisk | 18+ |
| Kamailio | 5.x |
| HylaFAX | 6.x (opcional) |
| FFmpeg | 4.x |
| PM2 | 5.x |

## Directorio de instalación

```
/opt/sipdoc/peke-web/
```

## Proceso de despliegue (rama v36)

El deploy se realiza automáticamente desde la rama `v36`:

```bash
# Acceder al directorio
cd /opt/sipdoc/peke-web

# Actualizar código
git pull origin v36

# Instalar dependencias PHP
composer install --no-dev --optimize-autoloader

# Ejecutar migraciones
php artisan migrate --force

# Limpiar cachés Laravel
php artisan config:cache
php artisan route:cache
php artisan view:cache

# Instalar dependencias Node.js
cd node-src && npm install --production

# Reiniciar procesos
pm2 restart PekeAMI
pm2 restart FastAGI
```

## Configuración inicial

### 1. Copiar y editar configuración
```bash
cp resources/pekepbx.conf.example resources/pekepbx.conf
nano resources/pekepbx.conf
```

Rellenar al menos:
- `APP_KEY` — Generar con `php artisan key:generate`
- `DB_*` — Credenciales de base de datos
- `REDIS_*` — Conexión Redis
- `AMI_*` — Credenciales AMI de Asterisk
- `JWT_SECRET` — Mínimo 32 caracteres aleatorios
- `PUBLICIP` / `LOCALIP` / `DOMAIN`

### 2. Crear base de datos
```sql
CREATE DATABASE peke_pbx CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'peke'@'localhost' IDENTIFIED BY 'contraseña';
GRANT ALL PRIVILEGES ON peke_pbx.* TO 'peke'@'localhost';
```

### 3. Migraciones iniciales
```bash
php artisan migrate
php artisan db:seed  # Si hay seeders de datos iniciales
```

### 4. Configurar Asterisk AMI

En `/etc/asterisk/manager.conf`:
```ini
[general]
enabled = yes
port = 5100
bindaddr = 127.0.0.1

[peke]
secret = tu-secreto-ami
deny = 0.0.0.0/0.0.0.0
permit = 127.0.0.1/255.255.255.0
read = all
write = all
```

### 5. Arrancar procesos con PM2
```bash
cd /opt/sipdoc/peke-web/node-src
pm2 start ecosystem.config.js
pm2 save
pm2 startup  # Configurar arranque automático al reiniciar el servidor
```

## Puertos utilizados

| Puerto | Servicio | Protocolo |
|--------|----------|-----------|
| 80/443 | Panel web (Nginx/Apache) | HTTP/HTTPS |
| 1443 | Super admin login | HTTPS |
| 3002 | WebSocket Cookie | WS/WSS |
| 3003 | Bridge PHP-Socket.io (FAX) | WS interno |
| 3004 | HTTP Bridge API | HTTP interno |
| 4573 | FastAGI | TCP (localhost) |
| 5050 | WebSocket JWT | WS/WSS |
| 5100 | AMI Asterisk | TCP (localhost) |
| 5060 | SIP (Kamailio) | UDP/TCP |
| 6379 | Redis | TCP (localhost) |

## Nginx — Ejemplo de configuración

```nginx
server {
    listen 443 ssl;
    server_name mi-pbx.com;

    root /opt/sipdoc/peke-web/src/public;
    index index.php;

    # WebSocket Cookie (puerto 3002)
    location /socket.io/ {
        proxy_pass http://localhost:3002;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }

    # API PHP
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    ssl_certificate /etc/ssl/certs/mi-pbx.crt;
    ssl_certificate_key /etc/ssl/private/mi-pbx.key;
}
```

## Logs

| Proceso | Log |
|---------|-----|
| PekeAMI | `pm2 logs PekeAMI` |
| FastAGI | `pm2 logs FastAGI` |
| Laravel | `/opt/sipdoc/peke-web/src/storage/logs/laravel.log` |
| Nginx | `/var/log/nginx/error.log` |
| Asterisk | `/var/log/asterisk/full` |

## Actualizaciones

Las actualizaciones siguen el patrón de deploy desde la rama `v36`. Antes de actualizar en producción:

1. Hacer backup de la base de datos
2. Revisar cambios en migraciones
3. Testear en entorno de staging
4. Ejecutar el deploy en horario de baja actividad
