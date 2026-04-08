---
title: "Write-Ahead Log"
category: data-structures
summary: "Write-Ahead Log (WAL) is a data structure that supports only append operations, providing excellent performance for sequential read/write access patterns commonly used in distributed message queues and databases."
sources:
  - raw/articles/_done/distributed-message-queue-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:56:33.774Z
---

# Write-Ahead Log

> Write-Ahead Log (WAL) is a data structure that supports only append operations, providing excellent performance for sequential read/write access patterns commonly used in distributed message queues and databases.

# Write-Ahead Log

A **Write-Ahead Log (WAL)** is a plain text file data structure that only supports appending operations, making it extremely efficient for sequential read/write access patterns. WAL is fundamental to many distributed systems, particularly [[Message Queue]] implementations and database systems.

## Key Characteristics

- **Append-only**: New data is always written to the end of the file
- **Sequential access**: Optimized for sequential read and write operations
- **Immutable**: Once written, data cannot be modified or deleted
- **HDD-friendly**: Takes advantage of traditional hard disk drive performance characteristics

## Performance Benefits

WAL files achieve exceptional performance with traditional HDDs due to sequential access patterns. While random disk access is slow, sequential access can achieve several MB/s read/write speeds. Modern operating systems aggressively cache disk data in memory, further improving performance.

## Segmentation Strategy

To avoid maintaining extremely large files, WAL implementations typically split data into **segments**:

- **Active segment**: The latest segment that accepts new writes
- **Read-only segments**: Older segments that only support read operations
- **Segment rotation**: When active segments reach size limits, they become read-only and new active segments are created

This segmentation enables efficient data management and supports features like data retention policies.

## Use Cases

### Message Queues

In distributed [[Message Queue]] systems, WAL stores messages in partitions. Each partition uses WAL segments to maintain message ordering and enable efficient sequential access for both producers and consumers.

### Database Systems

Many databases use WAL for transaction logging, ensuring durability and enabling crash recovery. Changes are first written to the WAL before being applied to the main database files.

## Implementation Considerations

- **Batching**: Writing multiple records together amortizes the cost of disk operations
- **Fsync frequency**: Balancing durability guarantees with performance
- **Compression**: Reducing storage requirements while maintaining access speed
- **Retention policies**: Automatically removing old segments based on time or size limits

## Advantages

- **High throughput**: Sequential writes are much faster than random writes
- **Simple implementation**: Straightforward append-only semantics
- **Crash recovery**: Easy to replay operations from the log
- **OS optimization**: Benefits from operating system disk caching strategies

## Limitations

- **No random updates**: Cannot modify existing records efficiently
- **Storage growth**: Files grow continuously without compaction
- **Read performance**: Reading old data may require scanning multiple segments

WAL is particularly effective in write-heavy systems with predominantly sequential access patterns, making it an ideal choice for [[Message Queue]] implementations and transaction logging systems.

---
*Related: [[Message Queue]], [[Database Replication]], [[Log-Structured Merge Trees]], [[Caching]]*
