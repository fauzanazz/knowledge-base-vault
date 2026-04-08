---
title: "Map Projection"
category: algorithms
summary: "Map projection is the process of translating 3D geographic coordinates from Earth's spherical surface to a 2D plane for digital mapping. Google Maps uses Web Mercator projection, which provides good performance for web applications despite geometric distortions."
sources:
  - raw/articles/_done/google-maps-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T10:00:00.583Z
---

# Map Projection

> Map projection is the process of translating 3D geographic coordinates from Earth's spherical surface to a 2D plane for digital mapping. Google Maps uses Web Mercator projection, which provides good performance for web applications despite geometric distortions.

# Map Projection

Map projection is the mathematical process of converting three-dimensional geographic coordinates from Earth's spherical surface into a two-dimensional plane suitable for digital maps and displays.

## The Challenge

Earth is a sphere rotating on its axis, with positions defined by:
- **Latitude**: Distance north or south from the equator
- **Longitude**: Distance east or west from the Prime Meridian

Converting this 3D coordinate system to 2D inevitably introduces distortions in geometry, area, distance, or direction.

## Projection Methods

There are numerous projection methods, each with specific trade-offs:
- **Mercator Projection**: Preserves angles but distorts areas, especially near poles
- **Equal-area Projections**: Maintain accurate area relationships but distort shapes
- **Equidistant Projections**: Preserve distances from specific points

## Web Mercator

Google Maps uses a modified version of Mercator projection called **Web Mercator** (EPSG:3857). This projection was chosen because:
- It provides good visual representation for most inhabited areas
- Enables efficient tile-based rendering systems
- Maintains consistent scale relationships for navigation
- Works well with rectangular coordinate systems used in computer graphics

## Technical Implementation

Web Mercator projection:
1. Treats Earth as a perfect sphere (not an ellipsoid)
2. Projects coordinates onto a cylinder tangent to the equator
3. Creates a square coordinate system suitable for tiling
4. Enables zoom levels that double resolution at each level

## Applications in Digital Mapping

Map projections are fundamental to:
- [[Google Maps System Design|Digital mapping systems]]
- [[Geospatial Indexing|Spatial indexing algorithms]]
- Navigation and routing calculations
- Tile-based map rendering systems

The choice of projection significantly impacts system performance, storage requirements, and user experience in location-based applications.

---
*Related: [[Google Maps System Design]], [[Geospatial Indexing]], [[Geohash Algorithm]]*
