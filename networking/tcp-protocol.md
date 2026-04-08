---
title: "TCP Protocol"
category: networking
summary: "TCP provides reliable communication over unreliable IP networks through mechanisms like sequence numbers, acknowledgments, flow control, and congestion control."
sources:
  - raw/articles/communication-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:17:50.258Z
---

# TCP Protocol

> TCP provides reliable communication over unreliable IP networks through mechanisms like sequence numbers, acknowledgments, flow control, and congestion control.

# TCP Protocol

TCP (Transmission Control Protocol) operates in the transport layer to provide reliable communication channels over unreliable IP networks. It ensures that data streams arrive at destinations without gaps, duplication, or corruption.

## Reliability Mechanisms

TCP achieves reliability through several key mechanisms:
- **Segmentation**: Splits byte streams into discrete segments with sequence numbers and checksums
- **Acknowledgments**: Every segment must be acknowledged by the receiver or it gets retransmitted
- **Error Detection**: Receivers can detect holes, duplicates, and corrupt segments using checksums

## Connection Management

TCP requires connection establishment before data transmission using a three-way handshake that exchanges starting sequence numbers. The OS manages connection state through sockets on both sides.

Connection overhead includes:
- Setup time driven by round-trip latency
- Teardown time when closing connections
- Socket cleanup delays that can drain available sockets

Mitigation strategies include using connection pools and keeping connections open between request/response pairs.

## Flow Control

TCP implements flow control to prevent sender overflow of receiver buffers. The receive buffer size is communicated back to senders, who use it to determine when to back off transmission rates.

## Congestion Control

The congestion window specifies total packets that can be in-flight (sent but not acknowledged). This window adjusts based on network feedback:
- Timeouts and missed packets decrease the window
- Successful transmissions increase the window

Slower round-trip times yield larger effective bandwidths, favoring server placement close to clients for optimal performance.

---
*Related: [[Network Communication]], [[UDP Protocol]], [[Flow Control]], [[Congestion Control]], [[Network Protocols]]*
