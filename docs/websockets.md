# WebSockets y Comunicación en Tiempo Real

PekePBX mantiene tres servidores de WebSocket/Socket.io para diferentes tipos de clientes.

## Servidor 1 — WebSocket JWT (Puerto 5050)

Para clientes de extensión (softphones, aplicaciones de escritorio).

**Autenticación:** Token JWT en el evento `authenticate`.

### Conexión

```javascript
const ws = new WebSocket('wss://mi-pbx.com:5050');

ws.onopen = () => {
  ws.send(JSON.stringify({
    event: 'authenticate',
    token: 'eyJ...'
  }));
};
```

### Eventos del cliente → servidor

| Evento | Descripción | Payload |
|--------|-------------|---------|
| `authenticate` | Autenticación inicial | `{ token: "eyJ..." }` |
| `ping` | Heartbeat | — |
| `click2call` | Iniciar llamada | `{ destination: "612345678" }` |
| `updateUserStatus` | Cambiar disponibilidad | `{ status: "available" \| "busy" \| "away" }` |

### Eventos servidor → cliente

| Evento | Descripción |
|--------|-------------|
| `pong` | Respuesta a ping |
| `PluginShow` | Mostrar panel de llamada |
| `PluginOpen` | Abrir plugin externo |

---

## Servidor 2 — WebSocket Cookie (Puerto 3002)

Para clientes web (navegador). La autenticación se realiza mediante la sesión Laravel (cookie).

**Autenticación:** Cookie de sesión válida de Laravel.

### Eventos recibidos del servidor

| Evento | Descripción |
|--------|-------------|
| `callStatus` | Actualización de estado de llamada |
| `extensionStatus` | Cambio de estado de extensión |
| `queueUpdate` | Actualización de cola (agentes, llamadas en espera) |
| `faxNotification` | Notificación de FAX recibido/enviado |
| `presenceUpdate` | Cambio de presencia de usuario |

### Ejemplo de uso en cliente web

```javascript
const socket = io('https://mi-pbx.com:3002', {
  withCredentials: true  // Envía cookie de sesión
});

socket.on('callStatus', (data) => {
  console.log('Estado llamada:', data);
});

socket.on('queueUpdate', (data) => {
  console.log('Cola actualizada:', data);
});
```

---

## Servidor 3 — Bridge PHP-Socket.io (Puerto 3003)

Uso interno para la integración de FAX. El código PHP legacy envía eventos de FAX a este servidor, que los enruta al servicio de FAX de Node.js.

**No está destinado a uso externo.**

| Evento | Descripción |
|--------|-------------|
| `sendFax` | Solicitud de envío de FAX desde PHP |

---

## Flujo de eventos AMI → WebSocket

```
Asterisk (evento AMI)
        │
        ▼
AmiManager (Node.js)
        │  Filtra y procesa eventos
        ▼
Event Listeners
        │
        ├──► Extensión conectada vía JWT (puerto 5050) → Notificación directa
        │
        └──► Clientes web vía Cookie (puerto 3002)   → Broadcast
```

### Tipos de eventos AMI procesados

- **Hangup** — Llamada finalizada
- **Dial** — Llamada en progreso
- **Hold / Unhold** — Llamada en espera
- **AgentCalled / AgentConnect** — Agente de cola recibiendo llamada
- **QueueMemberStatus** — Cambio de estado de agente
- **DeviceStateChange** — Cambio de estado BLF
- **VoicemailUserEntry** — Nuevo buzón de voz

---

## Heartbeat y reconexión

- El cliente debe enviar `ping` cada 30 segundos
- Si no hay respuesta `pong` en 10 segundos, reconectar
- El servidor cierra conexiones inactivas tras 60 segundos sin ping
