---
title: "FIX Protocol"
category: networking
summary: "FIX (Financial Information eXchange) is a standardized messaging protocol used for electronic trading and securities transaction communication. It enables efficient, low-latency communication between trading systems, brokers, and exchanges."
sources:
  - raw/articles/stock-exchange-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:20:49.881Z
---

# FIX Protocol

> FIX (Financial Information eXchange) is a standardized messaging protocol used for electronic trading and securities transaction communication. It enables efficient, low-latency communication between trading systems, brokers, and exchanges.

# FIX Protocol

FIX (Financial Information eXchange) is the industry-standard messaging protocol for electronic trading and securities transaction communication, widely used by exchanges, brokers, and institutional trading systems.

## Protocol Structure

FIX messages are composed of tag-value pairs separated by delimiters:
- **Tags**: Numeric identifiers for specific fields
- **Values**: Data associated with each tag
- **Delimiter**: SOH (Start of Header, ASCII 0x01) character

**Example FIX Message:**
```
8=FIX.4.2|9=176|35=8|49=PHLX|56=PERS|52=20071123-05:30:00.000|
11=ATOMNOCCC9990900|20=3|150=E|39=E|55=MSFT|167=CS|54=1|
38=15|40=2|44=15|58=PHLX EQUITY TESTING|59=0|47=C|32=0|
31=0|151=15|14=0|6=0|10=128|
```

## Key Message Types

**Order Messages:**
- New Order Single (35=D): Place new order
- Order Cancel Request (35=F): Cancel existing order
- Order Cancel/Replace Request (35=G): Modify order

**Execution Messages:**
- Execution Report (35=8): Trade confirmation
- Order Status (35=H): Order state updates

**Session Messages:**
- Logon (35=A): Establish session
- Heartbeat (35=0): Keep connection alive
- Logout (35=5): Terminate session

## Common FIX Tags

- **8**: BeginString (FIX version)
- **35**: MsgType (Message type)
- **49**: SenderCompID (Sender identifier)
- **55**: Symbol (Trading instrument)
- **54**: Side (Buy=1, Sell=2)
- **38**: OrderQty (Order quantity)
- **44**: Price (Order price)
- **40**: OrdType (Order type)

## Session Management

FIX implements reliable messaging through:
- **Sequence Numbers**: Every message has a unique sequence number
- **Acknowledgments**: Confirmations for message receipt
- **Resend Requests**: Recovery mechanism for missed messages
- **Heartbeats**: Regular keep-alive messages

## Integration with Trading Systems

In [[Stock Exchange System Design]], FIX protocol is used by:
- [[Client Gateway]] for institutional client communication
- Broker systems for order routing
- Market data distribution for real-time feeds
- Settlement and clearing systems

The protocol's standardization enables interoperability between different trading platforms and reduces integration complexity in financial markets.

---
*Related: [[Stock Exchange System Design]], [[Client Gateway]], [[Trading Protocols]], [[Financial Messaging]], [[Electronic Trading]]*
