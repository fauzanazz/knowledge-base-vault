---
title: "WebSocket Architecture"
category: fullstack-patterns
summary: "Persistent bidirectional connections enabling real-time communication, with production patterns for connection management, horizontal scaling, and choosing between Socket.io and native WebSockets."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# WebSocket Architecture

> Persistent bidirectional connections enabling real-time communication, with production patterns for connection management, horizontal scaling, and choosing between Socket.io and native WebSockets.

## What It Is

WebSockets provide full-duplex communication over a single TCP connection, upgraded from HTTP. Unlike HTTP request-response, the server can push data to clients at any time. Essential for: live dashboards, collaborative editing, chat, live notifications, multiplayer games.

## Native WebSocket vs. Socket.io

### Native WebSocket (Browser API)
```typescript
// Server (Node.js ws library)
import { WebSocketServer } from 'ws';

const wss = new WebSocketServer({ port: 8080 });

wss.on('connection', (ws, req) => {
  const clientId = req.headers['sec-websocket-key'];
  
  ws.on('message', (data) => {
    const message = JSON.parse(data.toString());
    broadcast(message, ws);  // send to all others
  });
  
  ws.on('close', () => cleanup(clientId));
  ws.on('error', (err) => handleError(err, clientId));
});

function broadcast(message: unknown, sender: WebSocket) {
  wss.clients.forEach(client => {
    if (client !== sender && client.readyState === WebSocket.OPEN) {
      client.send(JSON.stringify(message));
    }
  });
}
```

```typescript
// Client (browser native)
const ws = new WebSocket('wss://api.example.com/ws');

ws.onopen = () => ws.send(JSON.stringify({ type: 'join', room: 'general' }));
ws.onmessage = (event) => handleMessage(JSON.parse(event.data));
ws.onclose = () => scheduleReconnect();
ws.onerror = (err) => console.error('WS error', err);
```

### Socket.io
Socket.io adds: rooms, namespaces, auto-reconnection, event acknowledgements, broadcasting, and fallback to long-polling. Worth the overhead for complex applications.

```typescript
// Socket.io server
import { Server } from 'socket.io';

const io = new Server(httpServer, {
  cors: { origin: process.env.CLIENT_URL },
  transports: ['websocket', 'polling']  // polling fallback for proxies
});

io.on('connection', (socket) => {
  socket.on('join-room', (roomId: string) => {
    socket.join(roomId);
    socket.to(roomId).emit('user-joined', { id: socket.id });
  });

  socket.on('send-message', (data, ack) => {
    io.to(data.roomId).emit('new-message', { ...data, from: socket.id });
    ack({ status: 'delivered' });  // acknowledgement pattern
  });
});

// Socket.io client
import { io } from 'socket.io-client';
const socket = io(WS_URL, { autoConnect: true, reconnectionAttempts: 5 });
socket.on('new-message', (msg) => appendToChat(msg));
```

**When to use Socket.io**: rooms/namespaces needed, IE11 support, polling fallback behind restrictive proxies.  
**When to use native**: minimal overhead, custom protocol, services that own the full stack.

## Horizontal Scaling

Single-server WebSocket architecture fails when you need >1 instance. Connections are stateful — client A on server 1 can't receive events from client B on server 2.

### Redis Pub/Sub Adapter (Socket.io)
```typescript
import { createAdapter } from '@socket.io/redis-adapter';
import { createClient } from 'redis';

const pubClient = createClient({ url: process.env.REDIS_URL });
const subClient = pubClient.duplicate();
await Promise.all([pubClient.connect(), subClient.connect()]);

io.adapter(createAdapter(pubClient, subClient));

// Now: io.to('room').emit() works across all server instances
// Redis pub/sub fans out the message to correct servers
```

### Sticky Sessions (Simpler, Less Scalable)
Route each client to the same server instance using consistent hashing on session/user ID:
```nginx
upstream ws_backend {
    ip_hash;  # or use cookie-based stickiness
    server ws1:8080;
    server ws2:8080;
    server ws3:8080;
}
```

**Problem**: uneven load, failed server loses all connections.

### Dedicated WebSocket Service
Architecture pattern for large scale:

```
Client → Load Balancer → WebSocket Gateway (stateful, few instances)
                              ↓ Redis Pub/Sub
                        Message Bus (Kafka/Redis)
                              ↓
                        API Services (stateless, many instances)
```

The WebSocket gateway only handles connection management and message routing. Business logic stays in stateless services.

## Connection Management

### Heartbeat / Ping-Pong
```typescript
// Server-side heartbeat — detect dead connections
const HEARTBEAT_INTERVAL = 30_000;

wss.on('connection', (ws) => {
  let isAlive = true;
  ws.on('pong', () => { isAlive = true; });
  
  const interval = setInterval(() => {
    if (!isAlive) return ws.terminate();
    isAlive = false;
    ws.ping();
  }, HEARTBEAT_INTERVAL);
  
  ws.on('close', () => clearInterval(interval));
});
```

### Client Reconnection with Backoff
```typescript
class ReconnectingWebSocket {
  private retries = 0;
  
  connect() {
    this.ws = new WebSocket(this.url);
    this.ws.onclose = () => this.scheduleReconnect();
  }
  
  scheduleReconnect() {
    const delay = Math.min(1000 * 2 ** this.retries, 30_000); // cap at 30s
    this.retries++;
    setTimeout(() => this.connect(), delay + Math.random() * 1000); // jitter
  }
}
```

### Rate Limiting
```typescript
const rateLimiter = new Map<string, number>();

ws.on('message', (data) => {
  const now = Date.now();
  const lastMsg = rateLimiter.get(clientId) ?? 0;
  if (now - lastMsg < 50) return ws.send(JSON.stringify({ error: 'rate_limited' }));
  rateLimiter.set(clientId, now);
  processMessage(data);
});
```

## Production Capacity Planning

- 1 WebSocket connection ≈ 20–50KB RAM on Node.js
- Node.js single process: ~50,000 connections (OS fd limits)
- `ulimit -n 1000000` — increase file descriptor limits in production
- Connection load: 100K concurrent users = 2–3 Node.js instances minimum

## Alternatives to WebSockets

| Pattern | Use Case | Trade-off |
|---------|----------|-----------|
| **Server-Sent Events (SSE)** | Server → Client only (notifications, logs) | Simpler, HTTP/2 compatible, unidirectional |
| **HTTP Long Polling** | Behind strict firewalls | Higher latency, more overhead |
| **WebRTC Data Channel** | P2P, video/audio | Complex signaling, no server relay |
| **MQTT** | IoT devices | Binary protocol, broker required |

## Trade-offs

| Pro | Con |
|-----|-----|
| True real-time, low latency | Stateful = harder to scale |
| Bidirectional communication | Connection management overhead |
| Lower overhead than polling | Firewall/proxy issues (port 80/443 TLS mitigates) |
| Native browser support | Server resource cost per connection |

## When to Use

✅ Chat, collaborative editing (Figma, Notion-style)  
✅ Live dashboards, trading terminals  
✅ Multiplayer games  
✅ Live notifications where SSE isn't bidirectional enough  
❌ Mostly-read scenarios → use SSE (simpler)  
❌ Infrequent updates (>30s intervals) → use polling  
❌ Serverless deployments without state management

---
*Related: [[Edge Computing]], [[GraphQL Architecture]], [[Observability Stack]]*
