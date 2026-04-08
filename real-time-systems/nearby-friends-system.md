---
title: "Nearby Friends System"
category: system-design
summary: "A nearby friends system enables users to share their location and discover friends within a configurable radius. Unlike proximity services for static locations, this system handles constantly changing user locations with real-time updates."
sources:
  - raw/articles/nearby-friends-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:38:12.596Z
---

# Nearby Friends System

> A nearby friends system enables users to share their location and discover friends within a configurable radius. Unlike proximity services for static locations, this system handles constantly changing user locations with real-time updates.

# Nearby Friends System

A nearby friends system is a location-based service that allows users to share their current location and discover friends who are nearby. The key challenge is handling constantly changing user locations, unlike [[Proximity Service]] systems that deal with mostly static business locations.

## System Requirements

**Functional Requirements:**
- Display nearby friends within a configurable radius (typically 5 miles)
- Show distance and timestamp for each friend's location
- Update friend locations every few seconds
- Store location history for analytics

**Non-functional Requirements:**
- **Low latency**: Real-time location updates without significant delay
- **Reliability**: General system availability with acceptable occasional data loss
- **Eventual consistency**: Location data doesn't require strong consistency

## Architecture Components

The system uses a shared backend as a fan-out mechanism:

- **[[Load Balancer]]**: Distributes traffic across API and [[WebSocket]] servers
- **REST API servers**: Handle auxiliary tasks like friend management and profiles
- **WebSocket servers**: Stateful servers managing real-time location updates and client connections
- **Redis location cache**: Stores current location data with TTL for active users
- **Location history database**: Persistent storage for historical location data (often Cassandra)
- **Redis pub/sub**: Lightweight message bus with user-specific channels for location updates

## Location Update Flow

1. Mobile client sends location update via WebSocket connection
2. WebSocket server saves to location history database
3. Location cache is updated with new coordinates
4. Server publishes update to user's Redis pub/sub channel
5. Subscribed servers receive update and forward to relevant friends
6. Distance calculations determine which friends receive the update

## Scaling Considerations

**Redis Pub/Sub Scaling**: With millions of users, the system requires distributed Redis clusters managed by [[Service Discovery]] components like Zookeeper. [[Consistent Hashing]] minimizes channel redistribution during scaling operations.

**WebSocket Server Scaling**: Servers can be scaled horizontally with graceful connection draining during deployments.

**Database Scaling**: User data can be sharded by user_id, while location cache sharding handles high write loads.

## Advanced Features

**Nearby Random People**: Uses geohash-based pub/sub channels where users subscribe to location-based channels rather than friend-specific ones.

**Friend Management**: Real-time subscription/unsubscription to friend channels when relationships change.

The system successfully handles the unique challenges of real-time location sharing at scale while maintaining low latency and high availability.

---
*Related: [[Proximity Service]], [[WebSocket]], [[Redis Pub/Sub]], [[Geospatial Indexing]], [[Real-time Systems]]*
