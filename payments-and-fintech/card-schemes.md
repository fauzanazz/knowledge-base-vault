---
title: "Card Schemes"
category: system-design
summary: "Card schemes are organizations like Visa and MasterCard that process credit card operations, providing the infrastructure and standards for electronic payment transactions between merchants, banks, and cardholders."
sources:
  - raw/articles/payment-system-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:32:58.064Z
---

# Card Schemes

> Card schemes are organizations like Visa and MasterCard that process credit card operations, providing the infrastructure and standards for electronic payment transactions between merchants, banks, and cardholders.

# Card Schemes

**Card schemes** are organizations that process credit card operations and provide the infrastructure for electronic payment transactions. They establish the rules, standards, and networks that enable credit card payments between merchants, banks, and cardholders.

## Major Card Schemes
- **Visa**: Global payment technology company
- **MasterCard**: Multinational financial services corporation
- **American Express**: Integrated payment network and card issuer
- **Discover**: US-based payment network

## Role in Payment Processing

Card schemes work with [[Payment Service Provider|Payment Service Providers]] to facilitate the movement of money in [[Payment System|payment systems]]. They provide:

### Network Infrastructure
- Secure communication channels between banks
- Transaction routing and switching services
- Real-time authorization and settlement systems

### Standards and Compliance
- Security protocols and encryption standards
- Fraud prevention mechanisms
- Regulatory compliance frameworks
- PCI DSS (Payment Card Industry Data Security Standard) requirements

### Authorization Process
When a payment is initiated:
1. Merchant sends transaction to acquiring bank
2. Acquiring bank forwards to card scheme network
3. Card scheme routes to issuing bank
4. Issuing bank approves/declines and sends response back
5. Response travels back through the network to merchant

## Business Model
Card schemes typically earn revenue through:
- Interchange fees paid by merchants
- Network fees from participating banks
- Processing fees for transactions

## Integration Considerations
Direct integration with card schemes is rare and typically only done by large companies that can justify the significant investment in infrastructure, compliance, and regulatory requirements. Most companies use PSPs that handle card scheme integration.

---
*Related: *
