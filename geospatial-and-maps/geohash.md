---
title: "Geohash"
category: data-structures
summary: "Geohash is a geocoding system that encodes geographic coordinates into short alphanumeric strings. It enables efficient proximity searches by grouping nearby locations with common prefixes."
sources:
  - raw/articles/nearby-friends-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:38:12.597Z
---

# Geohash

> Geohash is a geocoding system that encodes geographic coordinates into short alphanumeric strings. It enables efficient proximity searches by grouping nearby locations with common prefixes.

# Geohash

Geohash is a geocoding system that converts two-dimensional geographic coordinates (latitude and longitude) into a single alphanumeric string. It's widely used in location-based services for efficient proximity searches and spatial indexing.

## How Geohash Works

Geohash uses a base-32 encoding system that recursively divides the world into smaller rectangular grids:

1. **Bit Interleaving**: Alternates between longitude and latitude bits
2. **Recursive Division**: Each bit narrows down the geographic area
3. **Base-32 Encoding**: Converts binary representation to readable string

**Example**: The coordinates (42.6, -5.6) might encode to "ezs42e44yx96"

## Key Properties

**Hierarchical Structure**: Longer geohashes represent smaller areas with higher precision
**Common Prefixes**: Nearby locations often share common geohash prefixes
**Variable Precision**: String length determines geographic precision (1 char ≈ 5000km, 12 chars ≈ 3.7cm)
**Deterministic**: Same coordinates always produce the same geohash

## Advantages

- **Simple Implementation**: Easy to encode/decode coordinates
- **Database Friendly**: Works well with standard string indexing
- **Proximity Approximation**: Common prefixes indicate nearby locations
- **Scalable**: Efficient for large datasets

## Limitations

**Edge Cases**: Nearby locations across geohash boundaries may have different prefixes
**Non-uniform Areas**: Grid cells vary in size, especially near poles
**Precision Trade-offs**: Shorter hashes cover larger areas but lose precision

## Applications

**[[Proximity Service]]**: Finding nearby restaurants, hotels, or points of interest
**[[Nearby Friends System]]**: Grouping users by geographic regions for efficient updates
**Database Sharding**: Distributing location data across servers
**Caching**: Geographic cache keys for location-based data

## Comparison with Alternatives

**vs [[Quadtree]]**: Geohash is simpler but less flexible for irregular data distributions
**vs Google S2**: S2 provides better geometric properties but higher complexity
**vs Simple Lat/Lng**: Geohash enables efficient range queries and indexing

## Implementation Considerations

For proximity searches, query multiple geohash prefixes to handle boundary cases. Consider the precision level based on your application's distance requirements and performance needs.

Geohash provides an excellent balance of simplicity and efficiency for most location-based applications requiring spatial indexing.

---
*Related: [[Proximity Service]], [[Quadtree]], [[Geospatial Indexing]], [[Nearby Friends System]], [[Google Maps System Design]]*
