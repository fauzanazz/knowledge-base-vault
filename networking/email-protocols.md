---
title: "Email Protocols"
category: networking
summary: "Email protocols define the standards for sending and receiving electronic messages between mail servers and clients. The main protocols include SMTP for sending, IMAP and POP for receiving, with modern systems also supporting HTTP-based APIs."
sources:
  - raw/articles/distributed-email-service-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-08T19:05:16.494Z
---

# Email Protocols

> Email protocols define the standards for sending and receiving electronic messages between mail servers and clients. The main protocols include SMTP for sending, IMAP and POP for receiving, with modern systems also supporting HTTP-based APIs.

# Email Protocols

Email protocols are standardized communication methods that enable the exchange of electronic messages between mail servers and email clients. Different protocols serve specific purposes in the email delivery chain.

## Core Protocols

**SMTP (Simple Mail Transfer Protocol)** is the standard protocol for sending emails from one server to another. It handles the transfer of messages between mail servers across the internet.

**POP (Post Office Protocol)** is used for receiving and downloading emails from a remote mail server to a local client. Once emails are retrieved, they are typically deleted from the remote server, making them available only on the downloading device.

**IMAP (Internet Message Access Protocol)** is similar to POP but keeps emails stored on the server-side. This allows users to access their emails from multiple devices while maintaining synchronization.

**HTTPS** is increasingly used by modern web-based email clients as an alternative to traditional email protocols, enabling rich web interfaces and RESTful APIs.

## DNS Configuration

Email servers require proper DNS configuration, particularly MX (Mail Exchange) records that specify which mail servers are responsible for receiving emails for a domain.

## Modern Implementations

Modern [[Distributed Email Service]] systems often use HTTP-based APIs for web clients while maintaining SMTP/IMAP support for traditional email clients. This hybrid approach provides flexibility for different client types while supporting legacy systems.

Email attachments are typically sent base64-encoded with size limits around 25MB, though this varies between providers and account types.

---
*Related: [[Distributed Email Service]], [[DNS]], [[HTTP]], [[API Design]]*
