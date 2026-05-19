# Guía de Instalación

Instalación completa de PekePBX desde cero en un servidor Debian 12 (Bookworm).

---

## Requisitos previos

| Componente | Versión |
|------------|---------|
| OS | Debian 12 (Bookworm) |
| PHP | 8.2 |
| Node.js | 18 LTS |
| MySQL / MariaDB | 8.0+ |
| Redis | 7.x |
| Asterisk | 18+ |
| Kamailio | 5.x |
| FFmpeg | 4.x |
| PM2 | 5.x |
| Composer | 2.x |
| HylaFAX | 6.x (solo si se usa FAX) |

---

## 1. Preparar el servidor

```bash
apt update && apt upgrade -y
apt install -y git curl unzip ffmpeg
```

---

## 2. Instalar PHP 8.2

En Debian 12 se usa el repositorio de Sury (equivalente al ondrej/php de Ubuntu):

```bash
apt install -y apt-transport-https lsb-release ca-certificates curl
curl -sSLo /usr/share/keyrings/deb.sury.org-php.gpg https://packages.sury.org/php/apt.gpg
echo "deb [signed-by=/usr/share/keyrings/deb.sury.org-php.gpg] https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list
apt update
apt install -y php8.2 php8.2-fpm php8.2-cli php8.2-mysql php8.2-redis \
    php8.2-xml php8.2-mbstring php8.2-curl php8.2-zip php8.2-gd \
    php8.2-ldap php8.2-intl php8.2-bcmath
```

---

## 3. Instalar Node.js 18

```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | bash -
apt install -y nodejs
npm install -g pm2
```

---

## 4. Instalar MySQL

```bash
apt install -y mysql-server
mysql_secure_installation
```

Crear base de datos y usuario:

```sql
CREATE DATABASE peke_pbx CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'peke'@'localhost' IDENTIFIED BY 'CONTRASEÑA_SEGURA';
GRANT ALL PRIVILEGES ON peke_pbx.* TO 'peke'@'localhost';
FLUSH PRIVILEGES;
```

---

## 5. Instalar Redis

```bash
apt install -y redis-server
systemctl enable redis-server
systemctl start redis-server
```

---

## 6. Instalar Composer

```bash
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
```

---

## 7. Desplegar el código

```bash
mkdir -p /opt/sipdoc
cd /opt/sipdoc
git clone <url-repositorio-peke-web> peke-web
cd peke-web
git checkout v36
```

---

## 8. Crear el fichero de configuración

El sistema lee su configuración desde `/etc/pekepbx.conf`. Copiarlo y editarlo:

```bash
cp /opt/sipdoc/peke-web/resources/pekepbx.conf /etc/pekepbx.conf
nano /etc/pekepbx.conf
```

### Valores obligatorios a rellenar

**[app]**
```ini
APP_ENV=production
APP_DEBUG=false
APP_KEY=                  # Generar en el paso 9
APP_URL=https://mi-pbx.com
APP_VERSION=36
```

**[database]**
```ini
DB_HOST=localhost
DB_DATABASE=peke_pbx
DB_USERNAME=peke
DB_PASSWORD=CONTRASEÑA_SEGURA
```

**[redis]**
```ini
REDIS_HOST=127.0.0.1
REDIS_PORT=6379
REDIS_DATABASE=1
```

**[system]**
```ini
PUBLICIP=X.X.X.X
LOCALIP=192.168.X.X
DOMAIN=mi-pbx.com
```

**[ami]**
```ini
AMI_PORT=5100
AMI_URL=localhost
AMI_USER=pekeweb
AMI_PASSWORD=CONTRASEÑA_AMI
```

**[web]**
```ini
JWT_SECRET=CADENA_ALEATORIA_MIN_32_CARACTERES
NODE_HTTP_BRIDGE_HOST=localhost
NODE_HTTP_BRIDGE_PORT=3004
```

**[regional]**
```ini
timezone=Europe/Madrid
lang=es
```

**[services]**
```ini
LOG_FILES_PATH=/var/lib/peke/logs/
```

**[license]**
```ini
id=ID_DE_LICENCIA
```

---

## 9. Instalar dependencias PHP y generar clave

```bash
cd /opt/sipdoc/peke-web/src
composer install --no-dev --optimize-autoloader

# Generar APP_KEY para pekepbx.conf
php artisan pbx:create-random-key
# Copiar el resultado al campo APP_KEY en /etc/pekepbx.conf
```

---

## 10. Crear directorios necesarios

```bash
mkdir -p /etc/asterisk/blf
mkdir -p /etc/asterisk/customdid
mkdir -p /etc/asterisk/customdidbackup
mkdir -p /var/lib/asterisk/fax
mkdir -p /var/lib/peke/logs
chown -R www-data:www-data /opt/sipdoc/peke-web/src/storage
chown -R www-data:www-data /opt/sipdoc/peke-web/src/bootstrap/cache
```

---

## 11. Ejecutar migraciones de base de datos

```bash
cd /opt/sipdoc/peke-web/src
php artisan doctrine:migrations:migrate --no-interaction
```

> Las migraciones crean todas las tablas del sistema. En instalaciones nuevas pueden tardar varios minutos (450+ migraciones).

---

## 12. Crear super administrador

```bash
php artisan pbx:create-super-admin "Nombre Apellido" admin@mi-pbx.com contraseña_segura
```

---

## 13. Configurar el scheduler de Laravel

Añadir al crontab del sistema:

```bash
crontab -e
```

```
* * * * * php /opt/sipdoc/peke-web/src/artisan schedule:run >> /dev/null 2>&1
```

Tareas programadas que ejecuta:
- Cada minuto: eliminar llamadas perdidas antiguas
- Cada 10 segundos: sincronizar extensiones desde Kamailio
- Cada hora: ping de estado del sistema
- Diariamente: calcular estadísticas del dashboard

---

## 14. Instalar dependencias Node.js

```bash
cd /opt/sipdoc/peke-web/node-src
npm install --production
```

---

## 15. Configurar Asterisk AMI

Editar `/etc/asterisk/manager.conf`:

```ini
[general]
enabled = yes
port = 5100
bindaddr = 127.0.0.1

[pekeweb]
secret = CONTRASEÑA_AMI
deny = 0.0.0.0/0.0.0.0
permit = 127.0.0.1/255.255.255.0
read = all
write = all
```

```bash
asterisk -rx "manager reload"
```

---

## 16. Arrancar procesos Node.js con PM2

```bash
cd /opt/sipdoc/peke-web/node-src
pm2 start ecosystem.config.js
pm2 save
pm2 startup  # Seguir las instrucciones del comando para habilitar arranque automático
```

Verificar que los procesos están activos:

```bash
pm2 list
```

Deberían aparecer:
- `PekeAMI` — estado `online`
- `FastAGI` — estado `online`

---

## 17. Configurar Nginx

Instalar Nginx:

```bash
apt install -y nginx
```

Crear el fichero de configuración del sitio:

```bash
nano /etc/nginx/sites-available/pekepbx
```

```nginx
server {
    listen 80;
    server_name mi-pbx.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name mi-pbx.com;

    root /opt/sipdoc/peke-web/src/public;
    index index.php;

    # WebSocket Cookie (clientes web)
    location /socket.io/ {
        proxy_pass http://localhost:3002;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }

    # API y panel web
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

# Super admin (puerto dedicado)
server {
    listen 1443 ssl;
    server_name mi-pbx.com;

    root /opt/sipdoc/peke-web/src/public;
    index index.php;

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

```bash
ln -s /etc/nginx/sites-available/pekepbx /etc/nginx/sites-enabled/
nginx -t && systemctl reload nginx
```

---

## 18. Verificación final

```bash
# Procesos Node.js
pm2 list

# Conexión AMI
asterisk -rx "manager show connected"

# Redis
redis-cli ping  # Debe responder PONG

# MySQL
mysql -u peke -p peke_pbx -e "SHOW TABLES;" | wc -l  # Debe haber >50 tablas

# Laravel scheduler
php /opt/sipdoc/peke-web/src/artisan schedule:run

# Logs de PekeAMI (no debe haber errores de conexión)
pm2 logs PekeAMI --lines 50
```

---

## Puertos que deben estar accesibles

| Puerto | Servicio | Acceso |
|--------|----------|--------|
| 80 | HTTP (redirect) | Público |
| 443 | HTTPS — Panel web | Público |
| 1443 | HTTPS — Super admin | Restringido |
| 3002 | WebSocket Cookie | Público (via Nginx proxy) |
| 5050 | WebSocket JWT | Público (softphones) |
| 5060 | SIP (Kamailio) | Público |
| 3004 | HTTP Bridge Node | Solo localhost |
| 4573 | FastAGI | Solo localhost |
| 5100 | AMI Asterisk | Solo localhost |
| 6379 | Redis | Solo localhost |

---

## Solución de problemas comunes

### PekeAMI no conecta con AMI
```bash
pm2 logs PekeAMI --lines 100
# Verificar credenciales AMI en /etc/pekepbx.conf y /etc/asterisk/manager.conf
```

### Error de permisos en storage
```bash
chown -R www-data:www-data /opt/sipdoc/peke-web/src/storage
chmod -R 775 /opt/sipdoc/peke-web/src/storage
```

### Las migraciones fallan
```bash
# Verificar conexión a base de datos
php /opt/sipdoc/peke-web/src/artisan doctrine:migrations:status
```

### WebSocket no conecta desde el navegador
```bash
# Verificar que el proxy WebSocket en Nginx está activo
nginx -t
curl -i http://localhost:3002/socket.io/?transport=polling
```
