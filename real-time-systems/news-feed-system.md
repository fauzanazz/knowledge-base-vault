---
title: "News Feed System"
category: system-design
summary: "A news feed system displays constantly updating posts from user connections in reverse chronological order. It requires careful design of feed publishing and retrieval flows to handle millions of users efficiently."
sources:
  - raw/articles/_done/news-feed-system-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T10:22:59.664Z
---

# News Feed System

> A news feed system displays constantly updating posts from user connections in reverse chronological order. It requires careful design of feed publishing and retrieval flows to handle millions of users efficiently.

# News Feed System

A news feed system displays a constantly updating list of posts (status updates, photos, videos, and links) from a user's connections. Examples include Facebook's news feed, Instagram's feed, and Twitter's timeline.

## Core Requirements

- Support both web and mobile platforms
- Users can publish posts and view friends' posts
- Posts sorted in reverse chronological order
- Handle up to 5,000 friends per user
- Support 10 million daily active users
- Include text, images, and videos

## System Architecture

The system consists of two main flows:

### Feed Publishing Flow
1. User publishes via `POST /v1/me/feed` API
2. [[Load Balancer]] distributes traffic to web servers
3. Web servers authenticate and enforce rate limits
4. Post service stores content in database and cache
5. Fanout service propagates to friends' feeds
6. Notification service alerts friends

### News Feed Retrieval Flow
1. User requests feed via `GET /v1/me/feed` API
2. News feed service fetches post IDs from cache
3. Complete post details retrieved from database or cache

## Fanout Strategies

**Fanout on Write (Push Model):**
- Pros: Real-time updates, fast retrieval
- Cons: Resource-intensive for high-connection users

**Fanout on Read (Pull Model):**
- Pros: Efficient for inactive users
- Cons: Slower feed retrieval

**Hybrid Approach:** Push model for regular users, pull model for celebrities with millions of followers.

## Cache Architecture

Five-layer [[Caching]] system:
- News Feed Cache: Stores post IDs
- Content Cache: Stores post details
- Social Graph Cache: User relationships
- Action Cache: User interactions
- Counter Cache: Likes, shares, followers

## Scaling Considerations

- [[Database Sharding]] for horizontal scaling
- [[Database Replication]] for read-heavy workloads
- [[Message Queue]] for decoupling components
- Stateless web tier for [[Horizontal Scaling]]
- Cache optimization for reduced latency

The fanout service uses graph databases to fetch friend lists, filters based on user preferences, and processes updates through message queues to maintain scalability.

---
*Related: [[Load Balancer]], [[Caching]], [[Database Sharding]], [[Message Queue]], [[Horizontal Scaling]], [[Database Replication]]*
