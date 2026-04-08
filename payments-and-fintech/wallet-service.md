---
title: "Wallet Service"
category: system-design
summary: "A wallet service maintains account balances for merchants and users in a payment system, tracking how much money each party has available. It works closely with the ledger system to ensure balance consistency and supports both pay-in and pay-out operations."
sources:
  - raw/articles/payment-system-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:32:58.065Z
---

# Wallet Service

> A wallet service maintains account balances for merchants and users in a payment system, tracking how much money each party has available. It works closely with the ledger system to ensure balance consistency and supports both pay-in and pay-out operations.

# Wallet Service

A **wallet service** is a critical component of [[Payment System|payment systems]] that maintains account balances for merchants and users. It tracks how much money each party has available and ensures balance consistency across all financial operations.

## Core Functionality

### Balance Management
The wallet service maintains real-time account balances by:
- Processing balance updates from successful payments
- Handling both incoming (pay-in) and outgoing (pay-out) transactions
- Ensuring balance accuracy through integration with [[Double-Entry Ledger System]]

### Account Types
- **Merchant Wallets**: Track seller account balances from completed sales
- **User Wallets**: Maintain buyer account information and available funds
- **System Wallets**: Handle platform fees and operational accounts

## Integration with Payment Flow

### Pay-in Process
1. [[Payment Service Provider]] processes customer payment
2. Payment executor confirms successful transaction
3. Wallet service updates merchant balance
4. Balance information is stored in wallet database
5. [[Ledger]] records the money movement

### Pay-out Process
1. Merchant requests payout
2. Wallet service verifies sufficient balance
3. Balance is debited from merchant wallet
4. Funds are transferred to merchant's bank account

## Data Consistency

The wallet service maintains consistency through:
- **Database transactions** ensuring atomic balance updates
- **Reconciliation flags** (`wallet_updated`) tracking successful updates
- **Regular reconciliation** with ledger system to detect discrepancies
- **Eventual consistency** mechanisms for distributed deployments

## Security Considerations

- Balance calculations use string data types to avoid floating-point precision errors
- All balance changes are logged for audit trails
- Access controls restrict balance modification to authorized services
- Regular balance verification against ledger records

The wallet service provides the foundation for financial operations while maintaining the accuracy and reliability required for monetary transactions.

---
*Related: *
