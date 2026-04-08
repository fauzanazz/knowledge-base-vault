---
title: "Distributed Email Service"
category: system-design
summary: "A distributed email service is a scalable system designed to handle billions of users sending and receiving emails, similar to Gmail or Outlook. It uses distributed architecture with multiple components including web servers, metadata databases, and real-time communication systems."
sources:
  - raw/articles/_done/distributed-email-service-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:52:10.259Z
---

# Distributed Email Service

> A distributed email service is a scalable system designed to handle billions of users sending and receiving emails, similar to Gmail or Outlook. It uses distributed architecture with multiple components including web servers, metadata databases, and real-time communication systems.

# Distributed Email Service

A distributed email service is designed to handle massive scale email operations for billions of users, similar to Gmail (1.8 billion users) or Outlook (400 million users).

## Key Components

- **Web servers**: Handle user requests, authentication, and basic email validation
- **Real-time servers**: Push new email updates using WebSockets or long-polling
- **Metadata database**: Stores email headers, body, and metadata with strong consistency
- **Attachment store**: Object storage (like S3) for large file attachments
- **[[Message Queue]]**: Handles asynchronous email processing and delivery
- **Search store**: Distributed document store for full-text email search
- **[[Caching]]**: Redis cache for recent emails to improve performance

## Email Flow

**Sending**: User composes email → [[Load Balancer]] → Web server validation → Message queue → SMTP workers → Destination server

**Receiving**: SMTP server → Mail processing workers → Storage/cache/real-time servers → User retrieval

## Database Design

Email metadata requires special characteristics:
- Headers are small and frequently accessed
- Body size varies but typically read once
- Operations are user-isolated
- Recent data is accessed more frequently
- High reliability requirements (no data loss)

Partitioning uses `user_id` as partition key to keep user data on single shards. Tables include folders, emails, and attachments with denormalized read/unread email tables for efficient filtering.

## Scalability Considerations

- **Email deliverability**: Requires dedicated IPs, reputation management, and authentication protocols
- **Search functionality**: Uses Elasticsearch or custom LSM-tree based solutions
- **[[Multi-Data Center Setup]]**: Leader-follower failover for high availability
- **Performance**: Handles 100k emails per second with petabytes of annual storage

The system prioritizes consistency over availability for critical email operations while maintaining horizontal scalability across all components.

---
*Related: [[Message Queue]], [[Load Balancer]], [[Caching]], [[Multi-Data Center Setup]], [[Database Sharding]]*
