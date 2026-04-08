---
title: "Lambda Architecture"
category: system-design
summary: "Lambda architecture is a data processing design that combines batch and stream processing in parallel paths to handle both real-time and historical data analysis. It maintains separate codebases for batch and streaming components."
sources:
  - raw/articles/ad-click-event-aggregation-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:51:06.807Z
---

# Lambda Architecture

> Lambda architecture is a data processing design that combines batch and stream processing in parallel paths to handle both real-time and historical data analysis. It maintains separate codebases for batch and streaming components.

# Lambda Architecture

Lambda architecture is a data processing framework that combines batch and stream processing systems to handle both real-time analytics and comprehensive historical data analysis.

## Architecture Components

**Batch Layer**: Processes large volumes of historical data using systems like Hadoop MapReduce or Apache Spark. Provides comprehensive, accurate views but with higher latency.

**Speed Layer**: Handles real-time data streams using technologies like Apache Storm or Apache Flink. Provides low-latency results but may sacrifice some accuracy.

**Serving Layer**: Merges results from both batch and speed layers to provide a unified view for queries. Often implemented using databases like HBase or Cassandra.

## Benefits

- **Fault Tolerance**: If the speed layer fails, the batch layer can still provide results
- **Accuracy**: Batch processing ensures comprehensive, accurate historical analysis
- **Low Latency**: Stream processing provides near real-time insights
- **Scalability**: Each layer can be scaled independently based on requirements

## Drawbacks

- **Complexity**: Maintaining two separate processing systems increases operational overhead
- **Code Duplication**: Similar logic must be implemented in both batch and streaming components
- **Data Consistency**: Reconciling results between layers can be challenging

## Alternative: Kappa Architecture

Kappa architecture simplifies the design by using only stream processing, treating batch processing as a special case of streaming with historical data replay. This approach reduces complexity but requires more sophisticated streaming frameworks.

Lambda architecture is commonly used in [[Ad Click Event Aggregation]] systems where both real-time bidding decisions and comprehensive billing reports are required.

---
*Related: [[Ad Click Event Aggregation]], [[Message Queue]], [[System Scaling]], [[Microservices Architecture]]*
