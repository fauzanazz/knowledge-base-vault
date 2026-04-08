---
title: "Real-time Gaming Leaderboard"
category: system-design
summary: "A real-time gaming leaderboard system displays top players and individual rankings with immediate score updates. It requires careful consideration of data storage, scalability, and performance optimization."
sources:
  - raw/articles/real-time-gaming-leaderboard-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:46:58.797Z
---

# Real-time Gaming Leaderboard

> A real-time gaming leaderboard system displays top players and individual rankings with immediate score updates. It requires careful consideration of data storage, scalability, and performance optimization.

# Real-time Gaming Leaderboard

A real-time gaming leaderboard system displays the top players in a game tournament and shows individual player rankings with immediate score updates. The system must handle millions of users while maintaining low latency for both score updates and leaderboard queries.

## Functional Requirements

- Display top 10 players on leaderboard
- Show a user's specific rank
- Real-time score updates reflected immediately
- Support for monthly tournaments with new leaderboards

## System Architecture

The high-level architecture consists of:

1. **Game Service**: Validates game wins and calls the leaderboard service
2. **Leaderboard Service**: Updates player scores and serves leaderboard data
3. **Data Storage**: Stores leaderboard data and user information

Security is critical - clients should not directly update scores to prevent cheating through man-in-the-middle attacks.

## Data Storage Solutions

### Redis Sorted Sets

Redis sorted sets are the preferred solution for real-time leaderboards. They maintain data in sorted order using a combination of hash maps and skip lists, enabling O(logN) operations for updates and queries.

Key Redis operations:
- `ZINCRBY`: Increment user score
- `ZREVRANGE`: Fetch top players
- `ZREVRANK`: Get user's rank

For 25 million monthly active users, storage requirements are approximately 650MB, easily fitting in a modern Redis cluster.

### Relational Database Alternative

For smaller scales, relational databases work well with indexed score columns. However, they don't scale efficiently for rank calculations across millions of users.

## Scaling Considerations

For larger user bases (500M+ DAU), [[Database Sharding]] becomes necessary. Two approaches:

1. **Range Partitioning**: Shard by score ranges, enabling efficient top-K queries
2. **Hash Partitioning**: Use Redis Cluster for automatic distribution

Range partitioning is preferred as it simplifies top player queries and rank calculations.

## Alternative: NoSQL Solutions

[[NoSQL]] databases like DynamoDB can handle heavy writes and provide good scalability. However, calculating exact user ranks becomes challenging, often requiring approximations like percentile rankings.

## Performance Optimization

- Use [[Caching]] for frequently accessed user profiles
- Implement Redis replicas for high availability
- Consider serverless architectures (AWS Lambda) for automatic scaling
- Cache top 10 player details for faster retrieval

The system can handle 2,500 score updates per second on a single Redis instance, with [[Horizontal Scaling]] available for higher loads.

---
*Related: [[Redis]], [[Database Sharding]], [[Caching]], [[System Scaling]], [[NoSQL]]*
