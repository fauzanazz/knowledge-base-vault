---
title: "Chat System"
category: system-design
summary: "A chat system supports real-time messaging between users through one-on-one and group conversations. It requires careful design for scalability, real-time communication protocols, and message synchronization across multiple devices."
sources:
  - raw/articles/_done/chat-system-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:55:18.200Z
---

# Chat System

> A chat system supports real-time messaging between users through one-on-one and group conversations. It requires careful design for scalability, real-time communication protocols, and message synchronization across multiple devices.

# Chat System

A chat system enables real-time messaging between users, supporting both one-on-one conversations and group chats. Modern chat systems must handle millions of daily active users while maintaining low latency and high availability.

## Core Features

- **One-on-one and group chat** (typically limited to 100 members for performance)
- **Real-time messaging** with instant delivery
- **Online presence indicators** showing user availability
- **Multiple device support** with message synchronization
- **Push notifications** for offline users
- **Persistent chat history** storage

## Communication Protocols

Chat systems use different protocols for sending and receiving messages:

- **HTTP for sending**: Leverages persistent connections for efficiency
- **WebSocket for receiving**: Provides bi-directional, persistent connections for real-time communication
- **Alternative approaches**: Polling and long polling are less efficient due to frequent requests or resource waste

## System Architecture

The architecture consists of both stateless and stateful services:

**Stateless Services:**
- Handle user authentication, profiles, and management
- Integrate with service discovery for optimal chat server allocation

**Stateful Services:**
- Chat servers maintain persistent WebSocket connections
- Manage message delivery and synchronization
- [[Presence servers]] track online/offline status

**Storage:**
- [[Key-value stores]] for chat history due to horizontal scaling capabilities and low latency
- Better than relational databases for handling large indexes and random access patterns

## Message Flow

**One-on-One Chat:**
1. Sender transmits message to chat server
2. Server assigns unique message ID and stores in database
3. Message forwarded to recipient's chat server if online
4. [[Push notifications]] sent if recipient offline

**Group Chat:**
- Messages copied to individual inboxes for each group member
- Simplifies synchronization but increases storage costs
- Each recipient maintains a message sync queue

## Message Synchronization

Devices track `cur_max_message_id` to identify new messages. Messages are considered new when:
- Recipient ID matches logged-in user
- Message ID exceeds device's current maximum

## Online Presence

**Heartbeat Mechanism:**
- Clients send periodic heartbeats to presence servers
- Users marked offline after missing heartbeat threshold (typically 30 seconds)

**Fanout Model:**
- Presence updates distributed via publish-subscribe channels
- Each friend pair maintains dedicated channel for status updates
- Effective for small user groups but requires optimization for larger networks

## Scalability Considerations

- **[[Service Discovery]]**: Uses tools like Apache Zookeeper to allocate optimal chat servers based on geography and capacity
- **[[Horizontal Scaling]]**: Add servers as user base grows
- **[[Load Balancer]]**: Distribute traffic evenly across chat servers
- **[[Caching]]**: Reduce database load and improve response times

Chat systems like Facebook Messenger and Discord demonstrate the effectiveness of key-value storage and WebSocket-based architectures for real-time communication at scale.

---
*Related: [[WebSocket]], [[Real-time Communication]], [[Message Queue]], [[Presence System]], [[Push Notifications]]*
