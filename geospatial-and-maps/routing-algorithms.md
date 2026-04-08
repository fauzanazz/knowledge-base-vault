---
title: "Routing Algorithms"
category: algorithms
summary: "Routing algorithms find optimal paths between locations in navigation systems by representing road networks as graphs with intersections as nodes and roads as edges. Modern systems use modified versions of Dijkstra's or A* algorithms with hierarchical tiling for scalability."
sources:
  - raw/articles/_done/google-maps-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T10:00:00.583Z
---

# Routing Algorithms

> Routing algorithms find optimal paths between locations in navigation systems by representing road networks as graphs with intersections as nodes and roads as edges. Modern systems use modified versions of Dijkstra's or A* algorithms with hierarchical tiling for scalability.

# Routing Algorithms

Routing algorithms are pathfinding techniques used in navigation systems to find optimal routes between origin and destination points. They form the core of services like [[Google Maps System Design|Google Maps]] navigation.

## Graph Representation

Road networks are modeled as graphs where:
- **Nodes**: Represent intersections, highway entrances/exits, and decision points
- **Edges**: Represent road segments with associated weights (distance, time, traffic)
- **Weights**: Include factors like distance, travel time, traffic conditions, and road restrictions

## Core Algorithms

### Dijkstra's Algorithm
- Finds shortest path from source to all other nodes
- Guarantees optimal solution but explores many unnecessary nodes
- Time complexity: O((V + E) log V) where V is vertices and E is edges

### A* Algorithm
- Uses heuristic function to guide search toward destination
- More efficient than Dijkstra's for single destination queries
- Heuristic typically uses straight-line distance (Euclidean or Manhattan)
- Maintains optimality while reducing search space

## Scalability Challenges

Pathfinding performance is sensitive to graph size. For global navigation systems:
- World-scale graphs contain billions of nodes and edges
- Memory bandwidth becomes a bottleneck
- Single-machine processing is insufficient

## Hierarchical Routing

Modern systems use **routing tiles** - a technique similar to map tiling:

### Multi-Level Approach
1. **Detailed Tiles**: High-resolution graphs for local routing
2. **Highway Networks**: Simplified graphs for long-distance routing
3. **Tile Interconnections**: References between neighboring tiles

### Benefits
- **Memory Efficiency**: Load only necessary tiles
- **Scalability**: Distribute processing across multiple servers
- **Performance**: Use appropriate detail level for route distance

## Traffic Integration

Modern routing algorithms incorporate:
- **Real-time Traffic**: Current congestion and incident data
- **Historical Patterns**: Time-of-day and day-of-week traffic trends
- **Predictive Models**: Machine learning for ETA calculations
- **Dynamic Rerouting**: Adaptive path updates during navigation

## Implementation Considerations

- **Preprocessing**: Precompute routing tiles and hierarchical networks
- **Caching**: Store frequently requested routes
- **Load Balancing**: Distribute routing requests across multiple servers
- **Fault Tolerance**: Handle server failures gracefully

Routing algorithms must balance accuracy, performance, and scalability to serve millions of navigation requests efficiently.

---
*Related: [[Google Maps System Design]], [[Geospatial Indexing]], [[System Scaling]], [[Caching]]*
