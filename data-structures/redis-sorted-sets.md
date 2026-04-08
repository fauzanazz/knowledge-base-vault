---
title: "Redis Sorted Sets"
category: database
summary: "Redis sorted sets are data structures that maintain elements in sorted order using hash maps and skip lists. They provide O(logN) performance for insertions, updates, and range queries."
sources:
  - raw/articles/real-time-gaming-leaderboard-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:46:58.799Z
---

# Redis Sorted Sets

> Redis sorted sets are data structures that maintain elements in sorted order using hash maps and skip lists. They provide O(logN) performance for insertions, updates, and range queries.

# Redis Sorted Sets

Redis sorted sets are specialized data structures that maintain a collection of unique elements in sorted order based on associated scores. They combine the benefits of hash maps for fast lookups with skip lists for efficient range operations.

## Internal Structure

Sorted sets use two underlying data structures:

1. **Hash Map**: Maps elements (keys) to their scores for O(1) score lookups
2. **Skip List**: Maintains elements in sorted order by score for efficient range operations

## Skip List Performance

A skip list is a probabilistic data structure that allows fast search in sorted data. It consists of multiple levels of linked lists, where higher levels act as "express lanes" for faster traversal.

For a dataset with 64 nodes:
- Basic linked list: 62 node traversals to find a value
- Skip list: Only 11 node traversals needed

This structure enables O(logN) complexity for most operations even with large datasets.

## Key Operations

- **ZADD**: Insert element with score, O(logN)
- **ZINCRBY**: Increment element's score, O(logN)
- **ZRANGE/ZREVRANGE**: Get elements by rank range, O(logN + M) where M is result size
- **ZRANK/ZREVRANK**: Get element's rank, O(logN)
- **ZREM**: Remove element, O(logN)

## Use Cases

### Gaming Leaderboards

```redis
ZINCRBY leaderboard_feb_2021 1 'player123'
ZREVRANGE leaderboard_feb_2021 0 9 WITHSCORES
```

### Real-time Analytics

Sorted sets excel at maintaining top-K lists, time-series data, and priority queues where elements need constant reordering.

## Memory Considerations

For large datasets, sorted sets require approximately double the memory of the raw data due to skip list overhead. However, they remain memory-efficient compared to maintaining sorted data in traditional databases.

## Scaling

When single Redis instances reach capacity limits, sorted sets can be distributed using [[Database Sharding]] strategies, though this complicates cross-shard operations like global rankings.

---
*Related: [[Redis]], [[Real-time Gaming Leaderboard]], [[Database Sharding]], [[Caching]]*
