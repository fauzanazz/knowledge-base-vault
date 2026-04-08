---
title: "Routing Tiles"
category: algorithms
summary: "Routing tiles are hierarchical graph subdivisions that enable scalable pathfinding algorithms by breaking the world into manageable pieces. They contain road network data at different detail levels and reference neighboring tiles for efficient route calculation."
sources:
  - raw/articles/google-maps-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:18:36.857Z
---

# Routing Tiles

> Routing tiles are hierarchical graph subdivisions that enable scalable pathfinding algorithms by breaking the world into manageable pieces. They contain road network data at different detail levels and reference neighboring tiles for efficient route calculation.

# Routing Tiles

Routing tiles are a hierarchical data structure that subdivides the world's road network into smaller, manageable graph segments to enable scalable pathfinding algorithms. This technique is essential for navigation systems handling global-scale route calculations.

## Problem with Global Graphs

Running pathfinding algorithms like Dijkstra or A* on the entire world's road network is computationally prohibitive due to:
- **Memory constraints**: Billions of intersections and road segments
- **Processing time**: Exponential complexity with graph size
- **Bandwidth limitations**: Loading entire global graph is impractical

## Hierarchical Tile Structure

### Multi-Level Detail
Routing tiles exist at different resolution levels:
- **High detail**: Local streets, parking lots, building access
- **Medium detail**: City roads, highways within regions
- **Low detail**: Major highways, interstate connections

### Tile Interconnection
Each tile maintains:
- **Internal graph**: Roads and intersections within the tile
- **Border nodes**: Connection points to neighboring tiles
- **References**: Pointers to adjacent tiles for graph traversal

## Algorithm Implementation

### Pathfinding Process
1. **Start tile identification**: Determine source location's routing tile
2. **Local search**: Run A* within the starting tile
3. **Tile traversal**: Follow border nodes to neighboring tiles
4. **Dynamic loading**: Load only necessary tiles during route calculation
5. **Multi-resolution**: Use appropriate detail level based on route distance

### Memory Optimization
Algorithms load tiles on-demand, significantly reducing:
- Memory bandwidth requirements
- Storage overhead
- Processing time for long-distance routes

## Data Storage

In [[Google Maps System Design]], routing tiles are:
- Stored in S3 object storage for scalability
- Compressed using binary adjacency list formats
- Cached aggressively for performance
- Generated from raw road data through offline processing pipelines

## Benefits

- **Scalability**: Enables global navigation without loading entire world graph
- **Performance**: Reduces memory usage and computation time
- **Flexibility**: Supports different detail levels for various route types
- **Maintainability**: Allows incremental updates to specific geographic regions

> ⚠️ *This article may be incomplete. Run `kb compile --full` to regenerate.*

---
*Related: [[Google Maps System Design]], [[Pathfinding Algorithms]], [[Graph Data Structures]], [[Geohash]]*
