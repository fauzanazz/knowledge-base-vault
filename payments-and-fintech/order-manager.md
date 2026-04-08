---
title: "Order Manager"
category: system-design
summary: "An order manager handles order lifecycle management in trading systems, performing risk checks, wallet validation, and state transitions. It serves as the intermediary between client gateways and matching engines."
sources:
  - raw/articles/stock-exchange-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:20:49.881Z
---

# Order Manager

> An order manager handles order lifecycle management in trading systems, performing risk checks, wallet validation, and state transitions. It serves as the intermediary between client gateways and matching engines.

# Order Manager

An order manager is a critical component in [[Stock Exchange System Design]] responsible for managing order lifecycle and ensuring proper validation before orders reach the [[Matching Engine]].

## Primary Responsibilities

**Risk Management:**
- Validates orders against predefined risk rules
- Enforces trading limits (e.g., maximum 1 million shares per day)
- Performs compliance checks for regulatory requirements
- Blocks orders that violate risk parameters

**Wallet Validation:**
- Verifies sufficient funds before order placement
- Reserves funds for pending orders
- Releases reserved funds when orders are canceled or filled
- Maintains real-time account balance tracking

**Order State Management:**
- Tracks order status throughout lifecycle (new, partial fill, filled, canceled)
- Manages state transitions using [[Event Sourcing]] patterns
- Handles order modifications and cancellations
- Maintains order history for auditing

## Order Processing Flow

1. **Receive Order:** Accept order from [[Client Gateway]]
2. **Risk Check:** Validate against risk management rules
3. **Wallet Check:** Verify and reserve sufficient funds
4. **Send to Sequencer:** Forward validated order for processing
5. **Receive Fills:** Process execution results from matching engine
6. **Update State:** Maintain current order status
7. **Notify Client:** Send updates back through client gateway

## State Transition Management

The order manager implements event sourcing to handle complex state transitions:
- Stores immutable order events instead of current state
- Enables replay capability for system recovery
- Provides audit trail for regulatory compliance
- Supports concurrent order processing

## Integration Patterns

**Library Packaging:**
- Order manager logic packaged as a library
- Embedded in multiple components to avoid extra network calls
- Reduces latency in critical trading paths
- Maintains consistency across system components

**Communication:**
- Receives orders via memory-mapped files for low latency
- Sends validated orders to [[Sequencer]]
- Processes execution confirmations asynchronously

The order manager must balance thoroughness in validation with speed requirements, as it sits on the critical path for trade execution.

---
*Related: [[Stock Exchange System Design]], [[Matching Engine]], [[Client Gateway]], [[Event Sourcing]], [[Risk Management]]*
