---
title: "Proximity Service"
category: system-design
summary: "A proximity service finds nearby locations like restaurants and hotels within a defined radius, used by applications like Google Maps and Yelp. It requires efficient geospatial indexing and low-latency search capabilities to handle millions of location-based queries."
sources:
  - raw/articles/_done/proximity-service-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:58:58.675Z
---

# Proximity Service

> A proximity service finds nearby locations like restaurants and hotels within a defined radius, used by applications like Google Maps and Yelp. It requires efficient geospatial indexing and low-latency search capabilities to handle millions of location-based queries.

# Proximity Service

A proximity service is designed to find nearby locations such as restaurants, hotels, gas stations, and other businesses within a defined radius. This functionality powers applications like Google Maps and Yelp to help users discover places near their current location.

## Functional Requirements

- Search for businesses based on user location (latitude, longitude) and search radius
- Allow business owners to add, update, or delete businesses
- Provide detailed business information when requested

## System Architecture

The system comprises two main components:

**Location-Based Service (LBS)**: Processes location-based search queries. This is a read-heavy, stateless service with high QPS during peak hours in dense areas.

**Business Service**: Handles business CRUD operations and serves detailed business information to customers.

The architecture uses a [[Load Balancer]] to route traffic and a [[Database Replication]] setup with primary-replica architecture for read-heavy workloads.

## Geospatial Indexing Approaches

### Geohash
Encodes latitude and longitude into a single alphanumeric string with hierarchical grid structure. Provides efficient proximity searching but has boundary issues where nearby locations may have different geohash prefixes.

### Quadtree
Recursively divides 2D space into four quadrants until each node contains fewer than a threshold number of businesses (typically 100). Supports k-nearest neighbor queries and dynamically adjusts grid size based on business density.

### Google S2
Maps spherical coordinates to 1D index using Hilbert curves. Excellent for geofencing with arbitrary area coverage and varying precision levels.

## Scaling Strategy

The system uses [[Database Sharding]] by business ID for even data distribution. [[Caching]] strategy employs geohash as cache keys to store business IDs, with separate caching for detailed business information.

For a system handling 100 million daily active users making 5 searches per day, the search QPS reaches approximately 5,000 queries per second.

---
*Related: [[Geospatial Indexing]], [[Location-Based Services]], [[Google Maps]], [[Database Sharding]], [[Caching]]*
