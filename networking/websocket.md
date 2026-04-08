---
title: "WebSocket"
category: networking
summary: "WebSocket is a communication protocol that provides full-duplex, persistent connections between clients and servers. It enables real-time, bi-directional communication essential for applications like chat systems and live updates."
sources:
  - raw/articles/chat-system-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-08T19:08:40.643Z
---

# WebSocket

> WebSocket is a communication protocol that provides full-duplex, persistent connections between clients and servers. It enables real-time, bi-directional communication essential for applications like chat systems and live updates.

# WebSocket

WebSocket is a communication protocol that establishes persistent, full-duplex connections between clients and servers over a single TCP connection. Unlike traditional HTTP request-response patterns, WebSocket enables real-time, bi-directional communication.

## Protocol Characteristics

- **Persistent Connection**: Maintains an open connection after the initial handshake
- **Full-Duplex**: Both client and server can send messages simultaneously
- **Low Overhead**: Minimal protocol overhead compared to HTTP polling
- **Real-Time**: Enables instant message delivery without polling delays

## Connection Establishment

WebSocket connections begin with an HTTP upgrade request:

1. Client sends HTTP request with `Upgrade: websocket` header
2. Server responds with `101 Switching Protocols` status
3. Connection upgrades from HTTP to WebSocket protocol
4. Both parties can now send messages freely

## Use Cases

WebSocket is ideal for applications requiring real-time communication:

- **[[Chat System]]**: Instant messaging and group conversations
- **Live Updates**: Stock prices, sports scores, news feeds
- **Gaming**: Real-time multiplayer interactions
- **Collaboration Tools**: Document editing, video conferencing
- **IoT Applications**: Sensor data streaming and device control

## Advantages Over Alternatives

**vs. HTTP Polling**:
- Eliminates constant server requests
- Reduces bandwidth and server load
- Provides true real-time communication

**vs. Long Polling**:
- No connection timeouts or reconnection overhead
- Better resource utilization
- Simpler client-side implementation

## Implementation Considerations

- **Connection Management**: Handle connection drops and reconnection logic
- **[[Load Balancer]]**: Requires sticky sessions or connection-aware routing
- **Scaling**: Stateful connections complicate [[Horizontal Scaling]]
- **Security**: Implement proper authentication and message validation

## Protocol Limitations

- **Firewall Issues**: Some corporate firewalls block WebSocket connections
- **Proxy Compatibility**: Older proxies may not support the protocol
- **Resource Usage**: Persistent connections consume server memory
- **Complexity**: Requires careful error handling and connection state management

WebSocket has become the standard for real-time web applications, providing the foundation for modern interactive experiences that demand instant communication between clients and servers.

---
*Related: [[Chat System]], [[Real-Time Communication]], [[HTTP]], [[Load Balancer]], [[Horizontal Scaling]]*
