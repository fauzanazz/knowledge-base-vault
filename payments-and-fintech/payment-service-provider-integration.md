---
title: "Payment Service Provider Integration"
category: system-design
summary: "Payment Service Provider (PSP) integration enables payment systems to process transactions through third-party providers like Stripe or Braintree. Integration can be done via direct API calls or hosted payment pages to handle compliance and security requirements."
sources:
  - raw/articles/payment-system-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:16:46.890Z
---

# Payment Service Provider Integration

> Payment Service Provider (PSP) integration enables payment systems to process transactions through third-party providers like Stripe or Braintree. Integration can be done via direct API calls or hosted payment pages to handle compliance and security requirements.

# Payment Service Provider Integration

**Payment Service Provider (PSP) integration** connects payment systems to third-party processors that handle the actual movement of money between accounts. PSPs like Stripe, Braintree, and Square provide the infrastructure to process credit card payments without requiring direct bank connections.

## Integration Methods

### Direct API Integration
Payment systems collect payment information and send it directly to PSP APIs:
- Requires handling sensitive payment data
- Demands strict PCI compliance
- Provides more control over user experience
- Suitable for systems with robust security infrastructure

### Hosted Payment Page
PSPs provide a secure payment page that handles sensitive data:
- Avoids storing credit card information
- Reduces compliance burden
- Simplifies security requirements
- Most common approach for e-commerce systems

## Hosted Payment Page Workflow

1. User clicks "checkout" button
2. Client calls payment service with order information
3. Payment service sends registration request to PSP
4. PSP returns a unique token identifying the payment session
5. User is redirected to PSP-hosted payment page
6. User enters payment details on secure PSP page
7. PSP processes payment and returns status
8. User is redirected back with payment result
9. PSP sends webhook notification to payment service

## Key Features

### Idempotency
PSPs use **nonce values** (typically UUIDs) to prevent duplicate payment processing:
- Same nonce = same transaction
- Prevents double-charging customers
- Enables safe retry mechanisms

### Webhook Notifications
Asynchronous payment status updates:
- Real-time payment completion notifications
- Handles delayed payment processing
- Supports reconciliation processes

### Security Benefits
- PCI compliance handled by PSP
- Encrypted payment data transmission
- Fraud detection and prevention
- Secure token-based payment references

PSP integration is essential for modern [[Payment System]]s, providing secure, compliant payment processing without the complexity of direct banking relationships.

---
*Related: *
