---
title: "Email Protocols"
category: networking
summary: "Email protocols are standardized communication methods for sending and receiving emails between servers and clients. The main protocols include SMTP for sending, POP and IMAP for receiving, with modern systems also using HTTP for web-based clients."
sources:
  - raw/articles/_done/distributed-email-service-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:52:10.259Z
---

# Email Protocols

> Email protocols are standardized communication methods for sending and receiving emails between servers and clients. The main protocols include SMTP for sending, POP and IMAP for receiving, with modern systems also using HTTP for web-based clients.

# Email Protocols

Email protocols define how email messages are transmitted between servers and clients in distributed email systems.

## Core Protocols

**SMTP (Simple Mail Transfer Protocol)**
- Standard protocol for sending emails between servers
- Used for server-to-server email transfer
- Handles outbound email delivery and routing

**POP (Post Office Protocol)**
- Downloads emails from server to local client
- Emails are typically deleted from server after download
- Suitable for single-device access
- Simpler but less flexible than IMAP

**IMAP (Internet Message Access Protocol)**
- Accesses emails while keeping them on the server
- Supports multiple device synchronization
- Allows server-side email management (folders, flags)
- Better for modern multi-device usage

**HTTP/HTTPS**
- Used by web-based email clients
- Enables rich user interfaces and real-time features
- RESTful APIs for email operations
- Supports modern web technologies like WebSockets

## DNS Configuration

**MX Records**: DNS records that specify mail servers responsible for receiving emails for a domain. Essential for email routing between different email providers.

## Modern Implementations

Modern [[Distributed Email Service]] systems often use HTTP APIs for web clients while maintaining SMTP/IMAP support for traditional email clients. This hybrid approach provides flexibility for different client types while enabling advanced features like real-time notifications and rich web interfaces.

**Email Authentication**: Protocols like SPF, DKIM, and DMARC help prevent spam and phishing by verifying sender authenticity.

---
*Related: [[Distributed Email Service]], [[DNS]], [[Web APIs]]*
