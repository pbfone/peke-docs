# Arquitectura del Sistema

## Visión general

PekePBX es un sistema multi-tenant que permite a un proveedor de servicios gestionar múltiples empresas clientes, cada una con sus propias extensiones, colas, DIDs y configuraciones aisladas.

## Diagrama de arquitectura

```
                    ┌─────────────────────────────────────┐
                    │   Teléfonos SIP / Clientes Web      │
                    └──────────────┬──────────────────────┘
                                   │
                    ┌──────────────┼──────────────────┐
                    │              │                  │
            WebSocket          WebSocket           REST API
            (Puerto 5050)      (Puerto 3002)       (Laravel)
            Auth JWT            Auth Cookie
                    │              │                  │
                    └──────────────┼──────────────────┘
                                   │
                    ┌──────────────▼──────────────────┐
                    │   Node.js Backend (PekeAMI)     │
                    │  - AMI Manager                  │
                    │  - Procesamiento de eventos     │
                    │  - Bridge HTTP (Puerto 3004)    │
                    └──────────┬──────────────────────┘
                               │
                ┌──────────────┼──────────────┐
                │              │              │
          AMI (5100)       FastAGI        MySQL / Redis
          Asterisk          (4573)
```

## Componentes principales

### 1. Asterisk
Motor PBX central. Gestiona las llamadas SIP, colas, IVRs, grabaciones y buzones de voz.
- Expone el AMI (Asterisk Manager Interface) en el puerto 5100
- Se comunica con el servidor FastAGI en el puerto 4573 para lógica de llamada personalizada

### 2. Kamailio
Capa de enrutamiento SIP. Actúa como proxy entre los teléfonos y Asterisk.

### 3. Node.js — PekeAMI
Proceso principal del backend Node. Responsable de:
- Conexión y escucha de eventos AMI
- Servidores WebSocket para clientes en tiempo real
- HTTP Bridge API (puerto 3004) para acciones de control de llamadas
- Integración con el servicio de FAX

### 4. FastAGI Server
Servidor de scripts de llamada. Ejecuta lógica personalizada durante el flujo de llamadas en Asterisk (IVR, routing dinámico, validaciones).

### 5. Laravel 12 (PHP 8.2)
API REST principal. Gestiona:
- Autenticación (JWT)
- CRUD de entidades: extensiones, colas, empresas, DIDs, etc.
- Configuración del sistema multi-tenant

### 6. MySQL
Base de datos relacional principal. Almacena:
- Configuración de empresas, extensiones, colas
- CDR (registros de llamadas)
- Buzones de voz y eventos

### 7. Redis
- Caché de sesiones
- Cola de mensajes (Laravel Queue)
- Estado en tiempo real de dispositivos (BLF)

## Flujo de datos típico

1. El cliente web/softphone se conecta via WebSocket autenticado
2. El AMI Manager captura eventos de Asterisk (llamadas, estados, colas)
3. Los eventos se procesan y distribuyen a los clientes WebSocket conectados
4. Las acciones de usuario (hacer llamada, transferir) van al HTTP Bridge → AMI → Asterisk
5. El estado se persiste en Redis y MySQL según el tipo de dato

## Multi-tenancy

Cada empresa tiene:
- Sus propias extensiones (aisladas por `companyId`)
- Colas y agentes propios
- DIDs asignados
- Reglas de enrutamiento independientes
- Acceso de administrador segregado
