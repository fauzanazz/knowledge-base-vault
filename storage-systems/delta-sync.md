---
title: "Delta Sync"
category: system-design
summary: "Delta sync is an optimization technique that transfers only the modified portions of files instead of entire files. It significantly reduces bandwidth usage and improves sync performance in distributed file systems."
sources:
  - raw/articles/_done/google-drive-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:54:34.692Z
---

# Delta Sync

> Delta sync is an optimization technique that transfers only the modified portions of files instead of entire files. It significantly reduces bandwidth usage and improves sync performance in distributed file systems.

# Delta Sync

Delta sync is a file synchronization optimization that transfers only the changed portions (deltas) of files rather than complete files, dramatically reducing bandwidth usage and sync times.

## How It Works

Instead of uploading or downloading entire files when changes occur, delta sync:

1. **Identifies Changes**: Compares file blocks using hash values to detect modifications
2. **Transfers Deltas**: Sends only the modified blocks between client and server
3. **Reconstructs Files**: Combines unchanged blocks with new deltas to rebuild complete files

## Implementation in File Systems

In systems like [[Google Drive System Design]], files are split into fixed-size blocks (typically 4 MB) with unique hash identifiers. When a file is modified:

- Only blocks with changed hash values are transferred
- Unchanged blocks remain in place
- File reconstruction happens by joining blocks in correct order

## Benefits

- **Bandwidth Efficiency**: Reduces data transfer by 80-95% for typical file edits
- **Faster Sync**: Significantly improves sync speed for large files with small changes
- **Cost Reduction**: Lower bandwidth costs for cloud storage providers
- **Better User Experience**: Reduces waiting time for file synchronization

## Use Cases

- **Document Editing**: Text documents with minor changes
- **Code Repositories**: Source code files with incremental updates
- **Large Media Files**: Videos or images with metadata changes
- **Database Backups**: Incremental backup systems

## Challenges

- **Block Size Optimization**: Smaller blocks increase metadata overhead; larger blocks reduce delta efficiency
- **Conflict Resolution**: Handling simultaneous edits to the same file blocks
- **Storage Overhead**: Maintaining block metadata and hash calculations

Delta sync is essential for modern cloud storage systems serving millions of users with limited bandwidth constraints.

---
*Related: [[Google Drive System Design]], [[Blob Storage]], [[Caching]], [[Content Delivery Network]]*
