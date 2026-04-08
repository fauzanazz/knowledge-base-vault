---
title: "Star Schema"
category: database
summary: "Star schema is a data warehouse design pattern that organizes data into fact tables surrounded by dimension tables. It enables fast aggregation queries and efficient data filtering for analytics workloads."
sources:
  - raw/articles/ad-click-event-aggregation-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:51:06.808Z
---

# Star Schema

> Star schema is a data warehouse design pattern that organizes data into fact tables surrounded by dimension tables. It enables fast aggregation queries and efficient data filtering for analytics workloads.

# Star Schema

Star schema is a dimensional modeling technique used in data warehouses that organizes data into a central fact table surrounded by dimension tables, resembling a star when visualized.

## Structure

**Fact Table**: Contains quantitative data (metrics) and foreign keys to dimension tables. Examples include sales amounts, click counts, or transaction volumes.

**Dimension Tables**: Contain descriptive attributes used for filtering and grouping. Examples include time, geography, products, or user demographics.

## Benefits

**Query Performance**: Optimized for OLAP queries with fast aggregations and filtering. Pre-computed joins reduce query complexity.

**Intuitive Design**: Business users can easily understand the relationship between facts and dimensions.

**Flexible Analysis**: Supports slicing and dicing data across multiple dimensions for comprehensive analytics.

## Example: Ad Click Analytics

In an [[Ad Click Event Aggregation]] system:

**Fact Table**: `ad_clicks`
- `ad_id`, `click_minute`, `country`, `count`

**Dimension Tables**:
- `ads`: ad details, campaign info
- `time`: minute, hour, day, month hierarchies
- `geography`: country, region, city

## Trade-offs

**Advantages**:
- Fast query performance for analytics
- Simple to understand and maintain
- Excellent for reporting and dashboards

**Disadvantages**:
- Data duplication increases storage requirements
- More complex ETL processes
- Not suitable for transactional workloads

## Implementation Considerations

**Pre-aggregation**: Computing and storing aggregated results for common query patterns improves performance but increases storage costs.

**Dimension Explosion**: Having many filtering criteria can create numerous dimension combinations, significantly increasing data volume.

Star schema is particularly effective for systems requiring fast analytical queries across multiple dimensions, such as business intelligence and reporting platforms.

---
*Related: [[Ad Click Event Aggregation]], [[Database Sharding]], [[Caching]], [[System Scaling]]*
