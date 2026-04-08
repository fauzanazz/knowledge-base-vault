---
title: "Hotel Reservation System"
category: system-design
summary: "A hotel reservation system manages room bookings, inventory, and payments for hotel chains. It requires handling concurrency issues, overbooking policies, and dynamic pricing while maintaining data consistency."
sources:
  - raw/articles/hotel-reservation-system-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:48:31.884Z
---

# Hotel Reservation System

> A hotel reservation system manages room bookings, inventory, and payments for hotel chains. It requires handling concurrency issues, overbooking policies, and dynamic pricing while maintaining data consistency.

# Hotel Reservation System

A hotel reservation system manages room bookings, inventory tracking, and payment processing for hotel chains. The system must handle high concurrency during peak seasons while supporting features like overbooking and dynamic pricing.

## Core Components

The system follows a microservice architecture with these key services:

- **Hotel Service**: Provides hotel and room information with aggressive caching
- **Rate Service**: Manages dynamic room pricing based on demand and availability
- **Reservation Service**: Handles booking requests and inventory management
- **Payment Service**: Processes payments and updates reservation status
- **Hotel Management Service**: Administrative functions for authorized personnel

## Data Model

The system uses a relational database with key tables:

- `room_type_inventory`: Tracks available rooms per hotel, room type, and date
- `reservation`: Records guest booking data with idempotency keys
- `room_type_rate`: Stores dynamic pricing information

The inventory table uses one row per (hotel_id, room_type_id, date) combination, enabling efficient availability queries and supporting overbooking policies (typically 110% of capacity).

## Concurrency Handling

To prevent double booking, the system implements:

- **Idempotent APIs**: Using unique reservation IDs to prevent duplicate bookings
- **Optimistic Locking**: Version numbers prevent conflicting updates
- **Database Constraints**: Ensure inventory never goes negative

## Scalability

For high-traffic scenarios, the system can scale through:

- **[[Database Sharding]]**: Partition data by hotel_id across multiple databases
- **[[Caching]]**: Use Redis for inventory data with CDC synchronization
- **Read Replicas**: Distribute read traffic across multiple database instances

The system maintains eventual consistency between cache and database, with the database serving as the authoritative source for preventing invalid reservations.

---
*Related: [[Database Sharding]], [[Caching]], [[Microservices]], [[Concurrency Control]], [[Payment Systems]]*
