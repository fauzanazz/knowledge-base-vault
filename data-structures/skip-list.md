---
title: "Skip List"
category: data-structures
summary: "A skip list is a probabilistic data structure that maintains sorted data with multiple levels of linked lists for fast search operations. It provides O(logN) search complexity through hierarchical indexing."
sources:
  - raw/articles/real-time-gaming-leaderboard-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:46:58.800Z
---

# Skip List

> A skip list is a probabilistic data structure that maintains sorted data with multiple levels of linked lists for fast search operations. It provides O(logN) search complexity through hierarchical indexing.

# Skip List

A skip list is a probabilistic data structure that allows fast search within sorted data by maintaining multiple levels of linked lists. Each level acts as an "express lane" that skips over elements, dramatically reducing traversal time.

## Structure

A skip list consists of:

1. **Base Level**: Complete sorted linked list containing all elements
2. **Index Levels**: Higher levels with fewer nodes that provide shortcuts
3. **Header Node**: Starting point with pointers to each level

## How It Works

Search operations start at the highest level and move right until finding a value greater than the target, then drop down one level and repeat. This creates a "staircase" pattern that minimizes node visits.

### Performance Example

For a 64-node dataset:
- **Linear search**: 62 node traversals (worst case)
- **Skip list search**: 11 node traversals (average case)

This represents an 82% reduction in traversal operations.

## Time Complexity

- **Search**: O(logN) average, O(N) worst case
- **Insert**: O(logN) average
- **Delete**: O(logN) average
- **Space**: O(N) average

## Probabilistic Nature

Node levels are determined randomly during insertion, typically with a 50% probability of promoting to the next level. This randomization maintains balance without complex rebalancing algorithms.

## Advantages

- Simpler implementation than balanced trees
- Good cache performance due to linear memory layout
- Concurrent access friendly
- No complex rebalancing required

## Use Cases

### Redis Sorted Sets

Redis uses skip lists as the underlying structure for [[Redis Sorted Sets]], enabling efficient leaderboard operations and range queries.

### Database Indexing

Some databases use skip lists for indexing sorted data, particularly in scenarios requiring frequent insertions and range queries.

## Implementation Considerations

Skip lists work best when:
- Data access patterns favor range queries
- Frequent insertions and deletions occur
- Simple implementation is preferred over optimal worst-case performance

The probabilistic nature means performance can vary, but average-case behavior is consistently good for most applications.

---
*Related: [[Redis Sorted Sets]], [[Data Structures]], [[Database Indexing]], [[Real-time Gaming Leaderboard]]*
