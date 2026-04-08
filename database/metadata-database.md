---
title: "Metadata Database"
category: database
summary: "A metadata database stores structured information about files, users, and system objects in distributed storage systems. It tracks relationships, permissions, and versioning information separate from the actual file content."
sources:
  - raw/articles/google-drive-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-08T19:07:44.992Z
---

# Metadata Database

> A metadata database stores structured information about files, users, and system objects in distributed storage systems. It tracks relationships, permissions, and versioning information separate from the actual file content.

# Metadata Database

A metadata database stores structured information about files, users, and system objects in distributed storage systems like [[Google Drive System Design]], separate from the actual file content stored in [[Block Storage Systems]].

## Schema Design

Typical metadata database includes core tables:

### User Table
- User profiles and authentication information
- Storage quotas and preferences
- Account settings and permissions

### File Table
- File metadata: name, size, path, creation/modification timestamps
- File status: pending, uploaded, deleted
- Owner and permission information
- Parent directory relationships

### Block Table
- Block identifiers and hash values
- File-to-block mappings for reconstruction
- Block locations in distributed storage
- Compression and encryption metadata

### File Version Table
- Revision history for each file
- Version timestamps and user attribution
- Delta information between versions
- Rollback and recovery data

## Performance Optimization

### Caching Strategy
- Frequently accessed metadata cached in memory
- User session data and recent file information prioritized
- Cache invalidation on file updates
- Reduces database load for common operations

### Database Scaling
- **[[Database Sharding]]**: Partition by user_id for horizontal scaling
- **[[Database Replication]]**: Master-slave setup for read scaling
- **Indexing**: Optimize queries on file paths, user ownership, and timestamps

## Consistency Requirements

### ACID Properties
Metadata operations require strong consistency:
- **Atomicity**: File creation includes all related metadata
- **Consistency**: Referential integrity between files and blocks
- **Isolation**: Concurrent updates don't interfere
- **Durability**: Metadata changes are permanently stored

### Synchronization
Metadata updates must coordinate with file operations:
- File upload sets status to "pending" until completion
- S3 callbacks trigger status updates to "uploaded"
- Notification service informed of metadata changes

The metadata database serves as the authoritative source for file system state, enabling efficient queries while maintaining consistency across distributed storage infrastructure.

---
*Related: [[Google Drive System Design]], [[Database Sharding]], [[Database Replication]], [[Block Storage Systems]], [[ACID Properties]]*
