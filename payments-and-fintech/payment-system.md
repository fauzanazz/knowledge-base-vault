---
title: "Payment System"
category: system-design
summary: "A payment system is a distributed architecture that handles financial transactions, transferring monetary value between buyers and sellers through secure, reliable money movement processes."
sources:
  - raw/articles/_done/payment-system-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:58:07.364Z
---

# Payment System

> A payment system is a distributed architecture that handles financial transactions, transferring monetary value between buyers and sellers through secure, reliable money movement processes.

# Payment System

A **payment system** is used to settle financial transactions, transferring monetary value between parties in e-commerce applications. Modern payment systems handle millions of transactions daily while ensuring reliability, security, and compliance.

## Core Components

### Pay-in Flow
The pay-in flow receives money from customers on behalf of merchants:
- **Payment Service**: Accepts payment events and coordinates the payment process, including risk checks
- **Payment Executor**: Executes payment orders via Payment Service Providers (PSPs)
- **Payment Service Provider (PSP)**: Third-party services like Stripe or Square that move money between accounts
- **Ledger**: Maintains financial records using [[Double-Entry Ledger System]]
- **Wallet**: Tracks account balances for merchants

### Pay-out Flow
The pay-out flow sends money from the platform to sellers, often using third-party providers like Tipalti for regulatory compliance and bookkeeping.

## Key Design Principles

### Reliability and Consistency
Payment systems prioritize strong consistency over performance. They use traditional SQL databases for ACID guarantees and implement [[Exactly-Once Delivery]] to prevent double-charging.

### Security
Security measures include HTTPS encryption, SSL certificate pinning, [[Idempotency]] keys, and PCI compliance. Most systems avoid storing credit card data directly, instead using hosted payment pages from PSPs.

### Communication Patterns
Payment systems use both synchronous and asynchronous communication. [[Message Queue]] systems enable multiple services to react to payment events while maintaining loose coupling.

## Reconciliation
Daily reconciliation processes compare internal system state against external PSP records to detect and resolve inconsistencies. This ensures data integrity across distributed components.

Payment systems represent critical infrastructure requiring careful attention to fault tolerance, security, and regulatory compliance.

---
*Related: [[Double-Entry Ledger System]], [[Exactly-Once Delivery]], [[Idempotency]], [[Message Queue]], [[PSP Integration]], [[Payment Security]]*
