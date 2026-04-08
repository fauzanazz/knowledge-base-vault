---
title: "Online Presence System"
category: system-design
summary: "An online presence system tracks and displays user availability status in real-time applications. It uses heartbeat mechanisms to detect user activity and fanout models to efficiently distribute presence updates to connected users."
sources:
  - raw/articles/chat-system-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:03:31.112Z
---

# Online Presence System

> An online presence system tracks and displays user availability status in real-time applications. It uses heartbeat mechanisms to detect user activity and fanout models to efficiently distribute presence updates to connected users.

# Online Presence System

An online presence system tracks and displays user availability status (online, offline, away) in real-time applications like chat systems, social networks, and collaboration tools. It provides immediate feedback about user availability to enhance user experience and communication efficiency.

## Core Components

**Presence Detection:**
- Monitors user activity and connection status
- Determines online/offline/away states
- Handles multiple device scenarios

**Status Distribution:**
- Propagates presence changes to relevant users
- Manages subscription relationships
- Optimizes update delivery for scale

## Heartbeat Mechanism

The heartbeat approach is the most common method for presence detection:

1. **Client heartbeats**: Clients send periodic signals (every 30 seconds)
2. **Server monitoring**: Presence servers track last heartbeat timestamp
3. **Timeout detection**: Users marked offline if no heartbeat within threshold
4. **Status updates**: Presence changes trigger notification events

**Advantages:**
- Simple implementation and debugging
- Reliable detection of disconnected clients
- Configurable timeout thresholds

**Considerations:**
- Network overhead from periodic messages
- Battery impact on mobile devices
- Delayed offline detection (up to timeout period)

## Fanout Models

Presence updates use fanout patterns to distribute status changes:

**Push Model (Fanout on Write):**
- Presence changes immediately pushed to all friends
- Uses publish-subscribe channels for each friend pair
- Effective for small friend networks
- Higher write load but faster read access

**Pull Model (Fanout on Read):**
- Clients fetch presence data when needed
- Reduces server push overhead
- Higher read latency but lower write load

## Scalability Challenges

Large-scale presence systems face several challenges:

- **Friend graph size**: Users with many connections create hotspots
- **Update frequency**: High-activity users generate many status changes
- **Storage requirements**: Maintaining presence state for millions of users
- **Network bandwidth**: Distributing updates across global user base

## Integration with Chat Systems

In [[Chat System]] architectures, presence systems:
- Display online indicators next to user names
- Influence message delivery strategies (push vs. notification)
- Support "last seen" timestamps for offline users
- Enable features like "typing indicators" and "read receipts"

Online presence systems are essential for creating engaging, responsive real-time applications that keep users informed about availability and activity status.

---
*Related: [[Real-time Systems]], [[Publish-Subscribe Pattern]], [[Heartbeat Protocol]], [[User Status Tracking]]*
