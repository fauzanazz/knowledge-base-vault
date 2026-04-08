---
title: "Fanout Service"
category: system-design
summary: "A fanout service propagates user posts to their friends' news feeds using push, pull, or hybrid strategies. It manages the distribution of content updates across social networks efficiently."
sources:
  - raw/articles/_done/news-feed-system-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T10:22:59.665Z
---

# Fanout Service

> A fanout service propagates user posts to their friends' news feeds using push, pull, or hybrid strategies. It manages the distribution of content updates across social networks efficiently.

# Fanout Service

A fanout service is responsible for propagating user posts to their friends' news feeds in social media systems. It determines how and when content updates are distributed across the network.

## Fanout Strategies

### Fanout on Write (Push Model)
Posts are immediately pushed to all friends' feeds when published.

**Advantages:**
- Real-time updates for friends
- Fast news feed retrieval
- Consistent user experience

**Disadvantages:**
- Resource-intensive for users with many connections
- High write amplification
- Storage overhead for inactive users

### Fanout on Read (Pull Model)
Posts are fetched from friends when users request their feed.

**Advantages:**
- Efficient for users with many followers
- No wasted resources on inactive users
- Lower storage requirements

**Disadvantages:**
- Slower feed generation
- Higher read latency
- Complex aggregation logic

### Hybrid Approach
Combines both strategies based on user characteristics:
- Push model for regular users (< 1000 friends)
- Pull model for celebrities and high-connection users
- Optimizes resource usage while maintaining performance

## Implementation Process

1. **Fetch Friend IDs:** Retrieve friend list from graph database
2. **Filter Friends:** Apply user preferences (muted friends, privacy settings)
3. **Queue Processing:** Send filtered list and post ID to [[Message Queue]]
4. **Worker Processing:** Fanout workers update news feed cache
5. **Cache Storage:** Store `<post_id, user_id>` mappings in [[Caching]] layer

## Optimization Techniques

- **Memory Efficiency:** Store post IDs instead of full objects
- **Configurable Limits:** Maintain only recent posts in cache
- **Batch Processing:** Group updates for efficiency
- **Priority Queues:** Handle celebrity posts differently

The fanout service is critical for [[News Feed System]] performance and scales through careful strategy selection and efficient caching mechanisms.

---
*Related: [[News Feed System]], [[Message Queue]], [[Caching]], [[Database Sharding]], [[System Scaling]]*
