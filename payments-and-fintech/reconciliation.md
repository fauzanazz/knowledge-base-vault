---
title: "Reconciliation"
category: system-design
summary: "Reconciliation is a background process in payment systems that detects and fixes inconsistencies between internal system state and external systems like PSPs or banks. It typically runs nightly using settlement files to identify mismatches that require manual or automated correction."
sources:
  - raw/articles/payment-system-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:32:58.066Z
---

# Reconciliation

> Reconciliation is a background process in payment systems that detects and fixes inconsistencies between internal system state and external systems like PSPs or banks. It typically runs nightly using settlement files to identify mismatches that require manual or automated correction.

# Reconciliation

**Reconciliation** is a critical background process in [[Payment System|payment systems]] that detects and corrects inconsistencies between internal system state and external systems like [[Payment Service Provider|PSPs]], banks, or internal services.

## Process Overview

### Nightly Reconciliation
Every night, PSPs send **settlement files** containing:
- All processed transactions for the day
- Final transaction statuses
- Settlement amounts and fees
- Transaction timestamps and identifiers

The reconciliation system compares this external data against internal records from the [[Ledger]] and [[Wallet Service]] to identify discrepancies.

### Internal Reconciliation
Reconciliation also detects inconsistencies between internal services:
- Ledger vs. Wallet Service balance mismatches
- Payment Service vs. Payment Executor status differences
- Missing or incomplete transaction records

## Types of Mismatches

### Classifiable and Automatable
Known discrepancies that can be automatically corrected:
- Network timeout causing missing status updates
- Delayed webhook notifications
- Standard fee adjustments

### Classifiable but Manual
Known issues requiring human intervention:
- Partial refunds not reflected in internal systems
- Currency conversion discrepancies
- Merchant-specific fee arrangements

### Unclassifiable
Unknown discrepancies requiring investigation:
- Unexplained transaction differences
- Data corruption issues
- System bugs causing incorrect processing

## Implementation Components

### Settlement File Processing
1. Download daily settlement files from PSPs
2. Parse transaction data and normalize formats
3. Match external transactions with internal records
4. Identify missing, extra, or mismatched transactions

### Discrepancy Resolution
- **Automated fixes**: Apply standard correction procedures
- **Manual queue**: Route complex issues to finance team
- **Investigation tools**: Provide detailed transaction history
- **Audit trails**: Log all reconciliation actions

## Benefits

- **Financial accuracy**: Ensures money movements are correctly recorded
- **Fraud detection**: Identifies unauthorized or suspicious transactions
- **System reliability**: Catches bugs and integration issues
- **Regulatory compliance**: Provides audit trails for financial reporting

Reconciliation is essential for maintaining trust and accuracy in payment systems, ensuring that all financial operations are properly accounted for across distributed systems.

---
*Related: *
