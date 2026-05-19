# PekePBX — Documentación Técnica

Sistema VoIP/PBX multi-tenant construido sobre Asterisk, con backend en Laravel + Node.js y comunicación en tiempo real via WebSockets.

## Índice

- [Instalación](docs/instalacion.md)
- [Arquitectura](docs/arquitectura.md)
- [Configuración](docs/configuracion.md)
- [API REST](docs/api.md)
- [WebSockets y Tiempo Real](docs/websockets.md)
- [Módulos y Funcionalidades](docs/modulos.md)
- [Integraciones](docs/integraciones.md)
- [Despliegue](docs/despliegue.md)

## Versión actual

**v36** — Desplegada el 12 de mayo de 2026.

## Stack tecnológico

| Capa | Tecnología |
|------|-----------|
| PBX | Asterisk + AMI |
| SIP Routing | Kamailio |
| Backend API | PHP 8.2 + Laravel 12 |
| Tiempo real | Node.js + Socket.io / WebSocket |
| Base de datos | MySQL + Redis |
| ORM PHP | Doctrine |
| ORM Node | Sequelize |
| Call flow | FastAGI |
| Procesos | PM2 |
