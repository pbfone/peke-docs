# Módulos y Funcionalidades

## Extensiones

Gestión de extensiones SIP asociadas a usuarios dentro de una empresa.

**Campos principales:**
- Número de extensión
- Nombre del usuario
- Empresa asociada (`companyId`)
- Contraseña SIP
- Buzón de voz activado/desactivado
- Grabación de llamadas
- Permisos de marcación (local, nacional, internacional)

**Integración:** Cada extensión se provisiona automáticamente en Asterisk al crearse o modificarse.

---

## Colas (Call Center)

Sistema de distribución de llamadas entrantes entre agentes.

**Características:**
- Estrategias de distribución: ringall, leastrecent, fewestcalls, random, rrmemory
- Música en espera configurable
- Timeout y fallback a otro destino
- Prioridad de llamadas
- Pausa/despausa de agentes en tiempo real
- Estadísticas en vivo: llamadas en espera, tiempo de espera, agentes disponibles

**Estados de agente:**
- `available` — Disponible
- `paused` — En pausa (con motivo opcional)
- `busy` — En llamada
- `offline` — Desconectado

---

## DIDs (Números directos entrantes)

Gestión de números de teléfono que enrutan llamadas entrantes al sistema.

**Destinos de enrutamiento disponibles:**
- Extensión
- Cola
- IVR / Menú
- Buzón de voz
- Conferencia
- Otro DID
- Horario (con reglas de tiempo)

---

## IVR / Menú de Bienvenida

Configuración de menús interactivos de voz:
- Locución de bienvenida (fichero de audio o text-to-speech)
- Opciones de teclado (0-9, *, #) con destino configurable
- Timeout y opción por defecto

---

## CDR (Registros de llamadas)

Registro completo de todas las llamadas del sistema.

**Datos registrados por llamada:**
- Fecha y hora de inicio/fin
- Extensión origen y destino
- Duración total y duración de conversación
- Dirección (entrante/saliente)
- Estado (respondida, no respondida, ocupado)
- Grabación asociada (si aplica)
- Cola y agente (si aplica)

**Retención:** 12 meses (configurable en `pekepbx.conf`)

---

## Grabación de llamadas

- Activable por extensión, cola o DID
- Grabaciones almacenadas en el servidor
- Accesibles desde el CDR
- Transcripción opcional (requiere servicio externo)

---

## Buzón de voz

- Activable por extensión
- Notificación por email con fichero de audio adjunto
- Gestión desde el panel web (escuchar, eliminar)
- Acceso por teléfono marcando el prefijo configurado

---

## FAX

Integración con HylaFAX para envío y recepción de faxes.

**Funcionalidades:**
- Recepción de fax → conversión a PDF → entrega por email (mail2fax)
- Envío de fax desde el panel web
- Estado de envío en tiempo real via WebSocket

**Componentes involucrados:**
- `HylaFAX` — Servidor de fax
- `socketPhp` — Bridge PHP → Node.js para eventos de envío
- `FAX Service` en Node.js — Gestión de colas de envío

---

## Conferencias

Salas de conferencia multi-participante.

- Salas con PIN de acceso
- Activación/desactivación de participantes
- Integración con Jitsi Meet para videoconferencia

---

## BLF (Busy Lamp Field)

Estado en tiempo real de extensiones para teléfonos físicos.

- El estado se almacena en Redis
- Actualizado via eventos AMI (`DeviceStateChange`)
- Consumido por teléfonos SIP compatibles mediante NOTIFY SIP

---

## Listas negras

Bloqueo de llamadas entrantes o salientes por número.

- Bloqueo por número exacto o prefijo
- Aplicable a nivel de empresa

---

## Métricas y Dashboard

Vista general del sistema en tiempo real:
- Llamadas activas
- Agentes conectados por cola
- Estadísticas del día (llamadas respondidas, abandonadas, tiempo medio)
- Histórico exportable

---

## Alias de teléfono

Asignación de nombres o identificadores alternativos a extensiones o números externos. Útil para el directorio corporativo.

---

## Reglas de horario

Enrutamiento de llamadas basado en franjas horarias y días de la semana:
- Horario laboral → Cola de atención
- Fuera de horario → Buzón de voz o locución
- Festivos configurables

---

## Autenticación LDAP

Integración opcional con directorio LDAP/Active Directory para autenticación de usuarios del panel de administración.
