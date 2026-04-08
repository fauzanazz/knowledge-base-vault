---
title: "Watermark"
category: system-design
summary: "Watermarks are timestamps used in stream processing to handle late-arriving events by extending processing windows. They balance between data completeness and processing latency in real-time systems."
sources:
  - raw/articles/ad-click-event-aggregation-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:51:06.808Z
---

# Watermark

> Watermarks are timestamps used in stream processing to handle late-arriving events by extending processing windows. They balance between data completeness and processing latency in real-time systems.

# Watermark

Watermarks are a stream processing technique used to handle late-arriving events by defining when a processing window should close, balancing data completeness with processing latency.

## Problem: Late Events

In distributed systems, events may arrive out of order due to:
- Network delays
- Clock skew between systems
- Processing delays in upstream components
- System failures and retries

Without proper handling, late events can be missed in time-based aggregations, leading to inaccurate results.

## How Watermarks Work

A watermark is a timestamp that indicates "no more events with timestamps earlier than this will arrive." It extends the processing window beyond the nominal end time.

**Example**: For a 1-minute aggregation window ending at 10:00:00:
- Without watermark: Window closes exactly at 10:00:00
- With 30-second watermark: Window stays open until 10:00:30

## Watermark Strategies

**Fixed Delay**: Add a constant time buffer (e.g., 30 seconds) to all windows.

**Percentile-Based**: Close windows when 95% of expected events have arrived based on historical patterns.

**Heuristic**: Use domain knowledge about data sources to estimate appropriate delays.

## Trade-offs

**Short Watermarks**:
- Lower latency for results
- Higher risk of missing late events
- Better for real-time dashboards

**Long Watermarks**:
- More complete data capture
- Higher processing latency
- Better for billing and financial calculations

## Implementation in Ad Systems

In [[Ad Click Event Aggregation]] systems, watermarks are crucial because billing accuracy is more important than immediate results. A 2-3 minute watermark might be acceptable to ensure financial data completeness, with end-of-day reconciliation handling any remaining stragglers.

---
*Related: [[Ad Click Event Aggregation]], [[Message Queue]], [[System Scaling]], [[Real-time Gaming Leaderboard]]*
