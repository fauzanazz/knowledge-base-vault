---
title: "File Conflict Resolution"
category: system-design
summary: "File conflict resolution handles situations where multiple users simultaneously modify the same file in distributed storage systems. It ensures data integrity while providing users with options to resolve conflicts."
sources:
  - raw/articles/_done/google-drive-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:54:34.693Z
---

# File Conflict Resolution

> File conflict resolution handles situations where multiple users simultaneously modify the same file in distributed storage systems. It ensures data integrity while providing users with options to resolve conflicts.

# File Conflict Resolution

File conflict resolution manages situations where multiple users simultaneously edit the same file in distributed storage systems, ensuring data integrity while preserving user work.

## When Conflicts Occur

Conflicts happen when:
- Two or more users modify the same file simultaneously
- Network partitions cause delayed synchronization
- Offline edits overlap with online changes
- Race conditions in distributed file operations

## Resolution Strategies

### First-Writer-Wins
The most common approach used in systems like [[Google Drive System Design]]:

1. **Processing Order**: First processed modification is accepted
2. **Conflict Detection**: Subsequent changes trigger conflict state
3. **Version Preservation**: Both versions are saved separately
4. **User Choice**: Conflicted user chooses resolution method

### Last-Writer-Wins
- Simpler implementation but risks data loss
- Suitable for systems where conflicts are rare
- Used in some [[Caching]] systems and databases

### Operational Transformation
- Real-time collaborative editing approach
- Transforms operations to maintain consistency
- Complex implementation but seamless user experience
- Used in Google Docs and similar collaborative tools

## Implementation Details

### Conflict Detection
- **Version Numbers**: Track file versions using timestamps or sequence numbers
- **Hash Comparison**: Compare file or block hashes to detect changes
- **[[Optimistic Locking]]**: Allow concurrent edits with conflict detection

### User Resolution Options
- **Manual Merge**: User manually combines both versions
- **Choose Version**: Select either local or server version
- **Side-by-Side Comparison**: Visual diff tools for informed decisions
- **Automatic Merge**: System attempts intelligent merging for compatible changes

## Best Practices

- **Clear Notifications**: Inform users immediately when conflicts occur
- **Preserve All Data**: Never automatically discard user work
- **Intuitive UI**: Make resolution process simple and understandable
- **Backup Conflicts**: Store conflicted versions for future reference

Effective conflict resolution balances system performance with user experience, ensuring no data loss while minimizing user friction in collaborative environments.

---
*Related: [[Google Drive System Design]], [[Optimistic Locking]], [[Pessimistic Locking]], [[Database Replication]], [[Notification System]]*
