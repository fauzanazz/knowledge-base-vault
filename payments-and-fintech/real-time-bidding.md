---
title: "Real-Time Bidding"
category: system-design
summary: "Real-time bidding (RTB) is a digital advertising process where ad inventory is bought and sold programmatically within milliseconds. It requires sub-second response times and high accuracy for financial transactions."
sources:
  - raw/articles/ad-click-event-aggregation-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:51:06.807Z
---

# Real-Time Bidding

> Real-time bidding (RTB) is a digital advertising process where ad inventory is bought and sold programmatically within milliseconds. It requires sub-second response times and high accuracy for financial transactions.

# Real-Time Bidding

Real-time bidding (RTB) is the automated process of buying and selling digital advertising inventory in real-time auctions that occur when users visit websites or apps.

## Process Overview

When a user visits a webpage, an auction occurs within 100 milliseconds:

1. **Ad Request**: Publisher sends user context to ad exchange
2. **Bid Request**: Ad exchange broadcasts to demand-side platforms (DSPs)
3. **Bid Response**: Advertisers submit bids based on user data
4. **Auction**: Highest bidder wins the ad placement
5. **Ad Delivery**: Winning ad is displayed to user

## Technical Requirements

**Latency**: Sub-second response times (typically <100ms) are critical since auctions happen in real-time as pages load.

**Scale**: Major platforms process millions of bid requests per second globally.

**Accuracy**: Precise bid calculations and financial tracking are essential since real money changes hands with each transaction.

## System Components

**Bid Management**: Algorithms determine optimal bid amounts based on user profiles, campaign budgets, and performance data.

**User Profiling**: Real-time analysis of user behavior, demographics, and interests to inform bidding decisions.

**Budget Control**: Real-time spend tracking to prevent campaign budget overruns.

**Performance Tracking**: Integration with [[Ad Click Event Aggregation]] systems to measure campaign effectiveness and adjust bidding strategies.

## Data Dependencies

RTB systems rely heavily on aggregated click and conversion data to make informed bidding decisions. This creates a feedback loop where historical performance data influences future bid amounts and targeting strategies.

The speed and accuracy requirements of RTB make it one of the most demanding real-time systems in digital advertising.

---
*Related: [[Ad Click Event Aggregation]], [[Microservices Architecture]], [[Caching]], [[Load Balancer]]*
